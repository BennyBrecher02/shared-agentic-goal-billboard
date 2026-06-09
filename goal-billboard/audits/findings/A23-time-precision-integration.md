# A23 — Time precision + scheduler/heartbeat/time three-way integration

**Audit type:** system infrastructure verification (read-only diagnostic)
**Run at:** 2026-05-29T01:53:56Z (system clock, `date -u`)
**Scope:** Heart-flatline root-cause + true-time discipline end-to-end + three-way clock consistency + staleness semantics
**Mode:** READ-ONLY. No files touched besides this one. No LaunchAgents installed. No git/server runs.

> **Note on "today's date":** the SessionStart context says *"Today's date is 2026-05-28."* The macOS Unix clock reads **2026-05-29T01:53Z** (UTC). Per `feedback_true-time-discipline`, the system clock is canonical; the injected context string is ~2h behind the rollover into 05-29 UTC (it is still 05-28 in local EDT, `-0400`, which is where that string is anchored). This is a display nuance, not a clock fault — but it is exactly the kind of text-vs-clock gap the discipline warns about. **All timestamps in this doc are `date -u` / file-mtime-derived, never inferred.**

---

## TL;DR

The Heart is **NOT** truly flatlined. **"28h stale" is largely a measurement artifact**: `organic-os-pulse.sh` judges Heart liveness by the **mtime of the event _outbox_ (`heartbeat-pending.jsonl`)** — a file that only advances when a *producer appends an event*, not when the Heart *beats*. No producer has appended in 28h (no monkey decisions, no event-bus dispatch), so the outbox looks stale even though:

- the **independent Heart tick** beat 5× as recently as **2026-05-28T21:11Z (~4.7h ago)** — `last-independent-tick.json`
- **daemon10** (the only launchd-installed cell) polled **2026-05-29T01:15Z (~38m ago)** — alive
- the **drain ack** updated **2026-05-29T01:46Z (this session)** — alive

There **is** a real, narrower outage underneath the artifact: the **scheduler-driven tier ticks stopped at 2026-05-26T23:11Z (~50h ago)** and the **event-producer cohort stopped at 2026-05-27T21:03Z (~28h ago)**. The Heart's *reactive high-tier path* (scheduler/agent-action-driven) is dead; only the *independent backstop* and daemon10 are beating — and the independent backstop is **dry-run, not on a launchd timer**, so it only fired because something invoked it by hand on 05-28.

True-time discipline is **mostly clean** (canonical `date -u` everywhere I checked), with **one real latent hazard** (stat `%Sm` + literal `Z` prints local time labelled as UTC) and **one contamination** (`synthetic-a23` strings written into Heart state by a prior A23 run).

---

## Master timestamp reconciliation (the evidence spine)

All epochs converted with `date -u -r <epoch>`; `now` = capture time.

| Item | Epoch | UTC ISO | Age | Source |
|------|-------|---------|-----|--------|
| **now** | 1780019636 | 2026-05-29T01:53:56Z | — | `date +%s` |
| tier-high/medium/low tick files | 1779837069 | **2026-05-26T23:11:09Z** | **~50h** | `.claude/cache/heartbeat/last-tick-{high,medium,low,hourly}.txt` (content = epoch) |
| scheduler last `tick_end` | 1779915792 | **2026-05-27T21:03:12Z** | **~28h** | `.claude/cache/scheduler-events/2026-05-26.jsonl:last` |
| **heartbeat-pending.jsonl mtime** | 1779915643 | **2026-05-27T21:00:43Z** | **~28h** | the file the pulse HEART check reads |
| newest event IN pending queue | — | 2026-05-27T21:00:43Z | ~28h | `heartbeat-pending.jsonl:last` (`monkey-decision-applied`) |
| independent Heart beat | 1780002717 | **2026-05-28T21:11:57Z** | **~4.7h** | `.claude/cache/heartbeat/last-independent-tick.json` |
| daemon10 last poll | 1780017358 | **2026-05-29T01:15:58Z** | **~38m** | `.claude/cache/daemon10-tier-low-poll.jsonl:last` |
| last-drain-ack | 1780019211 | **2026-05-29T01:46:51Z** | this session | `.claude/cache/heartbeat/last-drain-ack.txt` |
| clock-drift check | 1779836... | 2026-05-26T23:10:57Z | ~50h | `.claude/cache/clock-drift.json` (drift 0.048s, "ok") |

**Two distinct "stale" cohorts** fall out of this table — the 50h tier-tick cohort and the 28h producer cohort — and the pulse conflates both into one HEART glyph by reading a third thing (outbox mtime). That conflation is the headline finding.

---

## FINDING 1 — Heart "flatline" is a measurement artifact: the pulse reads the OUTBOX, not the BEAT

- **What:** `organic-os-pulse.sh` (the SessionStart vital-signs strip) computes the HEART glyph from the **mtime of `heartbeat-pending.jsonl`** — but that file is the **event outbox/queue**, written only when a *producer appends an event*. It is **not** a liveness beacon and is **never touched by the Heart's own beat**.
- **Evidence:**
  - `scripts/hooks/organic-os-pulse.sh:92` — `local hb="$CACHE_DIR/heartbeat-pending.jsonl"`; `:94` `age="$(_file_age_secs "$hb")"`; thresholds `:100-106` (`<6h ✓ / <24h ◑ / else ✗`).
  - Live run reproduced exactly: `HEART ✗ (28h stale) ... → flatlined: HEART.`
  - `heartbeat-pending.jsonl` mtime = **2026-05-27T21:00:43Z**, and its last line is `{"ts":"2026-05-27T21:00:43Z",...,"event":"monkey-decision-applied",...}` — mtime == newest-event ts, confirming the file only moves on append.
  - The Heart's actual beat beacon — `last-independent-tick.json` — is **4.7h fresh** (`"ts":"2026-05-28T21:11:57Z"`), and `last-drain-ack.txt` updated **this session** (01:46Z). The Heart machinery is demonstrably running; the pulse just isn't looking at it.
  - Confirmed no producer writes the outbox on a beat: `heartbeat-drain.sh` only **reads** `$PENDING` (lines 28/114/131-136; the only write is a >1000-line rotation that isn't triggered at 36 lines). The sole appenders are event *producers* — `monkey-click-fan-out.sh:244` and `consumer-heartbeat-tier-high.sh:64` (fired by `dispatch.py`).
- **Severity:** **HIGH** (false alarm that masks the real, narrower outage in Finding 2 — and erodes trust in the one observer the system has).
- **Recommendation (PROPOSE):** Change `check_heart()` to read the **beat beacon**, not the outbox. Concretely: judge HEART by `max(mtime)` over a small beacon set — `last-independent-tick.json`, `last-drain-ack.txt`, `last-tick-high.txt` — i.e. "did *any* part of the Heart move recently?" Keep the outbox age as a **secondary** signal ("queue N items, oldest Xh") on the detail line, not as the pass/fail glyph. This makes the glyph mean *"is the Heart beating?"* instead of *"has a producer emitted lately?"*.

## FINDING 2 — The REAL outage: scheduler-driven tier ticks (50h) + event-producer cohort (28h) are dead; only the independent backstop + daemon10 beat

- **What:** Underneath the artifact there is a genuine partial outage. The **scheduler-coordinated tier ticks** last fired **2026-05-26T23:11Z (~50h ago)**, and the **event-producer path** (scheduler `tick_end` → events) last fired **2026-05-27T21:03Z (~28h ago)**. The Heart's *reactive high-tier path* depends on these and is therefore dark. This is the exact A74-D1 SPOF the independent tick was built to survive.
- **Evidence:**
  - `last-tick-high.txt` / `-medium.txt` / `-low.txt` / `-hourly.txt` all contain epoch `1779837069` = **2026-05-26T23:11:09Z** (file mtimes match).
  - `scheduler-events/2026-05-26.jsonl` last line: `{"event":"tick_end","tick_id":"a857d977","ts":"2026-05-27T21:03:11Z",...}` — no scheduler events since. No `2026-05-28.jsonl` or `2026-05-29.jsonl` file exists.
  - **Firing mechanism is event-driven, not timer-driven** (this is *why* it can flatline): per `settings.json`, the three Heart-feeding hooks fire on agent/session events, never on a wall clock —
    - `organic-os-pulse.sh` → **SessionStart** (`settings.json:77`)
    - `heartbeat-drain.sh` → **UserPromptSubmit** (`:112`)
    - `scheduler-heartbeat-coordinator.sh` → **PostToolUse[Bash]** (`:290`, matcher `"Bash"`)
  - No wall-clock timer for the Heart is installed: `launchctl list | grep evium` returns only `com.evium.daemon10`, `daemon2`, `daemon3`, `daemon4`, `daemon5`. **`com.evium.heartbeat*` is NOT loaded** (template exists at `scripts/heartbeat/heartbeat-launchd.plist.template` but was never installed — matches the script header's own "tick.sh ... was NEVER installed" note).
  - The independent backstop **did** beat — but only **5 times, all 2026-05-28T21:08–21:11Z, all `"mode":"dry-run"`** (`heartbeat-independent-tick.jsonl`, 5 lines). With no launchd timer for it either, those 5 beats were a **manual/hook-triggered burst**, not autonomous cadence. So even the backstop is not actually backstopping on a schedule yet.
  - daemon10 **is** healthy (own launchd, `StartInterval 3600`): `daemon10-tier-low-poll.jsonl` last line `{"ts_iso":"2026-05-29T01:15:58Z","items_pending":0,"dry_run":1}`, `launchctl list com.evium.daemon10` → `LastExitStatus = 0`. (Its `.err` mtime of 00:58Z is a red herring — recent polls had nothing to log to stderr; the JSONL is the true liveness signal.)
- **Severity:** **HIGH** (real degradation: tier-medium/low Heart roles — bottleneck-detect cadence, banana-staleness, capacity self-throttle, cross-substrate watchdog — have not run on a timer; they only run when an agent turn happens to drive the hooks).
- **Recommendation (PROPOSE — user-gated install, do NOT auto-apply):** This is precisely what the **independent tick + its launchd plist** were designed for. Re-arming path (user runs the `launchctl` step — it is on the A66 high-risk-always-gated list):
  1. Render `scripts/heartbeat/heartbeat-launchd.plist.template` (replace `__REPO__`) → `~/Library/LaunchAgents/com.evium.heartbeat-independent.plist`.
  2. `launchctl load -w` it. It ships **DRY_RUN** (no `--apply`) — satisfies the 24h dry-run mandate. `StartInterval 600` (10 min).
  3. After 24h of clean dry-run beats (`last-independent-tick.json` advancing every ~10 min), flip to `--apply` (append `<string>--apply</string>`, bootout + reload).
  - The agent's role ends at "describe how." The template + procedure already exist and are correct; nothing in the *code* needs fixing to re-arm — it needs the **user-gated `launchctl load`**.

## FINDING 3 — Heart state contaminated by a prior A23 run: `synthetic-a23` written into liveness fields

- **What:** Two Heart state files contain the literal string **`synthetic-a23`** instead of a tick id / epoch — left behind by an earlier A23 audit/test that wrote synthetic values into live state and didn't clean up.
- **Evidence:**
  - `.claude/cache/heartbeat/last-scheduler-tick-info.txt` → `synthetic-a23`
  - `.claude/cache/heartbeat/last-seen-tick.txt` → `synthetic-a23`
  - (Both file mtimes are 2026-05-26T23:11Z, i.e. they were last *written* in the 50h-stale window, but the *content* is a synthetic sentinel, not a real tick id.)
- **Severity:** **MEDIUM** (any consumer that parses these as a tick id/epoch will mis-handle them; also a true-time-adjacent hazard — synthetic values masquerading as real liveness data, which is the same class of "don't trust the displayed value" the true-time rule guards).
- **Recommendation (PROPOSE):** Reset these two files to a real value on next legitimate tick (or blank them so consumers treat them as "no prior tick"). More importantly: tests/audits that touch live Heart state must use a **sandboxed cache dir** (the repo already has `g6-test-sandbox/` and `scripts/heartbeat/test-fixtures/` for exactly this) — never write sentinels into `.claude/cache/heartbeat/`. Add a guard/CoT note so a future A23-style run can't re-contaminate.

## FINDING 4 — True-time HAZARD: `stat -f %Sm ... -t "...Z"` prints LOCAL time labelled as UTC

- **What:** Formatting a file mtime with BSD `stat -f %Sm -t "%Y-%m-%dT%H:%M:%SZ"` renders the time in the **local timezone** (EDT, `-0400`) but appends a literal **`Z`** — producing a string that *claims* UTC but is 4h off. This is a live foot-gun: it nearly produced a false "4-hour drift" finding in this very audit before I cross-checked against the raw epoch.
- **Evidence:**
  - My own first probe: `stat -f "... mtime=%Sm" -t "%Y-%m-%dT%H:%M:%SZ" last-independent-tick.json` → `mtime=2026-05-28T17:11:57Z`.
  - But the file's `epoch` field (and raw `stat -f %m`) = `1780002717`, and `date -u -r 1780002717` = **`2026-05-28T21:11:57Z`**. The `17:11:57Z` was local EDT mislabelled `Z`. `date +%z` → `-0400` confirms the 4h offset.
  - The JSON's internally-written `"ts"` is **correct** (`21:11:57Z`) because the script uses `/bin/date -u +...` (`heartbeat-independent-tick.sh:110`). So the bug is **not** in the emitted state — it's a trap for *readers/auditors* using `stat %Sm` to interpret timestamps.
- **Severity:** **MEDIUM** (no current script persists a `stat %Sm`-formatted UTC string that I found — emission uses `date -u` correctly — but any hook/audit that *reads back* mtimes this way will compute wrong ages/drifts; it's a latent recurrence of the A36/A37 true-time class).
- **Recommendation (PROPOSE):** Codify in `feedback_true-time-discipline` + `agentic-script-design` time ref: **never** format a UTC string from `stat %Sm`. To get a file's UTC mtime, take the **raw epoch** (`stat -f %m` / `stat -c %Y`) and convert with `date -u -r <epoch>`. `organic-os-pulse.sh` already does the safe thing (`_file_age_secs` uses raw `%m` and subtracts `date +%s` — both epoch, no tz). Add a one-line lint/grep guard for `%Sm.*Z` in scripts.

## FINDING 5 — True-time discipline elsewhere: CLEAN (positive verification)

- **What:** Per scope item 2, I checked timestamp emission across the Heart/scheduler/pulse hooks and the independent tick for the four forbidden patterns (hardcoded dates in emitted text; duration arithmetic on natural-language text; time inferred from message/token counts; non-canonical sources). No violations found in emission paths.
- **Evidence:**
  - `heartbeat-independent-tick.sh:108,110,111` — `NOW_ISO="$(/bin/date -u +%Y-%m-%dT%H:%M:%SZ)"`, `NOW_EPOCH="$(/bin/date +%s)"`, log uses `/bin/date -u`. Pending-staleness math is epoch arithmetic (`(NOW_EPOCH - PMT)/3600`, `:225`), not text parsing.
  - `organic-os-pulse.sh:50-55` — ages from raw `stat %m` / `stat -c %Y` minus `date +%s`; no string-time math.
  - `clock-drift.json` — `"source":"sntp"`, `drift_sec: 0.048`, `"ok"` — NTP-verified path exists and the system clock agreed with SNTP to 48ms (caveat: that check is ~50h old; re-running it is cheap and worth wiring into a tier-low beat).
  - Canonical source confirmed present: `scripts/utils/now-iso.sh` exists (referenced across the system); the scripts inspected use `date -u`/`/bin/date -u` directly, which is equivalent and acceptable.
- **Severity:** **INFO** (this is the "good news" half of the verification — emission discipline holds; the only time bugs are the *reader-side* `%Sm` hazard (F4) and the synthetic contamination (F3)).
- **Recommendation:** Keep `now-iso.sh` as the documented single source; consider having scripts call it rather than inlining `date -u` so there's exactly one definition to audit (minor DRY win, not urgent).

## FINDING 6 — Three-way integration & clock consistency: CONSISTENT, but the drift check itself is stale

- **What (scope item 3):** scheduler-events, the heartbeat tier state, and the time source all reference the **same wall clock** with no detectable inter-substrate drift. The scheduler's last `tick_end` (21:03:12Z) and the outbox mtime (21:00:43Z) agree to ~2.5 min (the gap is just "tick ran, then the resulting event was appended"). All epoch↔ISO conversions in every file round-trip cleanly against `date -u`.
- **Evidence:**
  - scheduler ts strings are plain UTC `Z` (`"ts":"2026-05-27T21:03:11Z"`), matching the tier files' epoch semantics.
  - `ack-latency.jsonl` (the reports-side latency substrate; canonical copy at `.claude/cache/ack-latency.jsonl`, last write 2026-05-28T21:46Z local / mtime) uses `measured_at` + `received_at` UTC strings and an integer `latency_ms` — e.g. `{"measured_at":"2026-05-28T09:33:56Z","received_at":"2026-05-28T09:30:29Z","latency_ms":207000}`; 207000 ms = 3m27s, which matches the two ISO stamps exactly. **Latency math is consistent with the clocks** (no fabricated durations). (One historical entry shows `latency_ms: 20623000` ≈ 5.7h, a genuine long ack, not a clock error — `received` 05-27T23:43Z vs `measured` 05-28T05:27Z.)
  - `clock-drift.json` drift 0.048s "ok" — system clock ≈ NTP.
- **Severity:** **LOW** (integration is sound; the only weakness is staleness of the *verification*, not the clocks).
- **Recommendation (PROPOSE):** Wire the SNTP `clock-drift.json` refresh into a **tier-low Heart beat** (it's cheap, read-only, no LLM) so drift is re-verified on cadence instead of going 50h between checks. This is one of the under-utilized Role-6 cross-substrate checks the heartbeat ref already lists.

---

## Ranked fixes (most-leverage first)

| # | Fix | Addresses | Effort | Risk | Gated? |
|---|-----|-----------|--------|------|--------|
| **1** | **Re-point `check_heart()` to the beat beacon** (`last-independent-tick.json` / `last-drain-ack.txt` / `last-tick-high.txt`), demote outbox-age to detail line | F1 (false alarm) | S (single function in one script) | Low | No — but it's a hook edit; log + verify with `--probe` |
| **2** | **User-gated install of `com.evium.heartbeat-independent.plist`** (24h dry-run → `--apply`) to give the Heart a real wall-clock cadence | F2 (real outage; closes the A74-D1 SPOF for good) | S to render, but **launchctl = high-risk-gated** | Med | **YES — user runs `launchctl load`; agent never installs** |
| **3** | **Codify the `stat %Sm`+`Z` ban** in true-time ref + add a grep-lint guard; standardize on `epoch → date -u -r` for reading mtimes | F4 (latent reader-side time bug) | S | Low | No |
| **4** | **Clean `synthetic-a23` out of Heart state** + make audits/tests use the sandbox cache, never live `.claude/cache/heartbeat/` | F3 (contamination) | S | Low | No (touches cache state, not src/) |
| **5** | **Wire SNTP drift refresh + (eventually) the tier-medium/low Role-3/5/6 checks into the independent beat** so the under-utilized Heart roles actually run on cadence | F2, F6 | M | Low (dry-run first) | Partially (depends on #2 timer) |

**Sequencing:** #1 is the highest-leverage, lowest-cost move — it stops the Heart from crying wolf at every SessionStart and is a pure-read hook edit. #2 is the structural fix for the *actual* outage but is user-gated (launchctl). #3/#4 are cheap hygiene that prevent recurrence of the two time-class bugs this audit surfaced. #5 is the "make the Heart earn its keep" follow-on once #2 gives it a timer.

---

## Direct answers to the four scope questions

1. **Heart flatline root-cause:** The firing mechanism is **event-driven hooks**, not a timer — `organic-os-pulse.sh`@SessionStart, `heartbeat-drain.sh`@UserPromptSubmit, `scheduler-heartbeat-coordinator.sh`@PostToolUse[Bash] (`settings.json:77/112/290`). The **scheduler stopped emitting at 2026-05-27T21:03Z** and the **tier ticks stopped at 2026-05-26T23:11Z**, so the *reactive* Heart path went dark (the A74-D1 SPOF). It is **supposed to be both** — daemon-fired via launchd *and* agent/hook-fired — but the **launchd timer for the Heart (`com.evium.heartbeat-independent.plist`) was NEVER installed** (only daemon10/2/3/4/5 are loaded). So today the Heart only beats when an agent turn drives the hooks, plus 5 manual dry-run backstop ticks on 05-28.
2. **True-time discipline:** Emission is **clean** — `date -u`/`/bin/date -u`/epoch arithmetic throughout; `now-iso.sh` is the documented single source; SNTP drift was 0.048s "ok". **One real hazard** (F4): `stat %Sm -t "...Z"` prints local time mislabelled UTC (4h off in EDT) — reader-side trap, not currently persisted but used in ad-hoc checks. **One contamination** (F3): `synthetic-a23` sentinels in two Heart state files.
3. **Three-way integration:** scheduler-events ↔ heartbeat ↔ time source are **clock-consistent** — same UTC wall clock, no inter-substrate drift; scheduler `tick_end` (21:03Z) ↔ outbox mtime (21:00Z) agree to ~2.5 min; `ack-latency.jsonl` `latency_ms` values match their ISO stamps exactly. No drift between what the scheduler "thinks" the time is and the system clock. The only weakness is that the drift *check* is 50h stale.
4. **Staleness semantics:** **"28h stale" is mostly a measurement artifact** — the pulse reads the event **outbox** mtime (last producer append 2026-05-27T21:00:43Z), not a beat beacon. The Heart's actual beat (`last-independent-tick.json`) is 4.7h fresh and daemon10/drain-ack updated within the last hour. **BUT** there is a **real narrower outage** behind the artifact: the scheduler-driven tier ticks (50h) and the event-producer cohort (28h) genuinely stopped, so the reactive/timer Heart roles aren't running. Both true: the *glyph* is wrong (Finding 1), and the thing it's accidentally pointing at is *also* partly broken (Finding 2).

---

## Files inspected (repo-relative)

- `scripts/hooks/organic-os-pulse.sh` (the observer; F1)
- `scripts/hooks/heartbeat-independent-tick.sh` (the backstop beat; F2/F4/F5)
- `scripts/hooks/heartbeat-drain.sh` (read-only outbox surfacer)
- `scripts/heartbeat/heartbeat-launchd.plist.template` (re-arm path; F2)
- `.claude/settings.json` (hook wiring: 77 / 112 / 290)
- `.claude/cache/heartbeat/` (last-*.txt tick state, last-independent-tick.json, last-drain-ack.txt; F2/F3)
- `.claude/cache/heartbeat-pending.jsonl` (the outbox the pulse misreads; F1)
- `.claude/cache/heartbeat-independent-tick.jsonl` (5 dry-run beats; F2)
- `.claude/cache/scheduler-events/2026-05-26.jsonl` (last tick 21:03Z; F2/F6)
- `.claude/cache/clock-drift.json` (SNTP 0.048s; F5/F6)
- `.claude/cache/ack-latency.jsonl` (latency↔ISO consistency; F6)
- `.claude/cache/daemon10-tier-low-poll.jsonl` (daemon10 alive 01:15Z; F2)
- `~/Library/LaunchAgents/com.evium.daemon10.plist` (the only installed Heart-adjacent timer; F2)
- `scripts/event-bus/consumers/consumer-heartbeat-tier-high.sh` + `scripts/monkey-click-fan-out.sh:244` (the outbox producers; F1)
- `.claude/skills/agentic-quality-discipline/references/heartbeat-system-discipline.md` (architecture model)

BG-COMPLETE-SENTINEL
