---
id: A83
status: completed
type: reflective-audit
mode: READ-ONLY
captured: 2026-05-29T08:15Z
theme: "Built but never RUN / QUERIED / WIRED / SURFACED — the A74 meta-finding + Convention F, swept system-wide. Plus dropped-user-ask hunt."
trigger: "Agent built scripts/analyze-token-burn.py then answered a user cost question from memory instead of running it. Prior capability audit (A74) found crontab empty + autonomic daemons dark + designed-crons-never-installed + daemon10 erroring."
scope: "scripts/ orphan census · plans/ designed-not-wired · daemons dark/firing · producers-with-no-surfacer · disciplines that describe-but-don't-run (impact-domino, idea-gap)"
prior_art:
  - "context/markdowns/research/systems/built-but-not-used-analytics-audit.md (2026-05-29T07:45Z) — deep dive on the 7 ANALYTICS subsystems. A83 does NOT re-derive it; cites + extends with the broader orphan census + the discipline-enforcement check + a live daemon-state DELTA."
  - "A74 — originating 'built but not used/queried' meta-finding + Convention F."
  - "context/markdowns/research/systems/cot-brain-ledger-health.md — CoT ledger specific."
verified_counts:
  scripts_total: 239
  hook_scripts: 71
  hooks_wired_in_settings: 79
  plan_files: 130
  daemons_loaded: 5
  daemons_designed_per_organic_os_pulse: 8
---

# A83 — Internal Build-But-Surface Gaps (system-wide sweep)

**All numbers measured live this pass (2026-05-29T08:00–08:15Z) via `launchctl list`, `crontab -l`, `grep` over settings.json + plists + scripts, and direct execution of the detectors.**

## TL;DR

The A74 "built-but-not-used" meta-finding is **not analytics-specific** — it is a system-wide modal failure. Beyond the 7 analytics subsystems already documented in `built-but-not-used-analytics-audit.md`, this sweep found **4 fully-built immune/safety hooks wired to NOTHING**, an **entire triage discipline (impact-domino) enforced by zero code**, **5 daemons that fire only at SessionStart boundaries instead of their claimed hourly cadence and are stuck in dry-run 2+ days past the 24h graduation gate**, a **daemon-liveness glyph that can never read healthy** (8 designed, 5 built), and an **autonomic layer with no boot-persistence reloader**.

**The pattern, restated:** we are excellent at *building producers and detectors* and structurally undisciplined about *wiring them to a trigger and a surface*. The scariest instances are not the analytics (cosmetic/cost) — they are the **safety/immune hooks** that are built, tracked, documented as "highest-leverage," and connected to no hook event at all.

**Good news (not gaps, verified working):** the idea-gap detector (`detect-idea-gaps.py`) is fully wired, ran this session, classified 59 prompts, found 0 gaps — the user's "never miss an idea" fear is structurally closed. The token-budget hook, dashboard prime, and bug-billboard consolidate are all live. daemon10 ticks healthily.

---

## TOP 5 SCARIEST (lead)

### #1 — `bottleneck-detect.sh` is the Immune-system centerpiece enforcing the user's MOST-REPEATED rule, and it is wired to no hook event. [discipline-not-enforced + orphaned-script · RISK: CRITICAL]

- **What it is:** the INNATE-side Immune detector that enforces the constitutional chant *"i dont ever want testing becoming a time/tokencost bottleneck"* (user 2026-05-26T23:18Z, repeated 25×). 8 detection signals (cold-cache test >15min, ghost BG >2h, queue depth >50, token burn, recurrence 3+, hook latency >5s, dashboard chain >30s, snapshot drift >100). Its own header says A74-D1 already flagged it was "DESIGNED but NEVER BUILT," then it WAS built.
- **The gap:** `grep bottleneck-detect.sh .claude/settings.json` = **0 occurrences.** It is NOT a SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, or Stop hook. The only references are doc-comments in `organic-os-pulse.sh` (checks its *existence*, not invokes it) and `sample-resources.sh` (a "proposed wiring" comment). Its last write to `bottleneck-detect.jsonl` was **2026-05-28T17:22Z** in dry-run mode — and it caught REAL signals then (30 ghost BGs at 31.6h, A66 recurring 11×, `detect-script-candidates.sh` p95=15.6s) that **surfaced to nobody** because nothing runs it.
- **Why scary:** the system's single most-emphasized user constraint is enforced by a detector that fires only when manually invoked. The 6-step immune response (DETECT→PAUSE→DIAGNOSE→FIX→VERIFY→RESUME) can't start because step 1 never auto-runs.
- **Minimal fix:** add one Stop-hook (or UserPromptSubmit, throttled) line in settings.json: `bash -c '[ -z "$EVIUM_BOTTLENECK_DETECT_OFF" ] && bash scripts/hooks/bottleneck-detect.sh || true'`. The script already self-guards to dry-run and emits a `<system-reminder>` only when a signal trips. Per its own design + the chant memory rule, do the 24h dry-run window (it has had 1 day already) then flip to BLOCK-mode via its env flag. **Cost: one settings line.**

### #2 — `wrapper-only-write-guard.sh` ("the single highest-leverage move in the retro-sweep," closes 3 immunity holes) is DARK. [orphaned-script / designed-not-wired · RISK: CRITICAL]

- **What it is:** a PreToolUse `Write|Edit` shared hardware barrier its own header calls *"the single highest-leverage move in the retro-sweep,"* closing **three** adaptive-immunity instances at once (A57 dashboard-content, A48, A61) — enforcing wrapper-only writes to protected surfaces.
- **The gap:** `grep wrapper-only-write-guard.sh .claude/settings.json` = **0 occurrences.** Zero callers anywhere (`grep -rl` across scripts/workflows = only itself). It is git-tracked and fully written — it just was never added to the PreToolUse `Write|Edit` matcher block (which currently has `inbox-schema-validate-pre`, `settings-edit-guard`, `src-edit-guard`, `inbox-mutation-guard`).
- **Why scary:** three separate immunity incidents the team already spent effort diagnosing are "closed" on paper by a guard that intercepts nothing. The wall-of-text chamber post (A57) class can recur freely.
- **Minimal fix:** add to the existing PreToolUse `Write|Edit` hooks array in settings.json: `bash scripts/hooks/wrapper-only-write-guard.sh`. Verify it no-ops on non-protected paths first (read its allow-list logic). **Cost: one settings line.**

### #3 — Impact-domino triage discipline is 100% memory-only — NO script, hook, or workflow implements it. [discipline-not-enforced · RISK: HIGH]

- **What it claims (`feedback_impact-domino-analysis.md`, A67):** *"Every new plan/idea/comment/request gets domino-mapped + leverage-scored + stack-ranked... 5-step loop CAPTURE→DOMINO-MAP→SCORE→STACK→PROPOSE... slam-dunks rise to goal billboard as proposals."* Memory presents it as a STANDING PROTOCOL that fires on every new item.
- **The gap:** there is **no executable** anywhere. `ls scripts/*domino* scripts/*slam* scripts/hooks/*domino*` → none exist. The only "domino" code is `session-bundle/analyzer-domino-effectiveness.py` — which (a) measures *past* domino effectiveness retrospectively, not *prospective* triage, and (b) lives inside the manual-only session-bundle that has run exactly once. The `chamber-empty-detect.sh` and `idea-gap-surface.sh` grep hits are coincidental keyword overlaps (doc-pointer comments / unrelated `band_queue`), not enforcement. The `slam-dunk-handoff-workflow-plan.md` produced no script.
- **Why scary:** this is the textbook "mechanism that describes-but-doesn't-run." The discipline reads as automatic in memory; in reality every domino-map that has ever happened was the agent manually remembering to do it mid-turn — the exact failure mode that triggered THIS whole audit family (agent answering from memory instead of running the tool). It is the impact-domino equivalent of `analyze-token-burn.py`.
- **Minimal fix (two honest options):**
  - (a) **Demote the memory claim to reality:** edit `feedback_impact-domino-analysis.md` to say "agent-applied heuristic, on-demand" rather than "STANDING PROTOCOL that fires on every item" — removing the false-enforcement smell. (Cheapest; needs user OK per memory-edit discipline.)
  - (b) **Build the missing tether:** a UserPromptSubmit hook `impact-domino-prompt.sh` that, when a new-plan/idea trigger marker is present, emits a `<system-reminder>` "domino-map this before proceeding" (mirrors `user-ask-detect.sh`). This makes the protocol structurally fire instead of relying on memory.
  - Recommend (a) now (truth-in-labeling), (b) as a tracked follow-up. Do NOT leave it claiming automatic enforcement it does not have.

### #4 — All 5 autonomic daemons are stuck in dry-run 2+ days past the "24h-then-live" gate, AND fire only at SessionStart boundaries, not their claimed hourly cadence. [dark-daemon / designed-not-wired · RISK: HIGH]

- **Live state (measured 08:00Z):** all 5 daemons (2,3,4,5,10) ARE loaded (`launchctl list | grep com.evium` shows 5, all LastExit=0). **This is a DELTA from `built-but-not-used-analytics-audit.md` (07:45Z) which found only daemon10 loaded** — they got re-registered ~08:00Z, almost certainly by this session's startup. Good: the prior audit's "75% dark" is currently false.
- **But two deeper gaps remain:**
  1. **Not actually hourly.** Plists say `StartInterval 3600`, but the poll JSONLs prove otherwise: daemon2 has **3 polls total** across 34h (2026-05-27T21:56 → 2026-05-29T08:00), daemon3/4/5 have **only 2 polls each** (one on 05-27, one today). daemon10 alone has 35 polls (~hourly, healthy). So 2–5 do NOT survive between sessions — they die on logout/sleep and only re-fire at the next SessionStart-ish boundary. The "hourly autonomic grooming" A65 promised does not happen between sessions.
  2. **All 5 are dry-run (`dry_run:1`), and daemon3 reports `severity:critical gaps=6 xref=4` EVERY fire** — surfacing to nobody. The memory rule says "24h dry-run mandatory before live"; they have been dry-running since 2026-05-27 (2+ days). The graduation gate has long passed and nobody flipped them. So even when they DO fire, their critical findings evaporate.
- **Minimal fix:**
  - **Boot-persistence:** add `scripts/daemons/reload-all.sh` (idempotent `launchctl bootout`+`bootstrap gui/$UID` for the 5 plists) and call it from SessionStart (or fix the plists' `KeepAlive`/load semantics so logout doesn't drop them).
  - **Graduation:** run `daemon-graduation-check.sh` (it exists, logs to `daemon-graduation-check.jsonl`), confirm 24h clean, then flip each consumer's dry-run flag to live so daemon3's `gaps=6 xref=4` actually surfaces.
  - These are SAFETY-adjacent (daemon3 = audit-catalog integrity, daemon5 = bug-billboard) so rank above the analytics fixes.

### #5 — No daemon is in the services registry, so the SessionStart services-digest can NEVER detect a dark daemon; and the AUTONOMIC glyph can never read healthy (8 designed, 5 built). [producer-no-surfacer / monitoring-gap · RISK: MEDIUM-HIGH]

- **The monitoring blind spot:** `required-services.json` registers exactly 3 services (`agent-inbox-server` P0, `astro-dev` P1, `scheduler` P2). The 5 daemons are NOT in it. So `sessionstart-services-digest.sh` (which reads `services-check.sh` → that registry) **structurally cannot warn when a daemon is dark** — which is exactly why the prior audit's "daemons 2–5 silent since 05-27" went unnoticed for 2 days. The one surface that should catch runtime-orphaning is blind to it.
- **The unwinnable glyph:** `organic-os-pulse.sh` computes the AUTONOMIC vital sign as `live_count >= DAEMONS_DESIGNED` where `DAEMONS_DESIGNED=8`. Only **5 plists exist** (research-furthering has a `.plist.template` never deployed; heartbeat relies on `heartbeat-independent-tick.sh` not a plist; the A65 catalog listed 8 — cot-ledger-consolidator, snapshot-drift-scanner, statistical-correlator were never built as daemons). So the glyph reads `5/8 live ◑` **permanently** — it can never show `✓` even when everything that exists is running. A health indicator that can never be green is a broken indicator (alert fatigue → ignored).
- **Minimal fix:**
  - Add the 5 daemons to `required-services.json` with a daemon-class health check (`launchctl list | grep <label>` presence), so the digest flags a dark daemon. (Closes the monitoring blind spot — this is the structural fix for #4's "went unnoticed 2 days.")
  - Set `EVIUM_DAEMONS_DESIGNED=5` (or correct the constant) so the glyph reflects reality, OR build/deploy the 3 missing daemons. Recommend correcting the constant to 5 now + tracking the 3 unbuilt as explicit deferrals — don't leave a permanently-amber indicator.

---

## Full findings by zone

### Zone 1 — Orphaned scripts (zero callers)

Strict scan: for each of the 239 scripts, who references its basename in settings.json / plists / other scripts / workflows / package.json / plans / skill refs, *excluding self*. Excluded scheduler/ python-package internals, tests, `__init__`, `lib-common`, `now-iso.sh` (imported/sourced, not invoked-by-name).

**TRUE capability orphans (built, no trigger, should have one) — the gaps:**
| Script | Class | Note |
|---|---|---|
| `scripts/hooks/bottleneck-detect.sh` | discipline-not-enforced | **TOP #1** — immune centerpiece, no hook event |
| `scripts/hooks/wrapper-only-write-guard.sh` | orphaned-script | **TOP #2** — "highest-leverage," no PreToolUse wire |
| `scripts/hooks/inbox-server-guard.sh` | orphaned-script | A45 server-death closure ("makes monkey-click-loss STRUCTURALLY impossible") — DARK. Documented in A45 audit as the fix; never wired. **RISK: MEDIUM-HIGH** (chamber click-loss is a known recurring class). Fix: wire as PreToolUse or a SessionStart liveness check. |
| `scripts/hooks/bg-ghost-detect-dual-signal.sh` | orphaned-script | Refined ghost-BG detection (dual-signal). DARK. But `bg-auto-stop.sh` + `bg-stuck-warn.sh` ARE wired (Stop hook), so the ghost-BG class has *partial* coverage. **RISK: MEDIUM** — this is the better detector sitting unused behind weaker wired ones. Fix: either wire it (replace/augment the single-signal ones) or log a decision that the wired pair suffices and retire this. |
| `scripts/analyze-token-burn.py` | producer-no-surfacer | The trigger's exact answer. Zero callers (re-confirmed; now git-tracked, prior audit said untracked). Fully covered in `built-but-not-used-analytics-audit.md` §2/§A — fix = `--summary` mode + SessionStart digest line. |

**Legitimately orphaned BY DESIGN (one-shot migrations / on-demand CLIs — NOT gaps, but archival candidates):**
| Script | Why it's fine |
|---|---|
| `scripts/apply-set-005-batch.py` | one-shot batch applier for a specific design set (set-005); ran, done |
| `scripts/audit-checklist-backfill.py` | one-shot backfill of master-checklist severity codes |
| `scripts/backfill-improvement-tags.py` | docstring literally says "One-shot backfill … for the 10 already-shipped" |
| `scripts/run-inbox-archive-report.py` | on-demand report CLI (run manually when wanted) |
| `scripts/scheduler-cleanup-orphan.py` | on-demand worktree cleanup CLI (invoked by operator) |
| `scripts/heartbeat/test-fixtures/inject-ask.sh` | test fixture (invoked by hand during testing) |
| `scripts/dashboard/parse_bug_master.py`, `parse_pins.py` | parsers imported by the dashboard build at runtime via module path, not basename — flagged by the strict scan but actually used (see `dashboard-wave2-remaining.md`); NOT orphaned |

**Recommendation for the one-shot group:** move to `scripts/_archive/` (mirrors the `public/img/_archive` convention) OR add a top-of-file `# ONE-SHOT — retained for provenance, safe to archive` marker so future orphan scans auto-classify them. Low priority; zero risk. **Do not delete** (provenance value).

### Zone 2 — Designed-not-wired (plans propose, settings/crontab/launchctl lack)

Scanned 130 plan files for proposed hooks/daemons/crons/launchd/plists, cross-checked against settings.json + crontab + launchctl.

- **crontab: empty** (`crontab -l` → "no crontab for bennybrecher"). Confirms A74. No plan actually *requires* cron — the daemons use launchd `StartInterval`, which is correct for macOS. **Not a gap** (cron was never the chosen mechanism). The A74 phrasing "designed-crons-never-installed" appears to have meant the launchd daemons, which DO now exist.
- **A69 hook-wave inventory:** 32 designed hooks, **all wired** except `test.sh` (a glob artifact, not a real hook) and `tick.sh` (heartbeat tick — correctly launchd-driven, not settings-driven). **A69 is CLEAN.** Good.
- **git-hygiene-hooks-plan (5 designed):** `git-drift-warn.sh` ✓ wired, `scheduler-tick-drift-warn.sh` ✓ wired. **`git-hygiene-audit.sh` and `write-to-ignored-warn.sh` — designed in the plan, scripts MISSING (never built), not in settings.** [designed-not-built · RISK: LOW-MEDIUM] — these are warning-only hygiene hooks; the plan flagged them Phase-2. Fix: either build+wire the 2, or mark them explicitly deferred in the plan so they stop reading as "designed (implying coming)."
- **research-furthering daemon:** `.plist.template` exists at `scripts/daemons/research-furthering-launchd.plist.template`, **never deployed** to `~/Library/LaunchAgents/`. [designed-not-wired · RISK: LOW] — the never-idle research function is currently served by the `research-furthering` skill/workflow on-demand, so the daemon is redundant-for-now. Fix: log explicit "template retained, daemon deferred — on-demand skill covers it" decision, OR deploy it.
- **session-bundle Stop/SessionEnd trigger:** the plan (`end-of-session-realignment-ritual-plan.md`) intended an auto-trigger; **no hook fires `session-bundle-create.sh`** (confirmed: grep over hooks + settings = none). Covered in `built-but-not-used-analytics-audit.md` §1/§C. [designed-not-wired · RISK: LOW-MEDIUM] — manual-only, run once 3 days ago.

### Zone 3 — Dark daemons (see TOP #4 + #5)

Live state table (measured 08:00–08:15Z):
| Daemon | Loaded? | LastExit | Poll count | Cadence reality | dry-run? | Verdict |
|---|:---:|:---:|:---:|---|:---:|---|
| daemon2 (memory-consolidator) | ✅ | 0 | 3 | NOT hourly (3 in 34h) | yes | fires at session boundaries only; dry-run 2d+ |
| daemon3 (audit-catalog-scan) | ✅ | 0 | 2 | NOT hourly | yes | **reports severity:critical gaps=6 xref=4 to nobody** |
| daemon4 (steering-compact) | ✅ | 0 | 2 | NOT hourly | yes | severity:ok; low risk |
| daemon5 (bug-billboard-groom) | ✅ | 0 | 2 | NOT hourly | yes | severity:ok currently (0 inbox); but bug-integrity-adjacent |
| daemon10 (heartbeat-tier-low) | ✅ | 0 | 35 | ~hourly (healthy) | yes | the one healthy daemon; survives via own launchd cadence |
- **`daemon10-launchd.err`** contains 6 dry-run lines (last 2026-05-28T04:58Z) — these are *informational dry-run notices written to stderr*, not crashes. So "daemon10 erroring" from the trigger description is **a false alarm** — it's the dry-run channel, exit 0 throughout. Worth noting to avoid re-flagging.
- **No boot-persistence reloader** for any daemon (confirmed: no script does `launchctl bootstrap com.evium.daemon*`; SessionStart digest doesn't reload them). daemon10 survives only because its own plist cadence happens to re-fire; 2–5 don't survive logout.

### Zone 4 — Producers with no surfacer

(Largely covered by `built-but-not-used-analytics-audit.md`; re-confirmed live + adding the non-analytics ones.)
| Producer | Output | Who reads it? | Verdict |
|---|---|---|---|
| `correlate_stats.py` (A43) | `reports/stats/correlations.json` (8.8KB, 2026-05-28T22:32Z, ~10h stale) | **only the writer + the `research-furthering.js` workflow that triggers it.** Dashboard builders (`build_render_model.py`, `generate_dashboard.py`, `render-stats-data.py`) read it = **0.** | producer-no-surfacer — output reaches no human surface (confirmed) |
| `analyze-token-burn.py` | `reports/token-burn/operations-ranked.csv` (21,534 ranked turns) | **ZERO** | producer-no-surfacer (TOP-adjacent; see Zone 1) |
| `run-metrics-aggregator.py` | `.claude/cache/token-metrics.json` (fresh, live via token-budget hook) | token-budget hook reads it BUT never emits the `max()` "most expensive session" | partial — data surfaced, *answer* not |
| session-bundle (5 analyzers) | `reports/session-bundles/<iso>/insights*.md` | nothing auto; ran once | manual-only |
| daemon3 audit-catalog scan | `daemon3-audit-catalog-poll.jsonl` (`critical gaps=6 xref=4`) | **nobody** (dry-run, no surfacer) | producer-no-surfacer (the daemon IS the producer; its critical output evaporates) |
| `bottleneck-detect.jsonl` | real signals logged | **nobody** (script not hook-wired) | producer-no-surfacer (see TOP #1) |
| CoT/Brain ledger | `cot-ledger.jsonl` (live) | `correlate_stats.py` C4 + `organic-os-pulse` glyph — but feeds *mislabeled* data | live-but-low-fidelity (see `cot-brain-ledger-health.md`) |

### Zone 5 — Disciplines that describe-but-don't-run

- **impact-domino / slam-dunk:** MEMORY-ONLY, no enforcement. **TOP #3.** (The scariest discipline gap.)
- **idea-gap detector + asks-log:** ✅ **FULLY WIRED + FUNCTIONAL — NOT a gap.** `detect-idea-gaps.py` is run by `idea-gap-surface.sh` (SessionStart hook, in settings.json), reads `idea-gaps.jsonl`, surfaces a preview. Ran live this pass: classified 59 user prompts in the active transcript, serviced 20/20 of the classified window, **0 gaps**, exit 0. The `--no-write`, `--json`, env-override surface area is mature. `asks-log.md` is maintained (it's in the dirty working tree this session). **This is the model the impact-domino discipline should copy.** Explicitly calling this out as the counter-example: the user's "never miss an idea" anti-drop fear IS structurally closed; the "domino-map every new item" claim is NOT.
- **Convention F (analytics must land with a surfacer):** proposed in `built-but-not-used-analytics-audit.md` §F — **not yet adopted** into any skill ref. Its sibling disciplines (A40 idea-closure, A44 finding-closure) ARE enforced by hooks; the *capability*-closure version is not. [discipline-not-enforced · RISK: MEDIUM — it's the meta-fix that would prevent the next 10 orphans]. Fix: add to a skill ref + (optionally) a Stop-hook "new-script-without-caller" detector (a `plan-orphan-detector.sh` analog for scripts).

---

## Risk-ranked master list

| # | Gap | Class | Risk | Min fix | Cost |
|---|---|---|:---:|---|:---:|
| 1 | `bottleneck-detect.sh` not hook-wired | discipline-not-enforced | **CRITICAL** | 1 settings line (Stop/UserPromptSubmit) + graduate from dry-run | XS |
| 2 | `wrapper-only-write-guard.sh` dark | orphaned-script | **CRITICAL** | 1 settings line (PreToolUse Write\|Edit) | XS |
| 3 | impact-domino discipline = memory-only | discipline-not-enforced | **HIGH** | demote memory claim to honest "on-demand" OR build UserPromptSubmit tether | S |
| 4 | daemons 2–5 dry-run 2d+ & not hourly & no boot-persistence | dark-daemon | **HIGH** | reload-all.sh + graduate dry-run flags | S |
| 5 | daemons not in services registry; AUTONOMIC glyph 5/8 unwinnable | producer-no-surfacer | **MED-HIGH** | add daemons to required-services.json + fix DAEMONS_DESIGNED | S |
| 6 | `inbox-server-guard.sh` dark (A45 click-loss closure) | orphaned-script | **MED-HIGH** | wire as PreToolUse/SessionStart liveness | XS |
| 7 | `correlations.json` read by no surface | producer-no-surfacer | **MED** | ingest into build_render_model.py (after fixing CoT labels) | M |
| 8 | `analyze-token-burn.py` zero callers | producer-no-surfacer | **MED** | `--summary` + SessionStart digest line (per prior audit §A) | S |
| 9 | `bg-ghost-detect-dual-signal.sh` dark (weaker detectors wired instead) | orphaned-script | **MED** | wire it or log decision that bg-auto-stop+bg-stuck-warn suffice | XS |
| 10 | Convention F not adopted | discipline-not-enforced | **MED** | skill-ref + optional script-orphan Stop-hook | S |
| 11 | session-bundle no auto-trigger | designed-not-wired | **LOW-MED** | throttled Stop/SessionEnd hook | S |
| 12 | `git-hygiene-audit.sh` + `write-to-ignored-warn.sh` never built | designed-not-built | **LOW-MED** | build+wire or mark deferred | M |
| 13 | research-furthering daemon template never deployed | designed-not-wired | **LOW** | deploy or log "on-demand skill covers it" | XS |
| 14 | one-shot migration scripts uncategorized | orphaned-script (by-design) | **LOW** | move to scripts/_archive or add ONE-SHOT marker | XS |

---

## Cross-software domino observation (the audit's own meta-point)

The trigger incident (answer-from-memory-instead-of-running-the-tool) is **the same defect** across all of: `analyze-token-burn.py` (analytics), `bottleneck-detect.sh` (immune), the impact-domino discipline (triage), and the dry-run daemons (autonomic). In every case a real, working producer/detector exists and the *last mile* — wire it to a trigger, surface its output — is missing. The fixes are uniformly cheap (mostly one settings line each). **The expensive part was building them; the cheap part is what's undone.** Adopting Convention F (gap #10) as a Stop-hook "no script lands without a caller-or-explicit-on-demand-decision" is the single structural change that makes this whole class non-recurring — it is to *capabilities* what `detect-idea-gaps.py` is to *ideas* and `critical-finding-detect.sh` is to *findings*.

## What is NOT a gap (verified working — don't re-flag)
- idea-gap detector + asks-log (zone 5b) — fully wired, ran clean this session.
- daemon10 — healthy hourly ticks.
- daemon10 `.err` "errors" — actually dry-run informational notices, exit 0. False alarm.
- crontab empty — correct; launchd is the chosen mechanism, no plan needs cron.
- A69 hook wave — all 32 designed hooks wired (minus 2 non-hook artifacts).
- token-budget hook, dashboard-prime, bug-billboard-consolidate — live + firing.
- dashboard parsers `parse_bug_master.py`/`parse_pins.py` — used via module import, strict-scan false positive.

## Cross-references
- `context/markdowns/research/systems/built-but-not-used-analytics-audit.md` — the analytics deep-dive A83 extends (do not duplicate).
- `context/markdowns/research/systems/cot-brain-ledger-health.md` — CoT ledger fidelity.
- A74 — originating meta-finding + Convention F.
- A65 / `feedback_autonomic-system.md` — the 8-daemon catalog (5 built).
- A45 / `required-service-health-protection-plan.md` — inbox-server-guard's intended home.
- A58 / `immune-system-discipline.md` — bottleneck-detect's design + the chant.
- A67 / `feedback_impact-domino-analysis.md` — the unenforced triage discipline.
- A40 / `feedback_never-miss-an-idea.md` — the WORKING anti-drop model.
- `git-hygiene-hooks-plan.md` — the 2 unbuilt hygiene hooks.
