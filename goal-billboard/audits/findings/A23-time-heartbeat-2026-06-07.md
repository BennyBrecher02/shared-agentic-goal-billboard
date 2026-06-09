---
title: "A23 — time precision + scheduler/heartbeat/time three-way integration (READ-ONLY re-audit)"
date: 2026-06-07
run_at: 2026-06-07T19:35:03Z
audit_kind: system-infrastructure (heart / time-precision / three-way integration)
mode: READ-ONLY — no files touched besides this one; no LaunchAgents installed; no git/server runs; no spawn_task
trigger: "A25 audit flagged a Heart SPOF (launchd heartbeat uninstalled). A23 re-audit of (1) Heart SPOF, (2) time precision via now-iso single source, (3) scheduler↔heartbeat↔time integration."
supersedes_context: A23-time-precision-integration.md (2026-05-29), heart-subsystem-audit-2026-06-02.md
serves_northern_star: G2
belongs_to_goal: G1
serves_guiding_light: G7
---

# A23 re-audit — time precision + heartbeat SPOF + three-way integration (2026-06-07)

All timestamps below are from `date -u` / `scripts/utils/now-iso.sh` / file-mtime-derived epochs — never inferred.
Capture time: **2026-06-07T19:35:03Z** (epoch 1780860903).

---

## TL;DR (the headline)

The A25 "Heart SPOF — launchd heartbeat uninstalled" flag is **half right and now misleading**. Since the
prior A23 (2026-05-29) and the 2026-06-02 heart-subsystem audit, the two HIGH findings they raised were
**both fixed**:

- **F1 (pulse read the outbox, not the beat) — RESOLVED.** `organic-os-pulse.sh:95` now reads the beat
  beacon `heartbeat/last-independent-tick.json` directly.
- **The independent-tick launchd timer — INSTALLED + BEATING.** `com.evium.heartbeat-independent` is
  `launchctl`-loaded and has beaten **326 times on a clean ~600s cadence since 2026-06-02T06:21:50Z**
  (last beat 2026-06-07T19:25:26Z, beacon age 576s at capture). The A74-D1 SPOF backstop is live.
- **F3 (synthetic-a23 contamination) — RESOLVED.** `last-scheduler-tick-info.txt` / `last-seen-tick.txt`
  are gone; no `synthetic` strings remain in `.claude/cache/heartbeat/`.

So `heart_installed = Y` — the SPOF-proof core heart IS installed. **But the real, still-open exposure is
different and more subtle than "uninstalled":** the installed heart has run in **DRY-RUN for ~5.5 days**
(326 beats, **zero** apply-mode beats), so it records a fresh beacon (→ pulse shows HEART ✓) while
performing **none of its actual work** (event-bus cursor-advance + tier-high consumer firing are JOB 1/2,
**skipped in dry-run**). The pulse cannot tell a *working* heart from a *zombie* one. That, plus an
**unverified wall clock** (`verify-clock.sh` is wired to no cadence; `clock-drift.json` does not exist),
are the two findings that matter.

What the A25 flag conflates: the **tiered dispatcher** `tick.sh` (`com.evium.heartbeat`) is genuinely NOT
installed — but per the 2026-06-02 audit that is a deliberate CHOICE, not the SPOF. The SPOF core (the
independent tick) is installed. Don't re-install the tiered heart on the strength of the SPOF flag alone.

---

## Heart liveness — verified state (the evidence spine)

| Item | Value | Source |
|------|-------|--------|
| Independent-tick launchd job | **LOADED** (`com.evium.heartbeat-independent`, LastExitStatus 0) | `launchctl list` |
| Tiered dispatcher launchd job | **NOT loaded** (`com.evium.heartbeat`) | `launchctl list com.evium.heartbeat` → "Could not find service" |
| Independent beats logged | **326** | `.claude/cache/heartbeat-independent-tick.jsonl` |
| Beat cadence | **~600–605s, no gaps** | last 12 beats all +600/601s |
| First / last beat | 2026-06-02T06:21:50Z / 2026-06-07T19:25:26Z | beat log |
| **apply-mode beats** | **0** (all 326 are `"mode":"dry-run"`) | `grep '"mode": "apply"'` → 0 |
| Beacon freshness | 576s (✓ <6h) | `last-independent-tick.json` mtime |
| event-bus.jsonl | **ABSENT** | `.claude/cache/event-bus.jsonl` |
| event-bus cursor | **blank file** (0 bytes) | `.claude/cache/event-bus-cursor.txt` |
| scheduler-events/ | **EMPTY** (no `*.jsonl`) | `.claude/cache/scheduler-events/` |
| clock-drift.json | **ABSENT** | `.claude/cache/clock-drift.json` |
| 5hr-window-reset.txt | **ABSENT** | `.claude/cache/5hr-window-reset.txt` |
| now-iso single source | present + correct | `scripts/utils/now-iso.sh` |
| Time-precision audit verdict | **0 format anomalies, 0 TZ anomalies, 0 cross-event drift** | `run-time-precision-audit.py --dry-run` (336 scripts, 53 JSONLs, 3991 records) |

---

## SECTION 1 — Heart SPOF

### F1 [HIGH] — The installed heart is a "zombie": 5.5 days of dry-run, JOB 1 + JOB 2 never execute
- **What:** `com.evium.heartbeat-independent` runs `heartbeat-independent-tick.sh` with **no `--apply`**
  (the plist's ProgramArguments are bash + the script, nothing else). Per the script's own contract, dry-run
  does JOB 3 only (record the beat); **JOB 1 (advance event-bus cursor + fire tier-high consumers via
  `dispatch.sh`) and JOB 2 (bound/rotate `heartbeat-pending.jsonl`) are skipped.** It has been in this state
  for **~5.5 days / 326 beats** — far past the script + plist's own **24h dry-run mandate** before going live.
- **Why it matters:** the beacon (JOB 3) is written every 10 min regardless of mode, so the pulse's HEART
  glyph is ✓ and `last-independent-tick.json` is always fresh — **the heart looks fully alive while doing
  none of its real work.** If a producer *were* appending to the event bus, those events would sit unconsumed
  forever (cursor never advances), and tier-high reactions (the inbox-urgency surfacing the independent tick
  is meant to keep reactive) would never fire. This is the exact "looks-alive-but-isn't" failure class the
  true-time canary + the pulse were built to catch — and the pulse is blind to it.
- **Severity:** **HIGH** (silent: the one observer reports ✓; the heart's purpose is unmet).
- **Fix (one line, A66-GATED — PROPOSE, do not apply):** after confirming the (already-satisfied) 24h
  dry-run window, flip the plist to live — add `<string>--apply</string>` to ProgramArguments, then
  `launchctl bootout` + reload. *User runs the launchctl step; agent never installs.*

### F2 [MEDIUM] — Pulse HEART glyph has a dry-run blind spot (can't see mode)
- **What:** `organic-os-pulse.sh check_heart()` judges liveness purely by the beacon **file mtime**
  (`:97-111`). It never reads the beacon JSON's `"mode"` field, so it cannot distinguish `dry-run` (zombie)
  from `apply` (working). F1 is invisible precisely because of this.
- **Severity:** **MEDIUM** (observability gap that masks F1; not a liveness fault itself).
- **Fix (one line, ungated hook edit — PROPOSE):** in `check_heart()`, parse the beacon's `mode`; if
  `mode == "dry-run"`, render HEART as `◑` with detail `"beating (dry-run — not applying)"` instead of a
  clean `✓`. Pure-read, log + verify before/after.

### F3 [LOW] — No independent redundancy for the heart's own timer (single launchd job)
- **What:** the SPOF fix gave the heart its OWN launchd cadence (good — survives scheduler/agent-action
  death, the A74-D1 win). But that is now itself a single point: if `com.evium.heartbeat-independent` is
  booted out (or the plist removed), the heart stops with no second timer. `daemon10` (hourly) is the only
  other independent launchd cell and does NOT replicate the heart's JOB 1/2.
- **Severity:** **LOW** (the design explicitly trades the multi-driver SPOF for one robust launchd timer;
  launchd `KeepAlive` + `RunAtLoad` already auto-restart it on login/crash). Documented for completeness;
  no action recommended beyond F1.
- **Fix:** none required. If belt-and-suspenders is wanted later: a SessionStart guard that re-`load`s the
  plist if `launchctl list` doesn't show it (A66-gated, low value given launchd already persists it).

### Dependency map — what concretely breaks if the heart is down
- **If the beacon stops (timer booted out):** `organic-os-pulse.sh` flips HEART → ◑ (>6h) → ✗ (>24h) — the
  ONLY automated detector. Nothing else reads the beacon. No daemon/vital-sign hard-depends on it for
  correctness — its job is *surfacing*, not core logic.
- **If JOB 1 is dead (current dry-run state):** event-bus events (when produced) never reach tier-high
  consumers via the independent path; `heartbeat-pending.jsonl` is fed only by the agent-turn-driven paths
  (`consumer-heartbeat-tier-high.sh`, `monkey-click-fan-out.sh`) + surfaced by `heartbeat-drain.sh`
  (UserPromptSubmit). So between-session reactivity that the independent tick promises is absent.
- **Currently moot:** `event-bus.jsonl` is absent and `scheduler-events/` is empty, so there are **no events
  to drop** right now. The exposure is latent — it bites the moment a producer starts appending while the
  tick is still dry-run.

---

## SECTION 2 — Time precision

### F4 [HIGH] — Wall clock is UNVERIFIED: `verify-clock.sh` wired to no cadence; `clock-drift.json` absent
- **What:** the true-time discipline (Discipline 3) says NTP-verify the wall clock and consult
  `.claude/cache/clock-drift.json`. That file **does not exist** in this checkout, and `verify-clock.sh` is
  **not** in SessionStart (or any) hooks — grep of `settings.json` + all hooks finds no caller. The prior A23
  saw a 50h-stale drift check; it is now *infinitely* stale (never run here).
- **Why it matters:** every timestamp the system emits, and every staleness/drift judgment, rests on the Mac
  wall clock being correct. With no NTP check, a silently-drifted clock would make all "X happened at …",
  all heartbeat staleness bands, and the 5hr-window inference wrong with **zero** signal. This is the
  foundational dependency of the whole time discipline, unmonitored.
- **Severity:** **HIGH** (foundational; silent-failure mode — the very thing the discipline exists to prevent).
- **Fix (one line, ungated — PROPOSE):** add `bash scripts/utils/verify-clock.sh` (or a cached-mtime wrapper)
  to SessionStart hooks so `clock-drift.json` is produced + refreshed; surface `drift_status != ok` in the
  state-batch line. (Discipline doc already specifies SessionStart cadence + the warn/critical bands.)

### F5 [LOW] — Non-canonical timestamp form (`+00:00`) emitted by 4+ Python hooks instead of `Z`
- **What:** `recurrence-detect.sh:586`, `parallel-capacity-check.sh:165`, `hedge-detect.sh:190`,
  `ns-advance-check.sh:241` emit `datetime.now(timezone.utc).isoformat()` → `…+00:00`. The discipline (Python
  callers, time-precision-coordination.md L93-94) mandates `.strftime("%Y-%m-%dT%H:%M:%SZ")` — the `Z` form.
  Live JSONLs confirm it: `hedge-detections`, `ns-advance-check`, `parallel-capacity-log`, `recurrences`,
  `repeated-request-launches` all store `ts:iso-offset`.
- **Why it matters (mildly):** the values are correct UTC and sort identically, BUT they are NOT the
  canonical `Z` form, and `run-time-precision-audit.py` buckets `iso-offset` SEPARATELY and **does not count
  it in the anomaly totals** — so the audit reports "0 TZ anomalies" while a real format-inconsistency exists.
  Mixed `Z` + `+00:00` across files forces every reader to normalize.
- **Severity:** **LOW** (cosmetic/consistency; no wrong-time risk; not text-arithmetic).
- **Fix (one line each — PROPOSE):** replace `.isoformat()` with
  `.strftime("%Y-%m-%dT%H:%M:%SZ")` in those emitters; OR (better leverage) make `run-time-precision-audit.py`
  *count* `iso-offset` as a TZ anomaly so the discipline self-enforces.

### F6 [LOW] — 9 capture/stats scripts use local-time `date` / naive `datetime.now()`
- **What:** `run-time-precision-audit.py` flags 9 files: `android-emulator-capture.sh`,
  `ios-simulator-capture.sh`, `mobile-sim-sweep-orchestrator.sh`, `run-meta-loop-eval.sh`,
  `run-deep-audit.sh`, `migrate-memory-on-move.sh`, `daily-stats.sh` (local `date +%…`), and
  `run-metrics-aggregator.py`, `render-stats-data.py` (`datetime.now()` naive).
- **Why it matters:** none are in the heart/scheduler/time CORE; mostly local filename stamps or display
  strings. But naive `datetime.now()` in the metrics aggregator/renderer can leak local time into
  user-facing dashboard numbers.
- **Severity:** **LOW** (periphery; verify the aggregator's naive `now()` isn't used for a persisted `ts`).
- **Fix (PROPOSE):** swap local `date +%…` → `date -u +%…` (or `now-iso.sh`); swap `datetime.now()` →
  `datetime.now(timezone.utc)` in the two Python files. Cosmetic for the `.sh`, worth doing for the `.py`.

### Time-precision — positives (verified, not assumed)
- **Single-source emission is clean in the core:** the heart's own emission uses `/bin/date -u +%Y-%m-%dT%H:%M:%SZ`
  (`heartbeat-independent-tick.sh:114/116`) — the exact body of `now-iso.sh`; equivalent + canonical `Z`.
  No hardcoded dates, no text-arithmetic, no message/token-count duration inference anywhere in the heart/
  scheduler/coordinator paths.
- **Cross-event drift = 0** between scheduler-events and heartbeat-pending (vacuously, since both are empty/
  absent — see F7), and **format/TZ anomalies = 0** by the checker's count across 3991 records.

---

## SECTION 3 — Three-way integration (scheduler ↔ heartbeat ↔ time)

### F7 [MEDIUM] — The integration is wired + sound in design, but runs on ZERO live data (vacuous-pass risk)
- **What:** every integration mechanism exists and is correctly coded, but the producer side is dark:
  - `scheduler-events/` is **empty** — no scheduler tick has ever been logged in this checkout.
  - `event-bus.jsonl` is **absent**; cursor is **blank**.
  - therefore `check-scheduler-health.sh`'s **median-aware stuck-detection** (Part 3) and **tick-completion
    cross-event info** (which would repopulate `last-scheduler-tick-info.txt:259`) have **no input**;
  - the **cross-event drift check** (`run-time-precision-audit.py crosscheck_correlated_events`) passes
    **vacuously** — there are no `tick_end` ↔ hb-pending pairs to compare;
  - `scheduler-heartbeat-coordinator.sh` (PostToolUse[Bash]) early-outs on every command (no scheduler
    invocations happen), and even when it *would* fire it calls `tick.sh --force-tier medium` — but
    `tick.sh`'s launchd is uninstalled, so it runs the script ad-hoc (works, but its tier-state lives in
    `.claude/cache/heartbeat/last-tick-*.txt`, which are **also absent** here → no tier timer state at all).
- **Why it matters:** "0 drift / 0 anomalies / coordinator wired" reads GREEN, but it's GREEN-because-empty,
  not GREEN-because-correct. The first real scheduler tick would exercise paths that have **never run** in
  this checkout. The 5hr-window-reset inference (Discipline 6) is in the same state — `detect-5hr-reset.py`
  exists, `token-budget-check.sh` has the auto-refresh path (`:39-55`), but it only fires **when
  `5hr-window-reset.txt` already exists and is stale**; the file is absent, so the inference is **dormant**
  (nothing bootstraps the first write on cadence).
- **Severity:** **MEDIUM** (no active bug; meaningful silent-failure / untested-path risk — the integration's
  correctness is unproven against live data, and two inference subsystems are dormant by absence).
- **Fix (PROPOSE):**
  1. Bootstrap the 5hr inference: run `python3 scripts/detect-5hr-reset.py --apply` once (ungated, read-only
     over session JSONLs) so `5hr-window-reset.txt` exists and the auto-refresh loop in `token-budget-check.sh`
     can self-sustain. *(One command; not gated.)*
  2. Tie clock-verify + 5hr-refresh + (eventually) the tier-medium/low checks to the **independent beat once
     it is in `--apply`** (F1) so the under-utilized Role-3/5/6 heart checks actually run on cadence rather
     than only on agent turns. *(Depends on F1; A66-gated via the same plist flip.)*
  3. No silent-failure in the drift check itself — but add a `tick_count > 0` guard to its report line so a
     vacuous pass is labeled "no events to cross-check" rather than implying verified agreement.

### Three-way — positives (verified)
- All timestamps across the (currently sparse) substrates reference the **same Mac wall clock**; the heart's
  epoch arithmetic for staleness is epoch-based (`(NOW_EPOCH - PMT)/3600`), not string math — no inter-source
  drift is *possible* in the current data because everything derives from one `date` source.
- The coordinator's fire-decision uses wall-clock delta with a 0–60s window (`scheduler-heartbeat-coordinator.sh:73-78`)
  — sound; monotonic isn't needed there (it's an "is this event fresh?" check, not an interval measurement).

---

## Ranked fixes (most-leverage first)

| # | Fix | Addresses | Sev | Effort | A66-Gated? |
|---|-----|-----------|-----|--------|-----------|
| **1** | **Flip independent-tick plist to `--apply`** (24h dry-run long satisfied) so JOB 1/2 actually run | F1 (zombie heart) | HIGH | S (plist edit + reload) | **YES — PROPOSE; user runs launchctl** |
| **2** | **Wire `verify-clock.sh` into SessionStart** → produce/refresh `clock-drift.json`; surface non-ok drift | F4 (unverified clock) | HIGH | S (one hook line) | No (hook edit → settings.json: gated to USER) |
| **3** | **Teach `check_heart()` to read beacon `mode`** → render dry-run as ◑, not ✓ | F2 (pulse blind spot) | MED | S (one function) | No (hook script edit) |
| **4** | **Bootstrap 5hr inference** — `detect-5hr-reset.py --apply` once so the auto-refresh loop self-sustains | F7 (dormant inference) | MED | XS (one command) | No |
| **5** | **Label the vacuous cross-event-drift pass** ("0 events to compare") + count `iso-offset` as a TZ anomaly | F5, F7 | LOW | S (audit-script edits) | No |
| **6** | **Normalize `.isoformat()` → strftime `Z`** in the 4 hooks; `date -u` / `now(timezone.utc)` in the 9 periphery scripts | F5, F6 | LOW | S (mechanical) | No |

**Note on #2/#3:** these are hook/settings edits. Editing `.claude/settings.json` is itself on the
A66 high-risk-always-gated list (settings-edit-guard blocks the agent) — so #2's wiring is USER-applied;
#3 edits a hook *script* (ungated) but I am surfacing it as PROPOSE per the read-only mandate.

**Sequencing:** #1 + #2 are the two HIGHs and independent of each other — #1 makes the heart do its job, #2
makes the clock it relies on trustworthy. #3 closes the observability hole that hid #1. #4 is a one-command
freebie. #5/#6 are hygiene that make the audit self-enforcing.

---

## Direct answers to the three scope questions

1. **Heart SPOF:** The SPOF-proof CORE (`com.evium.heartbeat-independent`) **IS installed and beating**
   (326 clean 600s beats since 06-02) — so `heart_installed=Y`. The A25 "uninstalled" flag points at the
   **tiered `tick.sh` (`com.evium.heartbeat`)**, which is deliberately uninstalled (per the 06-02 audit) and
   is NOT the SPOF. The **real** exposure is that the installed heart is in **dry-run** (F1): it beats but
   does no work (JOB 1/2 skipped), and the pulse can't see that (F2). Redundancy: one robust launchd timer
   (acceptable by design, F3). Nothing hard-breaks today because there are no events to drop, but the moment
   a producer appends, the dry-run heart drops them silently.
2. **Time precision:** Emission through the single source is **clean in the core** (heart uses the exact
   `now-iso.sh` body; no text-arithmetic anywhere). The single REAL gap is that the **wall clock is never
   NTP-verified** here (F4 — `clock-drift.json` absent, `verify-clock.sh` unwired). Minor: 4 hooks emit
   `+00:00` instead of canonical `Z` (F5) and 9 periphery scripts use local/naive time (F6). No drift between
   tick times, heartbeat ts, and wall-clock was found — but only because everything derives from one
   `date` source and there's little live data.
3. **Three-way integration:** Wired and **sound in design** — coordinator wall-clock fresh-window check,
   median-aware stuck detection, tick-completion cross-event info, 1s cross-event drift tolerance all exist
   and are correctly coded. But it runs on **ZERO live data** (F7): scheduler-events empty, event-bus absent,
   so the drift check passes vacuously and the median/stuck/tick-completion paths have never executed here.
   The **5hr-window-reset inference is dormant** (file absent; auto-refresh only fires if the file already
   exists). Silent-failure modes: (a) the dry-run heart silently dropping future events (F1); (b) the
   unverified clock silently invalidating every time judgment (F4); (c) the vacuous "0 drift" pass reading
   as "verified" when it means "nothing to verify" (F7).

---

## Reconciliation with prior audits
- **A23 (2026-05-29):** its F1 (pulse-reads-outbox) and F2 (no launchd timer) are **FIXED**; F3
  (synthetic-a23) is **gone**; F4 (`stat %Sm`+Z reader hazard) — not re-checked exhaustively but no
  persisted `%Sm`-Z string found; F6 (wire SNTP refresh into a beat) is **still open** = this audit's F4.
- **heart-subsystem-audit (2026-06-02):** confirmed correct — independent-tick installed + beating, tiered
  uninstalled-by-choice. This audit ADDS the finding that 06-02 didn't flag: the installed tick has stayed
  **dry-run** ever since (zombie-heart, F1).

## Files inspected (repo-relative)
- `scripts/hooks/heartbeat-independent-tick.sh` (the beat; F1/F2)
- `scripts/hooks/organic-os-pulse.sh:89-112` (HEART glyph; F2)
- `scripts/hooks/scheduler-heartbeat-coordinator.sh` (PostToolUse coordinator; F7)
- `scripts/hooks/scheduler-tick-drift-warn.sh` (PreToolUse drift warn)
- `scripts/heartbeat/check-scheduler-health.sh` (hourly tier; median/stuck/tick-completion; F7)
- `scripts/hooks/token-budget-check.sh:34-55` (5hr auto-refresh; F7)
- `scripts/utils/now-iso.sh` (single source; F5/F6 baseline)
- `scripts/run-time-precision-audit.py` (the Layer-1 checker; F5/F7)
- `~/Library/LaunchAgents/com.evium.heartbeat-independent.plist` (dry-run ProgramArguments; F1)
- `.claude/settings.json` (SessionStart / UserPromptSubmit / PostToolUse wiring; F2/F4)
- `.claude/cache/heartbeat/` (beacon + absent tier-state; F1/F3), `heartbeat-independent-tick.jsonl` (326 dry-run beats)
- `.claude/cache/{event-bus.jsonl(absent), event-bus-cursor.txt(blank), scheduler-events/(empty), clock-drift.json(absent), 5hr-window-reset.txt(absent)}`
- `.claude/skills/agentic-script-design/references/time-precision-coordination.md` (the discipline)
- prior: `A23-time-precision-integration.md`, `heart-subsystem-audit-2026-06-02.md`

BG-COMPLETE-SENTINEL: (issues_found=7, critical_count=0, heart_installed=Y, status=read-only-audit-complete; 2 HIGH [F1 dry-run-zombie-heart, F4 unverified-clock], 2 MEDIUM [F2 pulse-blind-spot, F7 vacuous-integration], 3 LOW [F3 single-timer, F5 iso-offset-form, F6 periphery-local-time]; top gated fix=flip independent-tick plist to --apply [A66 PROPOSE])
