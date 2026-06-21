---
goal_id: G14
title: End-of-session realignment ritual + master system audit (Stage 3 of post-shutdown pipeline)
status: active
track: quality
northern_star: false
guiding_light: true
priority: P1
phase: "Active — the ACT half of the capture-vs-act seam with G11 (kept deliberately separate, per the 2026-05-31 rewrite). The end-of-session realignment ritual + master system audit (A34 origin) is the Stage-3 post-shutdown step that consumes G11's captured session bundle and drives the realignment. Concrete next: pin the ritual's trigger + the master-audit module set against the live bundle. (Phase line populated + stamp refreshed per goal-grooming-proposal-2026-06-21.md §4-G.)"
created: 2026-05-27T02:42:15Z
updated: 2026-06-21T21:01Z  # refreshed + empty phase populated per goal-grooming-proposal-2026-06-21.md §4-G
serves_northern_star: G2
linked_plans:
  - context/markdowns/plans/automation/end-of-session-realignment-ritual-plan.md
linked_audits:
  - A34 — End-of-session realignment ritual + master system audit (origin)
linked_skills:
  - .claude/skills/agentic-quality-discipline/references/session-bundle-discipline.md (G11 Phase 1)
  - .claude/skills/reactionary/ (sibling event-triggered pattern)
linked_bugs: []
linked_changes: []
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G14 — End-of-session realignment + master system audit

## Why it exists

User vision 2026-05-27 03:10Z: *"new hook at the end of every session after the bundle analyzer produces the Master session writeup we us it as a moment for realignment and then we would launch a master system audit, expand on all of this and all other ideas for the analyzer and post analysis hooks."*

**Boundary against G11 (capture-vs-act seam):** G11 = CAPTURE the session journey (within-session
bundle + analyzers). G14 = ACT on it across sessions (cross-session recurrence detection + a
realignment digest that pre-seeds the next session).

This goal **builds on G11 (bundle/analyzer)**. G11 captures the journey within a session; G14
detects patterns across sessions and triggers realignment — master system audit + realignment
digest + the post-analysis hooks that cascade *structural* action (the count is an outcome, not a
target).

## Why it's load-bearing

Per-session retrospection is incomplete without cross-session synthesis. G11 captures the journey within a session. G14 detects patterns across sessions and triggers realignment.

Specifically:
- Cross-session recurrence (Module F) catches systemic issues no within-session A19 can see
- Health scorecard quantifies agentic-OS state into a single comparable metric
- Realignment digest pre-populates next session's focus (no human-needed-to-remind-agent)
- Post-analysis hooks cascade STRUCTURAL action from insights (not just reports)

Per A29 zero-degradation criterion + A30 Phase 5: realignment digest fed into resume briefing means next session starts with FULL context + concrete priorities, never lazy.

## Current state

- Phase 1 ✅ (this turn): A34 audit + 8-phase master plan + this goal + memory rule queued
- Phase 2: G11 Phase 2 (4 more analyzers) — BG queued
- Phases 3-8: queued per master plan

## Blockers

- Phase 1: ✅
- Phase 2: queued behind A21 ceiling (BGs in flight)
- Phase 3+: depends on Phase 2
- Phase 8 (local AI): depends on A33 G12 pilot

## Next action

- Phase 2 BG launches when ceiling opens (FIFO queue)
- Phase 3 starts when Phase 2 lands

## Exit criteria

- The realignment digest is generated automatically at session end **and** feeds the resume
  briefing — so the next session opens with concrete priorities, never cold.
- The cross-session recurrence module is live and produces a real alert on ≥3 consecutive bundles.
- The post-analysis hooks that cascade *structural* action are in place (the count is an outcome,
  not a target — no literal hook-count is the goal).

**G14 exits as `achieved` when 5 consecutive sessions produce master audits + realignment digests + post-analysis cascades land successfully + recommendations from audit N are visibly addressed in session N+1.**

## History

- 2026-05-31: Goal-definition rewrite applied per user's 2026-05-31 chamber decision (goal-rewrite-drafts.md G14): added the G11-vs-G14 capture-vs-act boundary line to "Why it exists"; replaced the "13 post-analysis hooks" literal in the live body with "the post-analysis hooks that cascade structural action (count is an outcome, not a target)"; rewrote Exit-criteria to the 3 outcome bullets (auto realignment digest → resume briefing; cross-session recurrence alert on ≥3 bundles; structural-action cascade hooks, no literal count). The "13-post-hooks" mention in the 2026-05-27 dated history entry below is left verbatim as a record of what was said then. Per draft: G11's mirror boundary line is flagged but NOT written here (single-goal edit scope). Links + history otherwise untouched.
- 2026-05-27 03:15Z: G14 created. A34 audit + 8-phase plan landed. Extends G11 Phase 1 POC (just landed) with cross-session synthesis + master system audit + 13-post-hooks ecosystem.

## Notes

- G14 SERVES G2 indirectly (better retrospection = compounding learning = faster G2 delivery + future deliverables)
- Local AI (A33 G12 Phase 8) is THE enabler for token-free master audit pipeline
- Cross-references heavily into existing audits (A24 monthly, A28 skill health, A29 pushback, A19 recurrence) — A34 unifies their per-session retrospection in one ritual
- The "master writeup as TRIGGER not passive document" insight from user is the structural lever
