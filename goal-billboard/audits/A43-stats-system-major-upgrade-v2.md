---
audit_id: A43
title: Stats system major upgrade #2 — resource utilization + cross-system correlation + predictive + per-category dashboards + anomaly detection
status: in_progress
partial_state: "2026-07-08 (pin P-007): PARTIAL. SHIPPED — M2 correlation (dashboard/correlate_stats.py), M4 per-category dashboards (page-stats/scheduler/subagents + siblings), M8 rolling KPIs (longitudinal-metrics.py) + the run-metrics-aggregator pipeline. NOT BUILT — M1 per-process resource-util sampler, M3 predictive/forecast, M5 anomaly detection, M6 action-surface rule engine, M7 stats query CLI. NOT superseded (Mission Control is a parallelism cockpit, not this). Next: re-scope to the 5 unbuilt modules under NS G21 (M7 small / M1,M5 medium, Fable-era-cheap), or fold M1 into Mission Control CEILINGS. Triage: context/markdowns/audits/stale-audits-triage-2026-07-08.md."
catalogued: 2026-05-27T03:53:00Z
priority_when_run: P0
estimated_effort: large
trigger: |-
  2026-05-27T03:53Z — user explicit: *"this should be factored into our overall stats system which should get a major upgrade again."* Companion of A42 (the resource-saver that emits new metrics). User said "AGAIN" implying A14 was upgrade #1; A43 is #2.
deferral_reason: NONE — paired with A42; tightly scoped major upgrade
related_goals: [G14, G9, G11]
related_plans: [context/markdowns/plans/automation/stats-system-major-upgrade-v2-plan.md]
serves_northern_star: G2
belongs_to_goal: G18
serves_guiding_light: G14
related_refs:
  - A14 — stats audit (upgrade #1 — time + token + 5hr window foundations)
  - A37 — calculation audit (verified math substrate)
  - A42 — sibling (emits new emulator-resource metrics)
  - G11 / A34 analyzers (session bundle stats infrastructure)
  - scripts/run-metrics-aggregator.py + scripts/hooks/token-budget-check.sh (existing stats pipes)
findings: []
---

# A43 — Stats system major upgrade #2

## The user's framing

A14 was upgrade #1 (time + token + 5hr-window foundations + 7-tab UI). A43 is upgrade #2 — the next major leap. Triggered by A42 needing new resource-utilization metrics, but the scope is broader.

## What's missing from today's stats system (the gap audit)

### Gap 1: Resource utilization (per-process category)
- We track tokens but not RAM/CPU/disk
- Emulators (A42), BGs, scheduler workers, hooks — each has its own resource profile
- No cross-category view: "where is the system spending its compute?"

### Gap 2: Cross-system correlation (extends A14 correlation panel)
- Current: token-vs-time, token-vs-msg-count
- Missing: emulator-uptime vs BG-throughput, scheduler-queue-depth vs token-rate, hook-latency vs session-length
- A14 had Pearson; A43 needs multivariate

### Gap 3: Predictive stats
- Current: descriptive (what happened)
- Missing: forecast (what's coming)
- "Based on current trend, you'll hit weekly token limit in 4.2 days"
- "Based on recent matrix-run pattern, iOS Sim will be needed in next 35 min — don't kill yet"

### Gap 4: Per-category dashboards
- Current: one billboard with overlapping concerns
- Missing: dedicated views (Emulator, BG, Scheduler, Hook, Memory)
- Drill-down from billboard to category

### Gap 5: Anomaly detection (auto-flag)
- Current: user has to look at numbers to notice drift
- Missing: structural alerts when metric deviates >N stdev from baseline
- "Hook X took 5x its p95 budget — investigate"

### Gap 6: Stats-driven decisions (auto-flags surface)
- Current: stats are descriptive
- Missing: actionable surface
- "Scheduler queue depth is 8 — consider reducing parallelism"
- "Cache hit rate dropped 20% — last 3 sessions trending down; check breakpoint placement"

### Gap 7: Stats query CLI
- Current: read dashboards manually
- Missing: `stats query "emulator iOS uptime last 24h"` → answer
- DSL or natural-language hybrid

### Gap 8: Multi-session rolling KPIs
- Current: per-session snapshots
- Missing: 7-day rolling, 30-day trend
- Per-KPI sparkline showing direction

## The 8 modules (one per gap)

| Module | Audit ID slot | Scope | Effort |
|---|---|---|---|
| M1 Resource utilization | A43.1 | per-process RAM/CPU/disk via `ps` + `du` + sampling | medium |
| M2 Cross-system correlation | A43.2 | extend A14 multivariate; new panel | medium |
| M3 Predictive stats | A43.3 | linear extrapolation + rolling regression | large |
| M4 Per-category dashboards | A43.4 | new UI panels (Emulator, BG, Scheduler, Hook, Memory) | large |
| M5 Anomaly detection | A43.5 | baseline + stdev + alert hook | medium |
| M6 Stats-driven action surface | A43.6 | rule engine surfacing recommendations | medium |
| M7 Stats query CLI | A43.7 | `stats` CLI + DSL | small |
| M8 Multi-session rolling KPIs | A43.8 | 7-day + 30-day rolling per-KPI | medium |

## Module dependencies

```
M1 (resource util) ← independent
M2 (correlation) ← depends on M1 data + A14
M3 (predictive) ← depends on M1 + M8 (need history)
M4 (dashboards) ← depends on M1+M2+M3
M5 (anomaly) ← depends on M1 + M8 (need baseline)
M6 (action surface) ← depends on M5 + M2
M7 (query CLI) ← can be done in parallel; useful early
M8 (rolling KPIs) ← independent; foundational
```

Phase ordering:
- Phase 1 (this turn or next session BG): M1 + M8 (foundational + independent)
- Phase 2: M7 (query CLI — early usefulness)
- Phase 3: M2 + M5 (depend on M1 + M8)
- Phase 4: M3 + M4 (depend on Phase 1-3)
- Phase 5: M6 (depends on everything else)

## The A42 integration (sibling-bridge)

A42 emits 7 emulator metrics. A43-M1 (resource utilization) hosts them. A43-M2 (correlation) tests for: when emulator_uptime ↑, does BG_throughput ↓ (resource contention)? When emulator_miscall ↑, does session_token_burn ↑ (re-spawn overhead)? Etc.

A43-M5 (anomaly) auto-flags: "emulator_kills_count = 0 for 6 sessions in a row — is killer disabled / stuck?"

A43-M6 (action surface) emits: "iOS sim threshold has held at 45 min for 3 weeks with miscall rate 0 — consider tightening to 30 min for further savings."

## The token-budget connection (the user's binding constraint)

A43-M3 (predictive) directly addresses the user's weekly-limit concern (from RH-011 launch):

```
Predicted weekly token usage at current rate: 290M ± 15M
Weekly ceiling: <N> (per Anthropic Pro plan)
Days until projected exhaust: 4.2
Recommendations:
  - Defer 3 BGs to next week (saves ~1.2M)
  - Adopt RH-011's cache-warmup pattern (saves ~30% on prefix re-reads)
```

This makes RH-011's adoption candidates CONCRETELY measurable.

## Cost gates

Per-module phase BG: <400k each. Total <3M across all 8 modules over multiple sessions. Per A14 / G9 amortized via cache reuse.

## Cross-references

- A14 — stats upgrade #1 (foundation; A43 extends)
- A42 — sibling (emits resource metrics A43 hosts)
- A37 — calculation audit (verified math A43 builds on)
- A36 — true-time (timestamps in all new stats)
- G11 / A34 — bundle analyzers (cross-session stats feed A43-M8 rolling KPIs)
- RH-011 — token cheap-out (A43-M3 makes RH-011's savings measurable)
- G9 — testing toolbelt (anti-bottleneck depends on accurate stats)

## Status

IN PROGRESS — design landed this turn; Phase 1 module BGs queue for capacity.

## Lessons (preliminary)

- "Major upgrade" implies 8 modules — not one. A14 was 5 modules. Cumulative stat system depth doubles.
- The A42-A43 pairing is a TEMPLATE for future feature-pairs: build the metric source AND the surfacing in lockstep.
- M3 predictive stats are the FIRST forecasting capability we'd have. Today we're reactive; M3 makes us proactive on cost.
- M5 anomaly detection completes the feedback loop user has wanted since A18-A29: today metric drift is invisible; M5 makes it surface.

