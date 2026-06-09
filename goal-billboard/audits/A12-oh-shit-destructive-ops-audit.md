---
audit_id: A12
title: "Oh-shit audit: destructive operations on uncommitted/staged work"
status: in_progress
catalogued: 2026-05-26T15:55:00Z
priority_when_run: P0
estimated_effort: small (1-2 hours including hook design)
trigger: Triggered NOW by a near-miss 2026-05-26T15:44 — agent ran `git checkout HEAD -- src/styles/app.css` while P0-03 was staged but not committed; staged changes were wiped (recovered from /tmp/g2-contact-form.patch only because the saved patch file existed)
deferral_reason: NONE — run immediately
related_goals: [G1, G2]
related_plans: []
related_refs:
  - .claude/skills/agentic-quality-discipline/references/git-hygiene.md
  - .claude/settings.json
findings: context/markdowns/goal-billboard/audits/findings/A12-oh-shit-2026-05-26-initial.md
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'scheduler' -> GL G1; NS defaults to G2
serves_guiding_light: G1
---
# A12 — "Oh-shit" destructive-operations audit

## Why this audit matters

The agent has a deny list (`.claude/settings.json`) that blocks the most obvious destructive operations: `rm -rf`, `git reset --hard`, `git clean`, `git stash drop`. **It does NOT cover operations that are normally benign but become destructive in specific states** — like `git checkout HEAD -- <file>` on a file with staged-but-not-committed changes.

On 2026-05-26T15:44Z, the agent ran exactly that to revert a P1-08 working-tree edit, **not realizing the same file had staged P0-03 changes from earlier today**. Both reverted. The staged P0-03 verified fix was wiped from working tree. Recovery succeeded only because a separate `/tmp/g2-contact-form.patch` file existed — but that's luck, not architecture.

**This is one example of a class.** The audit identifies the full class + lands countermeasures.

## Scope

In-scope:
1. Destructive `git` commands that LOOK benign but corrupt state in specific contexts
2. File-overwrite operations (Write tool, `cp -f`, `mv`, redirect `>`) that clobber uncommitted work
3. Branch/worktree operations that delete artifacts holding the only reference to verified state
4. Stash/restore round-trips that drop the stash silently
5. The agent's own assumption that "revert is safe" — when the assumption breaks

Out-of-scope:
- Hardware/permission destruction (covered by A9)
- Remote git operations (push --force etc.; covered by user's panic-mode "I push myself" rule)
- Subagent-side destruction (subagent inherits deny rules; covered by parent settings)

## Initial findings (2026-05-26 PM — immediate pass)

See [findings/A12-oh-shit-2026-05-26-initial.md](findings/A12-oh-shit-2026-05-26-initial.md). Summary:

**Class 1 — git checkout HEAD --: wipes staged changes silently.**
- Today's near-miss. The most insidious because `git checkout HEAD --` is normally the right revert command. It becomes destructive only when the file has staged content.
- Countermeasure: PreToolUse hook that intercepts `git checkout HEAD -- <path>` (and `git restore --staged --worktree <path>`) and refuses if `<path>` has staged content (`git diff --cached --name-only` includes it). Workaround for legitimate use: stash first OR set `EVIUM_CHECKOUT_OK=1` env var.

**Class 2 — git stash without push -m and ID tracking.**
- Stashes are easy to lose track of. `git stash pop` on the wrong stash overwrites work silently.
- Countermeasure: standing protocol — always `git stash push -m "<descriptive>"` (named); never `git stash` alone.

**Class 3 — Write/Edit clobbering uncommitted edits made outside the agent.**
- If the user manually edits a file while the agent is running, then the agent Writes that file, the user's edits are lost.
- Countermeasure: Write tool already requires Read first, which catches some cases. The risk remains for fast-edit-by-user scenarios.

**Class 4 — Shell redirects (`>`) that overwrite existing files without warning.**
- Bash deny list does NOT cover `>` redirects to existing files.
- Countermeasure: shell wrapper that warns on `>` to existing tracked files. Lower priority — Write/Edit covers most cases.

**Class 5 — Stuck artifacts the deny prevents cleaning up.**
- Today's BG agent left a stuck worktree because the deny refused both the path-shape match AND the absolute-path form. We can't even clean up our own mess.
- Countermeasure: an explicit "agent-initiated cleanup" mode (`EVIUM_CLEANUP_OK=1` env) that the agent can set only when it's cleaning its own owned artifact (verified via registry).

## Recommended immediate actions

1. **Add `git checkout HEAD --` guard** (new hook `scripts/hooks/git-checkout-safety.sh`) — see findings doc for the script design. Blocks the exact scenario that just happened.
2. **Memory protocol:** never `git checkout HEAD -- <file>` without first `git diff --cached --name-only` and confirming the file is NOT staged. Stash explicitly if it is.
3. **Add `git stash push -m` discipline** to standing-protocols memory.
4. **Cleanup-mode env** for the agent's own owned worktrees. Carefully scoped.
5. **Re-audit when a new "oh shit" moment happens** — append to this audit's findings; don't open a parallel audit.

## Open questions for user

- Whether to add the new hook to settings.json now (requires user approval per agent's no-touch-settings rule)
- Whether the cleanup-mode env should be approved as a standard scheduler subprocess pattern (low risk; high friction otherwise)

## When this audit unsets

- When all 5 classes have countermeasures in place AND a steering-log review shows no oh-shit incidents in the prior 7 days (or whatever the next stability window is).
- Otherwise re-audit on each incident.
