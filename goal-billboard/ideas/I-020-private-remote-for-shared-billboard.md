---
idea_id: I-020
title: Create a private remote for ~/.claude/shared-billboard (push = backup + cross-node foundation) — timing NOW
lifecycle: captured        # captured = awaiting your review (the user-gate per the ideas-lane safety contract)
serves_northern_star: G21  # the portable agentic-OS — a backed-up, pushable shared store is part of "portable + cross-node"
belongs_to_goal: G21       # repo-creation todo #1 of 3, tracked under the Northern Star
user_gated: true           # USER-ONLY: creating a remote + the first push are gated (push is a user-owned op)
timing: NOW
source: user-prompt        # the user-approved goal-grooming triage (2026-06-21) appended these 3 repo-creation todos
source_ref: context/markdowns/audits/goal-grooming-proposal-2026-06-21.md
created: 2026-06-21T21:01Z
updated: 2026-06-21T21:01Z
---

# I-020 — Private remote for the shared-billboard store (timing: NOW)

**What it is.** `~/.claude/shared-billboard/` is **already a local git repo** (the goal-billboard shared store
that this billboard now lives in). It has **no remote** — so there is no off-machine backup and no foundation
for cross-node sync. This todo: create a **private** remote (e.g. a private GitHub repo) and push, giving the
shared store (a) a real backup and (b) the foundation for the cross-node / multi-machine future (G17/G21).

**Why NOW.** It is the lowest-effort, highest-safety item of the three — the repo already exists locally, so
this is `git remote add` + an initial push, nothing to extract or restructure. Until it has a remote, the
entire goal-billboard + ideas + pins + audits live on exactly one disk.

**Why USER-GATED.** Creating the remote and running the **first push** are user-owned operations (push is on
the gated list; the user owns history + remotes). Claude prepares; the user creates the private repo and
pushes. Privacy matters — this store carries internal project state, so the remote MUST be private.

**Concrete first step (for the user).** Create a private remote, then from `~/.claude/shared-billboard/`:
`git remote add origin <private-url>` and `git push -u origin <branch>`. (Claude can stage/commit locally on
request, but will not create the remote or push.)

**Timing: NOW.** Already a local repo; this is pure upside (backup) with near-zero effort.
