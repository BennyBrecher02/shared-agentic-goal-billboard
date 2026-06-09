---
goal_id: G7
title: Cluster + true parallelization (multi-Mac + Raspberry Pi)
status: paused
superseded_by: G17
paused_reason: collapsed into G17 umbrella (user decision 2026-05-29). G7's cluster vision is now a sub-part of G17's heterogeneous-cluster scope. Reversible: unpause if cluster work needs standalone tracking.
track: infrastructure
northern_star: false
guiding_light: true
priority: P1
created: 2026-05-26T23:00:00Z
updated: 2026-05-29
serves_northern_star: G2
umbrella_relationship: G17  # 2026-05-28 A74-D2: G17 declares absorbs_goals:[G7]. But G7 is a guiding_light (the cluster PRINCIPLE); G17 is the active EXECUTION umbrella. Relationship documented; NOT paused — a guiding light's state change is the user's call. Likely correct resolution: G7 stays guiding-light, G17 executes it.
linked_plans:
  - context/markdowns/plans/automation/cluster-kickoff-plan.md (Phase 1)
  - context/markdowns/plans/automation/cluster-readiness.md (architectural)
  - context/markdowns/plans/automation/m2-ssh-scheduler-integration.md (Phase 2)
  - context/markdowns/plans/automation/m2-mac-test-farm.md (Phase 3)
linked_audits:
  - A11 — concurrency-ceiling (single-machine bottleneck)
  - A21 — parallel capacity (single-machine BG ceiling)
  - A22 — cluster + NS integration gap (origin of this goal)
linked_bugs: []
linked_changes: []
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G7 — Cluster + true parallelization

## Why it exists

Single-machine ceilings hit hard today:
- A11: Astro dev server saturates at 4+ concurrent Playwright workers → 3-shard parallel ceiling
- A21: BG-agent ceiling at 5 in-flight (token + coordination + context limits)

Wall-clock work today bottlenecked repeatedly. Cluster nodes are the only way past these ceilings: each additional Mac contributes ~3 parallel shards; each Raspberry Pi contributes ~2. A 4-node cluster gets us ~10 effective parallel shards — enough for the full Playwright matrix in one wall-clock window instead of 4 batches.

User framing 2026-05-26 PM: *"if i have a second mac slave then we could be running many tasks and going for many guiding lights to work in truer parallelization, so lets kick this off."*

## Current state (2026-05-26 PM)

- Architectural plans exist: `cluster-readiness.md`, `m2-ssh-scheduler-integration.md`, `m2-mac-test-farm.md` (all PARKED since 2026-05-26 AM)
- Today: only M4 active; cluster math = 1 node × 3 shards = 3 effective
- No hardware-independent foundation built yet — that's Phase 1 of cluster-kickoff-plan landing this turn
- Hardware status: user signal pending. May have second Mac available; may not. Foundation is independent.

## Phases

### Phase 1 — Hardware-independent foundation (THIS TURN)
- `scripts/cluster/` scaffolding (inventory, readiness-check, dispatch shim, lib-common)
- Dashboard "Cluster" panel + lefter mini-block
- Renderer for `data.json` cluster block
- BG agent launching this turn implements Phase 1

### Phase 2 — Hardware-gated (M2 SSH integration)
- Real SSH dispatch in `dispatch.sh`
- Scheduler accepts `target_node:` on Change records
- Worktree path normalization
- Artifact transfer (rsync)
- **Trigger: user signals "M2 ready" + provides SSH config**

### Phase 3 — Hardware-gated (Test-farm + Pi)
- Distributed Playwright matrix execution
- Cross-node coordination
- Raspberry Pi Linux node integration
- Node-failure handling

### Phase 4 — Optimization
- Load balancing across nodes
- Per-node cost tracking
- Auto-failover

## Blockers

- Hardware availability for Phase 2+ (user signals)
- A21 parallel-launch-discipline (just landed; hooks for foundation are now ready to land on cluster's dispatch primitive)

## Next action

- Phase 1 BG agent launching this turn → `scripts/cluster/` + dashboard panel
- After Phase 1 lands: foundation tested against localhost; ready for hardware
- Ask user: "M2 hardware status? Do you have SSH access to a second Mac, or is this still TBD?"

## Exit criteria

- Phase 2: M2 accepts dispatched tasks via SSH; scheduler routes Changes to it; matrix runs return artifacts
- Phase 3: ≥10 effective parallel shards measured; full matrix in 1 wall-clock window
- Phase 4: load balancer + failover ON
- **Final exit:** cluster sustains 12-project Playwright matrix in <5min wall-clock (currently 15+ min on M4 alone)

## History

- 2026-05-26 AM: cluster-readiness.md + m2-ssh-integration plans created; PARKED
- 2026-05-26 PM (this turn): A22 audit catches the parking gap; G7 created; Phase 1 kickoff plan written; impl BG launching

## Notes

- This goal SERVES the Northern Star (currently G2). Cluster speed-up makes future NS work faster.
- After G2 ships, G7 becomes a candidate for the next Northern Star (or stays infra if next NS is content/feature work).
- Each cluster node is its own resource pool — A21 ceiling of 5 BGs applies PER MACHINE; cluster effectively multiplies the agent throughput.
