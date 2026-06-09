---
title: "Local-BG coordination failure — the spawn-and-also-do-it-inline race (post-mortem + reaudit)"
date: 2026-06-02
type: findings
audit_kind: system-infrastructure (local-BG coordination / concurrency / visibility)
severity: HIGH (process + concurrency; benign OUTCOME this time, real failure class)
trigger: "user 2026-06-02 — two spawn_task LOCAL-BG chips ran cleanup the main session had ALSO done inline; one raced the git index with a main-session commit; the user: 'this violates everything we've been building and guarding... why spawn an agent and then do its work again??? did you even keep track of it?'"
---

# Local-BG coordination failure (2026-06-02 post-mortem)

## What happened
Two `spawn_task`-style LOCAL-BG chips (the `.pyc` cleanup + the `.scratch/*.new` cleanup) ran as separate
sessions. **The main session had ALREADY done both inline** (committed in `6065265`). The spawned sessions:
- redid/re-verified the same work (redundant),
- operated on the **shared main git index** (NOT isolated worktrees — `.claude/worktrees/` never existed),
  so the main session's `git commit 6065265` **absorbed a spawned session's staged `git add .gitignore`**,
- were **completely invisible** to the main session (no awareness, no tracking).

## What HELD (the honest other side)
- The **permission gate blocked** the spawned session's `git rm` (denied twice) → no bad mutation, no push.
- The race **resolved benignly** (the cleaner `.gitignore` won; no data loss).
- The spawned sessions themselves behaved **cautiously + correctly** (investigated, stopped at the denial,
  surfaced clearly — they were NOT brash; the brashness was the main session's: spawning/allowing
  redundant work and not tracking it).

## THREE failures — each cuts across what we built TODAY
1. **Redundant spawn (coordination).** Work done inline was ALSO queued as a chip. Discipline gap:
   `spawn_task` is for OUT-OF-SCOPE work the main session will NOT do; never flag a chip for work being
   done inline. (And the suggested-task surface needs dedup against in-flight/just-committed work.)
2. **Git index = unguarded shared mutable state (the concurrency audit MISSED it).** Today's
   concurrency-synchronization audit covered `.claude/cache`/memory/billboard/dashboard — **not the git
   index/working-tree.** Two sessions sharing one checkout race on `git add`/`commit`/`rm`. Isolated
   worktrees (per-worktree index) prevent this; these sessions were NOT isolated. **Open question: why
   did they run on the main checkout instead of a worktree?** (spawn_task is *supposed* to isolate — needs
   the spawn mechanism investigated.)
3. **Zero visibility — Consumer C didn't track them (the thing built TODAY to track exactly this).**
   `brain-streams/` never existed; the heartbeat wire is **uncommitted** (working-file only, applied this
   turn — AFTER the chips ran). Worktrees inherit the COMMITTED settings, which has 0 heartbeat hook. And
   detect-frozen/harvest **only scan `.claude/worktrees/`** — structurally blind to a local-BG on the main
   checkout. So three independent reasons the visibility layer saw nothing.

## The meta-lesson (banked)
We built Consumer C (visibility) + the concurrency audit (races) TODAY, and **both had gaps this incident
exposed**: Consumer C wasn't live (wire uncommitted), and the audit's surface list omitted the git index.
"Built" ≠ "verified in production." A new safety layer must be exercised against a real instance before
it's trusted.

## FIXES
| # | Fix | Who | Status |
|---|---|---|---|
| F1 | **Concurrency model: add the git index/working-tree as a shared-mutable-state surface** + the rule "never two sessions on one checkout; local-BG isolates via a worktree (own index) OR the main session does not commit while a BG holds staged changes." | claude | extending `concurrency-synchronization-audit.md` |
| F2 | **Discipline: don't `spawn_task` work you're doing inline** + suggested-task dedup vs in-flight/committed work. | claude | bank into `bg-lifecycle-discipline` |
| F3 | **Make local-BGs trackable:** commit the heartbeat wire so worktree BGs inherit + pulse the registry; widen detect-frozen/harvest to also notice a main-checkout spawned session (or MANDATE worktree isolation so the worktree-only scan is sufficient). | user (commit settings) + claude | the SessionStart paste-over already adds the sentinel; heartbeat-wire commit is gated |
| F4 | **Investigate WHY these ran on the main checkout** (spawn_task isolation expectation vs reality). | claude | open |

## Verify (the detectors' LOGIC is sound — the gap is wiring/isolation, not logic)
`scripts/test-frozen-spawned-sessions.sh` → 16/16 · `scripts/tests/test-brain-coordination.sh` → 16/16.
The detectors work; they were never given a worktree to look at, and the heartbeat wire wasn't live.
