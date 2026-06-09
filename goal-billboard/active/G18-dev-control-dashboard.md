---
goal_id: G18
title: Dev Control dashboard — the operational command surface
status: active
track: tooling
northern_star: false
guiding_light: true
priority: P1
phase: "Active build — 15 views live; deep iteration in flight (git-page UX overhaul, board goal-wiring fix, rail redesign); animated-monkey chamber feature pinned (P-019)"
created: 2026-05-31T12:42Z
updated: 2026-05-31T12:42Z
serves_northern_star: G2
linked_plans:
  - context/markdowns/plans/dashboard-monolith-split-plan.md
  - context/markdowns/plans/post-split-dashboard-builds.md
linked_research:
  - context/markdowns/research/systems/board-organization-upgrade.md
  - context/markdowns/research/systems/board-refocus-second-pass.md
  - context/markdowns/research/systems/git-page-uiux-overhaul.md
  - context/markdowns/research/systems/dashboard-rail-redesign.md
  - context/markdowns/research/systems/goal-wiring-analysis.md
  - context/markdowns/research/systems/board-ideas-backlog.md
  - context/markdowns/research/systems/monkey-chamber-animated-monkeys-research.md
linked_audits:
  - A78 — dashboard scrutiny + plan consolidation
  - A91 — git-page before/after fidelity
  - A92 — git-history forensic fact-check
linked_pins:
  - P-019 — animated-monkey SVG chamber (pinned feature)
linked_bugs: []
linked_changes: []
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G18 — Dev Control dashboard (the operational command surface)

## Why it exists

Created at the user's request (2026-05-31: *"make a new goal for our manifesto"*) to formalize what had been the dominant **un-goaled** build thread for days. The Dev Control dashboard is the user's single window into — and control over — the entire agentic OS: decisions (Monkey Communication Chamber), goals + all work (Board / corkboard), what changed on the site (Git / Rollback), test coverage (Matrix), plus Improvement, Stats, Subagents, Plans / Audits, Bugs, Scheduler, Dev Handoff, Asks. It is a 0-runtime-fetch single HTML file.

The goal-wiring analysis (`research/systems/goal-wiring-analysis.md`) showed this meta-work was scattered/orphaned with no home goal — dashboard plans were even mis-homing to unrelated goals for lack of a parent. **G18 is that home.**

## Why it's load-bearing

- It's the surface the user steers EVERYTHING from — including G2 (the Northern Star). A better command surface = faster, safer steering of all work.
- The "bottleneck" the user keeps naming (dashboard changes serializing on one monolith file) is THIS goal's central engineering tension — addressed by `dashboard-monolith-split-plan.md`.
- Every other initiative (scheduler/G1, local-AI/G12–G13, lifecycle/G10, rabbit-holes/G15…) becomes visible + actionable through it.

## Current state (2026-05-31)

- **15 views live**, 0-fetch, 4 live-swappable skins (Blueprint default).
- **Landed recently:** board refocus (Mission-default, G2-lead, ideas demoted) + P1 org upgrades (lifecycle Flow view, meaningful stack sort, stale auto-surfacing, 🐇 Rabbit-holes view); chamber reverted to clean-simple; matrix 2nd pass (full-width + Matrix-film); git trust work (A91/A92 + recreated before/afters + per-commit sub-card split).
- **In flight:** git-page UX overhaul (declutter 8→3 + honest "20→38 parts" counts + hybrid badge/flatten granularity); board goal-wiring fix (parser parent-field + linkage — the cohere fix); rail redesign (one-line label, regroup, wonky-height fix).
- **Pinned:** animated-monkey SVG chamber (P-019).

## Exit criteria

**The USER owns "done"** (per the manifesto rule — Claude never marks it solved). G18 reaches `achieved` only when the user signs off that the dashboard is at a finalized, professional state across the core views (Overview / Board / Chamber / Git / Matrix / rail), the board coheres post-wiring, and 0-fetch holds. Until then it stays `active`, however much is "built."

## History

- 2026-05-31T12:42Z: G18 created at the user's request to formalize the Dev Control dashboard buildout (the dominant un-goaled thread; manifesto §9 "Dashboard — the bottleneck"). Animated-monkey chamber feature pinned as P-019 the same turn.

## Notes

- The agentic-OS *system* goals (G8 deep-integration, G11 session-bundle, etc.) are the engine; **G18 is the surface.**
- Many orphan dashboard plans / research / audits should re-home here (`belongs_to_goal: G18`) — to be applied in the goal-wiring decisions phase (currently some are mis-homing elsewhere for lack of this parent).
