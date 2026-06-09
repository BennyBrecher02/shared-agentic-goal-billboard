# Audit → Workflow Transformation Catalog

> Generated 2026-05-31T07:30Z from per-audit classifications (86 audits). Each audit was tagged
> recurring-vs-one-off, given a workflow-shape, a subsystem (nervous / immune / both), a trigger,
> value/effort, and an already-wired note. This catalog buckets them into: convert-now slam-dunks,
> by-subsystem candidate tables, already-wired (don't rebuild), and genuine one-offs.

---

## 1. Headline

| Metric | Count |
|---|---|
| **Total audits classified** | **86** |
| Recurring-workflow candidates | **65** |
| Genuine one-offs (leave as catalog audits) | **21** |
| Audits with *some* wiring already in place (full or partial) | **73** |

**Recurring candidates by subsystem:**

| Subsystem | Count | Meaning |
|---|---|---|
| **Both** (nervous + immune) | **34** | Scheduled sweep *and* a failure-detector trip |
| **Immune** (detect-and-respond) | **19** | Fires on a bad-state predicate / PreToolUse guard |
| **Nervous** (scheduled / cadence) | **11** | Fires on a clock / heartbeat tier / threshold |
| (none — recurring but a pure on-demand gate) | **1** | A77 idea-conflict-check (agent-invoked) |

Headline read: **the system already metabolised most of its own audits.** 73 / 86 have at least
partial wiring; the recurring *disciplines* are overwhelmingly live as hooks/daemons. The
transformation work that remains is **closing the recurring LOOP on a small set where only the
substrate (scripts/producers) was built but no schedule/detector fires it** — plus one fully-unwired
HIGH-value audit (A39).

---

## 2. 🎯 Slam-dunk conversions

The HIGH-value, recurring audits whose **recurring mechanism is NOT yet firing** — substrate may
exist, but no schedule/detector/hook closes the loop. Ranked by leverage (value × how-load-bearing ÷ effort).

| # | Audit | Becomes (the workflow) | Subsystem | Concrete trigger | Value / Effort |
|---|---|---|---|---|---|
| 1 | **A39** — High-potential idea handoff + slam-meter | Stop-hook counts artifacts a trigger-firing prompt produced, assigns SLAM/GOOD/MID/FUMBLE tier, flags FUMBLE on 0 artifacts; cross-session slam-rate KPI | both | Stop on any prompt with flesh-out/stew/idea-handoff markers; roll-up at graceful shutdown | **high / med** — *only fully-unwired HIGH audit in the set* |
| 2 | **A2** — Critical-script test coverage | Guard test: a new/changed script under `scripts/hooks/` or wired in settings.json with **no matching `tests/` file** trips a coverage-gap warn | immune | On new wired script without a sibling test; weekly backstop sweep | **high / med** — hooks-health-check verifies existence but NOT coverage |
| 3 | **A4** — Skill-graph dead-link / orphan-ref health | Resolve every `[[link]]`/`references/foo.md` mention (flag dead), flag refs unmentioned in parent SKILL.md (orphans), diff descriptions for rot/overlap | immune | Chain into the skill-cross-link-rebuild PostToolUse hook so a dead link trips on edit; quarterly | **high / med** — existing hooks do siblings + staleness only, NOT graph-health |
| 4 | **A34** — End-of-session realignment ritual (master system audit) | SessionEnd→bundle→Stage-3 master-system-audit (7 modules) → 13 post-analysis hooks; feeds next-session resume | both | SessionEnd fires bundle; analyzer-complete fires Stage 3. **`session-bundle-create.sh` has 0 refs in settings.json** | **high / large** — ~40% built, Stage 3 + 13 hooks missing, bundle not wired |
| 5 | **A37** — Calculation audit (verify every metric/count) | Per-metric formula extract + independent recompute + diff vs JSONL/`find\|wc` ground truth; VERIFY assertion that fails on formula drift | both | Bolt onto every recurring master/run-group audit; guard trips when a memory-claimed count drifts from filesystem | **high / large** — one-shot passes ran; NO recurring calc-verify hook; verified-counts hand-maintained + STALE |
| 6 | **A28** — Skill-health refresh sweep | Periodic per-skill health compute → dispatch a 1-at-a-time refresh BG for whichever skill crosses cadence/use/scratchpad trigger | both | P0 monthly / P1 quarterly; event when ≥5 audits land in a skill's domain or scratchpad ≥30 | **high / large** — staleness *detection* wired (lint-skill-staleness.sh); the refresh-BG sweep is agent-discretionary |
| 7 | **A86** — Spawned-task self-clean pipeline | SessionStart PULL-scan spawned-worktree dir → recover real work to staging → prune dry-run; detector trips on unrecovered work / noise-only past TTL | both | main-session SessionStart poll. **#1 flip (harvest auto-surface) ALREADY landed**; recover→stage→prune pipeline unbuilt | **high / large** — user-asked 30+×; auto-prune walled behind A66 mass-delete gate |
| 8 | **A27** — Tiered testing + anti-bottleneck cost-gate | Affected-tests-only selection + result cache + P0/P1/P2 tiers + cost-gate that PAUSES a tier on median-P0>30s / single>60s / cache-hit<50% | both | nightly P2 + per-change P0; bottleneck-detect trips on a tier breaching its ceiling | **high / large** — `bottleneck-detect.sh` substrate exists; tier system + selector + cache + cost-gate unwired |
| 9 | **A43** — Stats upgrade #2 (correlation / predictive / anomaly) | Sample metrics on a cadence → recompute correlations/forecasts → fire alert + recommendation when a metric deviates from baseline | both | heartbeat tier-medium/low or a stats daemon; anomaly trip (>N-stdev) fires the alert | **high / large** — `correlate_stats.py` is a **producer-with-no-surfacer**; M3/M5/M6 + scheduled recompute unwired |
| 10 | **A58** — Organic-OS Immune centerpiece (bottleneck-detect on the tick) | `bottleneck-detect.sh` fires on the heartbeat-medium tick to enforce "testing never becomes THE bottleneck" via 8 signals | both | heartbeat tier-medium sweep + trip on any of 8 signals (cold-cache matrix >15min, ghost BG >2h, queue >50, hook latency >5s, recurrence 3+) | **high / large** — Brain/Heart hooks wired; `bottleneck-detect.sh` is **built, dry-run, zero hook callers** |
| 11 | **A83** — Capability-closure census (Convention-F) | Standing detector: any new script committed with no caller + no on-demand marker, or a daemon overrunning its 24h dry-run gate, trips a Stop-hook | both | weekly orphan census + Convention-F Stop-hook on new-script landing | **high / med** — `convention-f-closure.sh` is **BUILT but UNWIRED** (its own finding); slices covered by established-workflows-check + plan-orphan-detector |
| 12 | **A36** — Post-shutdown hook-timing re-evaluation (Track 4) | After every clean bundle, re-measure each hook's actual elapsed vs budget → recommend tighten/loosen/remove; immune canary on launched-but-never-closed BG | both | nervous: post-graceful-shutdown re-run; immune: stuck-ghost canary trip | **high / med** — Phase-2 instrumentation LIVE; Track-4 timing-reeval explicitly unbuilt (no script) |
| 13 | **A22** — Parked-plan readiness sweep | Periodic re-check of each design-parked plan's unblock-condition (e.g. 2nd Mac now SSH-reachable) against live capacity → surface ones now actionable | both | SessionStart or heartbeat sweep | **high / med** — NS/GL half wired (ns-advance-check.sh); the parked-plan-readiness cluster sweep is NOT wired |

**Cheap wins inside the slam set** (HIGH value, *small/med* effort — do these first): **A39** (med),
**A2 / A4 / A22 / A36** (med). A83's Convention-F flip is a med-effort wire of an already-built script —
the single highest leverage-per-hour conversion (it makes the whole orphan-script class self-detecting).

---

## 3. By subsystem

Recurring candidates only. ✅ = recurring loop already firing · ◐ = substrate built, loop not wired · ○ = unbuilt.

### NERVOUS-SYSTEM candidates (scheduled / cadence / threshold)

| id | Becomes | Trigger | V/E | State |
|---|---|---|---|---|
| **A24** | Monthly meta-loop-eval: scan hooks for unconnected writer/reader pairs, goals-without-audit-links → "Untapped Integrations" report | Monthly cron 06:00 (template exists, **not in live crontab**) | high/sm | ◐ built, cron not live |
| **A65** | Autonomic daemon framework — 8 maintenance daemons on Heart tiers w/ 7-gate safety contract | Heart tiers fire each daemon; per-daemon 24h-dry-run → live | high/lg | ◐ 5/8 daemons as launchd agents (dry-run); 3 not daemonized |
| **A67** | Persistent leverage-ranking daemon above the goal billboard (re-rank impact×unblock÷effort+risk) | per-prompt stub-append wired (domino-check.sh); the re-rank Daemon #9 deferred to A65 Ph2 | high/med | ◐ capture wired; re-ranker unbuilt |
| **A17** | T-30/T-15/T-5 pre-reset countdown ladder w/ dedup + drift-detection | heartbeat tick, 5-min near reset | high/sm | ◐ ladder fires via heartbeat; interval-downgrade + drift ladder + per-response surface missing |
| **A38** | SessionStart appends new prompts; refresh "look-through-Ben's-eyes" skill at +50 prompts | SessionStart extractor + 50-prompt refresh | med/lg | ○ substrate built, extractor **NOT a SessionStart hook**, skill ref absent in live skills |
| **A72** | Per-session USD cost-ledger + daily/weekly/monthly rollup widget | session-end cost-tracker + rollup | med/med | ○ `cost-tracker.py`/`cost-rollup.py`/`anthropic-pricing.json` do **not exist**; token JSONL is the input |
| **A8** | Read-only fan-out classifying SKILL.md prose incantations → wrap / cite-existing / drop / leave | before skill-consolidation; recurrence flags re-typed incantation 3+× | med/sm | ○ not wired |
| **A1** | Monthly drift-cluster of agentic-vs-scripted calls in steering-log → calibration recs | monthly OR steering-log +50-entry boundary (already 7× past) | med/med | ○ not wired |
| **A3** | Hook signal-to-noise ranking from a firing log: (value×firings)/cost → keep/merge/on-demand/remove | every ~2wk OR ≥50 sessions | med/med | ◐ needs a `hook-firings.log` substrate that doesn't exist |
| **A26** | agentic-script-design compliance-pass scoring code against refs | threshold: recommendations file >~30 entries | med/med | ✅ consolidation loop covered by reactionary skill; only compliance-scoring unfired |
| **A15** | Research-folder nudge + cross-link harvest + surface-on-resume | UserPromptSubmit / PostToolUse / SessionStart | med/sm | ✅ fully wired |

### IMMUNE-SYSTEM candidates (detect-and-respond / PreToolUse guard)

| id | Becomes | Trigger | V/E | State |
|---|---|---|---|---|
| **A2** | New wired script w/o matching test → coverage-gap warn | new script without sibling test; weekly backstop | high/med | ◐ existence-check only, no coverage assert |
| **A4** | Skill-graph dead-link + orphan-ref + description-overlap check | chain into skill-cross-link-rebuild PostToolUse; quarterly | high/med | ◐ siblings/staleness only |
| **A12** | Guard-coverage detector for clobber paths (Class-2/4 stash-naming, redirect-clobber) | PreToolUse on git checkout/restore/stash + write-clobber | high/sm | ◐ Class-1 (`git-checkout-safety.sh`) live; 2/4 protocol-only |
| **A12 (re-run)** | Class-B Python-subprocess clobber detector: grep scheduler for ungated `git apply`/`worktree remove --force`/`--update-snapshots` | on change to `scripts/scheduler/**` or new reaper | high/med | ◐ Bash surface wired; F1/F2/F3 subprocess paths **ungated** |
| **A16** | Idle-stop blocker: pending work + headroom → block idle stop, track idle-responses KPI, 3× escalation | Stop when pending work exists despite headroom | high/med | ◐ detect+nudge LIVE (goal-queue/pending-parallel/capacity/ns-advance); KPI + 3× ladder unwired |
| **A50** | Dry-run/LOG_ONLY-liveness detector: input-log has real events but action-log shows 0 did-fire | input-log ≥1 real event + 0 did-fire over window | high/sm | ◐ A45/A50 click-loss closed; the *general* dry-run-liveness detector unwired |
| **A57** | Pre-write content lint on user-facing surfaces (5-pillar: length/voice/format/hierarchy/density) | Write/Edit to data.json / billboard / asks / goal / plan | high/sm | ◐ lint+wrapper+guard built (5/5), guard **not in settings.json** (WARN→BLOCK user-gated) |
| **A18** | Plan/audit-creation gap: detect "make a plan/audit" → log pending ask → Stop checks for matching artifact | UserPromptSubmit + Stop | high/med | ✅ user-ask-detect + user-ask-artifact-check wired |
| **A20** | Hedge-holdoff detector: scan turn for "want me to/let me know if" → re-ask next prompt | Stop + UserPromptSubmit | high/sm | ✅ hedge-detect + drain wired |
| **A21** | Parallel-candidate scan + capacity-underuse warning | UserPromptSubmit + Stop | high/med | ✅ pending-parallel-work-scan + parallel-capacity-check |
| **A44** | Critical-finding reflex: Stop scans for 11 markers → CLASSIFY→SPAWN→ESCALATE→CROSS-CHECK reminder | every Stop | high/med | ✅ critical-finding-detect.sh wired (WARN); skill ref still missing |
| **A45** | Required-sidecar health-check + bounded auto-restart (3/hr, never kill squatter) | SessionStart + on-demand | high/med | ✅ inbox-server-guard + services-digest wired; CLOSED |
| **A47** | BG ghost-reaper: 5-condition predicate → SIGTERM→grace→SIGKILL, rate-limited | Stop / heartbeat tick | high/sm | ✅ bg-auto-stop wired (DRY-RUN; live-flip user-gated) — most-wired audit |
| **A66** | Scope-of-permission guard: high-risk edit (src/settings/baselines) WARN/BLOCK unless explicit auth | every PreToolUse Write/Edit on high-risk target | high/sm | ✅ src-edit-guard + settings-edit-guard wired |
| **A68** | Poll-gap drain: Notification + Stop + Heart-tier-low drain inbox + 8 mid-turn substrates | inbox-arrival events + hourly backstop | high/sm | ✅ drain hooks + daemon10 wired; only VERIFY regression test unwired |
| **A71** | Inbox-never-lost triple-tether: per-session ack + mutation-guard + Stop-drain + integrity-check | PreToolUse ack-write + Stop + integrity scan | high/sm | ✅ VERIFIED COMPLETE (5/5) |
| **A49** | Chamber-empty domino: pending→0 transition → refresh G2 plow plan → "plow?" pivot | PostToolUse on chamber substrate, 5-min recency window | med/med | ✅ chamber-empty-detect wired |
| **A13** | Snapshot-baseline staleness + overwrite warn before `--update-snapshots` | PreToolUse(Bash) on matrix/snapshot-regen | med/med | ✅ staleness-check + drift-pre + Path-D regen wired |
| **A7** | Style-token audit: rg src/ for hardcoded hex/font/easing → drop-in-fix vs design-decision | PostToolUse on src/ CSS/.astro; before client handoff | med/med | ○ not wired |

### BOTH (scheduled sweep + failure-detector)

| id | Becomes | Trigger | V/E | State |
|---|---|---|---|---|
| **A34** | Master-system-audit ritual (7 modules + 13 post-analysis hooks) | SessionEnd→bundle→Stage-3 | high/lg | ◐ ~40% built, bundle not wired |
| **A37** | Recurring calculation-verification + count-drift guard | every master audit; count-drift trip | high/lg | ◐ one-shot only, no recurring hook |
| **A28** | 1-at-a-time skill-refresh BG sweep | monthly/quarterly + domain/scratchpad event | high/lg | ◐ detection wired, sweep agent-discretionary |
| **A86** | Spawned-task recover→stage→prune pipeline | SessionStart poll; TTL/noise detector | high/lg | ◐ harvest wired, pipeline unbuilt |
| **A27** | Tiered test cost-gate enforcement | nightly P2 + per-change P0; ceiling trip | high/lg | ◐ detector substrate only |
| **A43** | Sample→correlate→forecast→anomaly-alert stats loop | heartbeat tier; >N-stdev trip | high/lg | ◐ producers exist, unsurfaced |
| **A58** | bottleneck-detect on the heartbeat tick (8 signals) | tier-medium tick + 8 failure signals | high/lg | ◐ built, dry-run, 0 callers |
| **A83** | Convention-F capability-closure census | weekly + new-script Stop-hook | high/med | ◐ `convention-f-closure.sh` built, unwired |
| **A36** | Post-shutdown hook-timing re-eval (Track 4) | post-clean-bundle; stuck-ghost canary | high/med | ◐ Track 4 unbuilt |
| **A22** | Parked-plan-readiness sweep | SessionStart/heartbeat | high/med | ◐ cluster half unwired |
| **A30** | Shutdown-snapshot + SessionStart resume + nightly round-trip self-test | SessionEnd / SessionStart / nightly | high/lg | ◐ scripts exist; no dedicated SessionEnd/SessionStart hooks; test not nightly |
| **A32** | Affected-hook-tests on change + Phase-5 effectiveness re-run + Phase-6 post-shutdown runner | hook edit + post-shutdown async | high/med | ◐ 235-test suite built; effectiveness re-run + post-shutdown-runner.sh absent |
| **A60** | Goal-staleness detector (P0=2d/P1=5d/P2=10d no-CoT-activity) | Heart tier-medium sweep + staleness trip | med/sm | ◐ staleness hooks wired; schema fields not in goal frontmatter |
| **A48** | data.json auto-derive from inbox-history on every dashboard refresh + raw-write barrier | last step of dashboard-data-prime + PreToolUse barrier | med/sm | ◐ auto-derive built; wrapper-guard not in settings.json |
| **A9** | Deny-list re-classification sweep + friction-frequency detector (≥3 denies/session) | monthly/on-settings-change + ≥3-deny trip | med/med | ◐ edit-safety wired; utility classifier + frequency detector unwired |
| **A79** | Repo-exposure detector: .gitignore default-deny gap + secret-pattern scan + bloat/LFS | pre-commit warn + weekly tree/history sweep | med/med | ◐ drift-warn covers working-tree only |
| **A85** | Git rollback-safety sweep: dirty-tree-before-handoff + Cursor perms-widening + reflog force-push | SessionStart drift + periodic perms-diff | med/med | ◐ drift/guards wired; one-time recs need user action |
| **A84** | Swarm escalation+back-off protocol: sample load/mem/window-burn at each K, back off on 429/>80%/swap | sampler during escalation; back-off ladder trip | med/med | ◐ samplers exist; cloud-light cohort + window-burn fields unbuilt |
| **A42** | Idle-emulator killer: 7 situational signals + adaptive threshold + miscall watchdog | ~5-min sweep + miscall-rate self-disable | med/lg | ◐ session-boundary teardown only; periodic dynamic-threshold sweep + tracker unbuilt |
| **A6** | Scheduler worktree-hygiene sweep: keep-vs-reap + disk-pressure trigger | weekly OR >5 worktrees OR <10GB free | med/sm | ◐ post-tick cleanup auto-runs; standing sweep/disk-pressure audit unwired |
| **A14** | tokens+wall-clock+burn-rate+time-to-reset together every surface | SessionStart + per-Bash + midnight + burn-governor trip | high/med | ✅ FULLY SHIPPED |
| **A19** | Recurrence detector + workflow-bypass detector | UserPromptSubmit + Stop after src/dash/scheduler edit | high/lg | ✅ recurrence-detect + established-workflows-check |
| **A29** | Parallel-launch FLOOR enforcer + calm-mode-trap + BGs/response KPI | UserPromptSubmit + Stop + rolling-3 | high/med | ✅ core loop wired; calm-mode-trap sub-detector may be partial |
| **A40** | Idea-gap detector: classify last-N prompts vs 4 closure paths within K-window | SessionStart (wired) + Stop backstop + daily sweep | high/med | ✅ SessionStart surface wired; Phase-3/4 backstop+sweep unbuilt |
| **A63** | Idle-capacity research-furthering refill (7 safety boundaries) | daemon idle-check + in_flight<cap condition | high/sm | ✅ daemon + capacity hooks wired |
| **A80** | Never-idle "go to the well" protocol (ranked research well) | daemon idle-detect + condition | high/med | ✅ daemon + skill wired; well-prioritization agent-side |
| **A81** | research-furthering Workflow (parallel queue.map, slot-refill) | idle capacity + non-empty queue; chain on batch-complete | high/med | ✅ `research-furthering.js` wired |
| **A23** | Time-precision verification sweep (verify-clock + precision-audit + 5hr-reset) | heartbeat-low/daily drift sweep + malformed-ts trip | high/med | ◐ integration half wired; periodic verification sweep not scheduled |
| **A5** | /consolidate-memory run on bloat/threshold trip | monthly + index>24.4KB / >35 entries / 3+ stale | high/med | ◐ detector wired; consolidation RUN still manual |
| **A25** | settings.json spine-hygiene: orphan hooks + broken paths + chain latency + inline-bash | SessionStart sweep + orphan/over-budget trip | high/med | ◐ hooks-health-check partial; 24 patches unapplied |
| **A25 (re-run)** | Full census: orphan-entry + reverse-orphan + Rule-1 ordering + per-surface perf-budget | on settings.json change / weekly; orphan/budget-bust trip | high/med | ◐ no single workflow does the full census |
| **A56** | Sim/emu idle teardown at session boundary + weekly tighter sweep | Stop/SessionEnd + weekly | high/sm | ✅ teardown hook built (dry-run); manual audit/killer exist |
| **A61** | Inbox/chamber 5-layer defense (schema-guard, staleness ticker, auto-recovery, recurrence, consistency) | PreToolUse guards + 15-min heartbeat ticker | high/med | ◐ L1/L4 wired (WARN); L2 ticker + L3 auto-recovery unwired (13 tests skipped) |

---

## 4. Already wired — do NOT rebuild

These audits' recurring disciplines are **already a live workflow / hook / daemon**. Re-running the
audit-the-doc is pointless; the discipline self-fires. (Name = the wired mechanism.)

| Audit | Wired mechanism (don't rebuild) |
|---|---|
| **A14** Stats/burn-rate | `token-budget-check.sh` + `claude-5hr-window.py` + `daily-stats.sh` + `detect-5hr-reset.py` + `burn-rate-governor.py` — FULLY SHIPPED |
| **A18** Plan/audit-creation gap | `user-ask-detect.sh` (UserPromptSubmit) + `user-ask-artifact-check.sh` (Stop) + `asks-log.md` |
| **A19** Recurrence + workflow-bypass | `recurrence-detect.sh` + `established-workflows-check.sh` + `state-batch-digest.sh` |
| **A20** Hedge-holdoff | `hedge-detect.sh` (Stop) + `hedge-detect-drain.sh` (UserPromptSubmit) |
| **A21** Parallel-capacity | `pending-parallel-work-scan.sh` + `parallel-capacity-check.sh` |
| **A29** Parallel FLOOR / calm-mode | same two hooks + `dispatch-metrics-log.sh` KPI |
| **A40** Idea-gap | `idea-gap-surface.sh` (SessionStart) + `detect-idea-gaps.py` |
| **A44** Critical-finding reflex | `critical-finding-detect.sh` (Stop, WARN) — *skill ref still missing* |
| **A45** Required-service health | `inbox-server-guard.sh` + `sessionstart-services-digest.sh` (CLOSED) |
| **A47** BG ghost-reaper | `bg-auto-stop.sh` + `bg-stuck-warn.sh` + `bg-ghost-detect-dual-signal.sh` + `dispatch-bg.sh` (DRY-RUN; live-flip gated) |
| **A66** Scope-of-permission | `src-edit-guard.sh` + `settings-edit-guard.sh` + memory rule |
| **A68** Poll-gap inbox drain | `agent-inbox-drain-{notification,stop}.sh` + `agent-inbox-stop-drain.sh` + `daemon10` |
| **A71** Inbox-never-lost | `inbox-mutation-guard.sh` + `agent-inbox-stop-drain.sh` + `inbox-integrity-check.py` (5/5) |
| **A49** Chamber-empty domino | `chamber-empty-detect.sh` (PostToolUse) |
| **A13** Snapshot baseline drift | `snapshot-baseline-staleness-check.sh` + `snapshot-drift-pre.sh` + Path-D regen |
| **A15** Research-folder discipline | `research-folder-advise.sh` + `research-cross-link-harvest.sh` + `research-surface-on-resume.sh` |
| **A63 / A80 / A81** Never-idle | `research-furthering-daemon.sh` + `research-furthering.js` + capacity hooks |
| **A77** idea-conflict-check | `.claude/workflows/idea-conflict-check.js` + skill ref — reusable gate, agent-invoked |
| **A56** Sim/emu teardown | `sim-emu-session-teardown.sh` (Stop/SessionEnd, dry-run) |

> Note: several "wired" entries ship in **DRY-RUN / WARN mode with a user-gated WARN→BLOCK or live-kill
> flip** (A47, A48, A56, A57, A61, A66's BLOCK). Those flips are *user actions*, not rebuild work.

---

## 5. Genuine one-offs — leave as catalog audits

Not workflows. Read once; value flows into other builds or is a settled decision. Do not schedule.

- **A11** — Phase-B concurrency-ceiling diagnosis → outcome encoded (`max_workers=4` cap). Re-runs only on new hardware/shard-type.
- **A31** — Pre-restart 6-front bundle; each front landed as its own infra. The bundle act isn't recurring.
- **A35** — Wide external agentic-systems research sweep; reusable refill loop (research-furthering) already built.
- **A41** — "Powerful uses of session JSONLs" brainstorm catalog (25 ideas); items graduate individually (A1/A5 already wired).
- **A59** — Scheduler-upgrade brainstorm (15 ideas, "don't change now" verdict); individual ideas tracked elsewhere.
- **A62** — Adaptive-immunity *meta-discipline* (the template behind every immune response); the triple-tether/VERIFY-coverage sub-check is an unautomated recurring sliver but the doctrine itself isn't a runnable scan.
- **A64** — Stuck-state + SOLID-grade + 5 per-letter refs investigation; recurring slivers owned by A45/A19.
- **A69** — CoT auto-realign gap — CLOSED (309-entry ledger, all cot-* hooks wired); orphan-hook generalization lives in hooks-health-check.
- **A73** — Emulator/simulator matrix-profile cut; DECISION SHIPPED (the 2-per-OS cap is the recurring guard, not this audit).
- **A74** — 4.8 systemwide birdseye; generalized by `model-upgrade-benchmark.js`; top recs (organic-os-pulse, audit-close-check) already built.
- **A75** — Workflow 4.7-vs-4.8 comparison; point-in-time, verdict reached (no conflict).
- **A76** — Semantic code-index downside hunt; generalized by `idea-conflict-check.js`; DEFER verdict delivered.
- **A78** — Dashboard scrutiny + plan-consolidation + Terminal-Garden direction; process lives in agentic-page-scrutiny.
- **A82** — TRUE agent-plow ceiling synthesis verdict (read-only); follow-ups are separate gated items.
- **A87** — Manifesto realignment per-item drift re-read; generalized by `session-request-audit.js`.
- **A88** — Git-rollback visible-vs-invisible reclassification; lasting value is a classifier code change (add `visible` field), not a re-run.
- **A89** — Dashboard-migration completeness diff; a generalized feature-parity guard could be recurring but THIS is a one-shot.
- **A33** — Local-AI mass-wielding dual-track research initiative; founding vision consumed once (dispatch mechanics already generic).
- **A10** — Scheduler deny-list-workaround catalog; only actionable after A9 moves a deny entry (gated, not standing).
- **A46** — Monkey-chamber UX de-retardize; one-shot UI build (Phase 1 shipped).
- **A51** — Dashboard typing survives HMR; single ~50-line code fix, verified once.

---

### Transformation summary

- **Convert now (13 slam-dunks):** A39, A2, A4, A34, A37, A28, A86, A27, A43, A58, A83, A36, A22.
- **Highest leverage-per-hour:** **A83** (wire the already-built `convention-f-closure.sh` → the whole orphan-script class self-detects) and **A39** (the only fully-unwired HIGH audit).
- **Don't rebuild:** 19 disciplines already firing (§4) — many just await a user-gated dry-run→live flip.
- **Leave as catalog:** 21 genuine one-offs (§5).
