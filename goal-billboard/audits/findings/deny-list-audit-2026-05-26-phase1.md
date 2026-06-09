---
created: 2026-05-26T11:50:00Z
updated: 2026-05-26T11:50:00Z
audit_id: A9
phase: 1 — read-only discovery
status: findings-locked; awaiting per-entry user approval before Phase 2
---

# A9 Phase 1 — Deny-list findings (read-only)

**No settings.json changes made.** This is a categorized inventory of every deny entry in `.claude/settings.json` with a recommendation column. Per the A9 safety protocol, each row needs separate user approval before any allowlist change is applied.

## Categorization framework

| Category | Meaning | Default action |
|---|---|---|
| **CATASTROPHIC** | If removed, single mistake = irreversible data loss or external exposure | **KEEP — NEVER LOOSEN** |
| **LOAD-BEARING** | Protects against a real class of "agent goes wrong" failures; specific narrow allows OK | **KEEP. Allow narrowly only with explicit reason** |
| **FRICTION** | Blocks legitimate workflow with no commensurate safety value at our scale | **CANDIDATE for narrow allow** |
| **DEFENSIVE-PARANOID** | Rare operations; deny costs nothing because agent never needs them | **KEEP. Cheap insurance** |

## Per-entry verdict (62 deny entries)

### 1. Filesystem destruction (NEVER LOOSEN)

| Entry | Category | Notes |
|---|---|---|
| `Bash(rm -rf*)` | CATASTROPHIC | Recursive force-delete. The classic "oh no" line. |
| `Bash(rm -r*)` | CATASTROPHIC | Recursive. Same risk class. |
| `Bash(rm -f*)` | CATASTROPHIC | Force flag = no prompt = no chance to catch typo. |
| `Bash(rm /*)` | CATASTROPHIC | Absolute-root targets. |
| `Bash(rm ~/*)` | CATASTROPHIC | Home directory targets. |
| `Bash(rm ../*)` | CATASTROPHIC | Parent-of-project targets. |
| `Bash(sudo*)` | CATASTROPHIC | Privilege escalation. Agent never needs this. |
| `Bash(find * -delete*)` | CATASTROPHIC | Mass-deletion via find. |
| `Bash(find * -exec rm*)` | CATASTROPHIC | Same. |
| `Bash(find * -exec mv*)` | CATASTROPHIC | Mass-rename — equivalent to delete-and-move. |
| `Bash(find * -execdir rm*)` | CATASTROPHIC | Same as exec but per-dir context — still mass-destructive. |
| `Bash(xargs rm*)` | CATASTROPHIC | Pipeline → mass-delete. |
| `Bash(xargs mv*)` | CATASTROPHIC | Same. |
| `Bash(mv /*)` | CATASTROPHIC | Absolute-source mv — easy to misroot. |
| `Bash(mv ~/*)` | CATASTROPHIC | Home-source. |
| `Bash(mv * ~/.Trash/EviumOverhaul)` | CATASTROPHIC | Mass-trash. |
| `Bash(rsync*--delete*)` | CATASTROPHIC | rsync delete-mode wipes destination. |

**Verdict: 17 entries. ALL KEEP. NEVER LOOSEN regardless of A10 findings.**

### 2. Git remote / history-mutating (NEVER LOOSEN — user pushes/pulls themselves)

| Entry | Category | Notes |
|---|---|---|
| `Bash(git push*)` | CATASTROPHIC | User explicitly stated they push themselves. |
| `Bash(git pull*)` | LOAD-BEARING | Avoid accidental remote merge that introduces conflict-resolution mistakes. |
| `Bash(git fetch*)` | DEFENSIVE | Pure-read but agent rarely needs it; the cost of denying is zero. |
| `Bash(git merge*)` | LOAD-BEARING | History modification + conflict risk. |
| `Bash(git rebase*)` | LOAD-BEARING | History rewrite — can lose commits if done wrong. |
| `Bash(git cherry-pick*)` | LOAD-BEARING | Cross-branch commit copying — needs human judgment. |
| `Bash(git revert*)` | LOAD-BEARING | Creates real commits with reversal semantics — user discretion. |
| `Bash(git clone*)` | DEFENSIVE | New-repo creation; rare; agent shouldn't be doing it. |
| `Bash(git remote remove*)` | DEFENSIVE | Severs upstream — irreversible without re-add. |
| `Bash(git remote rm*)` | DEFENSIVE | Alias of remove. |

**Verdict: 10 entries. ALL KEEP. The remote/history surface stays user-controlled.**

### 3. Working-tree mutators (KEEP with rare allow)

| Entry | Category | Notes |
|---|---|---|
| `Bash(git reset --hard*)` | LOAD-BEARING | "Make my mistake go away silently." Forces agent to face consequences via Edit tool revert. **Keep.** |
| `Bash(git restore*)` | FRICTION (mild) | Modern Git replacement for parts of checkout. Could enable narrow allow for `git restore <specific-file>` but easy to mis-fire. **Keep for now.** |
| `Bash(git checkout --*)` | FRICTION (mild) | Old-school file restore. Same friction as restore. **Keep.** |
| `Bash(git clean*)` | DEFENSIVE | Removes untracked files; agent never needs this (Edit handles single-file undo). **Keep.** |
| `Bash(git stash drop*)` | DEFENSIVE | Loses stashed work. **Keep.** |
| `Bash(git stash clear*)` | DEFENSIVE | Same. **Keep.** |
| `Bash(git tag -d*)` | DEFENSIVE | Deletes tags. We don't use tags yet. **Keep.** |
| `Bash(git update-ref -d*)` | DEFENSIVE | Low-level ref manipulation. **Keep.** |
| `Bash(git config --unset*)` | DEFENSIVE | Removes config keys. **Keep.** |
| `Bash(git rm*)` | DEFENSIVE | Removes from index + working tree. Edit tool handles file-by-file. **Keep.** |
| `Bash(git switch*)` | DEFENSIVE | Branch switching; agent does not currently need to switch branches. **Keep.** |

**Verdict: 11 entries. ALL KEEP. Mostly low-frequency or already-handled-by-Edit-tool.**

### 4. Branch deletion (FRICTION — recommend narrow allow)

| Entry | Category | Notes |
|---|---|---|
| `Bash(git branch -d*)` | DEFENSIVE | Safe-delete (only merged branches). Reasonable to deny by default. |
| `Bash(git branch -D*)` | DEFENSIVE | Force-delete. The risky one. |
| `Bash(git branch -m*)` | DEFENSIVE | Rename. Agent doesn't need. |
| `Bash(git branch -M*)` | DEFENSIVE | Force-rename. Agent doesn't need. |
| `Bash(git branch -c*)` | DEFENSIVE | Copy. Agent doesn't need. |
| `Bash(git branch -C*)` | DEFENSIVE | Force-copy. Agent doesn't need. |
| `Bash(git branch -d calibration-*)` | DEFENSIVE | Specific double-deny. **Keep — calibration branches may contain side-agent work.** |
| `Bash(git branch -D calibration-*)` | DEFENSIVE | Same. **Keep.** |
| **`Bash(git branch -d change-*)`** | **FRICTION ← KEY** | **The scheduler creates change-* branches that hold no commits (just an empty branch pointing at HEAD that the worktree's working-tree-only diff sits atop). They accumulate forever. Recommend allowlist.** |
| **`Bash(git branch -D change-*)`** | **FRICTION ← KEY** | **Same. Force-delete is safe for change-* since no commits exist on these branches. Recommend allowlist.** |

**Recommended specific allows (Phase 2 approval required):**

```jsonc
// Tightly scoped: only change-* branches, only the cases that need it
"Bash(git branch -d change-*)",          // safe-delete: only deletes if merged (or trivially clean)
"Bash(git branch -D change-*)",          // force-delete: needed because change-* branches point at HEAD with the working-tree diff (technically "unmerged")
```

Justification: change-* branches are scheduler-created with `git worktree add -b change-<id>`. The diff lives in the worktree's working tree, not in any commit on the branch. So the branch effectively aliases main HEAD. Deleting it = no data loss. The deny pattern was inherited from "general branch deletion is risky" without distinguishing this specific safe case.

**Risk assessment for the proposed allow:**
- Could a non-scheduler operation accidentally match `change-*`? Only if a human creates a branch named `change-<anything>`. Unlikely + user-controlled. Acceptable.
- Could a malicious change.md inject a path? change.md doesn't dictate branch names — scheduler constructs them from change_id. change_id IS user-influenced (it's in the frontmatter) but only as a filename / identifier, not as a shell expansion.
- Is `--force` (-D) safe here? Yes — branch has no unique commits.

**Verdict: 8 keep + 2 candidate allows. The 2 are the only friction-only entries in branch-deletion.**

### 5. Worktree-removal (CATASTROPHIC — never loosen)

| Entry | Category | Notes |
|---|---|---|
| `Bash(git worktree remove .)` | CATASTROPHIC | Would remove the worktree we're operating in. |
| `Bash(git worktree remove ..)` | CATASTROPHIC | Parent. |
| `Bash(git worktree remove ../EviumOverhaul)` | CATASTROPHIC | The main checkout. |
| `Bash(git worktree remove ../EviumOverhaul/)` | CATASTROPHIC | Trailing-slash variant. |
| `Bash(git worktree remove /*)` | CATASTROPHIC | Absolute paths. |
| `Bash(git worktree remove ~/*)` | CATASTROPHIC | Home paths. |
| `Bash(git worktree prune*)` | LOAD-BEARING | Prune cleans stale worktree refs. Mostly safe but could lose user side-session worktrees if their dir was moved/removed. **Keep.** |

**Verdict: 7 entries. ALL KEEP.**

### 6. `*--force*` blanket deny (FRICTION-OVERLOADED — recommend narrow allows)

| Entry | Category | Notes |
|---|---|---|
| `Bash(*--force*)` | FRICTION (overloaded) | Blanket-deny matches `git push --force` (catastrophic), `git worktree remove --force` (need this for scheduler change worktrees with applied diff), `npx playwright test --update-snapshots --force` (rare), etc. The blanket is too coarse. |

**Recommended specific allow (Phase 2 approval required):**

```jsonc
"Bash(git worktree remove --force ../EviumOverhaul-change-*)",
"Bash(git worktree remove --force ../EviumOverhaul-calibration-*)",
```

Justification: scheduler-owned worktrees need `--force` to clean (the applied diff makes them "modified" → plain `git worktree remove` fails). The allow is path-scoped to scheduler/calibration-owned patterns. The blanket `*--force*` deny still catches everything else (especially `git push --force`).

**Risk assessment:**
- Could `--force` here delete a side-agent worktree? No — pattern is specifically `EviumOverhaul-change-*` or `EviumOverhaul-calibration-*`. Bro's worktree at `.claude/worktrees/quirky-curie-bc56ce` doesn't match.
- Could `--force` on a change-* delete unsaved work? Only if the scheduler is running and someone manually put unsaved work in the worktree. The positive-ownership registry + 24h TTL mean only scheduler-created worktrees with expired TTL get removed.

**Verdict: 1 entry. KEEP blanket. ADD specific allows for the 2 scheduler-owned paths.**

### 7. CI hook bypass (KEEP)

| Entry | Category | Notes |
|---|---|---|
| `Bash(*--no-verify*)` | LOAD-BEARING | Skips commit hooks. We don't want agents bypassing pre-commit checks. **Keep.** |

**Verdict: 1 entry. KEEP.**

### 8. Snapshot regeneration (FRICTION — recommend narrow allow OR keep with workflow note)

| Entry | Category | Notes |
|---|---|---|
| `Bash(npm run test:e2e:snapshots*)` | FRICTION (real) | Blocks visual baseline regen. When CSS legitimately changes, agent can't update baselines — user must manually. |
| `Bash(npx playwright test*--update-snapshots*)` | FRICTION (real) | Same workflow gap, different invocation. |

**Two paths to consider (Phase 2 decision):**

**Path A — Keep both denied (load-bearing safety stance):**
Visual baselines are decisions, not computations. The user reviews + commits them. Agent never touches. The friction is intentional.

**Path B — Add tightly-scoped allow:**
```jsonc
"Bash(npm run test:e2e:snapshots -- --grep \"*\")",  // require an explicit --grep filter
```
This still requires the agent to name the specific tests being updated; can't blanket-regen.

**My recommendation:** Path A. Visual baselines are the kind of thing where "agent decided to regenerate" is suspicious. Keep the human-gated workflow. Friction is correct here.

**Verdict: 2 entries. KEEP both. Friction is intentional safety stance.**

### 9. External-exposure (NEVER LOOSEN)

| Entry | Category | Notes |
|---|---|---|
| `Bash(npm publish*)` | CATASTROPHIC | Publishes to npm registry — public + irreversible. **Keep.** |
| `Bash(curl -X DELETE*)` | CATASTROPHIC | Arbitrary DELETE to external APIs. **Keep.** |
| `Bash(npm uninstall*)` | DEFENSIVE | Could remove a critical dep + break the build. **Keep.** |

**Verdict: 3 entries. ALL KEEP.**

### 10. Mobile device state (KEEP — modifying physical-ish devices)

| Entry | Category | Notes |
|---|---|---|
| `Bash(~/...adb push*)` | DEFENSIVE | Pushes files to Android device/emulator. |
| `Bash(~/...adb install*)` | DEFENSIVE | Installs APK. |
| `Bash(~/...adb uninstall*)` | DEFENSIVE | Removes app from device. |
| `Bash(~/...adb shell rm*)` | DEFENSIVE | Removes files on device. |
| `Bash(xcrun simctl erase*)` | DEFENSIVE | Wipes iOS Sim. |
| `Bash(xcrun simctl delete*)` | DEFENSIVE | Deletes iOS Sim. |

**Verdict: 6 entries. ALL KEEP.** Device-state mutation is user's domain.

---

## Summary

| Category | Count | Action |
|---|---|---|
| CATASTROPHIC (never loosen) | 30 | Keep |
| LOAD-BEARING (keep with narrow allows where justified) | 11 | Keep |
| DEFENSIVE-PARANOID (cheap insurance) | 17 | Keep |
| FRICTION → recommend narrow allow | 4 | **Phase 2 candidate** |

**4 entries recommended for narrow allows (each is a SEPARATE Phase 2 decision):**

1. `Bash(git branch -d change-*)` — allow safe-delete on scheduler-created branches
2. `Bash(git branch -D change-*)` — allow force-delete on scheduler-created branches (safe because no unique commits)
3. `Bash(git worktree remove --force ../EviumOverhaul-change-*)` — allow scheduler cleanup of its own worktrees
4. `Bash(git worktree remove --force ../EviumOverhaul-calibration-*)` — same for calibration worktrees

**0 entries recommended for full removal.** The blanket denies stay; only narrow scoped allows added.

## Snapshot regen (separate decision)

`Bash(npm run test:e2e:snapshots*)` + `Bash(*--update-snapshots*)` — KEEP both. Visual baseline regen is intentional human-gated workflow. Not a real friction worth loosening.

## What this audit DID NOT do

- ❌ Modified settings.json
- ❌ Made any of the recommended changes
- ❌ Bundled multiple decisions

## What happens next (Phase 2 — user-gated)

For each of the 4 recommended allows:
1. User reads the justification + risk assessment above
2. User explicitly approves OR rejects per row
3. For approved entries: Phase 3 applies ONE at a time with backup + JSON validate + scheduler-tick verification + separate git commit per change

**Nothing happens automatically.** Each row is a separate yes/no from you.
