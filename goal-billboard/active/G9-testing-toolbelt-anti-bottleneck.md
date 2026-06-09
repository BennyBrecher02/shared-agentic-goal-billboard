---
goal_id: G9
title: Testing toolbelt — cluster-integrated, anti-bottleneck (NEVER a time/token bottleneck)
status: active
track: quality
northern_star: false
guiding_light: true
priority: P0 (elevated — user 25× stress)
created: 2026-05-27T00:18:00Z
updated: 2026-05-27T00:18:00Z
serves_northern_star: G2
user_stress_level: maximum (25× repetition)
linked_plans:
  - context/markdowns/plans/automation/testing-toolbelt-plan.md
linked_audits:
  - A27 — Testing toolbelt + anti-bottleneck (origin)
  - A11 — Single-machine concurrency ceiling (related)
  - A14 — Token-burn analysis (related)
linked_skills:
  - .claude/skills/agentic-testing-discipline/  (NEW this turn)
linked_bugs: []
linked_changes: []
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G9 — Testing toolbelt (cluster-integrated, anti-bottleneck)

## Why it exists

User explicit framing 2026-05-26 23:18Z, repeated **25 times** in one message: *"i dont ever want testing becoming a time/tokencost bottleneck."*

The repetition IS the spec. Build a full testing toolbelt — TDD plus 13 other mindsets — iteratively over the cluster/sharding/scheduler/opsys push. Every phase has cost gates. If breached: PAUSE + reassess.

The current testing reality:
- 5 Playwright spec files (a11y, rail, responsive, visual, vq-capture)
- npm scripts for matrix runs
- A11: 15+ min full-matrix on single Mac
- **No unit tests for any of the ~30 hooks, ~10 scheduler files, ~15 utility scripts**

The opportunity: expand coverage to substantial logic that's currently unverified — WITHOUT slowing velocity.

## Strategy

**Cost-discipline first** (Pillar 1 of the new skill): tier every test (fast/medium/slow/nightly); affected-tests-only mode; aggressive caching; cluster distribution.

**Diverse mindsets** (Pillar 2): TDD + BDD + property-based + mutation + contract + fuzz + chaos + visual + smoke + load + static + integration + e2e + affected-only + cached.

**Cluster-integrated** (Pillar 3): use G7 cluster substrate to distribute slow tiers.

## Current state

- Phase 1 (this turn): A27 audit + testing-toolbelt-plan + new skill `agentic-testing-discipline` + 2 initial refs (taxonomy + anti-bottleneck) + 3 parallel BGs (substrate baseline, affected-tests, caching)
- Phases 2-5: queued

## Blockers

- Phase 5 depends on G7 cluster Phase 2 (hardware-gated SSH dispatch). Phases 1-4 land independently.

## Next action

- Phase 1 BGs complete + reports reviewed
- Phase 2 (tier system + scheduler integration) queued

## Exit criteria

Phase 1 acceptance:
- A27 + plan + skill exist
- 3 BGs land their reports
- Memory rule live
- Skill cross-link from related skills (agentic-device-testing, agentic-script-design)

Phase 5 (long-term) exit:
- Median P0 fast tier <30s
- Cluster-distributed full matrix <5min wall-clock
- Cache hit rate >70% steady state
- 14-type toolbelt has ≥1 proof-of-concept test per type
- **Testing never measurably slows development velocity** — measured by the conversation-pattern KPI showing no user complaints about test latency

## History

- 2026-05-27T00:18Z: G9 created. A27 audit catches the gap + designs the toolbelt + 10 anti-bottleneck strategies. User's 25× stress encoded as Pillar 1 (cost-discipline) of the new skill. 3 Phase-1 BGs launching: baseline + affected-only + caching. Anchor goal for cluster (G7) integration.

## Notes

- This goal SERVES the Northern Star (G2). Better testing makes future client-deliverable work faster, not slower.
- After Phase 5 lands + cluster Phase 2 is real, this goal becomes a candidate for the next Northern Star (anti-bottleneck testing infra IS the deliverable that lets every future deliverable be fast).
- Memory rule (testing-cost-discipline) is non-negotiable. If I ever consider running tests in a way that slows velocity, abort.
