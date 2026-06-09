---
goal_id: G3
title: Audit dashboard v2 → v3 evolution
status: paused
track: tooling
phase: v2 shipped; v3 deprioritized — resumes after G1 Phase A.5
priority: P1
created: 2026-05-25T10:00Z
updated: 2026-05-31T12:55Z
superseded_by: G18  # 2026-05-31 — G3's dashboard/board/timelapse items re-homed to the Dev Control dashboard goal (G18). G3 stays paused as the historical v2→v3 audit-timelapse record; new dashboard work lives under G18.
paused_reason: Deprioritized in favor of scheduler MVP. v2 fully meets current needs. Resume when G1 stabilizes (post-A.5) or when a v3 feature becomes a blocker.
linked_plans:
  - context/markdowns/notes/timelapse-design.md
  - context/markdowns/plans/operational-dashboard-expansion.md  (NEW 2026-05-26 — broadens G3 scope to multi-page operational console)
linked_refs:
  - .claude/skills/agentic-page-scrutiny/SKILL.md
linked_bugs: []
linked_changes: []
serves_northern_star: G2  # migrated 2026-05-26 - goal_id=G3 (not NS) → GL; NS defaults to G2
serves_guiding_light: G3
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G3 — Audit dashboard evolution

## Why it exists

The audit time-lapse dashboard turns 1400+ per-section captures into a navigable, comparable, time-progressing artifact. v1 was a screenshot grid; v2 added multi-run + scratchpad rail + scrub-bar time-lapse + tabs.

This is the "make audit results usable" capability — without it, the audit captures are noise.

## Current state

- **v2 SHIPPED** (2026-05-25 PM):
  - Multi-run support (landing page lists every run)
  - Scratchpad rail (per-section findings inline next to image)
  - Before/after slot
  - Scrub-bar time-lapse (not autoplay slop)
  - Tabs (Worst sections + Reels)
  - Jump-strip with severity dots
  - Heatmap (single — duplicate dropped)
- Output at `public/audit-timelapse/` (symlinked from `reports/timelapse/`)

## Blockers

- None blocking; deprioritized in favor of scheduler MVP

## Next action (queued)

Not the current focus. Resume after G1 Phase A.5.

## Exit criteria

- [x] v2 ships (multi-run, scratchpad rail, scrub bar, tabs, jump-strip)
- [ ] v3.1: Per-route walkthrough reels (opt-in autoplay)
- [ ] v3.2: Per-section cross-device strips
- [ ] v3.3: Diff overlay between runs (visual delta highlighting)
- [ ] v3.4: Severity-trend chart over time

## History

- 2026-05-25 AM — original dashboard design (autoplay strip-reel) rejected as "slop"
- 2026-05-25 PM — 3-layout-toggle proposal → consolidated to tabs after iteration
- 2026-05-25 PM — v2 ships
- 2026-05-26 — deprioritized to background; G1 takes the foreground

## Notes

- NEVER autoplay anything on dashboard load (see `feedback_dashboard-centerpiece.md`)
- Per-route walkthrough reels are OPT-IN micro-reels, not global filmstrips
- Archive every run as immutable (see `feedback_audit-dashboard-shape.md`)
