---
goal_id: G12
title: Local AI mass-wielding (single Mac) — token-free heavy lifting via MLX/Ollama
status: paused
paused_reason: deferred until after Evium (G2); was live-but-empty. Research still attaches under G12 as a paused constellation.
track: infrastructure
northern_star: false
guiding_light: true
priority: P1
created: 2026-05-27T02:34:13Z
updated: 2026-05-27T03:15:25Z
serves_northern_star: G2
linked_plans:
  - context/markdowns/plans/automation/local-ai-single-mac-mass-wielding-plan.md
linked_audits:
  - A33 — Local AI mass-wielding research initiative (origin)
linked_skills:
  - .claude/skills/agentic-script-design/  (distributed-systems-patterns + bg-dispatch-architecture)
linked_bugs: []
linked_changes: []
related_goals: [G13]  # 2026-05-31 — sibling local-AI goal (single-Mac → cluster). Linked, NOT merged: G12 is the single-Mac path, G13 the multi-Mac cluster extension; both paused-until-after-Evium (G2). Per user: "12/13 pause makes sense they dont need to be merged but can be linked."
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G12 — Local AI mass-wielding (single Mac)

## Why it exists

User vision 2026-05-27 00:55Z: *"orchestrate insanely powerful swarms all local no token bottleneck at all for the heavy work while we save all our token usage for you."*

This goal is the **single-Mac path** to that vision. One Mac running MLX/Ollama; multiple specialist models swapped/loaded; high-volume mechanical work served locally. Claude reserved for orchestration.

## Why it's load-bearing

Token economy reshape:
- Claude 5hr-window ceiling stops being the binding constraint for high-volume work
- Per-BG token cost drops from ~100-500k → essentially zero (compute only)
- Project velocity decouples from Claude API limits

For G2 (Northern Star) specifically:
- Per-cell visual audit on 696 cells becomes free — full matrix joyride mode
- Diff review on every change runs locally
- Test generation expands without budget concern

## Current state

- Phase 0 ✅ (this turn): A33 audit + master plan + this goal + sister G13 + memory rule queued
- Phase 1: Research Front A (model landscape) — BG queued; launches when A21 ceiling opens
- Phases 2-7: queued per master plan

## Blockers

- Phase 1: queued behind A21 ceiling (5 BGs in flight)
- Phases 5-7 depend on Phases 1-4 research outputs
- No hardware blocker (this Mac is the substrate)

## Next action

- Wait for A21 ceiling to open
- Launch Research Front A (landscape survey) as first BG when slot available
- Subsequent fronts queue per master plan

## Exit criteria

Phase 5 (pilot): one local AI worker runs at least one task class with measurable token-savings vs Claude API equivalent.

Phase 6 (production): ≥3 use cases routed to local AI; cumulative token-savings ≥40% on those task classes.

**G12 exits as `achieved` when local AI absorbs >50% of mechanical BG work without quality regression.**

## History

- 2026-05-27 01:00Z: G12 created. A33 audit + master plan + sister G13 landed. Architecture sketch + 7-phase plan + 5-research-front scoping ready. Token economy paradigm shift identified.

## Notes

- G12 SERVES G2 indirectly — better infra = faster G2 work
- After G2 ships + G12 lands, G12 may become candidate for next Northern Star (infrastructure that unlocks ALL future deliverables faster)
- Per A33 cost-discipline matrix: certain task classes ALWAYS stay Claude (orchestration, strategy, novel design). Local AI never replaces those.
- Sister G13 (cluster swarms) extends this; G13 is gated on G7 Phase 2 hardware.
