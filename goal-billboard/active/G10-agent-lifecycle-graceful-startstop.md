---
goal_id: G10
title: Agent lifecycle — graceful start/stop with zero-degradation resumption (laptop-suspend-resume model)
status: active
track: quality
northern_star: false
guiding_light: true
priority: P0 (elevated — user gated actual shutdown on completion of this overhaul)
created: 2026-05-26T23:50:00Z
updated: 2026-06-21T21:01Z
status_tag: awaiting-user
gated_on: "USER signal — the explicit 'OK-to-shut-down' for Phase 4. Phase 3 LANDED (round-trip test 99/100, wired into npm test:medium, A30 Phase-4 verification gate CLEARED on the structural-test side). Phase 4 (the actual graceful shutdown) cannot proceed without the user's deliberate shut-down OK. A decision to put in front of the user, NOT a stall. (Surfaced per goal-grooming-proposal-2026-06-21.md §4-D.)"
phase: "Phase 3 LANDED — round-trip test built + first run 99/100 PASS; wired into npm test:medium; A30 Phase 4 verification gate CLEARED on the structural test side; AWAITING USER explicit OK-shut-down signal for Phase 4 (user-gate — see gated_on)"
serves_northern_star: G2
linked_plans:
  - context/markdowns/plans/automation/graceful-shutdown-startup-overhaul-plan.md (master; this overhaul)
  - context/markdowns/plans/automation/graceful-shutdown-infrastructure-plan.md (foundation; superseded by overhaul)
linked_audits:
  - A30 — Graceful shutdown/startup overhaul (origin)
  - A29 — Lazy regression-to-mean (zero-degradation criterion derives from)
linked_bugs: []
linked_changes: []
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G10 — Agent lifecycle (graceful start/stop)

## Why it exists

User explicit framing 2026-05-26 23:50Z: *"similar to how context swaps state capturing type stuff alows laptops to close and stop and restart wihtout apps even realizing... when we start again we dont work in a lazier slower state, it shouldnt even be noticeable, so lock in on this."*

The laptop suspend/resume model applied to our agent session. Existing graceful-shutdown work (from gap-fill BG #71) is FOUNDATION; the overhaul makes resume INDISTINGUISHABLE from no-stop.

## Why it's load-bearing

Without this:
- User drives home + opens new session next morning → no context preserved → agent starts in "lazy mode" (A29 regression)
- BG agents in flight during shutdown → their completions invisible on resume
- Pending substrate (monkey decisions, settings patches, audits in_progress) → re-discovered from scratch
- Token-budget continuity (5hr-window, cumulative) → lost
- Northern Star focus → has to be re-derived

The user has called out the lazy-startup pattern explicitly. This goal closes it.

## Current state

- Phase 1 ✅ (2026-05-26 23:50Z): A30 audit + overhaul plan + this goal landed in foreground
- Phase 2 ✅ (2026-05-27 00:08Z): snapshot + resume scripts landed; skill ref v2; foundation plan superseded; memory rule added; refresh-history logged
- Phase 3 ✅ (2026-05-27 00:14Z): round-trip test built (`scripts/test-shutdown-resume-roundtrip.sh`) + first run scored 99/100 PASS; wired into `npm run test:medium`; skill ref `shutdown-resume-roundtrip-test.md` shipped
- Phase 4: actual shutdown user-gated — awaiting explicit "OK shut down now" signal
- Phase 5: first post-resume verification (next session)

## Blockers

- Phase 2 ✅ unblocked (BG completed 2026-05-27 00:08Z).
- Phase 3 ✅ unblocked (test landed + first run 99/100 PASS 2026-05-27 00:14Z).
- Phase 4 (actual shutdown) blocked ONLY on user explicit "OK shut down now" signal — structural verification gate is cleared.

## Next action

- ~~Wait for current 5 BGs to land~~ ✅
- ~~When 1+ ceiling slot opens → launch Phase 2 BG (snapshot + resume scripts)~~ ✅
- ~~After Phase 2 lands → launch Phase 3 BG (round-trip test)~~ ✅
- Notify user: A30 Phase 4 verification gate CLEARED — graceful shutdown is now safe to execute on user explicit go
- Await user "OK shut down now"; then execute Phase 4
- Phase 5 (next session): verify resume works in real life

## Exit criteria

Phase 1: ✅ audit + plan + goal landed
Phase 2: snapshot script exists + runs <2s + resume script exists + emits comprehensive briefing
Phase 3: round-trip test exists + passes with score ≥95/100
Phase 4: snapshot executes cleanly + user confirms ready
Phase 5: first post-resume session demonstrates ZERO degradation per A29 criteria (≥1 BG launched in first response; FLOOR rule held; no user pushback)

**Goal exits as `achieved` after Phase 5 verifies in real session.**

## History

- 2026-05-26 23:50Z: G10 created; A30 audit + master plan landed; Phase 1 complete; subsequent phases queued.
- 2026-05-27 00:08Z: Phase 2 LANDED. `scripts/graceful-shutdown-snapshot.sh` (1s wall-clock, 13-field schema, atomic write, archive copy, PID-validated lock state) + `scripts/graceful-startup-resume.sh` (silent on absence, freshness banding, PID-validated stale-lock cleanup, BG-completion-fetch during gap, comprehensive `<system-reminder>` briefing, marker write) shipped. Skill ref v2 (snapshot/resume model + resume failure modes + "what NOT to do at resume"). Foundation plan marked superseded. Memory rule added to `feedback_standing-protocols.md` (+ mirror). 8-of-8 verification checks passed. Phase 3 (round-trip test) next.
- 2026-05-27 00:14Z: Phase 3 LANDED. `scripts/test-shutdown-resume-roundtrip.sh` (7-phase 0-100 scoring; tier=medium; --help/--json/--verbose flags; trap cleanup; ALWAYS exit 0; uses --path against resume + cache fallback) + `.claude/skills/agentic-testing-discipline/references/shutdown-resume-roundtrip-test.md` (12.5K skill ref with failure-mode catalog) + `scripts/run-tests.sh` wired (medium tier now includes the round-trip test in --dry-run listing + actual invocation). **First run: PASS 99/100** (b=10/10, c=24/25, d=30/30, e=15/15, f=15/15, g=5/5). A30 Phase 4 verification gate CLEARED on the structural side. Awaiting user explicit "OK shut down now" signal.

## Notes

- This goal SERVES the Northern Star (G2) — even client deliverable work is at risk if session continuity breaks. A failed resume next session could cost hours of context-rediscovery time.
- Zero-degradation criterion is A29's lazy-regression problem applied to session boundaries.
- Cluster (G7) Phase 2+ will eventually require multi-node graceful shutdown — out of scope for G10 v1; cluster Phase 4 will extend.
- Round-trip test is a new test class for the testing toolbelt (G9); reinforces G9's anti-bottleneck discipline.
