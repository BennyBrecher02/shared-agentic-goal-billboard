---
audit_id: A6
title: Scheduler worktree hygiene audit
status: available
catalogued: 2026-05-26T09:50:00Z
priority_when_run: P2
estimated_effort: small
trigger: When `git worktree list` shows >5 scheduler-created `EviumOverhaul-change-*` worktrees OR weekly OR before any major branch operation
deferral_reason: Currently 4 stale change-* worktrees (from Phase A.2 attempts 3-6) — operationally manageable. TTL cleanup logic exists in scheduler but only runs explicitly; user prefers manual --force removes per earlier feedback.
related_goals: [G1]
related_plans:
  - context/markdowns/plans/multi-change-scheduler-implementation-plan.md
related_refs:
  - .claude/skills/agentic-quality-discipline/SKILL.md
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'scheduler' -> GL G1; NS defaults to G2
belongs_to_goal: G1
serves_guiding_light: G1
---
# A6 — Scheduler worktree hygiene audit

## Why this audit matters

The scheduler creates a new worktree per `git worktree add` on each tick. If the change verifies, the worktree could be reaped after merge. If it fails partway, it lingers. After enough ticks, the filesystem fills with `EviumOverhaul-change-*` worktrees + change-* branches.

Layer 1 (positive-ownership registry) prevents the scheduler from touching unowned worktrees, but it doesn't proactively clean up its own retired worktrees unless `cleanup_expired_worktrees` is called. The TTL is 24h; manual `--force` removal is currently denied to the agent.

## What it would look at

- `git worktree list` → all worktrees, age, branch
- `.claude/cache/scheduler-owned-worktrees.json` registry → owned items + TTL status
- `git branch | grep "^  change-"` → orphan branches (no worktree)
- Disk space used by scheduler-owned worktrees
- Identification: which to keep (recent verified), which to reap (failed/expired)

## Expected outputs

- A list of safe-to-remove worktrees (matching `EviumOverhaul-change-*` pattern, TTL expired or status=verified+merged)
- A list of safe-to-remove orphan branches
- Manual commands the user can run (since `--force` is agent-denied)
- Or: a one-time cleanup batch the user approves

## How to trigger

Whichever fires first:
- `git worktree list` shows >5 scheduler-created worktrees
- Disk-headroom advisory pressure (DiskHeadroom shows < 10 GB free)
- User says "clean up the worktrees"
- Weekly (if we adopt a weekly rhythm)

## Notes

Currently we have 4 stale ones (from Phase A.2 attempts 3, 5, 6, 7). One was the verified one which IS the desired artifact. They're operationally fine right now.

Long-term: the scheduler's existing `cleanup_expired_worktrees` function should be wired to fire automatically post-tick if TTL has elapsed. But that needs the `--force` removal to be allowed for the scheduler specifically (path-restricted permission).

## Relationship to A9 + A10

A6 is the **reactive cleanup audit** — fires when stale state has accumulated. A9 is the **structural-decision audit** about whether scheduler should be allowed to clean its own worktrees. A10 is the **code-simplification audit** that wires the cleanup in if A9 approves.

**Sequence:**
1. A9 → user approves narrow allows for `git branch -D change-*` + `git worktree remove --force ../EviumOverhaul-change-*`
2. A10 → wires immediate post-verify cleanup into scheduler.tick()
3. A6 → **becomes mostly unnecessary** because cleanup is now automatic

If A9 stays at the current deny state → A6 remains a recurring chore (user-triggered manual cleanup).

A6 findings doc (if/when run) lands at `audits/findings/worktree-hygiene-audit-<date>-phase1.md`.
