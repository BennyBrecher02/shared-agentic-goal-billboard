---
audit_id: A14
title: "Stats system: correlate time-passed with token usage + activity"
status: in_progress
catalogued: 2026-05-26T20:00:00Z
priority_when_run: P1
estimated_effort: |-
  medium (4-6 hours: aggregator augment + 5hr-window tracker + hook upgrade + daily-reset + memory protocol)
trigger: User pushback 2026-05-26T15:55 local — "are you relating time passed to our token usage warnings? lets get this in better shape, stats audit initiated. also we should add daily global resets for consistency etc, maybe we should match claudes 5hr window even"
deferral_reason: NONE — running now
related_goals: []
related_plans: []
related_refs:
  - .claude/skills/agentic-quality-discipline/references/timestamp-convention.md
  - .claude/skills/agentic-quality-discipline/references/stat-aggregation-patterns.md
  - scripts/run-metrics-aggregator.py
  - scripts/hooks/token-budget-check.sh
findings:
  - context/markdowns/goal-billboard/audits/findings/A14-stats-2026-05-26-initial.md
  - context/markdowns/research/agent-os/stats-correlation-analysis.md  # analytical resolution — recommend status flip to `resolved` after user review
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'dashboard' -> serves NS
belongs_to_goal: G18
---
# A14 — Stats system: correlate time-passed with token usage + activity

## Why this audit matters

The current token-budget system tells us **HOW MUCH** we've spent without telling us **HOW FAST**. Without a time dimension:

- We can't tell whether a session at 45M tokens has been running 12h (sustainable burn) or 90min (about to hit ceiling)
- We can't tell how long until Claude's 5-hour rolling window resets
- We can't project whether the next 30min of work will cross the limit
- We can't compare today's burn rate against historical baselines for sanity-checking

The user surfaced this at 2026-05-26T15:55 local with reset at 17:00 local (~65min remaining), token tracker silent on both timing facts.

**Net:** the system measures one axis (tokens). To be useful for pacing decisions it needs three: tokens, wall-clock, and rate.

## Scope

In-scope:
1. Augment `run-metrics-aggregator.py` per-session output with session age + duration + tokens-per-minute
2. Build `scripts/claude-5hr-window.py` — user-input mechanism (`.claude/cache/5hr-window-reset.txt`) tracking the next reset time; `status` / `set` / `--watch` subcommands; projected at-reset usage
3. Extend `scripts/hooks/token-budget-check.sh` to include session age, time-to-reset countdown, projected usage
4. Daily global reset support — `.claude/cache/daily-token-snapshot.json` at midnight local; `scripts/daily-stats.sh` helper
5. Standing-protocols memory update — "token-or-budget question → run BOTH aggregator + 5hr-window status"

Out-of-scope (deferred):
- ~~7th dashboard tab (write data files; rendering deferred)~~ **DONE 2026-05-26 PM** — UI render layer shipped: `scripts/render-stats-data.py` emits the `stats` block into `reports/timelapse/<run>/data.json`; new `#page-stats` section in dashboard renders the burn-rate gauge / 30-min heatstrip / 24h histogram / recent-sessions table / 5hr-window card; auto-refresh wired (10s polling while tab visible); data-prime + billboard-refresh hooks now also spawn the stats renderer. Analysis (WHY tokens correlate with time) still pending — that's the audit's actual remaining work.
- Automatic 5hr-reset detection (Claude doesn't expose this API; user-provided is the only mechanism)
- Cross-project token rollup (single-project for now)

## Expected outputs

- A14 catalogued + initial findings doc
- `scripts/run-metrics-aggregator.py` extended (duration / rate / activity heatmap fields)
- `scripts/claude-5hr-window.py` (new)
- `scripts/hooks/token-budget-check.sh` extended
- `scripts/daily-stats.sh` (new) + `.claude/cache/daily-token-snapshot.json` schema
- Memory: `feedback_standing-protocols.md` + mirror updated with the new "always-run-both" protocol

## How to trigger

Already triggered by user pushback 2026-05-26 PM. If re-running later, the trigger is: "token usage feels surprising relative to wall-clock," "user asked 'how much budget left' or 'how long have we been at this,'" or "hit a session limit without seeing it coming."

## Resolution criteria

- Token-budget hook output shows time + tokens + rate together in one block
- `python3 scripts/claude-5hr-window.py status` returns time-to-reset + projected usage
- Standing-protocols memory has the always-run-both rule
- One real-data check confirms the math works on the live session

## Cross-references

- `feedback_timestamp-logging-discipline.md` — ISO 8601 UTC for cache files
- `feedback_standing-protocols.md` — the always-run-both rule lands here
- A1 (reactionary-automation-audit) — periodic recheck whether the stats system is still earning its keep

