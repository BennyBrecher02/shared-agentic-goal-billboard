---
audit_id: A10
title: Scheduler deny-list-workaround audit (simplify candidates)
status: available
catalogued: 2026-05-26T11:30:00Z
priority_when_run: P2
estimated_effort: medium
trigger: After A9 (deny-list audit) identifies specific denies as friction-only OR after Phase A.5 stabilization (more scheduler usage = more workaround data) OR before any major scheduler refactor
deferral_reason: Scheduler just shipped Phase A.3; need A.4-A.5 evidence of which workarounds actually pinch in production usage vs which are theoretical
related_goals: [G1]
related_plans:
  - context/markdowns/plans/multi-change-scheduler-implementation-plan.md
related_refs:
  - .claude/skills/agentic-quality-discipline/references/git-hygiene.md
serves_northern_star: G2  # migrated 2026-05-26 - path 'scheduler' -> GL G1; NS defaults to G2
belongs_to_goal: G1
serves_guiding_light: G1
---
# A10 — Scheduler deny-list-workaround audit

## Why this audit matters

The scheduler was built honoring the `.claude/settings.json` deny list. In multiple places, the design chose a workaround over a direct operation because the direct operation would have been blocked at agent-Bash-tool level. Some workarounds are GOOD discipline (forced explicit ownership, registry-based safety). Others are extra code that exists ONLY because of the deny pressure.

If A9 surfaces specific deny entries as friction-only, A10 catalogs which workarounds those entries forced — so simplification follows naturally.

This is the "give back" audit. A9 identifies what could loosen. A10 identifies what could simplify.

## What it would look at

Walk `scripts/scheduler/` + `scripts/hooks/` and identify code that:

- Uses `subprocess.run([...])` to bypass Bash-tool denies (e.g., `git worktree remove --force` in `cleanup_expired_worktrees`)
- Implements complex retry/fallback because direct operation is denied (e.g., `mv in-flight/X scheduled/X` instead of `git restore X`)
- Adds defensive checks that duplicate deny-list protection (e.g., path-shape regex in cleanup_expired_worktrees IS justified for registry corruption, but might be over-engineered if the underlying deny were less brittle)
- Defers cleanup tasks to humans (`git branch -D change-*` denied → branches accumulate → user manually cleans via A6)

## Expected outputs

- `audits/findings/scheduler-workaround-audit-{date}-phase1.md` (Phase 1 catalog)
- Additional findings docs per phase (`-phase3.md` after applied simplifications)
- Per finding: file:line, what the deny is, what the workaround does, complexity cost, simplification recipe if deny loosens
- A prioritized list of "if A9 loosens deny X, simplification Y becomes available"

## SAFETY PROTOCOL (insane levels)

### Phase 1 — Read-only catalog
1. Walk scheduler + hook code
2. Identify deny-list-driven patterns
3. Categorize: NECESSARY-WORKAROUND (keep even if deny loosens) / FRICTION-WORKAROUND (simplify if deny loosens) / DEFENSIVE-LAYER (independent of deny — keep regardless)
4. **No code changes during audit phase. Findings doc only.**

### Phase 2 — Cross-reference with A9 findings
For each FRICTION-WORKAROUND, check if the corresponding deny was identified as friction-only in A9. If yes, candidate for simplification.

### Phase 3 — Simplify ONE workaround at a time
For each candidate:
1. Write the simplified version as a separate commit
2. Update or add unit tests for the simplified code path
3. Run full scheduler test suite (75+ tests must stay GREEN)
4. Run a real scheduler tick — verify the simplified code path actually fires
5. Commit ONLY after all of the above

### Non-negotiables
- **DEFENSIVE-LAYERS stay regardless of A9 outcome.** The path-shape regex in `cleanup_expired_worktrees`, the positive-ownership registry, the 4-layer worktree safety architecture — these are independent of agent permissions. They protect against registry corruption + bugs in the scheduler itself.
- **Tests must precede simplification.** Don't remove a workaround without a test that exercises the simplified path.
- **One-at-a-time.** Don't bundle multiple simplifications into one PR.

### Roll-back protocol
If a simplification surfaces a regression:
1. `git revert` the simplification commit
2. Update audit findings doc
3. Mark the underlying deny as "actually load-bearing" in A9's update pass

## How to trigger

Whichever fires first:
- A9 finishes Phase 1 (deny-list findings doc lands)
- Phase A.5 finishes (more usage data accumulates)
- Major scheduler refactor planned (clean before adding complexity)
- User says "let's simplify scheduler workarounds"

## What this does NOT do

- Doesn't remove any defensive layer (registry, regex, ownership checks)
- Doesn't loosen tests
- Doesn't bundle multiple changes
- Doesn't change anything until A9 has produced specific allowlist additions to react to

## Notes

A9 + A10 are paired but A9 leads. Running A10 first would produce a list of workarounds with no clear simplification path (since deny list hasn't moved). Run A9 first to set the targets, then A10 to simplify.

If A9 outcome is "all current denies are load-bearing as-is" → A10 produces no actionable findings (everything is NECESSARY-WORKAROUND). That's a valid result — confirms current design is right for current safety stance.
