---
audit_id: A3
title: Hook effectiveness audit
status: in_progress
catalogued: 2026-05-26T09:50:00Z
last_run: 2026-05-26T19:48:00Z
priority_when_run: P2
estimated_effort: small
trigger: After 50+ sessions OR every 2 weeks of active development OR when a SessionStart digest is so noisy it's getting ignored
deferral_reason: Need usage data — too few session-runs of the new hooks (git-drift-warn, scheduler-tick-drift-warn, goal-staleness-warn, goal-billboard-status) to know which earn their keep
related_goals: [G4]
related_plans:
  - context/markdowns/plans/automation/git-hygiene-hooks-plan.md
related_refs:
  - .claude/settings.json
serves_northern_star: G2  # migrated 2026-05-26 - path 'hook' -> GL G4; NS defaults to G2
belongs_to_goal: G4
serves_guiding_light: G4
findings:
  - context/markdowns/goal-billboard/audits/findings/A3-hook-effectiveness-2026-05-26.md
---
# A3 — Hook effectiveness audit

## Why this audit matters

We've accumulated ~15 hooks across SessionStart, PreToolUse, PostToolUse, Stop, SessionEnd. Some are clearly earning their keep (token-budget-check, bug-billboard-status). Others may be silent forever (snapshot-drift-pre never fires because we rarely regen snapshots manually). Hooks that never fire = noise on review without value; hooks that fire but produce no agent response = wasted infra. This audit ranks them by signal-to-noise.

## What it would look at

- For each hook: count of firings in last N sessions (instrument via wrapper that logs to `.claude/cache/hook-firings.log`)
- For each firing: did the agent's next action reference the output? (heuristic: the output mentions something + the agent's next response touches it)
- For each hook: estimated cost (time + tokens consumed by SessionStart bloat)
- Rank by (value · firings) / cost

## Expected outputs

- `notes/hook-effectiveness-{date}.md` with the ranking
- Recommendations: which hooks to remove / merge / convert to on-demand
- Settings.json delta if hooks should be re-shaped
- Possibly: new convention "hooks must be measured for 5 sessions before being permanent"

## How to trigger

Whichever fires first:
- 50+ sessions have run since 2026-05-26
- 2 weeks of active development
- SessionStart output exceeds a screen of text without action

## Notes

Instrumentation requires a wrapper around all hook invocations — could be added to settings.json `hooks` schema's `command` field by piping through a logger. Adopt during the audit, not pre-emptively.
