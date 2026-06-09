---
id: A85
title: Git rollback safety / etiquette audit (Cursor dual-author era)
type: audit
status: completed
mode: read-only
created: 2026-05-29
audited_by: subagent (BG dispatch, read-only)
scope: rollback safety of actual history + the guards + etiquette-doc accuracy + Cursor dual-author risk
cross_refs:
  - .claude/skills/agentic-quality-discipline/references/rollback-discipline.md
  - .claude/skills/agentic-quality-discipline/references/git-hygiene.md
  - .claude/memory-mirror/feedback_git-hygiene-discipline.md
  - .claude/settings.json (permissions.deny)
  - scripts/hooks/git-checkout-safety.sh
  - scripts/hooks/worktree-remove-safety.sh
  - scripts/hooks/settings-edit-guard.sh
  - scripts/hooks/src-edit-guard.sh
  - scripts/hooks/git-drift-warn.sh
  - scripts/scheduler/worktree.py
---

# A85 — Git rollback safety / etiquette audit

**Premise (user, 2026-05-29):** audit our rollback safety + etiquette, especially as the user transitions to fine-grained hand-editing in **Cursor** — a dual-author era (Claude + user-in-Cursor).

**Bottom line:** rollback safety of the *actual history* is **excellent** — no force-push, no rewrites, clean linear ancestry, genesis recoverable, 90-day reflog. The *agent-side guards* are strong and layered. But there are **5 real findings**, two of them material for the Cursor transition:

- **F1 (HIGH, doc-correctness):** `rollback-discipline.md` recommends `git revert` as the #1 strategy, but `git revert*` is in the **deny-list** — the agent cannot run it. The doc never says so. The recommended path and the enforced reality contradict.
- **F2 (HIGH, Cursor):** **No GitHub branch protection** on `main` + **Cursor does not read the Claude deny-list**. Every "structural" protection in `rollback-discipline.md` is harness-local. In Cursor's terminal the user has an unguarded gun (force-push, reset --hard, clean -fdx all work).
- **F3 (MEDIUM, Cursor):** **Uncommitted-work clobber** is the #1 dual-author hazard. Working tree is currently dirty (19 files, incl. `public/dashboards.html` +195, `generate_dashboard.py` +540). If Claude commits/stashes while the user has unsaved Cursor edits to the same files (or vice-versa), work is lost. No lock, no coordination protocol exists today.
- **F4 (MEDIUM, guard-gap):** The scheduler removes worktrees via Python `subprocess` (`git worktree remove --force`), which **bypasses both the Bash deny-list and `worktree-remove-safety.sh`** (hooks fire only on the Bash *tool*). Positive-ownership registry + path regex are the only backstops there. Adequate today, but the "guard" is not where you'd think.
- **F5 (LOW, tracked-config):** `.claude/settings.json` is git-**tracked**. `settings-edit-guard.sh` protects only the *Claude Edit/Write tool* path. A user editing settings.json in Cursor and committing it has no guard — and that commit can silently widen the agent's own permissions on the next session.

---

## (1) ROLLBACK SAFETY OF THE ACTUAL HISTORY — VERDICT: CLEAN ✅

### Evidence

| Check | Command | Result |
|---|---|---|
| Force-push tokens | `git reflog --all \| grep -i forced-update` | **none** — "NO 'forced-update' tokens anywhere" |
| Destructive action-tokens | reflog action-token histogram | only `branch: Created` (23), `commit:` (n), `update by push` (8), `fetch: storing` (7), `Branch: renamed` (11). **Zero `reset`, `rebase`, `filter`, `forced-update`.** |
| origin/main ⊆ HEAD | `git rev-list HEAD \| grep origin/main-sha` | **CONFIRMED** origin/main (`621ba18`) is an ancestor of HEAD |
| Ahead/behind | `git rev-list --left-right --count origin/main...HEAD` | **`0  82`** — 0 behind, 82 ahead → strict fast-forward, no divergence |
| Genesis present | `git cat-file -t b77620d` | `commit` — `b77620d "first commit"` 2026-05-21, author BennyBrecher02 |
| Genesis reachable | `git rev-list HEAD \| grep b77620d-sha` | **YES** reachable from HEAD |
| Genesis tree recoverable | `git ls-tree --name-only b77620d` | full tree intact: `.claude .gitignore astro.config.mjs package*.json public src tsconfig.json` |
| Object integrity | `git fsck --full` | clean — only dangling **blobs/trees** + **2 dangling commits** (both recoverable, see below) |
| Reflog retention | `git config gc.reflogExpire*` | defaults: 90 days (reachable) / 30 days (unreachable); `main` reflog depth = **152 entries** |

### The 2 dangling commits are recoverable stash snapshots, not lost work
- `3912ad2` — "On main: scheduler-test-stash-pre-apply-to-main" (2026-05-26 11:22)
- `a9b5835` — "On main: p1-08-recapture" (2026-05-26 11:51)
- Plus live `stash@{0}` = "On main: g2-patch-test-tmp"

All three are recoverable via `git stash list` / `git show <sha>`. Dangling ≠ lost; they linger until a manual `gc --prune` (which is itself denied — see guards).

### Two non-`/private/tmp` `update by push` observation
The reflog shows 8 `update by push` rows — **all on `refs/remotes/origin/main`**, all monotonically forward (`621ba18` is the latest remote tip; HEAD is 82 ahead). This is normal push/fetch tracking, **not** history rewriting. The earlier rows (`02359cd`, `b77620d`) are the genesis-era pushes.

### Safe rollback path PER CHANGE-CLASS (corrected for this repo's actual deny-list)

| Change class | Recommended path | Works in THIS harness? |
|---|---|---|
| **Pushed commit to undo** | `git revert <sha>` → new inverse commit | ⚠ **NO — `git revert*` is DENIED.** See F1. Workaround: user runs revert (Cursor/manual), or temporarily lift the deny. |
| **Single file to a past state** | `git checkout <sha> -- path` then commit | ⚠ `git checkout --*` is denied (the `--` form). `git show <sha>:path > file` works and is *not* denied — use that. |
| **Exploration of an old state** | `git checkout <sha>` (detached, read-only browse) | ✅ Bare-SHA checkout is **not** matched by `git checkout --*`; `git-checkout-safety.sh` only blocks checkout/restore that target *staged files*. So detached browse is allowed. Return via `git checkout main` (allowed). |
| **Local-only mistake (unpushed)** | `git reset --soft`/`--mixed` | `git reset --hard*` is denied; soft/mixed are **not** denied. Prefer soft/mixed (they never touch the working tree). |
| **NEVER on pushed work** | `git reset --hard` + `git push --force`, `git clean -fdx`, `git filter-branch`, `gc --prune=now` | ✅ All denied in this harness. Correct posture. |

**Read-only, NEVER-RUN-ON-PUSHED-WORK list confirmed denied:** `git reset --hard*`, `git push*` (so `--force` can't even reach), `git clean*`, `git worktree prune*`, `git stash drop/clear*`, `git tag -d*`, `git update-ref -d*`, `git rebase*`, `git filter-*` (via `*--force*` for filter-repo; filter-branch is history-rewriting and there's no non-force path to remote). Good.

---

## (2) THE GUARDS — enumerated, wired?/sufficient?/gaps

### 2a. Bash deny-list (`.claude/settings.json` → `permissions.deny`)

| Dangerous op | Denied? | Pattern | Note |
|---|---|---|---|
| force-push | ✅ | `git push*` (blanket) + `*--force*` | push is fully blocked, so `--force-with-lease` can't reach either |
| `--no-verify` (hook bypass) | ✅ | `*--no-verify*` | substring match — blocks even in unrelated commands (over-broad but safe) |
| `reset --hard` | ✅ | `git reset --hard*` | soft/mixed intentionally allowed |
| `git clean` | ✅ | `git clean*` | blanket |
| `git rm` | ✅ | `git rm*` | blanket |
| `git revert` | ✅ DENIED | `git revert*` | **F1 — this contradicts rollback-discipline.md's #1 recommendation** |
| `git merge` / `rebase` / `cherry-pick` | ✅ | each blanket-denied | prevents agent-driven history surgery |
| `git switch` / `restore` / `checkout --` | ✅ | each denied | working-tree-clobber prevention |
| branch delete/rename/copy | ✅ | `git branch -d/-D/-m/-M/-c/-C*` | (change-* deletes re-allowed in allow-list for scheduler) |
| worktree prune / remove of `.`/`..`/repo | ✅ | explicit denies | + the safety hook (2c) |
| `git stash drop/clear` | ✅ | denied | protects the stash snapshots |
| `git tag -d`, `update-ref -d` | ✅ | denied | ref-deletion blocked |
| `gc --prune` | ⚠ **partial** | no explicit `git gc*` deny | relies on "never invoked" — `rollback-discipline.md §What-I-will-NEVER-do` lists it but there's **no deny pattern**. LOW risk (agent has no reason to run it) but it's a documented-but-unenforced item. |
| `git config --unset` | ✅ | `git config --unset*` | prevents un-setting protections |
| `git clone` | ✅ | denied | |

**Sufficiency:** The deny-list is the **primary and (server-side) only** enforcement layer. It is comprehensive for the agent. Two gaps: (i) F1 `git revert` denied-but-recommended; (ii) no `git gc*` deny (LOW). Stylistic over-breadth: `*--force*` / `*--no-verify*` substring patterns block these tokens even inside `echo`/comments (observed twice during this audit when an `echo` mentioning `--force` was denied). Safe, mildly annoying — leave as-is.

### 2b. `git-checkout-safety.sh` (PreToolUse Bash) — WIRED ✅
- **Wired:** yes, settings.json PreToolUse(Bash) line 159.
- **What it does:** blocks `git checkout HEAD -- <file>` / `git restore` when the target path has **staged-but-uncommitted** changes (the 2026-05-26 app.css silent-wipe incident, Class 1 of A12). Override `EVIUM_CHECKOUT_OK=1`.
- **Sufficient?** For its scope, yes. **Gaps:** (i) only triggers when `--`/`HEAD` is present *and* something is staged — an **unstaged** working-tree edit is not protected (git checkout would clobber it; but `git checkout --*` is also deny-listed, so the Bash-tool path is covered by the deny anyway). (ii) Path-parsing is heuristic (sed/awk flag-stripping) — exotic flag orders could slip, but the deny-list is the backstop. (iii) **Does not fire on Cursor** (see F2).

### 2c. `worktree-remove-safety.sh` (PreToolUse Bash) — WIRED ✅
- **Wired:** yes, line 151.
- **What it does:** refuses `git worktree remove/delete` on any path not matching `../EviumOverhaul-(calibration-…|change-…)`. Protects main + personal-branch worktrees.
- **Sufficient?** For the **Bash-tool** path, yes. **Gap (F4):** the scheduler calls `git worktree remove --force` via **Python subprocess** (`scripts/scheduler/worktree.py` `_cleanup_change_artifacts`, and `cleanup_expired_worktrees`), which **never goes through the Bash tool**, so neither this hook nor the deny-list applies. The Python side has its own defense (positive-ownership `WorktreeRegistry` + `_OWNED_WT_PATH_RE = .*/EviumOverhaul-(change|calibration)-…`). That's adequate, but the protection is structurally *different* from where a reader expects it.

### 2d. Worktree safety architecture (defense-in-depth) — SOUND ✅
From `worktree.py` docstring, four layers:
1. Positive-ownership registry (scheduler never deletes a WT it didn't create) — JSON, atomic tmp+rename write.
2. settings.json deny patterns (`.`/`..`/repo-path blocked).
3. TTL respected (no premature delete).
4. `--dry-run` default for first N invocations (bash wrapper).
- **Scheduler reality vs docs:** `create_worktree` uses **`git worktree add <path> -b <branch>`** (a fresh branch off current HEAD) — **not** literally `git worktree add HEAD` as `git-drift-warn.sh` / `git-hygiene.md` phrase it. The drift *concern* is still valid (the new branch starts at HEAD; a diff generated against the dirty working tree can fail to apply), but the phrasing in those two files is imprecise. Minor doc nit.

### 2e. settings-edit-guard.sh (PreToolUse Write|Edit) — WIRED, WARN-mode ✅⚠
- **Wired:** yes, line 204. Currently **WARN mode** (`EVIUM_SETTINGS_GUARD_MODE` defaults WARN; dry-run window per A47 Rule 5).
- **What it does:** deep-diffs `.permissions` (jq -S) of on-disk vs proposed; allows hook-only edits, would-block any `.permissions` change. Fails closed on unparseable/missing perms. `settings.local.json` fully blocked.
- **Sufficient?** Strong design (closes the "Bash-deny doesn't cover the Write tool" gap). **Gaps:** (i) still WARN — a permission-widening edit *proceeds* today, only logged. (ii) **F5 — does not cover Cursor:** a user editing `.claude/settings.json` in Cursor and committing it is invisible to this hook; the widened deny/allow takes effect next session.

### 2f. src-edit-guard.sh (PreToolUse Write|Edit) — WIRED, WARN-mode ✅⚠
- **Wired:** yes, line 208. WARN mode (`EVIUM_SRC_EDIT_GUARD_MODE` default WARN). Override `EVIUM_SRC_EDIT_OK=1`.
- **What it does:** WARN/BLOCK on any Write/Edit to `src/**` (A66 high-risk item #1). Logs to `.claude/cache/src-edit-guard.jsonl`.
- **Relevance to rollback:** this is the closest thing to "who-edits-what" enforcement, and it's exactly the boundary the Cursor transition formalizes (user owns `src/` hand-edits). Recommend flipping to **BLOCK** once the dual-author etiquette (below) is adopted — it cleanly encodes "Claude does not touch src/ in the dual-author era without per-action auth."

### Guards summary
- **Strong + wired:** deny-list (primary), checkout-safety, worktree-remove-safety, worktree registry, settings-edit-guard, src-edit-guard, git-drift-warn (SessionStart surface), scheduler-tick-drift-warn (PreToolUse).
- **Gaps:** F1 (revert denied-but-recommended), F4 (subprocess bypasses Bash guards — mitigated by registry), F5 (tracked settings.json + Cursor blind spot), no `git gc*` deny (LOW), two guards still in WARN dry-run.

---

## (3) ROLLBACK ETIQUETTE DOC ACCURACY

### `rollback-discipline.md` — mostly excellent, ONE material error

**Correct + valuable:**
- Foundational guarantees (content-addressable/immutable, push appends, force denied, reflog 30+ days, unreachable-not-immediately-deleted) — all accurate and verified above.
- "Find the starting point" recipes (`git log --reverse`, `git checkout <sha>` browse, `git show <commit>:path`, `git diff`) — accurate; the `git show <sha>:path` recipe is the *safest* file-recovery path and (importantly) is **not deny-listed**.
- Strategy 3 (branch from old commit) and the "What I will NEVER do" list — accurate posture.
- The screenshot→rollback workflow + the manifest-SHA enhancement proposal — sound, still unbuilt.

**ERROR (F1):** §"Strategy 1 — `git revert` (recommended)" presents `git revert` / `git revert --no-commit` / `git revert <range>` as the recommended, runnable rollback for pushed commits. But `.claude/settings.json` line 529 has **`Bash(git revert*)` in the deny-list.** The agent **cannot execute** the recommended strategy. The doc has no note of this. A future agent following the doc will hit a denial mid-rollback.
- **Fix:** add a callout to Strategy 1: *"`git revert*` is currently deny-listed in this harness — the agent cannot run it. To revert a pushed commit, either (a) the user runs `git revert` directly, or (b) temporarily lift the deny with explicit user authorization, or (c) use `git show <sha>:path > file` per-file recovery (not deny-listed) for surgical undo."* Then either keep the deny (and demote revert to "user-run") or carve a revert allow.
- **Decision needed (surface to user):** is `git revert` denied *intentionally* (treat all history-touching ops as user-gated) or *incidentally* (caught by the blanket git-verb sweep)? If intentional, the doc must say "user-run only." If incidental, add `Bash(git revert*)` to the allow-list — revert is the *safest* rollback (append-only, never rewrites) and denying it pushes toward more dangerous workarounds.

**Minor:** Strategy 2 (`git checkout <commit> -- path`) uses the `--` form, which `git checkout --*` deny-lists. The doc should prefer `git show <sha>:path > file` (un-denied) as the agent-runnable equivalent.

### `git-hygiene.md` skill ref — accurate, one phrasing nit
- The 4 lessons (blanket-parent ignore / symlink trailing-slash / HEAD-vs-working-tree drift / dry-run gitignore), the drift thresholds (0–9/10–29/30–99/100+), the diff strategies (`--3way` default, `--reject`, "Bro"), and the hook severity matrix are all accurate and match the live `git-drift-warn.sh` bands.
- **Nit:** repeats "scheduler's `git worktree add` creates worktrees from HEAD" — the actual code is `git worktree add <path> -b <branch>` (branch *off* HEAD). The drift conclusion still holds; the mechanism phrasing is loose.
- **Gaps it flags as unbuilt (still unbuilt, confirmed):** `gitignore-lint.sh`, `git-hygiene-audit.sh`, `reactionary-git-audit.md`. Not blocking.

### `feedback_git-hygiene-discipline.md` (memory-mirror) — accurate, current
- Faithful 4-lesson recap + apply-rules; cross-links resolve. No corrections. It does **not** mention rollback-discipline.md's revert/deny contradiction (out of its scope; F1 lives in rollback-discipline.md).

### Etiquette completeness gap
**None of the three docs cover the dual-author (Cursor) era.** They're written for a single-author (Claude-in-harness) world. That's the substance of section (4) — it should be captured as a new section in `rollback-discipline.md` (or a sibling `dual-author-git-etiquette.md` ref under agentic-quality-discipline).

---

## (4) THE CURSOR DUAL-AUTHOR RISK + SAFE ETIQUETTE

**Context that makes this live now:** git config already shows `branch.*.vscode-merge-base origin/main` on many branches — the repo is already opened in a VS Code-family editor (Cursor is a VS Code fork). The transition isn't hypothetical.

### The structural asymmetry (the root of every risk below)
Every guard audited in §2 — the Bash deny-list, all the PreToolUse hooks — is a **Claude Code harness construct**. **Cursor does not read `.claude/settings.json`'s deny-list or run these hooks.** When the user runs git in Cursor's integrated terminal (or via Cursor's Source Control UI), they have an **unguarded** git: `push --force`, `reset --hard`, `clean -fdx`, `branch -D`, `rebase` all work. And **`main` has no GitHub branch protection** (`gh api …/protection` → 404 "Branch not protected"). So the "structurally impossible" claims in `rollback-discipline.md` are true *only inside the harness*.

### New risks

| # | Risk | Mechanism | Severity |
|---|---|---|---|
| R1 | **Uncommitted-work clobber** | User has unsaved/uncommitted Cursor edits to a file; Claude (or scheduler) commits, stashes, or checks out that file → user's edits lost (or vice-versa). Working tree is dirty *right now* (19 files, `public/dashboards.html` +195, `generate_dashboard.py` +540). | HIGH |
| R2 | **Mixed agent+human commits** | Both authors commit to `main` interleaved. Hard to attribute, hard to revert a "Claude change" without catching a user change in the same range. | MEDIUM |
| R3 | **Divergence / non-fast-forward** | User commits in Cursor + pushes; Claude's local `main` (82 ahead, 0 behind today) diverges. Next Claude `git push*` is denied anyway, but Claude's view of HEAD goes stale; a user `pull`/`rebase` could silently relocate Claude's unpushed commits. | MEDIUM |
| R4 | **Worktree drift breaks scheduler** | The 17 worktrees include stale ones (`quirky-curie` is "behind 67"; many pin genesis `02359cd`). User edits in main while scheduler diffs against HEAD → `git apply` fails (the exact Phase A.2 failure git-hygiene.md documents). User opening a *worktree* dir in Cursor and editing there compounds it. | MEDIUM |
| R5 | **Settings.json widened in Cursor** | User edits `.claude/settings.json` (tracked) in Cursor — intentionally or via a merge — loosening deny/allow. `settings-edit-guard.sh` never sees it (not the Claude tool path). Next session, agent has wider powers. | MEDIUM (F5) |
| R6 | **Force-push from Cursor** | User (or a Cursor AI action) force-pushes `main`. No branch protection → remote history rewrites → Claude's 82 unpushed-ahead commits' base moves; reflog still saves locally, but the shared remote is now rewritten. | LOW-prob / HIGH-impact |
| R7 | **17 stale worktree dirs as edit traps** | User opens the wrong dir in Cursor (e.g. `.claude/worktrees/quirky-curie-bc56ce` at genesis) and edits "the site," producing commits on a dead branch that never reach main. | LOW |

### Recommended safe etiquette for the dual-author era

**A. Commit hygiene (both authors)**
1. **Commit-before-handoff rule (the single most important one).** Whoever is about to stop working commits (or stashes with a labeled message) *before* handing the working tree to the other author. Never leave dirty state across an author switch. Directly kills R1.
2. **Keep drift low.** `git-drift-warn.sh` already surfaces at SessionStart; treat any score ≥30 as "commit before letting the other author touch the tree." (The repo is at 19 now — fine, but `dashboards.html`/`generate_dashboard.py` are big uncommitted blocks; commit them before a Cursor session.)
3. **Author-tagged commits.** Claude commits keep the `Co-Authored-By: Claude` trailer (already standing); user-in-Cursor commits stay un-trailered. Makes R2 attribution + selective revert tractable (`git log --author`, `git log --grep "Co-Authored-By: Claude"`).

**B. Who-commits-what (ownership split)**
4. **`src/` is the user's hand-edit zone in Cursor.** This is already the spirit of `src-edit-guard.sh` (A66 #1). Recommend **flipping `src-edit-guard.sh` to BLOCK mode** now that the user owns fine-grained `src/` edits — it cleanly encodes "Claude does not touch `src/` without per-action auth," removing the largest R1/R2 surface. Claude continues to own `scripts/`, `.claude/skills/`, `context/markdowns/`.
5. **`.claude/settings.json` permission block stays USER-only, by hand, in Cursor or here.** Flip `settings-edit-guard.sh` from WARN→BLOCK (its dry-run window has data) so the agent-tool path is locked; the user editing it in Cursor is the *intended* path. Mitigates F5/R5. (Optionally add a SessionStart check that diffs committed `.permissions` vs last-known to surface a Cursor-side widening.)

**C. Branch / worktree conventions**
6. **User works on `main` in the root checkout only** (`<REPO_ROOT>`). Do **not** hand-edit inside `.claude/worktrees/*` or `/private/tmp/*` worktrees — those are scheduler/calibration-owned and several are stale. Kills R4/R7.
7. **Prune the stale worktrees** (user-run, since `git worktree prune*` + non-pattern `git worktree remove` are denied to the agent): the 9 genesis-pinned `02359cd` branches and `quirky-curie` (behind 67) are dead weight and edit-traps. (Out-of-scope to do here — read-only — but flag for the user.)
8. **Remote pushes stay one-author-at-a-time.** Since Claude can't push at all (denied), the user is the sole pusher. Keep it that way: **enable GitHub branch protection on `main`** (require-non-force, optionally require-PR) so even a Cursor-side `--force` is rejected server-side. This is the one fix that makes `rollback-discipline.md`'s "can't rewrite history" claim *actually* true rather than harness-local. Mitigates R6.

**D. Divergence handling**
9. **User integrates, not Claude.** If the user pushes from Cursor, the integration step (`pull --ff-only` / `rebase`) is the user's — Claude's `pull/fetch/merge/rebase` are all denied, so Claude *cannot* silently relocate commits. Good default; just make sure the user uses `--ff-only` to notice divergence instead of auto-merging. Mitigates R3.

### Two concrete config changes to recommend to the user (gated — not done, read-only audit)
1. **Enable branch protection on `main`** (the only server-side safety; closes R6 + makes the docs honest).
2. **Flip `src-edit-guard.sh` and `settings-edit-guard.sh` to BLOCK** (both have dry-run data; formalizes the ownership split for the Cursor era).

### One doc action to recommend
3. **Fix F1 in `rollback-discipline.md`** (revert denied-but-recommended) and **add a "Dual-author (Cursor) etiquette" section** capturing A–D above (or spin a sibling ref). Decide whether `git revert` should be allow-listed (it's the safest rollback) or remain user-run.

---

## Findings ledger

| ID | Severity | Finding | Fix owner |
|---|---|---|---|
| F1 | HIGH (doc) | `git revert` recommended in rollback-discipline.md but DENIED in settings.json | user decision: allow-list revert OR mark "user-run"; fix doc |
| F2 | HIGH (Cursor) | No GitHub branch protection + Cursor ignores deny-list → harness-local safety only | user: enable branch protection on main |
| F3 | MEDIUM (Cursor) | Uncommitted-work clobber risk; no cross-author coordination | etiquette rule A1 (commit-before-handoff) |
| F4 | MEDIUM (gap) | Scheduler `subprocess` worktree-remove bypasses Bash deny + safety hook (registry mitigates) | note; consider asserting registry-ownership louder |
| F5 | LOW-MED | `.claude/settings.json` tracked; Cursor edits bypass settings-edit-guard | flip guard to BLOCK + SessionStart perms-diff surface |
| F6 | LOW | No `git gc*` deny pattern (documented-but-unenforced "never") | optional: add `Bash(git gc*)` deny |
| F7 | LOW (nit) | git-drift-warn.sh + git-hygiene.md say `worktree add HEAD`; actual code is `add -b <branch>` | doc phrasing fix |

## What was NOT done (read-only audit)
- No config edits, no doc edits, no worktree pruning, no guard flips. All recommendations above are **gated** for explicit user decision per the high-risk-always-gated list (settings.json, branch protection, deny-list changes).
