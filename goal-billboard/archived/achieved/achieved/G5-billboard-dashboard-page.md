---
goal_id: G5
title: Billboard dashboard page (HTML view of goal billboard)
status: achieved
phase: MVP shipped 2026-05-26T13:58Z; polish (auto-render hook + click-through links) deferred
priority: P1
created: 2026-05-26T14:35:00Z
updated: 2026-05-26T14:00:00Z
achieved: 2026-05-26T14:00:00Z
linked_plans:
  - context/markdowns/plans/operational-dashboard-expansion.md  (page-3 portion only; pages 2 + 4 remain parked under G3)
linked_refs:
  - .claude/skills/agentic-quality-discipline/references/goal-billboard.md
linked_bugs: []
linked_changes: []
serves_northern_star: G2  # migrated 2026-05-26 - goal_id=G5 (not NS) → GL; NS defaults to G2
belongs_to_goal: G18
serves_guiding_light: G5
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G5 — Billboard dashboard page

A single rendered HTML view of the goal billboard — goals (active + paused), audits (catalog + archived), plans (grouped by goal), unscoped plans. The "what are we working on right now" page made browsable.

## Why it exists

The terminal output from `goal-billboard-status.sh` is good for SessionStart digest but cramped for browsing. A web page with proper hierarchy + click-through to source markdown makes the billboard navigable.

User explicitly requested this (2026-05-26 PM): "i mainly just wanted a goal billboard dashboard page."

## Scope (carved out from operational-dashboard-expansion.md)

**In scope (this goal):**
- Page-3 from the operational-dashboard-expansion plan
- HTML rendering of:
  - Active goals with title, priority, phase, exit criteria, history
  - Paused goals (collapsed)
  - Audit catalog (table view)
  - Archived audits (collapsed)
  - Plans grouped by goal
  - Unscoped plans
- Output to `public/billboard.html` (or symlink target)
- Reuse `scripts/goal-billboard-status.sh` frontmatter-parsing logic (rewrite in Python for HTML output)

**Out of scope (stays paused under G3):**
- Page-2 (scheduler live peek)
- Page-4+ (skill graph, decision timeline, hook firings, token spend)
- Multi-page navigation between dashboard pages
- Auto-refresh / polling

## Current state

Just kicked off. Building the renderer.

## Blockers

None. All data sources are filesystem-readable + already parsed by `goal-billboard-status.sh`.

## Next action

Build `scripts/render-billboard-page.py` that:
1. Reads frontmatter from goal-billboard files + plans/**/*.md
2. Groups by goal_id, sorts, renders HTML
3. Outputs to `reports/billboard.html` (symlinkable under public/)

## Exit criteria

- [x] HTML page renders all 4 active goals + 1 paused + 10 audits + 1 archived + 19 plans (verified)
- [x] Page is browsable in a browser (static HTML, no JS needed; verified via preview server)
- [x] Page regenerates when invoked (`python3 scripts/render-billboard-page.py`)
- [x] Standalone render script (deferred: post-audit-timelapse.sh integration)
- [ ] (deferred polish) Click-through to source markdown when curious

## History

- 2026-05-26T14:35Z — Goal created. Carved out from G3's operational-dashboard-expansion plan.
- 2026-05-26T14:00Z — MVP shipped. `scripts/render-billboard-page.py` writes `reports/billboard.html` + symlinked at `public/billboard.html`. Verified rendering via preview server. Click-through links + auto-render hook deferred as polish.

## Notes

- Reuses dashboard CSS where reasonable (don't reinvent)
- Static HTML; no JS framework
- Frontmatter parser shared with billboard-status.sh — refactor into shared Python module
- Auto-regenerate at SessionStart? — Defer; manual invocation first
