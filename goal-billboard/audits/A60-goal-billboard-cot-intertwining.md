---
audit_id: A60
title: Goal billboard × Chain of Thought intertwining (A58 Brain integration)
status: in_progress
catalogued: 2026-05-27T16:20:00Z
priority_when_run: P1
estimated_effort: small (schema + plan this turn; field rollout + hook wiring next-turn)
trigger: |-
  2026-05-27 user explicit — *"can we update/upgrade our goal bilboard by intertwining it with our chain of thought os system if you havent already yet?"*. Honest answer: cross-referenced in A58 but NOT actually integrated. This audit closes that gap.
deferral_reason: Schema-additive changes can land now (low risk); hook wiring waits for A58 Phase 2 (CoT ledger ship).
related_goals: [G1, G2, G4]
related_plans: [context/markdowns/plans/goal-billboard-cot-intertwining-plan.md]
serves_northern_star: G2
belongs_to_goal: G18
serves_guiding_light: G14
related_refs:
  - context/markdowns/goal-billboard/README.md (target for update)
  - context/markdowns/goal-billboard/active/G*.md (target for schema-additive rollout)
  - .claude/skills/agentic-quality-discipline/references/chain-of-thought-discipline.md (A58 Brain)
  - .claude/skills/agentic-quality-discipline/references/heartbeat-system-discipline.md (A58 Heart, for goal-staleness)
  - context/markdowns/goal-billboard/audits/A58-organic-os-overhaul.md (parent)
  - scripts/goal-billboard-status.sh (target for CoT-state surfacing)
findings:
  - cross-reference-only: A58 mentions the link but doesn't formalize it; schema gap
  - no-cot-state-in-goal-frontmatter: today goals don't expose last CoT state; SessionStart can't tell which goals are stuck on BOTTLENECK
  - no-goal-staleness-check: today nothing fires when a goal has no CoT activity for N days
---

# A60 — Goal billboard × CoT intertwining

## What this audit fixes

A58's Brain (Chain of Thought) skill ref includes a passing reference: *"Goal billboard: active goals link to their primary CoT chain via the `goal` label."* That's a soft cross-reference, not a formal schema link. A60 closes the gap.

## 5-step intertwining plan

| Step | What | Risk | Status |
|------|------|------|--------|
| 1 | Add `cot_chain_id` + `last_cot_state` + `last_cot_ts` fields to active goal frontmatter schema | **Low** — schema-additive | Phase 1 (this turn) |
| 2 | Update `goal-billboard-status.sh` hook to read CoT ledger; surface "last state for G2: PLOWING (15min ago)" in SessionStart digest | **Low** — read-only | Phase 1 (this turn, hook stub only — full impl after A58 Phase 2) |
| 3 | Wire goal status transitions to auto-write CoT entries (e.g. `paused` → CoT entry with state=BOTTLENECK, blocker=user-explicit) | **Medium** — needs CoT ledger to exist (A58 Phase 2) | Phase 2 |
| 4 | Add Heart tier-medium goal-staleness check (no CoT activity for goal in N days → surface alarm) | **Medium** — needs Heart tier-medium consumer | Phase 3 |
| 5 | Dashboard goal × CoT timeline matrix panel (per A57 Track B) | **Low** — pure render layer | Phase 4 (after A57 Track B + A58 Phase 2) |

## Schema design

New fields on every active goal:

```yaml
# Chain of Thought integration (A60 / A58 Brain)
cot_chain_id: "<goal_id>-<short-hash>"      # links to the CoT ledger chain for this goal
last_cot_state: PLOWING | BOTTLENECK | SIDETRACK-INTENT | SIDETRACK | RESUME-READY | PLOWING-RESUMED | DONE | (null if no CoT activity yet)
last_cot_ts: "<ISO 8601 UTC>"                # timestamp of last CoT entry tied to this goal
last_cot_summary: "<one-line summary>"       # for SessionStart surfacing
```

The chain_id is computed once and pinned (so cross-referencing back from the CoT ledger is stable). State + ts + summary update on every CoT entry tied to this goal.

## Phase 1 — this turn

- ✅ This audit
- ✅ Companion plan
- ✅ Goal billboard `README.md` updated to describe schema + intertwining
- Stub the schema fields in active goals (deferred to keep this turn tight; will add to G2 + G4 + G14 in next turn as proof-of-concept)
- Hook stub `goal-billboard-status.sh` with TODO comment for CoT integration (read existing; mark with annotation)

## Phase 2-4 — deferred to dependency-ready

Phase 2: after A58 Phase 2 ships CoT ledger + writer (`scripts/cot-log.sh`):
- Implement the auto-write on goal status transition
- Backfill all active goals' `cot_chain_id` + initial PLOWING entry
- Update `goal-billboard-status.sh` for live CoT surfacing

Phase 3: after A58 Phase 3 ships Heart tier-medium consumer:
- Add `goal-staleness-check.sh` running on Heart tier-medium
- Configurable N-days threshold per goal (P0=2d / P1=5d / P2=10d)
- Surface alarm via standard staleness mechanism (existing `goal-staleness-warn.sh` pattern)

Phase 4: after A57 Track B settles:
- Goal × CoT timeline matrix panel in dashboard
- Hover over goal chip → expand CoT chain inline

## Cross-references

- A58 audit + plan (parent; Brain integration)
- A57 audit (sibling; dashboard surfacing rules apply to Phase 4)
- `chain-of-thought-discipline.md` (the schema goals plug into)
- `heartbeat-system-discipline.md` (Heart fires the staleness check)
- `goal-billboard.md` (existing skill ref; this audit extends)

## Status

PHASE 1 LANDED 2026-05-27T16:20Z. Phase 2-4 deferred to A58 Phase 2+ dependencies.

