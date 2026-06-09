---
audit_id: A37
title: Calculation audit — stats/metrics + scheduler/OS arithmetic verified end-to-end
status: in_progress
updated: 2026-05-27T03:23:00Z
catalogued: 2026-05-27T03:02:02Z
priority_when_run: P0
estimated_effort: large
trigger: 2026-05-27 — sibling of A36 (true-time audit). User explicit — *"also at the same time launch a calculation audit that should target not just our stats and metrics but also our operatingsystem stuff like scheduler and etc."* If timing claims can be hallucinated, what other calculations can't be trusted?
deferral_reason: NONE — running immediately, parallel with A36
related_goals: [G14, G9, G7]
related_plans: [context/markdowns/plans/automation/calculation-audit-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - .claude/skills/agentic-quality-discipline/references/stat-aggregation-patterns.md (existing)
  - scripts/scheduler/ (scheduler's resource math)
  - scripts/session-bundle/analyzer-*.py (new from A34 Phase 2 — already-computed analytics)
findings: []
---

# A37 — Calculation audit (stats + scheduler + OS arithmetic)

## The trigger event

Sibling audit to A36. User reasoning: if BG completion times can be wrong (A36 trigger), what other numbers in our system are wrong? Especially the ones we make decisions on.

## Scope

### Track A: Stats & metrics arithmetic
1. **Token aggregations** — sum of in + out + cache_create; cache_read amortization ratios; per-session totals; per-day totals. Are formulas correct? Spot-check against raw JSONL.
2. **BG dispatch metrics** — "45 ok · 0 failed · avg 0s" type rollups. What does "avg 0s" mean — is the average correctly computed or is it a placeholder?
3. **Hit-rate calculations** — cache hit rates, hook fire rates, test pass rates.
4. **Sparkline/histogram bins** — bucket boundaries; off-by-one in bin assignment.
5. **Heartbeat-tier-stats** — high/med/low frequency math; are tier transitions deterministic?
6. **A14 stats correlation** — Pearson coefficients; verify implementation.
7. **G11 analyzers** — kpi-deltas / token-efficiency / recurrence-detection / domino-effectiveness math (just landed today; high risk of subtle errors).

### Track B: Scheduler / OS arithmetic
1. **Resource availability calculations** — capacity = nominal − in_use; verify monotonic; verify deadlock-prevention ordering math.
2. **Wait-vs-acquire economics** — when does the scheduler choose to wait vs reroute? Is the cost function correct?
3. **Worktree count vs capacity** — A12 destructive-ops protections rely on counts; are they accurate?
4. **Tick boundaries** — does each tick start when the previous ended, or is there drift?
5. **Stale-lock detection** — PID lockfile age threshold math.
6. **Priority routing (G6 Phase 2)** — priority score formula; tie-breakers.
7. **5hr rolling window** — Claude's 5-hour reset window; right now the file is stale by 5h 23min per session start hook. Why? Audit the math.

### Track C: Hook chain rollups
1. **goal-billboard-status** — "1 inbox file" counts; verify against directory listing.
2. **recurrence-detect** — "3 A19 events" tallies; verify against raw `user-asks-pending.jsonl`.
3. **state-batch-digest** — per-substrate counts; aggregator correctness.
4. **token-budget-check** — 140% ceiling display math; effective tokens formula.
5. **subagent-data-refresh** — "45 ok / 0 failed" — agreement with raw dispatch log.

### Track D: Memory + steering log arithmetic claims
1. Counts in memory rules ("33 audits, 15 goals, 95+ skill refs, 33+ plans") — verify against filesystem `find | wc -l`.
2. "180 BGs in session" claims — verify against dispatch log row count.
3. Skill count claims — verify against `find .claude/skills/ -name "*.md" | wc -l`.

## Concrete deliverables (Phase 1 — BG this turn)

The BG will:
1. For each Track, sample 3-5 concrete calculation sites
2. For each site: extract the formula, run an independent recomputation, compare
3. Classify each: ✅ correct / ❌ wrong (with specific delta) / ⚠ ambiguous (formula unclear)
4. For each ❌: propose patch (file + line; correct formula)
5. Cross-verify the major today's claims (token totals, BG counts, audit counts) against filesystem ground truth
6. Produce `reports/audit-findings/A37-calculation-pass-1.md` with full delta table
7. Land in <30 min wall-clock; <500k tokens

## Phase 2 — Apply patches + recompute claims (later)

Apply patches. Re-emit corrected memory-rule counts. Update session-bundle aggregators.

## Cross-references

- A36 — true-time audit (sibling launched same turn)
- A14 — stats audit (predecessor; A37 extends with verification)
- G11 / A34 analyzers — direct subject of audit
- G7 — cluster infrastructure (resource math)
- G9 — testing discipline (test-rate math)

## Lessons (preliminary)

- The two audits A36 + A37 together form a **measurement-integrity layer**. Without it, all our optimizations and decisions are made on potentially-wrong data.
- The G11 analyzers just landed today (Phase 2) — auditing their math BEFORE they accumulate cross-session history is the right time (cheapest to fix at the source).
- This audit pairs naturally with future master-audit cadence: a calculation-pass should run on every recurring master audit (Phase 4 of A34).

## Status

**Phase 2 PATCHES APPLIED — 9 patches; 244 tests pass.** (P1-P4, P5, P6-P8, P10 landed; P9 deferred — requires SessionStart hook + settings.json change.)

- Phase 1: COMPLETE (29 calculation sites verified, 11 ❌ flagged, 10 patches proposed) — see `reports/audit-findings/A37-calculation-pass-1.md`.
- Phase 2: COMPLETE 2026-05-27T03:23Z — see `reports/audit-findings/A37-calculation-phase-2-apply.md`.
- G6 Phase 2 priority-routing gap filed as `audits/findings/A37-g6-phase2-priority-routing-gap.md` (separate task — scheduler-semantics change requires deliberation).
