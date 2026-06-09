---
id: A12
title: Destructive-ops "oh-shit" audit — what can clobber uncommitted/staged work (Cursor dual-author era)
type: audit
status: completed
mode: read-only
created: 2026-05-29
audited_by: subagent (BG dispatch, read-only)
scope: every code path / script / hook / scheduler op / git command that could destroy uncommitted or staged work; cross-ref A85 + the new in-Python worktree guard; the dual-author (Claude+Cursor) clobber surface; Bash-guard-but-not-Python-subprocess gaps (the F4 class)
serves: manifesto §3 (git-safety)
cross_refs:
  - context/markdowns/goal-billboard/audits/A85-git-rollback-safety-audit.md
  - scripts/scheduler/worktree.py            # assert_safe_to_remove / _remove_worktree (the NEW guard)
  - scripts/scheduler/scheduler.py           # _apply_diff_to_main / _regenerate_main_baselines / _cleanup_change_artifacts
  - scripts/scheduler/__main__.py            # cmd_tick (uses REPO_ROOT = real main) + cmd_regenerate_baselines
  - scripts/scheduler-cleanup-orphan.py      # standalone worktree reaper (does NOT use the new guard)
  - scripts/hooks/git-checkout-safety.sh
  - scripts/hooks/worktree-remove-safety.sh
  - scripts/hooks/src-edit-guard.sh
  - scripts/hooks/settings-edit-guard.sh
  - .claude/settings.json                    # permissions.deny / allow
---

> ⚠ **ARCHIVED STALE SNAPSHOT (reconciled 2026-06-03).** This is an earlier 164-line copy of the A12 findings.
> The **canonical, current findings (185 lines) live at**
> [`../../audits/findings/A12-destructive-ops-audit.md`](../../audits/findings/A12-destructive-ops-audit.md).
> Kept (body intact, recoverable) only as the dashboard's "completed audits" marker — read the canonical for
> the up-to-date findings. (Best-handle per the 2026-06-03 sweep: no content loss, no dashboard/`A*`-prefix touch.)

# A12 — Destructive-ops "oh-shit" audit

**Premise (user, 2026-05-29):** what current operations could DESTROY uncommitted/staged work — now especially dangerous in the dual-author Cursor era. Live state at audit time: **27 dirty files, ~17 worktrees, two authors editing** (Claude-in-harness + user-in-Cursor). This is the original A12 "oh-shit" survey, refreshed against the A85 findings and the NEW in-Python worktree guard.

**Bottom line.** The Bash-tool surface is well-guarded (deny-list + 2 PreToolUse hooks; A85 verified). The new `worktree.py` guard (`assert_safe_to_remove` + `_remove_worktree` dirty-check) **closed A85-F4 for the scheduler's own removal paths** — that fix is real and good. But three destructive surfaces remain, two of them NOT covered by the new guard:

- **F1 (HIGH, UNGATED):** `scripts/scheduler/scheduler.py:191 _apply_diff_to_main` runs `git apply --3way` **directly into the real main repo's working tree via Python subprocess** with **NO dirty-check first**. The Bash deny-list and `git-checkout-safety.sh` never see it. If the user has uncommitted Cursor edits to a file the diff touches, `--3way` silently writes conflict markers into (or fails partway through) that file — the #1 dual-author clobber vector, and the very thing the new worktree guard protects worktrees from but this path does NOT protect main from.
- **F2 (MEDIUM-HIGH, UNGATED):** **TWO** standalone worktree reapers run `git worktree remove <path> --force` (+ `git branch -D`) using **only a path-regex** — neither routes through the new `assert_safe_to_remove`/`_remove_worktree` dirty-check: `scripts/scheduler-cleanup-orphan.py:49` (with `--force` reaps **every** registry entry regardless of TTL) and `scripts/scheduler/cli/cleanup.py:59` (with `--all` reaps everything; has a `--dry-run` flag but defaults to live removal). Both `--force`-clobber any uncommitted work a user hand-edited inside a scheduler worktree. This is the A85-F4 gap **still open in two sibling code paths** the worktree.py fix didn't reach.
- **F3 (MEDIUM, UNGATED):** `scripts/scheduler/scheduler.py:233 _regenerate_main_baselines` runs `npx playwright test --update-snapshots` against main's working tree (spawned **detached** via `__main__.py cmd_regenerate_baselines`), **overwriting committed/uncommitted PNG baselines under `src/**/__screenshots__`** with no dirty-check. Auto-fires after a successful `apply_to_main` when `expected_visual_diff:true`.

Then the A85-carried items that are still exposed (F4–F8 below), and the dual-author asymmetry that underlies all of it (F9).

---

## (1) THE DESTRUCTIVE-OP SURVEY — every path that can clobber uncommitted/staged work

### Class A — Bash-tool git ops (GUARDED — the well-covered surface)

These are the ops a reader first worries about. All are **denied at the Bash tool** (`.claude/settings.json` → `permissions.deny`) and additionally hook-guarded. Verified present:

| Op | Clobbers what | Guard | Evidence |
|---|---|---|---|
| `git reset --hard` | working tree + index | DENIED | settings.json:534 `Bash(git reset --hard*)` |
| `git clean [-fdx]` | untracked + ignored files | DENIED | settings.json:553 `Bash(git clean*)` |
| `git checkout -- <path>` / `git restore` / `git switch` | uncommitted file edits | DENIED + hook | settings.json:535/533/532 + `git-checkout-safety.sh` (PreToolUse, settings.json:159) |
| `git stash drop` / `clear` | the stash snapshots | DENIED | settings.json:555/556 |
| `git rm` | tracked working file | DENIED | settings.json:531 |
| `git worktree remove <non-pattern>` / `prune` | a whole worktree's uncommitted work | DENIED + hook | settings.json:546-552 + `worktree-remove-safety.sh` (settings.json:151) |
| `git branch -D` (non change-/calibration-) | a branch's unique commits | DENIED | settings.json:537-541 |
| `git push --force` / `rebase` / `merge` / `cherry-pick` / `revert` / `filter-*` | shared history | DENIED (push fully blocked) | settings.json:523/527/526/528/529 + `*--force*` |
| `git update-ref -d` / `git tag -d` / `git config --unset` | refs / protections | DENIED | settings.json:557/558(tag)/41-region |

**`git-checkout-safety.sh` scope (the original A12 Class-1 incident guard):** blocks `git checkout HEAD -- <file>` / `git restore` only when `--`/`HEAD` is present **and** the target path is **staged** (`git diff --cached`). Evidence: `git-checkout-safety.sh:33` (the `--`/`HEAD` heuristic gate), `:53` (reads `git diff --cached --name-only`). Override `EVIUM_CHECKOUT_OK=1`. **Gap (carried from A85):** an **unstaged** working-tree edit is not detected by this hook (it keys on `git diff --cached`) — but the same command is also deny-listed (`git checkout --*`), so the *Bash-tool* path is covered by the deny anyway. The hook's residual value is the override-with-warning ergonomics; the deny is the real wall.

**Verdict on Class A:** comprehensive for the agent's Bash tool. This is NOT where the remaining danger lives.

### Class B — Python subprocess git ops (the F4 class — Bash guards DO NOT fire here)

The harness hooks (`PreToolUse(Bash)`) and the deny-list match the **Bash tool** only. Anything a Python script shells out via `subprocess.run(["git", ...])` **bypasses all of it**. This is the structural gap A85 named F4. Inventory of every such call that can touch a working tree:

| # | Call | File:line | Targets | New guard? | Dirty-check? |
|---|---|---|---|---|---|
| B1 | `git apply --3way --whitespace=nowarn` (stdin diff) | `scheduler.py:210-216` | **the REAL main working tree** (`cwd=main_root`, default `REPO_ROOT`) | ❌ no | ❌ **NONE** → **F1** |
| B2 | `git worktree remove <path> --force` | `worktree.py:137-140` (via `_remove_worktree`) | a change/calibration worktree | ✅ `assert_safe_to_remove` | ✅ `allow_dirty` param + `_worktree_is_dirty` |
| B3 | `git worktree remove <path> --force` + `git branch -D` | `scheduler-cleanup-orphan.py:49-53,57-61` **AND** `scheduler/cli/cleanup.py:59-64,75-80` | a change/calibration worktree | ❌ **regex only, NOT the new guard (both reapers)** | ❌ **NONE** → **F2** |
| B4 | `git worktree add <path> -b <branch>` | `worktree.py:228-229` | creates a worktree off HEAD | n/a (additive) | n/a |
| B5 | `git branch -m`/`-D change-<id>` | `scheduler.py:162-163,177-178` | a change-* branch (no unique commits by design) | scoped to `change-<id>` name | n/a (branch has no commits) |
| B6 | `npx playwright … --update-snapshots` | `scheduler.py:258-260` (sync) + `__main__.py` detached | **PNG baselines under main's working tree** | ❌ no | ❌ **NONE** → **F3** |

**The asymmetry that makes F1/F2/F3 sharp:** the new `worktree.py` guard proves the team *knows* the F4 class exists and *how to fix it* (path-pattern + registry-ownership + `_worktree_is_dirty` fail-safe). B2 is fully fixed. But the **same discipline was not applied** to (a) the diff-apply-into-main path (B1/F1 — arguably the highest-value clobber target since it writes the user's actual main tree, not an ephemeral worktree), nor to (b) the standalone orphan reaper (B3/F2), nor to (c) the snapshot regen (B6/F3).

### Class C — filesystem writes / overwrites (scanned — all SAFE)

Surveyed every `write_text` / `open('w')` / `os.replace` / `shutil.rmtree` / `unlink` in `scripts/` (excluding tests). Findings:

- **Dashboard / billboard regen do NOT bake over hand-edits.** `generate_dashboard.py:55` writes `reports/dashboard-build/dashboard.html` only (docstring `:36-38` explicitly: "Writes EXACTLY ONE new path… NEVER writes/deletes the live blob, template.html, any data.json, or generate.py"). `render-billboard-page.py:31` writes `reports/billboard.html`. **Nothing regenerates into `public/` or `src/`.** Confirmed: zero script writes target `src/` or `public/` (grep for `/src/`+`/public/` ∩ write ops → empty). The A85-R-noted `public/dashboards.html` +195 is a **dirty-state hazard** (uncommitted block, F6 below), not a regen-clobber.
- **Every state/JSONL/markdown writer uses atomic tmp+rename** (e.g. `worktree.py:179-181`, `state.py` docstring "uses `mv` for state changes; never trusts partial writes", `handoff-from-rollback.py:107-109`, `inbox-write.py:142`). Atomic writes can't *interleave* a corrupt file, but a tmp+rename onto a **tracked file the user is hand-editing in Cursor** still overwrites the user's on-disk copy — none of the audited writers target a dual-author-contended tracked path, so this stays theoretical here.
- **`audit-close.sh` uses `git mv`** (`:473`) to archive a closed audit (falls back to plain `mv`). Move, not clobber — safe. Only touches the audit file being closed.
- **`scripts/*` `os.remove`/`unlink` hits** (`session-bundle-create.sh:717` temp cleanup, `claude-5hr-window.py:104` removes its own state file, `inbox-integrity-check.py:204` removes a malformed inbox card, `migrate-add-ns-field.py:299` tmp cleanup) — all operate on their own scratch/state files or quarantined inbox cards, **never** a user source file. Safe.

### Class D — autonomous execution (the "fires without an agent in the loop" question)

**Verified: NO autonomous destructive path.** The 5 launchd daemons (`~/Library/LaunchAgents/com.evium.daemon{2,3,4,5,10}.plist`) run **read-only event-bus consumers** (memory-consolidator, audit-catalog-scan, steering-log-compact, bug-billboard-groom, heartbeat-tier-low). Surveyed daemon4/daemon5/daemon2: only `grep`/`find`/`wc`/`stat` + writes to their own `*.jsonl` log dirs — **no `git reset/checkout/clean`, no `rm` on tracked files, no `git apply`.**

The scheduler tick (which contains B1/F1, B3 sibling, B6/F3) is **NOT** on any daemon. It fires only when an agent or the user runs `scripts/multi-change-tick.sh` → `scheduler/__main__.py cmd_tick`. **But** `cmd_tick` constructs the Scheduler with `wt_root_template = REPO_ROOT.parent / "EviumOverhaul-change-…"` and the apply path resolves `main_root = self.main_repo_root or REPO_ROOT` (`scheduler.py:724`) — i.e. **the real main checkout**. So when a queued change carries `apply_to_main_on_verify:true`, a single agent-invoked tick **will** run `git apply --3way` into the live, dirty, dual-author main tree (F1) with no dirty-check. Not autonomous, but it's one Bash call away and the agent has no guard catching it.

---

## (2) CROSS-REFERENCE: A85 findings + the NEW worktree guard — what's still exposed

The new in-Python guard (`worktree.py` `assert_safe_to_remove` + `_remove_worktree`, added post-A85) does three things on the scheduler's own removal paths: (1) path-pattern regex, (2) positive-ownership registry check, (3) `_worktree_is_dirty` refusal unless `allow_dirty=True`. Wired into `cleanup_expired_worktrees` (`worktree.py:268`) and `_cleanup_change_artifacts` (`scheduler.py:144`). **This genuinely closes A85-F4 for those two call sites.** What the fix did NOT reach:

| A85 finding | Status after the worktree.py fix | A12 ID |
|---|---|---|
| **F4** (subprocess worktree-remove bypasses Bash guards) | **Partially closed.** `worktree.py` + `scheduler.py` removal paths now guarded. **BUT `scheduler-cleanup-orphan.py` is a separate reaper that still uses bare regex + `--force` with no dirty-check** — the same class of gap, still open. | **F2** |
| (NEW, not in A85) diff-apply into main | **Never addressed.** `_apply_diff_to_main` has no equivalent guard. Highest-impact clobber (writes the real main tree). | **F1** |
| (NEW, not in A85) snapshot regen into main | **Never addressed.** `--update-snapshots` overwrites baselines. | **F3** |
| **F1** (`git revert` recommended but denied) | Open — doc-vs-deny contradiction, `rollback-discipline.md` still recommends revert; settings.json:529/10-region still denies it. | **F4 (this doc)** |
| **F2** (no GitHub branch protection + Cursor ignores deny-list) | Open — `main` unprotected; every Class-A guard is harness-local. | **F9 (this doc)** |
| **F3** (uncommitted-work clobber, no cross-author coordination) | Open at the *protocol* level — no commit-before-handoff lock exists. Worktree-level clobber is now blocked (B2), but main-tree clobber (F1) and orphan-reaper clobber (F2) are live. | **F5 (this doc)** |
| **F5** (tracked settings.json, Cursor edits bypass settings-edit-guard) | Open — `settings-edit-guard.sh` is WARN-mode + Bash/Write-tool only. | **F7 (this doc)** |
| **F6** (no `git gc*` deny) | Open — LOW. | **F8 (this doc)** |

---

## (3) THE DUAL-AUTHOR ANGLE — where Claude's writes clobber the user's Cursor edits (or vice-versa)

The root asymmetry (A85-F2, restated): **every Class-A guard is a Claude Code harness construct. Cursor reads none of it.** When the user runs git in Cursor's terminal or Source-Control UI, `reset --hard` / `clean -fdx` / `checkout -- .` / force-push all work unguarded, and `main` has no server-side branch protection. So the dual-author clobber is bidirectional:

**Claude → user (Claude's writes clobber the user's uncommitted Cursor edits):**
- **F1 is the sharpest case.** A queued change with `apply_to_main_on_verify:true` → `git apply --3way` into main. If the user is mid-edit in Cursor on (say) `src/styles/app.css` and hasn't committed, the 3-way apply writes conflict markers into the file *on disk*. Cursor's editor buffer may still hold the user's version; whoever saves last wins, and if Cursor auto-saved, the markers are now in the user's working copy. No dirty-check, no warning. (`scheduler.py:210` / call site `:736`.)
- **F2:** if the user opened a scheduler worktree dir in Cursor (the A85-R7 edit-trap — 17 worktrees, several stale) and hand-edited there, `scheduler-cleanup-orphan.py --force` `--force`-removes it, discarding those edits with no dirty-check. (`scheduler-cleanup-orphan.py:49`.)
- **F3:** `--update-snapshots` overwrites PNG baselines under main; if the user had hand-curated/uncommitted baseline images, they're regenerated away. (`scheduler.py:258`.)
- **src-edit-guard is the relevant boundary but it's WARN-mode.** `src-edit-guard.sh:34` defaults `MODE=WARN` → an agent Write/Edit to `src/**` **proceeds**, only logging to `.claude/cache/src-edit-guard.jsonl`. In the dual-author era where `src/` is the user's hand-edit zone, a WARN-mode guard does not prevent Claude from overwriting the user's in-progress `src/` work via the ordinary Write/Edit tool. (And it only covers the Write/Edit *tool* — not the F1 subprocess apply.)

**User → Claude (the user's Cursor ops clobber Claude's uncommitted/staged work):** fully unguarded by design — the harness can't gate Cursor. A user `git reset --hard` / `git checkout -- .` / `git stash` in Cursor discards anything Claude staged-but-didn't-commit. Mitigation is purely protocol (commit-before-handoff, F5) — there is no technical guard and cannot be one from inside the harness.

**The F4-class gap, stated precisely (the part the user asked for):**
> Guards that exist on the **Bash tool** but NOT on the **Python subprocess** path:
> - `git-checkout-safety.sh` (staged-file checkout block) — no subprocess equivalent. F1's `git apply` is conceptually a working-tree overwrite, exactly what this hook guards against on the Bash side, with **zero** coverage on the subprocess side.
> - `worktree-remove-safety.sh` (path-pattern remove block) — **HAS** a subprocess equivalent now (`assert_safe_to_remove`) for B2, but `scheduler-cleanup-orphan.py` (B3/F2) re-implements the bare regex and skips the dirty-check.
> - the deny-list `git apply`/`git worktree remove --force` matching — never applies to subprocess at all.

---

## Findings ledger

| ID | Severity | Finding | Evidence | Mitigation | GATED? |
|---|---|---|---|---|---|
| **F1** | **HIGH** | `_apply_diff_to_main` runs `git apply --3way` into the REAL main working tree via subprocess with NO dirty-check; bypasses deny-list + checkout-safety hook. Dual-author main-tree clobber. | `scheduler.py:191,210-216`; call site `:723-739` (`main_root = self.main_repo_root or REPO_ROOT`) | **UNGATED.** Add a pre-apply `git status --porcelain` check on the touched paths (reuse `worktree.py:_worktree_is_dirty` pattern): if any path the diff modifies is dirty in main, refuse + emit `applied_to_main_refused_dirty` and require explicit opt-in (env flag or `change.apply_over_dirty:true`). Mirrors the worktree dirty-guard onto the main-tree path. | **UNGATED** (apply currently proceeds with no check) |
| **F2** | MED-HIGH | TWO standalone reapers (`scheduler-cleanup-orphan.py` + `scheduler/cli/cleanup.py`) `git worktree remove --force` + `git branch -D` using path-regex ONLY — neither calls the new `assert_safe_to_remove`/`_remove_worktree`; `--force`/`--all` reaps ALL registry entries regardless of TTL, `--force`-clobbering a user's hand-edits inside a worktree. | `scheduler-cleanup-orphan.py:49-53,57-61`; `scheduler/cli/cleanup.py:54-65,68-81` (each its own `SAFE_*_RE`, no dirty-check) | **UNGATED.** Route BOTH scripts through `worktree._remove_worktree(..., allow_dirty=False)` so the dirty-check + registry-ownership apply. Standalone reapers should not have a weaker guard than the in-tick path. | **UNGATED** |
| **F3** | MED | `_regenerate_main_baselines` (`--update-snapshots`) overwrites PNG baselines under main's working tree; auto-fires (detached) after `apply_to_main` when `expected_visual_diff:true`; no dirty-check. | `scheduler.py:233,258-260`; spawn via `__main__.py cmd_regenerate_baselines` | **UNGATED.** Guard with a baseline-dir dirty-check, or commit/stash baselines before regen; at minimum emit a loud event when it overwrites uncommitted `__screenshots__`. | **UNGATED** |
| **F4** | HIGH (doc) | (= A85-F1, still open) `rollback-discipline.md` recommends `git revert` as strategy #1 but it's DENIED (`settings.json:529`). Agent following the doc hits a wall; pushes toward riskier workarounds. | `settings.json:529` `Bash(git revert*)`; `rollback-discipline.md` §Strategy 1 | **GATED.** User decides: allow-list `git revert` (it's the safest, append-only rollback) OR mark it "user-run only" in the doc. Either way fix the doc. | **GATED** (touches deny-list) |
| **F5** | MED | (= A85-F3) Uncommitted-work clobber, no cross-author coordination protocol. Working tree dirty NOW (27 files). F1/F2 are the live technical vectors; the protocol gap is the human one. | A85 §4 R1; live `git status` (27 dirty) | **GATED** (protocol). Adopt commit-before-handoff rule; treat `git-drift-warn.sh` score ≥30 as a hard "commit before the other author touches the tree." | **GATED** (workflow change) |
| **F6** | MED | Big uncommitted blocks sitting in the working tree are themselves the hazard (any user-side `reset --hard`/`checkout -- .` in Cursor discards them). `public/dashboards.html`, `generate_dashboard.py` etc. carry large uncommitted deltas. | A85 §4 R1; `git status` shows modified `scripts/dashboard/generate_dashboard.py`, `public/`-region edits | **GATED.** Commit the large dashboard blocks before any Cursor session. (Read-only here — flag for user.) | **GATED** (commit decision) |
| **F7** | LOW-MED | (= A85-F5) `.claude/settings.json` is git-tracked; `settings-edit-guard.sh` is WARN-mode + only the Write/Edit *tool* — a Cursor edit (or any subprocess) that widens deny/allow is invisible and takes effect next session. | `settings-edit-guard.sh` (WARN default); `src-edit-guard.sh:34` (`MODE=WARN`) | **GATED.** Flip `settings-edit-guard.sh` + `src-edit-guard.sh` to BLOCK (both have dry-run data per A85); add a SessionStart `.permissions` diff-vs-last-known surfacing a Cursor-side widening. | **GATED** (guard-mode flip) |
| **F8** | LOW | (= A85-F6) No `git gc*` deny pattern; `gc --prune=now` would purge recoverable dangling stash snapshots. Documented-but-unenforced "never." | `settings.json` deny-list (no `git gc*`); `rollback-discipline.md` §never-do | **GATED.** Optional: add `Bash(git gc*)` deny. LOW (agent has no reason to run it). | **GATED** (deny-list add) |
| **F9** | LOW-prob / HIGH-impact | (= A85-F2) No GitHub branch protection on `main` + Cursor ignores the deny-list → every Class-A guard is harness-local; a Cursor-side force-push rewrites shared history. | A85 §4 (R6; `gh api …/protection` → 404) | **GATED.** Enable branch protection on `main` (require-non-force). The one server-side fix that makes the rollback-discipline "can't rewrite history" claim actually true. | **GATED** (GitHub config) |

---

## Mitigation summary — GATED vs UNGATED

**UNGATED (code-only fixes inside Claude's own zone — `scripts/scheduler/`, no settings/deny/branch change; these are the new-guard-not-applied gaps and the highest-leverage clobber-prevention):**
- **F1** — add a main-working-tree dirty-check to `_apply_diff_to_main` before `git apply --3way` (refuse / require explicit opt-in if a touched path is dirty). *This is the single highest-value fix — it protects the user's real main tree, the thing the new worktree guard does NOT cover.*
- **F2** — route `scheduler-cleanup-orphan.py` through `worktree._remove_worktree(..., allow_dirty=False)` so the new dirty-check + registry-ownership guard actually applies there too.
- **F3** — dirty-check (or commit/stash) the baseline dir before `--update-snapshots`; loud event when it overwrites uncommitted `__screenshots__`.

**GATED (require explicit user decision per the A66 high-risk-always-gated list — settings.json / deny-list / branch protection / workflow):**
- **F4** — allow-list `git revert` OR mark user-run; fix `rollback-discipline.md` (deny-list touch).
- **F5/F6** — adopt commit-before-handoff; commit the large uncommitted dashboard blocks (workflow + commit decisions).
- **F7** — flip `settings-edit-guard.sh` + `src-edit-guard.sh` WARN→BLOCK; add SessionStart perms-diff (guard-mode change).
- **F8** — optional `Bash(git gc*)` deny (deny-list add).
- **F9** — enable GitHub branch protection on `main` (GitHub config; the only server-side safety; also covers the Cursor force-push case).

---

## What was NOT done (read-only audit)
No code edits, no guard flips, no deny-list changes, no worktree pruning, no commits, no branch-protection changes. Every mitigation above is a recommendation; the UNGATED ones are code-only and safe to implement next but were NOT applied here. The GATED ones are deferred to explicit user decision per the high-risk-always-gated list (settings.json, branch protection, workflow change, deny-list edits).

**Honest caveats:** (a) The diff-apply / orphan-reaper / snapshot-regen paths fire only on an agent- or user-invoked scheduler tick / cleanup, NOT autonomously (Class D verified clean) — so F1/F2/F3 are "one Bash call away," not "happening on a timer." That lowers probability, not severity. (b) Class-C filesystem writes were surveyed and found safe (atomic, scoped to scratch/reports, none target `src/`/`public/`); the dual-author file-overwrite risk there is theoretical absent a contended tracked path. (c) Whether `git revert` (F4) is denied intentionally or incidentally is a question for the user, carried from A85.
