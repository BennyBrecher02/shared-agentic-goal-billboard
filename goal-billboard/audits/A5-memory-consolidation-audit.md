---
audit_id: A5
title: Memory consolidation audit
status: available
catalogued: 2026-05-26T09:50:00Z
priority_when_run: P2
estimated_effort: small
trigger: When MEMORY.md index entries cross 35 (currently 29) OR when 3+ entries have been confirmed-stale-but-not-removed
deferral_reason: 29 entries is below the 30-entry threshold our skill-maintenance-cadence calls for. Some entries (e.g., dashboard-centerpiece, audit-dashboard-shape) might consolidate but it's not urgent.
related_goals: [G4]
related_plans: []
related_refs:
  - ~/.claude/projects/<PROJECT_SLUG>/memory/MEMORY.md
  - .claude/memory-mirror/MEMORY.md
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'skill ref' -> GL G4; NS defaults to G2
belongs_to_goal: G4
serves_guiding_light: G4
---
# A5 — Memory consolidation audit

## Why this audit matters

Memory entries decay in value over time. Some get superseded (e.g., dashboard centerpiece v1 thinking is now obsolete after v2 shipped). Some get redundant (audit-dashboard-shape + dashboard-centerpiece overlap). Some grow stale facts (file paths that have moved). The `/consolidate-memory` skill handles this but needs to be RUN — this audit defines when.

## What it would look at

- For each entry in MEMORY.md: still relevant? superseded? redundant with another?
- File paths cited inside: still valid? (grep + filesystem check)
- Date-based entries with "2026-05-X" markers: are they still actionable?
- Identify merge candidates (e.g., two feedback entries about the same convention)
- Identify deletion candidates (e.g., a scratch decision that's been formalized into a skill ref since)

## Expected outputs

- Memory cull list with reasoning per entry
- Merge proposals (combine 2-3 entries → 1)
- Updated MEMORY.md after consolidation
- Mirror sync verification

## How to trigger

Whichever fires first:
- MEMORY.md crosses 35 entries (currently 29)
- 3+ entries confirmed stale but not yet removed
- The `/consolidate-memory` skill recommends a pass

## Notes

The `/consolidate-memory` skill is the canonical tool; this audit just defines when to invoke it for this project specifically.
