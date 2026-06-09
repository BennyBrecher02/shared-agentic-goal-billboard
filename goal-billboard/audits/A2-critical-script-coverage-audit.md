---
audit_id: A2
title: Critical-script test coverage audit
status: available
catalogued: 2026-05-26T09:50:00Z
priority_when_run: P1
estimated_effort: medium
trigger: After Phase A.5 (scheduler stable through 5 verified changes) OR after a real bug surfaces in any of the listed scripts OR client deadline behind us (post-2026-05-27)
deferral_reason: Panic-mode focus is shipping G2 (Evium Phase 5 overhaul). Test scaffolding for bash scripts is foundational but not urgent.
related_goals: [G4]
related_plans: []
related_refs:
  - .claude/skills/agentic-quality-discipline/SKILL.md
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'scheduler' -> GL G1; NS defaults to G2
belongs_to_goal: G9  # re-routed 2026-06-08 G4->G9 (UNCLEAR Decision 4): "critical script COVERAGE" is the testing-toolbelt domain (G9). Prior B4 pass defaulted it to G4 substrate; coverage signal is stronger for G9.
serves_guiding_light: G1
---
# A2 — Critical-script test coverage audit

## Why this audit matters

We have 18 bash scripts in `scripts/` with ZERO tests. Five of them run on every SessionStart or in destructive code paths. A bug in any one silently corrupts shared state (memory, billboard, worktrees, settings). We caught the equivalent of two such bugs today inside the scheduler module because it has unit tests; the bash scripts have no such safety net.

## What it would look at

**Tier 1 (critical, zero tests, runs on every session):**
- `scripts/sync-memory.sh` — memory mirror sync. Bug = subagent drift.
- `scripts/bug-billboard-consolidate.sh` — modifies shared master.md.
- `scripts/calibration-cleanup.sh` — runs destructive ops on worktrees.

**Tier 2 (PreToolUse blockers — bug = either too permissive (data loss) or too strict (broken workflow)):**
- `scripts/hooks/worktree-remove-safety.sh` — already had a too-narrow-regex bug fixed 2026-05-26
- `scripts/hooks/settings-edit-guard.sh` — same risk class

**Tier 3 (scheduler internals with thin coverage):**
- `scripts/scheduler/__main__.py` — CLI entry + PID lock + stale-lock recursive cleanup (132 lines, 0 test files)
- `scripts/scheduler/resources/advisory.py` — 2 bugs fixed today (page-size, pmset streaming); 2 test files isn't enough
- `scripts/scheduler/shards/playwright_project.py` — 1 test file; just had artifact-path bug

## Expected outputs

- `bats` (Bash Automated Testing System) installed and a `tests/bash/` dir established
- Test files for the 5 Tier 1+2 scripts (~8-15 tests each)
- Additional pytest files for Tier 3 (~5-10 tests per module)
- A new `npm script` or `bash scripts/run-tests.sh` that aggregates everything
- Documented in `agentic-script-design` skill ref: "all critical-script tests live here, run via …"

## How to trigger

Whichever fires first:
- Phase A.5 milestone met (G1 stable through 5 changes)
- A real bug surfaces in any of the listed scripts (incident-driven)
- Post-deadline (after 2026-05-27)
- User explicitly says "let's harden the script coverage"

## Notes

Bash testing is harder than Python (no monkeypatch, subprocess mocking is awkward). `bats` is the standard; we'd adopt it.
