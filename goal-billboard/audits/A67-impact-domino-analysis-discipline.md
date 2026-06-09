---
audit_id: A67
title: Impact-domino analysis + leverage-ranked goal-billboard intake discipline
status: in_progress
catalogued: 2026-05-27T17:48:00Z
phase_1_landed_at: 2026-05-27T17:48:00Z
priority_when_run: P0
estimated_effort: medium (BG one-shot analysis this turn; discipline + daemon design foreground; daemon implementation deferred)
trigger: 2026-05-27 user verbatim — *"run another bg task that goes through all my plans/ideas from the last day and lines up all their possible dominoes for each individually and then make some stack so the highest win domino slam dunk effecting stuff gets put on our goal billboard and is actually given the attention it requires. this shouldnt be exclusive to old plans/ideas/comments/requests but rather a thing we keep and apply to all new incoming stuff too."*
deferral_reason: NONE — BG dispatched + framework lands this turn; daemon implementation deferred to A65 Phase 2 (after Daemon #1 ships).
related_goals: [G2, G4]
related_plans: []
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - reports/audit-findings/A67-domino-stack-2026-05-27.md (BG one-shot output; in flight)
  - .claude/skills/agentic-quality-discipline/references/impact-domino-analysis-discipline.md (NEW)
  - .claude/memory-mirror/feedback_impact-domino-analysis.md (NEW; both auto-load + mirror)
  - A65 audit (autonomic daemon framework — A67 becomes Daemon #9 candidate OR extends Daemon #5/#3)
  - A63 audit (idle-capacity — related; A67 ranks WHAT to research; A63 ranks WHEN to dispatch)
  - feedback_goal-billboard memory (never add goals unilaterally — A67 PROPOSES not APPLIES)
  - feedback_idea-handoff-slam-meter (A39 — A67 is the systematic version of slam-meter)
findings:
  - intake-triage-was-missing: between idea-arrival and goal-billboard, there was no leverage-ranking step. High-leverage items competed for attention with low-leverage items.
  - slam-meter-was-per-instance: A39 slam-meter measures per-handoff; A67 is the persistent rank-everything-against-everything version.
  - daemon-shaped: per user "thing we keep and apply to all new incoming stuff" — daemon, not discipline.
---

# A67 — Impact-domino analysis discipline

## The principle

**Every plan / idea / comment / request gets domino-mapped + leverage-scored. High-leverage items rise to goal-billboard with attention; low-effort wins get fast-tracked; low-leverage items get triaged into "later" or "no."**

This is a TRIAGE LAYER above the goal billboard — between intake and prioritization.

## The 5-step analysis loop (applies to every new item)

1. **CAPTURE** — log the item in the appropriate substrate (steering log / asks-log / chamber / bug billboard / etc.)
2. **DOMINO-MAP** — identify 1st / 2nd / 3rd-order downstream effects (what this unblocks / enables / requires next)
3. **SCORE** — leverage = (impact × downstream-unblock-count) / (effort + risk); scale 0-10
4. **STACK** — sort against existing items; identify slam-dunks (top-5) + low-effort wins (top-3)
5. **PROPOSE** — for slam-dunks: propose goal-billboard goal OR Phase-X attachment; for wins: propose fast-track; for low-leverage: triage to "later" or "no" (user confirms; never unilateral)

## Leverage formula

```
leverage = (impact_weight × downstream_unblock_count) / (effort_weight + risk_weight)
```

Where:
- `impact_weight`: closes-class=4 / unlocks-feature=3 / nice-to-have=2 / cosmetic=1
- `downstream_unblock_count`: number of OTHER items this directly unblocks
- `effort_weight`: S=1 / M=2 / L=4 / XL=8
- `risk_weight`: low=1 / medium=2 / high=4

Slam-dunk threshold: leverage ≥ 5.0 + downstream_unblock_count ≥ 3.
Low-effort win threshold: effort=S + impact ≥ 3 (unlocks-feature or higher).

## Where this fires

- **One-shot** (this turn): A67 BG runs against last-24h surface
- **Persistent** (Phase 2 deferred): Daemon #9 (NEW in A65 catalog OR extends #3 audit-catalog maintainer / #5 bug-billboard groomer) runs on Heart tier-medium; reads new items from steering log + asks log + chamber decisions + recent inbox-archive; produces ranked stack
- **Manual trigger**: user-invoked when they want a re-rank

## Integration with existing disciplines

- **A39 slam-meter** — per-instance qualitative; A67 is the quantitative persistent version
- **Goal billboard** — A67 PROPOSES additions; user confirms (per `feedback_goal-billboard`)
- **Chamber posts** — A67 may propose chamber posts for "deserves explicit user decision" items
- **A63 idle-capacity** — A67 ranks WHAT to research; A63 ranks WHEN to dispatch
- **A66 decision-handling** — A67 outputs are proposals, never auto-applied (high-risk to bypass user)
- **A65 autonomic** — A67 becomes Daemon #9 OR extends existing daemon

## Anti-patterns

| Anti-pattern | Why bad | Fix |
|--------------|---------|-----|
| Auto-add to goal billboard | Violates `feedback_goal-billboard` rule | Always propose; user confirms |
| Rank items in isolation | Misses comparative leverage | Always stack against existing |
| Score without domino map | Misses downstream unblock count | Map first; score second |
| Treat as one-shot | User explicitly named "thing we keep" | Daemon-shape it |
| Skip the calibration tag | Loses discipline-vs-instance learning | Tag each item with which discipline catches it |

## What this turn delivers

- ✅ A67 audit (this file) — framework definition
- ✅ Skill ref `impact-domino-analysis-discipline.md`
- ✅ Memory rule `feedback_impact-domino-analysis.md` (both paths)
- ✅ Audit catalog + MEMORY.md updates
- 🟡 A67 BG running — one-shot last-24h analysis + ranked stack + goal-billboard proposals
- ⏸ Daemon #9 implementation deferred (Phase 2 after Daemon #1 ships per A65 sequencing)

## Phase 2-N — deferred

- Daemon #9 design (or extension of #3/#5) per A65 framework
- 24h dry-run + VERIFY per A47 Rule 5
- Goal-billboard intake-form schema (if user wants chamber-post-style proposal flow)
- Auto-tag last-24h scope so the daemon knows where to look

## Cross-references

- `impact-domino-analysis-discipline.md` skill ref (the full discipline)
- `feedback_impact-domino-analysis.md` memory rule
- A39 slam-meter (per-instance qualitative version)
- A65 autonomic daemon framework (potential Daemon #9 home)
- A63 idle-capacity (sibling; A67 + A63 pair as WHAT + WHEN)
- A66 decision-handling (A67 outputs are proposals, never auto-applied)
- `feedback_goal-billboard.md` (intake convention preserved)
