---
kind: audit-findings
audit: floated-ideas (Pass 4 — the SUGGESTION/MUSING register)
date: 2026-06-07
method: DEDUP 390 raw floated candidates → ~52 distinct ideas → CLASSIFY built|declined|DROPPED → cross-ref on-disk + goal-billboard {ideas,proposals,plans,goals,active} + asks-log → FLESH OUT the confirmed dropped → feed verdict.jsonl
critical-constraint: the user HATES phantom gaps. Every DROPPED below was confirmed NOT built AND NOT a tracked idea/proposal/plan. Anything found on disk or in ideas/ was demoted to BUILT/DECLINED with the evidence path.
sibling-audits:
  - full-history-request-audit-2026-06-07.md (the IMPERATIVE-register sweep — 3 open asks)
  - orphaned-implementations-audit-2026-06-07.md (built-but-unwired)
verdict_feed: .claude/cache/alignment/verdict.jsonl (lobe=floated-idea-dropped)
spawned_ideas: [I-018, I-019, I-011, I-015, I-016, I-012, I-014, I-017, I-013]
---

# Floated-ideas audit — 2026-06-07 (Pass 4)

**The register this pass hunts:** not imperatives ("build X") but the SUGGESTION / MUSING layer — *"maybe we
should…", "can we…", "i think we could benefit…", "what about…", "flesh this out", "lets talk it out"*. This
is the audit blind spot that historically lost the **memory-mirror** idea (a pure "could be useful… what do
you think" that turned load-bearing) and the **agentic-preview-tools** skill (pushed 3× as "i think we
should" and still never built).

## Tally

| | Count | Notes |
|---|---:|---|
| **Total floated (raw candidates)** | **390** | heavily duplicated across 7 session JSONLs |
| **Distinct ideas (after dedup)** | **~52** | e.g. skill-maker appears in 3 sessions; parallel-work mechanism in 3; emulator-killer in 2; token-cheap-out in 2 |
| **BUILT** | **34** | exists on disk / wired / shipped as a skill-ref/plan/goal |
| **DECLINED** | **8** | explicitly deferred-with-reason or trigger-gated (clean defers, watched) |
| **DROPPED** | **10** | floated, never acted, never declined — confirmed absent on disk + absent from ideas/proposals/plans |

> **Headline:** like the sibling full-history audit, *most* floated ideas landed or were cleanly deferred.
> The musing register is leakier than the imperative one, but only **10 genuine drops** survive cross-ref —
> and **3 are high-leverage** (preview-tools skill, the skill-maker generative skill, token-saving
> adoptions). The rest are enhancements/sub-features whose parent shipped but whose softer rider didn't.

---

## DROPPED — ranked by leverage (the gold)

### HIGH leverage

#### 1. `agentic-preview-tools` skill — the missing sibling to `agentic-chrome-tools`
- **first floated:** 2026-05-25 · **last floated:** 2026-05-25 (session 2842f478) · **recurrence: 3× in one session**
- **User words:** *"i think we should flesh this area out more so our system better differentiates the
  benefits and use cases and deciding factors between chrome vs preview in real time decisions."*
- **What it was:** a dedicated skill cataloguing the Claude **Preview** plugin's capabilities (parallel to
  the 15-ref `agentic-chrome-tools`), so the agent can decide Preview-vs-Chrome in real time per task.
- **Why it matters:** the user pushed this **3×** in one session and was deflected each time ("future skill,
  defer — needs Preview capabilities catalogued from real use first"). The chrome-vs-preview gray line
  recurred for the rest of that session and into later ones — an unresolved tool-overlap question. The Chrome
  side got a full 10-mode skill; Preview never got its counterpart.
- **Confirmed DROPPED:** `.claude/skills/` has 12 skills — **no `agentic-preview-tools`**. Not in `ideas/`,
  `proposals/`, or any plan. The `mcp__Claude_Preview__*` toolkit exists and is used ad-hoc but has no skill.
- **First concrete step:** catalogue the `mcp__Claude_Preview__*` tools (preview_start/eval/inspect/network/
  resize/screenshot/console_logs) into `.claude/skills/agentic-preview-tools/SKILL.md` + a Preview-vs-Chrome
  decision-matrix ref that cross-links `agentic-chrome-tools` Mode-decision ref. **Leverage: HIGH** (closes a
  3×-asked tool-overlap gap; one skill file + one matrix ref).

#### 2. The master skill-maker GENERATIVE skill — "looks through MY eyes" (A38 stalled at substrate)
- **first floated:** 2026-05-27 · **last floated:** later sessions (0d739f65, 23f93bfd, 23492bcf) · **recurrence: 3 sessions**
- **User words:** *"…utilize that info to make a master skill maker skill that literally looks through MY
  eyes, lets flesh this out."*
- **What it was:** mine the user's own prompt corpus to learn how he writes his sharpest, highest-net prompts,
  then a **generative skill that AUTHORS new skills in his voice**. The deliverable is the skill-maker, not the analysis.
- **Why it matters:** the *substrate* shipped (`scripts/extract-user-prompts.py` + 296 prompts in
  `reports/conversation-extracts/ALL-PROMPTS.md`) — but the headline artifact, the skill that writes like Ben,
  was scoped to "voice analysis" and **A38 is still `status: in_progress`** with the plan frozen at *"Phase 1
  analysis BG queues for capacity."* The high-value half never built.
- **Confirmed DROPPED (the generative skill):** `plans/automation/master-skill-maker-plan.md` = Phase 0 done,
  Phase 1+ never run; `A38` still `in_progress`; the look-through-bens-eyes.md ref **does not exist**. (The
  plugin `anthropic-skills:skill-creator` is generic, NOT the bespoke voice-matched one.)
- **First concrete step:** run A38 Phase 1 — analyze `ALL-PROMPTS.md` for the slam-dunk + voice patterns,
  emit `look-through-bens-eyes.md` (a skill-ref the agent loads pre-response). **Leverage: HIGH** (substrate
  already paid for; this is the unrealized payoff of a slam-dunk the user explicitly flagged).

#### 3. Token-saving ADOPTIONS — the 5 RH-011 candidates (the user's weekly-limit motive, still research-only)
- **first floated:** 2026-05-27 (0d739f65) · **recurrence: 2 sessions**
- **User words:** *"how did they maximize token cost saves because my weekly limit is something i wanna
  consider without altering our workflows."*
- **What it was:** the RH-011 token-cheap-out research surfaced 5 adoption candidates — **Batch API for
  recurring BGs (#1), tool-result compression, stable-prefix audit, multi-model tier hint, weekly-window
  visibility surface** — to protect the weekly limit WITHOUT changing workflows.
- **Why it matters:** this was the user's *stated motive*, not idle curiosity. The research landed
  (`research/rabbit-holes/token-cheap-out-pass-2/`) and audits A40-A45 were catalogued, but **none of the 5
  adoptions were implemented** beyond the pre-existing `token-budget-check.sh` pre-flight. The savings he
  wanted remain unrealized research.
- **Confirmed DROPPED:** no Batch API integration anywhere (only `bg-launch-pattern.jsonl` cache); no
  tool-result compression script; no stable-prefix audit; no weekly-window visibility surface (only
  `TODO-after-weekly-limit-resets.md`, a parking note). Research exists; adoption does not.
- **First concrete step:** pick the highest-ROI/lowest-risk of the 5 — **Batch API for recurring BGs** — and
  scope it as a billboard proposal (it's the #1 saver and recurring BGs are the dominant token sink).
  **Leverage: HIGH** (directly serves the stated weekly-limit constraint; the rest can follow as a series).

### MEDIUM leverage

#### 4. Cross-substrate idea-gap sweep — apply the A40 lens to event-bus / dispatch-log / bug-billboard inbox
- **floated:** 2026-05-27 (0d739f65), as the explicit deferred follow-up to the A40 build.
- **User words (the meta-observation A40 captured):** *"the substrate was sitting there FOR WEEKS —
  built-but-not-queried"* + the named follow-up to sweep the OTHER substrates with the same gap lens.
- **What it was:** A40 built `detect-idea-gaps.py` to sweep **session JSONLs**. The named-but-undone
  extension: run the same dropped-request lens over `event-bus.jsonl`, `dispatch-log.jsonl`, and the
  `bug-billboard/inbox/` — so an idea dropped in ANY substrate (not just chat) is caught.
- **Why it matters:** the system's own lesson was "built-but-not-queried for weeks." The fix was scoped to one
  substrate; the other three coordination substrates have no gap-detection.
- **Confirmed DROPPED:** no script/hook references a cross-substrate gap sweep. (This very floated-ideas audit
  is itself a downstream of the original gap-mining ask — proof the class is still live.)
- **First concrete step:** extend `detect-idea-gaps.py` (or a sibling) to also scan `event-bus.jsonl` +
  `dispatch-log.jsonl` + `bug-billboard/inbox/` for unserviced items. **Leverage: MED.**

#### 5. Dynamic self-readjusting emulator "sweet-spot" timeout (the adaptive half of A42)
- **floated:** 2026-05-27 (0d739f65, 23f93bfd) · **recurrence: 2 sessions**
- **User words:** *"…some intense logic to not only back this up but also dynamically readjust the sweet spot
  time."*
- **What it was:** the emulator-killer should not use a fixed threshold — it should **learn and re-tune** the
  idle-timeout sweet-spot from usage, folded into the stats system.
- **Why it matters:** the basic killer shipped and is wired (`scripts/hooks/sim-emu-session-teardown.sh`,
  1 wire in settings.json) — but it's a **static** teardown. The *adaptive* learning loop the user asked for
  twice is absent (grep for dynamic/adaptive/readjust/sweet-spot in the script = 0 hits).
- **Confirmed DROPPED (the adaptive half):** the hook exists (parent BUILT); the self-readjusting logic does not.
- **First concrete step:** add a small learner that records emulator idle→reuse intervals and re-tunes the
  teardown threshold (a reactionary-style post-run recompute). **Leverage: MED.**

#### 6. Longitudinal "prove we're actually getting better" improvement-verification metric
- **floated:** 2026-05-26 (23f93bfd).
- **User words:** *"the longer we develop the better our skills have gotten right? but how can we confirm
  that? we need an audit and new metrics and stat tracking."*
- **What it was:** a metric that demonstrates the system improves **over time** — distinct from the
  point-in-time skill-health audit (A28). A trend line, not a snapshot.
- **Why it matters:** the skill-health audit (A28) + `run-skill-usage-count.sh` answer "what's the state
  now"; they do NOT answer "are we trending better." The user's actual question ("how can we confirm") has no
  measurement.
- **Confirmed DROPPED:** no longitudinal/trend improvement metric on disk (skill-metrics is current-state only).
- **First concrete step:** snapshot the skill-metrics JSON per session-bundle and chart skill-count /
  ref-depth / use-coverage over time. **Leverage: MED.**

#### 7. "All agents TIME their testing" — the universal timing mandate
- **floated:** 2026-05-25 (2842f478).
- **User words:** *"maybe all agents should now time their testing so we can get a clearer wider view of our
  matrixs time complexity and further improve our parallelization backed by logic and agents proof."*
- **What it was:** make per-test timing a **universal enforced discipline** across every testing agent, to
  build a hard-data view of matrix time-complexity.
- **Why it matters:** the *mechanism* exists (`scripts/record-matrix-timing.sh`) but invoking it is **opt-in**;
  no rule mandates that every testing agent times its run, so the "wider view" the user wanted is never
  systematically collected.
- **Confirmed DROPPED (the mandate):** mechanism BUILT, universal rule never written into memory/skill as enforced.
- **First concrete step:** add a one-line discipline to `agentic-device-testing` SKILL ("every matrix run
  pipes through `record-matrix-timing.sh`") + note it in the device-matrix memory rule. **Leverage: MED.**

### LOW leverage

#### 8. Graceful mid-plow rate-limit handling mechanism
- **floated:** 2026-05-26 (23f93bfd). **User words:** *"how do we potentially eventually handle randomly
  getting hit by rate limit when we're plowing…"* — answered-only; no hook beyond the existing token-budget
  pre-flight check. **First step:** a Stop-hook checkpoint-write so an abrupt limit-hit always leaves a
  resumable CoT marker. **Leverage: LOW** (filesystem state already survives; this is a nicety).

#### 9. The ~12 unbuilt dashboard additions (from the 15-item menu)
- **floated:** 2026-05-26 (23f93bfd). **User words:** *"do you have any more ideas of useful additions to our
  system dashboard."* The assistant offered 15; only **3** were pursued (BG ghost watcher, pinned next-step,
  decision-handoff drawer). The other ~12 were enumerated and never built. **First step:** triage the 12 into
  the `ideas/` folder under G18 (dev-control-dashboard) so they're tracked, not lost in a chat reply.
  **Leverage: LOW** (idea-capture, not build — but exactly the evaporation class this audit targets).

#### 10. "Feedback-skill-building after every turn" end-of-turn automation
- **floated:** 2026-05-25 (2842f478). **User words:** *"maybe some feedback skill building stuff … that run
  at the end of every turn."* The end-of-turn script framework + `record-matrix-timing.sh` shipped, but the
  specific "auto-build a feedback skill each turn" idea was never built as such (the skill-maintenance-cadence
  is periodic-consolidation, not per-turn). **First step:** decide if per-turn feedback-skill-building is even
  desirable vs the existing periodic cadence; if yes, scope it; if no, explicitly decline it (it's been in
  limbo since May 25). **Leverage: LOW.**

---

## DECLINED (clean defers / trigger-gated — watched, NOT dropped)

- **iOS 18 simulator runtime** — deferred-until-asked with a trigger (install when launch nears / a real
  iOS-18 bug appears). Logged in `A73-emulator-coverage-justification.md` + `pins/P-017`. Clean.
- **`hook-timing-reevaluation`** — `plans/automation/hook-timing-reevaluation-plan.md`,
  `status: deferred — build_gate: next-graceful-shutdown-bundle-landed`. **WATCH:** the gate condition
  (a trustworthy post-shutdown bundle) — verify on next consolidation that the gate fired and didn't lapse.
- **M3/M2 Mac slave node** — `plans/automation/m2-mac-test-farm.md`, gated behind "after we prove everything
  works." Seeded the whole cluster/Pi-cluster initiative. Clean.
- **CI/CD GitHub Actions** — the dataset flagged it "staged-not-applied," but `.github/workflows/matrix.yml`
  + `nightly.yml` **exist on disk** → BUILT/applied. (Re-shelve: NOT dropped.)
- **Unattended "submit 10 changes before bed" cron-tick mode** — designed-for-later in the multi-change
  scheduler plan (cron-tick as a 2nd invocation path); explicitly deferred, default = agent-tick.
- **"Separate future mode for coupled changes from main only"** — folded into the scheduler plan as a design
  decision (coupled work runs serial from main). Designed, not built — deliberate.
- **Async-throughput rabbit hole** — landed as `research/rabbit-holes/RH-014-async-throughput-and-token-saves/`.
- **Sharding "max cost ceiling" decision** — discussed across RH-021/swarm plans + `cost-tests-plan.md`;
  cost framing exists. Borderline, but covered enough to not be a clean drop.

## BUILT (floated → shipped — the demotions, with evidence)

The following were flagged in the dataset as possibly-dropped but are **confirmed on disk** — do NOT re-flag:

- **Parallel-work mechanism rule (Agent BG vs spawn_task default)** — flagged "durable artifact MISSING" in
  3 sessions, but **CAPTURED LATER**: `audits/findings/spawn-task-keep-or-drop-verdict-2026-06-02.md` holds
  the explicit DECISION RULE ("THIS session will consume the result → AGENT subagent ← THE DEFAULT") +
  `feedback_bg-lifecycle-discipline.md` Rule 6 (spawn-when-it-earns-it). The musing got its durable rule.
- **Skill use-count tracker** — `scripts/run-skill-usage-count.sh`, **wired** into `dashboard-merge-run.sh`.
- **Kanban / zoomed-out / column board view** — tracked as `ideas/I-005-zoomed-out-board-view.md` (+ I-001
  threads, I-002 card-slide) under G18. Stockpiled, not dropped.
- **the reference design design-system refs** — dataset said "only 1 of 8 built"; **5 exist** (section-archetypes,
  typography-roles, light-section-pattern, staged-reveal-choreography, color-mortise).
- **Heartbeat "fullest potential"** — `organic-os/heart/README.md` documents the 3-tier hierarchy +
  "what heartbeat means" — the multi-meaning expansion the user asked for. Substantially addressed.
- **Organs subsystem** (`organic-os/organs/README.md`), **Brain/CoT** (`organic-os/brain/README.md`),
  **Immune** — all built; CoT↔goal-billboard cross-ref present.
- **Cross-skill "Related skills" auto-link hook** — `scripts/hooks/skill-cross-link-rebuild.sh`.
- **Git deny-list expansion** (pull/fetch/merge/rebase/rm) — in `settings.json` (5 matches).
- **Chrome plugin dynamic 10-modes** — `agentic-chrome-tools` (15 refs).
- **never-idle / research-furthering under-fill** — `research-furthering-daemon.sh` +
  `never-idle-refill-surface.sh` + skill (the recurring under-fill complaint got a structural guard).
- **A40 idea-gap detector, A44 critical-finding reflex, A39 slam-meter, A36/A37 true-time, A41 JSONL-uses
  brainstorm, A42 emulator-killer (base), G11 session-bundle, dual-worktree calibration, multi-change
  scheduler (G1)** — all have artifacts on disk.
- **"Surpass the reference design on responsive," "bring over from last Evium attempt," "source-dive reflex"** —
  absorbed into shipped work / the page-scrutiny "measure before declaring" reflex (NB: the *measure-via-
  Chrome* reflex shipped; a literal "read reference SOURCE before Chrome" standing rule did not — low-value,
  not separately flagged).

---

## Recommendation

Surface the **3 HIGH-leverage drops** (preview-tools skill · skill-maker generative skill · token-saving
adoptions starting with Batch API) as goal-billboard **proposals** for the user — each is a real,
confirmed-unbuilt slam-dunk the user personally floated. The 7 MED/LOW drops are idea-capture candidates for
`ideas/` (especially #9, the 12 dashboard items, which is itself an evaporation example). All 10 fed to the
watchdog (`verdict.jsonl`, lobe `floated-idea-dropped`, `acted:false`).
