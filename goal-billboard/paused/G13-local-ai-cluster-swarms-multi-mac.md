---
goal_id: G13
title: Local AI cluster swarms (multi-Mac) — distributed local inference across G7 cluster
status: paused
paused_reason: deferred until after Evium (G2); was live-but-empty. Research still attaches under G13 as a paused constellation.
track: infrastructure
northern_star: false
guiding_light: true
priority: P2 (depends on G7 Phase 2 hardware + G12 single-Mac patterns)
created: 2026-05-27T02:34:30Z
updated: 2026-05-27T03:15:25Z
serves_northern_star: G2
linked_plans:
  - context/markdowns/plans/automation/local-ai-cluster-swarms-plan.md
linked_audits:
  - A33 — Local AI mass-wielding research initiative (origin)
linked_skills:
  - .claude/skills/agentic-script-design/  (distributed-systems-patterns + bg-dispatch-architecture)
linked_bugs: []
linked_changes: []
related_goals: [G12]  # 2026-05-31 — sibling local-AI goal (single-Mac → cluster). Linked, NOT merged: G13 is the multi-Mac cluster extension of G12's single-Mac path; both paused-until-after-Evium (G2). Per user: "12/13 pause makes sense they dont need to be merged but can be linked."
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G13 — Local AI cluster swarms (multi-Mac)

## Why it exists

User vision 2026-05-27 00:55Z: *"a completely separate similar second research for macs with clusters like mine, and try to find some way where we can orchestrate insanely powerful swarms all local."*

This goal is the **multi-Mac cluster path** to that vision. N nodes orchestrated as one swarm. Specialist-per-node (each Mac runs a different model) OR replicated-for-throughput (all nodes run same model; load-balanced). Sister track to G12 single-Mac.

## Why it's load-bearing

Cluster multipliers on G12:
- 4 nodes × 3 shards/node = 12 parallel inference channels
- Specialist swarm: distinct workloads simultaneously (vision + code + triage + test)
- Cross-cluster experimentation: 3 different fix strategies tested in parallel on 3 nodes; pick winner — ENTIRELY NEW capability class

For G2 + G7:
- Full 12-project matrix WITH per-cell vision evaluation in parallel
- Project velocity bounded by cluster size, not by Claude tokens or single-Mac ceiling

## Current state

- Phase 0 ✅ (this turn): A33 + master plan + G12 sister + this goal landed
- Phase 1: Research Front D (cluster swarm patterns) — BG queued
- Phases 2-6: gated on G7 Phase 2 hardware (M2 SSH integration)

## Blockers

- Phase 1 research: queued behind A21 ceiling
- Phase 2+ impl: HARD blocked on G7 Phase 2 (user signals "M2 ready" + SSH config done)
- Research can advance independently of hardware

## Next action

- Phase 1 research BG queues when A21 ceiling opens
- Watch G7 Phase 2 progress (user-gated; depends on hardware availability)
- When G7 Phase 2 lands → activate G13 Phase 2-6

## Exit criteria

Phase 5 (production cluster swarm): cluster runs production workload with ≥3× throughput vs single-Mac for at least one use case class.

Phase 6 (cross-cluster experimentation): demonstrated capability to run N candidate fixes in parallel on N nodes + decision-making from aggregated results.

**G13 exits as `achieved` when "full matrix joyride" runs in <2min wall-clock with cluster + local AI.**

## History

- 2026-05-27 01:00Z: G13 created. A33 + master plan + G12 sister landed. 7-phase plan; Phase 1 research achievable without hardware; Phases 2-6 gated on G7 Phase 2.

## Notes

- G13 BUILDS ON G12. Single-Mac patterns extend horizontally.
- HARD dependency: G7 Phase 2 (cluster hardware impl)
- Cross-cluster experimentation is the FORCE-MULTIPLIER capability — try 3 strategies in parallel, evidence-based decision
- Per user explicit: "completely separate similar second research" — distinct from G12 in scope + timeline
