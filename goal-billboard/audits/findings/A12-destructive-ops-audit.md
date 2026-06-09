# A12 — Oh-shit audit: destructive operations on uncommitted / staged / untracked work

**Type:** READ-ONLY research audit (system-protection assessment). No files mutated except this findings doc. No git/server/delete ops run.
**Date:** 2026-05-28 (evening; UTC timestamps in body)
**Scope:** Every path by which uncommitted / staged / untracked work could be destroyed, and what currently protects it.
**Method:** Inspected `git reflog`, `git config` gc settings, `.gitignore`, `.claude/settings.json` (full allow/deny), `scripts/hooks/{git-checkout-safety,worktree-remove-safety,src-edit-guard,settings-edit-guard}.sh`, the blob + its copies (md5), `scripts/timelapse/generate.py`, `scripts/serve-dashboard.sh`, snapshot baselines, worktree list.

---

## EXECUTIVE SUMMARY — the headline corrections

The audit prompt's two stated premises are **both now partly STALE** (the situation is safer than feared), but **new gaps** exist that the premises didn't anticipate:

1. **"37 unpushed commits, irrecoverable if reset."** → The 37 commits are **strongly recoverable**: all 37 are in `git reflog` (106 entries; oldest 2026-05-21), and `git reset --hard` / `git clean` / `git restore` / `git checkout --` / `git branch -D` / `git update-ref -d` / `git reflog expire`-adjacent ops are **all denied** in settings.json. Default gc reflog expiry is 90 days (reachable) / 30 days (unreachable). **Residual risk: the commits are NOT on `origin` — `origin/main` is at `621ba18`, far behind `HEAD=2472a51`.** Reflog is local-and-machine-only. A disk-loss / repo-corruption / `.git` deletion event = total loss of all 37. The single biggest real exposure is **not a git command — it's that the work has never left this one machine.**

2. **"Un-regenerable dashboard blob's only safety net is the untracked backup."** → **STALE.** The backup at `context/markdowns/dashboard-source-backup/operational-dashboard-2026-05-29.html` is now **TRACKED and committed** (commit `52991a8`, "BACK UP the irreplaceable dashboard blob"). The blob now has **FOUR copies**, three byte-identical (md5 `adf4e2d2…`): the live blob (`reports/timelapse/2026-05-25-overnight/index.html`), the tracked backup, and `dist/audit-timelapse/2026-05-25-overnight/index.html`. A fourth, *different/older* copy exists at `reports/timelapse-archive/2026-05-25-overnight/dashboard/index.html` (md5 `9a316608…`). **The blob is well-protected now** — the tracked backup rides into every commit + (when pushed) the remote. The real fix is still pending: give the operational view a real generator (A78 wave 2).

**Net posture:** The git layer is **hardened** (deny list is thorough; two relevant PreToolUse hooks are wired). The filesystem-overwrite layer (`>`, `cp`, `mv`, single-file `rm`) is **largely unprotected** but lower-likelihood. The **off-machine durability gap** (no push, reflog-only) is the top-ranked finding because its blast radius is *everything* and it's the one class no local hook can mitigate.

---

## RANKED FINDINGS

Ranking = (blast radius × likelihood-in-normal-agent-op) ÷ current-protection. Highest exposure first.

---

### F1 — [HIGH] 37 commits + ALL gitignored work live on ONE machine only (no push, reflog is local)

- **Exposure:** `origin/main` = `621ba18`; `HEAD` = `2472a51`; **37 commits ahead, none on the remote.** Every gitignored-but-precious asset (the 1.9 GB `reports/` tree, 2.0 GB client Drive export, audit findings, baselines) exists nowhere but this disk. Triggers: disk failure, accidental `.git` corruption, OS reinstall, `rm -rf` on the repo root from *outside* the agent (user shell), filesystem-level loss. The reflog that protects the 37 commits (F2) is itself stored in `.git/logs/` on this same disk — it does not survive disk loss.
- **Current protection:** None at this layer. `git push` is **denied** in settings.json (line 523) — correct for agent safety, but it means the agent *cannot* be the one to close this gap; only the user can push. There is no off-machine mirror, no bundle backup, no second clone.
- **Gap:** Single point of failure for 100% of the work product. This is the one destructive-op class where *no local hook can help* — recovery requires a copy that already left the machine.
- **Recommendation (PROPOSE, do not install):**
  - **(a)** User pushes `main` to `origin` at end of each working block. This alone closes the 37-commit exposure (commits become remote-recoverable). Since `git push*` is agent-denied, surface a Stop-hook *reminder* (warning-only, not a push) when `rev-list origin/main..HEAD` exceeds a threshold (e.g. ≥10). A `git-drift-warn.sh` already runs at SessionStart — extend it (or add a Stop sibling) to count unpushed commits and nag.
  - **(b)** For the gitignored precious set (reports/, audit-findings/, client Drive), propose a periodic `git bundle create` or `rsync` to an external/cloud location. These are too large/ephemeral to track, so a bundle/rsync mirror is the right tool. PROPOSE only — user owns the destination + cadence.

---

### F2 — [LOW, well-covered] `git reset --hard`, `git clean -fd`, `git checkout -- .`, `git restore`, `git branch -D`, `git update-ref -d`

This is the class the prompt feared most. It is **the best-protected class.**

| Op | Would destroy… | Denied in settings.json? |
|---|---|---|
| `git reset --hard <ref>` | uncommitted + staged changes (NOT committed history — commits survive in reflog) | **YES** — `Bash(git reset --hard*)` (line 534) |
| `git clean -fd` | **untracked files** incl. the blob, all of `reports/` | **YES** — `Bash(git clean*)` (553) AND `Bash(*--force*)` substring (522) catches `-f`/`--force` |
| `git checkout -- .` / `git checkout HEAD -- f` | uncommitted + staged changes to those paths | **YES** — `Bash(git checkout --*)` (535) + **PreToolUse hook** `git-checkout-safety.sh` (wired, settings line 159) |
| `git restore [--staged] f` | uncommitted/staged changes | **YES** — `Bash(git restore*)` (533) + same hook |
| `git stash drop` / `clear` | stashed work (1 stash exists: `g2-patch-test-tmp`) | **YES** — `Bash(git stash drop*)` (554), `Bash(git stash clear*)` (555) |
| `git branch -D <b>` | unmerged branch commits (recoverable via reflog) | **YES** — `Bash(git branch -D*)` (537) — note: also in *allow* list for `change-*`, but **deny wins** in Claude Code precedence, so even `change-*` deletes are blocked |
| `git update-ref -d` / `git tag -d` | refs (commits survive in reflog until expiry) | **YES** — lines 556–557 |
| `git push --force` / `--force-with-lease` | remote history | **YES** — `git push*` (523) + `*--force*` (522), double-covered |
| `git worktree prune` / `remove` | worktree (commits pinned by worktree HEAD survive in main repo) | **YES** — prune denied (552); remove gated to calibration/change patterns by both deny rules AND `worktree-remove-safety.sh` hook (wired, line 151) |

- **Exposure:** An agent running any of the above during "clean up the tree" or "revert this." Historically real: `git-checkout-safety.sh`'s header documents the **2026-05-26T15:44Z incident** where an agent reverted `src/styles/app.css` to HEAD and wiped a staged P0-03 fix (recovered only by luck — a `/tmp` patch existed). That incident is what motivated the hook.
- **Current protection:** Belt **and** suspenders. The Bash deny list blocks the command string; two PreToolUse hooks (`git-checkout-safety.sh`, `worktree-remove-safety.sh`) provide a second semantic layer for the checkout/restore + worktree cases.
- **Recoverability of the 37 commits if HEAD were somehow reset:** **YES, for ~30–90 days.** All 37 are in reflog (`HEAD@{…}` entries through 2026-05-21). `git config` shows no overrides → defaults apply: `gc.reflogExpire`=90d (reachable), `gc.reflogExpireUnreachable`=30d, `gc.pruneExpire`=2w. After a hard reset the orphaned commits become "unreachable" → recoverable via `git reflog` / `git fsck --lost-found` for **30 days** before gc can prune. **Caveat:** this protection is local-disk-only (see F1).
- **Gap:** Two minor ones:
  1. **`git reset` (mixed/soft, no `--hard`) is NOT denied.** This is *correct* — mixed/soft reset only moves HEAD/index and preserves the working tree; commits stay in reflog. No action needed, but noting it so the deny list isn't "tightened" by mistake.
  2. **`git reset --keep` / `--merge` not explicitly denied.** `--keep` is actually *safer* than `--hard` (it aborts if local changes would be lost); `--merge` similar. Low risk. Could add for completeness but not urgent.
  3. **Doc-staleness bug:** `git-checkout-safety.sh` line 17 says *"Not wired in settings.json yet"* — but it **IS** wired (settings line 159). Stale comment; harmless but should be corrected. (Flag, don't fix — read-only audit.)
- **Recommendation:** No new guard needed; this class is covered. PROPOSE only: (a) optionally add `git reset --keep*` / `git reset --merge*` to deny for symmetry; (b) correct the stale "not wired yet" comment in `git-checkout-safety.sh`.

---

### F3 — [MEDIUM] Filesystem overwrite of the precious blob via `>`, `cp`, `mv`, or single-file `rm` — NOT protected

The git layer is hardened, but the **plain-filesystem layer has holes** specifically around the irreplaceable, gitignored paths.

- **Exposure (each is reachable in normal agent op):**
  - **`> reports/timelapse/2026-05-25-overnight/index.html`** (truncate via redirect): **NOT denied.** There is no deny rule for bare output redirection. An agent doing `something > <that path>` (e.g. regenerating, or a typo'd output path) silently truncates the blob. *Mitigant:* three other copies exist (F2 table / summary), so a single truncation is recoverable from the tracked backup.
  - **`cp <x> reports/.../index.html`**: `Bash(cp -R*)` and `Bash(cp -r*)` are **ALLOWED** (lines 421–422). A `cp` onto the blob path overwrites it. *Mitigant:* same — tracked backup recovers it.
  - **`mv <x> reports/.../index.html`**: `Bash(mv*)` is **ALLOWED** (line 420; only `mv /*` and `mv ~/*` denied). An `mv` onto the path clobbers. Same mitigant.
  - **`rm reports/timelapse/2026-05-25-overnight/index.html`** (single-file, no flags): **ALLOWED** by `Bash(rm reports/*)` (line 429). Single-file `rm` of any report — including the live blob — is permitted. *Mitigant:* tracked backup + dist copy recover it. (Note `rm -rf`/`rm -r`/`rm -f` ARE denied — only flagless single-target `rm` of an allowed prefix slips through, which is the intended ergonomics for cleaning up stray captures.)
- **Current protection:** **Indirect only** — the multi-copy redundancy (F2) means the blob survives any *single* filesystem clobber. There is **no hook** guarding writes/overwrites to gitignored-precious paths, and no Trash-grace / safe-rm wrapper (`scripts/` has `bg-dispatch-wrapper.sh` but no `safe-rm`/`trash` wrapper).
- **Gap:** The protection is "we happened to make copies," not a guard. If the *next* irreplaceable artifact isn't manually backed up, the same vectors hit it with no safety net. The class is structural; the blob's safety is incidental.
- **Recommendation (PROPOSE):**
  - **(a)** A PreToolUse(Bash) **precious-path write-guard** hook: maintain a small allowlist-of-protected-globs (e.g. `reports/timelapse/*/index.html`, anything under `dashboard-source-backup/`) and WARN/BLOCK on `>`, `cp …<glob>`, `mv …<glob>`, `rm …<glob>` targeting them. Mirror the dry-run→BLOCK pattern of `src-edit-guard.sh`. High-risk to install (PreToolUse + Bash) → user-gated.
  - **(b)** Cheaper interim: make the **operational dashboard regenerable** (A78 wave 2 — give the 6 hand-authored tabs a real template/generator). Once regenerable, the blob stops being "irreplaceable source masquerading as an artifact" and this whole finding downgrades to LOW. This is the durable fix the backup README itself names.

---

### F4 — [MEDIUM] Tooling-triggered regen that DESTROYS hand-authored work — `generate.py` + `--update-snapshots`

Two tools overwrite hand-curated artifacts and are **explicitly allowed**.

- **F4a — `scripts/timelapse/generate.py`:**
  - **Exposure:** `generate.py` writes `reports/timelapse/{run-id}/index.html` (confirmed: docstring lines 9, 25, 470 — `(timelapse_root/"index.html").write_text(...)`). Per the backup README + A78-S4 F1: the operational dashboard is a **hand-authored 7-tab monolith** where generate.py only builds **1 of the 7 tabs**; the other 6 were hand-written and accreted A14→A48. **Re-running generate.py against run-id `2026-05-25-overnight` would overwrite the blob with a 1-tab regen, destroying the 6 operational tabs.**
  - **Current protection:** The tracked backup (F2) is the recovery path. `generate.py` invocation is not specifically denied (it'd run via `python3 scripts/timelapse/generate.py …`, which isn't in allow → hits default-ask, so it's at least not auto-run silently). No guard prevents pointing it at the operational run-id.
  - **Gap:** A "refresh the dashboard" instruction could trigger a destructive regen. Recovery exists (backup) but the foot-gun is live.
  - **Recommendation (PROPOSE):** (a) `serve-dashboard.sh` was inspected and is **safe** — it only *serves* (finds the largest index.html, `python3 -m http.server`), no regen. Keep it that way. (b) Make generate.py refuse (or `--force`-gate) writing over a run-id whose existing index.html is larger/multi-tab than its output — i.e. detect "I'm about to shrink a 7-tab dashboard to 1 tab" and abort. (c) Durable fix = same as F3(b): real generator for the operational tabs.

- **F4b — `--update-snapshots` (visual baseline regen):**
  - **Exposure:** `Bash(npx playwright test*--update-snapshots*)` (line 498) and `Bash(npm run test:e2e:snapshots*)` (line 497) are **ALLOWED**. There are **108 tracked baseline PNGs** under `tests/visual.spec.ts-snapshots/`. Running `--update-snapshots` overwrites all of them with whatever the page currently renders — if the current render has a regression, the regression becomes the new baseline (silent acceptance of a bug).
  - **Current protection:** Baselines are **tracked** → `git diff` shows the overwrite, and they're recoverable via `git checkout`. There's also a `snapshot-drift-pre.sh` (PreToolUse, line 147) and `snapshot-baseline-staleness-check.sh` (line 163). The drift-pre hook is the relevant guard.
  - **Gap:** Tracked → recoverable, so blast radius is bounded (unlike the blob). The risk is *silent baseline poisoning*, not data loss. Lower priority than F1–F3.
  - **Recommendation (PROPOSE):** Confirm `snapshot-drift-pre.sh` actually warns before `--update-snapshots` (it's wired; verify its logic covers this command — out of scope for this read-only pass). Optionally require regen to run on a clean tree (no uncommitted src changes) so the diff is reviewable.

---

### F5 — [LOW] `find -delete` / `find -exec rm` / `xargs rm` mass-deletion

- **Exposure:** Bulk deletion across the tree — could hit gitignored precious paths (reports/, client data) that git can't recover.
- **Current protection:** **Denied, thoroughly:** `Bash(find * -delete*)` (561), `Bash(find * -exec rm*)` (562), `Bash(find * -exec mv*)` (563), `Bash(find * -execdir rm*)` (564), `Bash(xargs rm*)` (565), `Bash(xargs mv*)` (566). Plus `Bash(rsync*--delete*)` (570). The leading-`*` `Bash(*--force*)` (522) also catches `--force` anywhere.
- **Gap:** The `find * …` patterns are prefix-anchored on `find ` — a `find` invocation whose deletion is expressed differently (e.g. `find . -name x | xargs rm` is caught by `xargs rm*`; but `find . -name x -ok rm {} ;` interactive, or piping to a non-xargs deleter) could in theory slip. Very low likelihood in agent op. Empirically confirmed during this audit: even an `echo` containing the string "rm " inside a compound command got blocked — substring matching is **aggressive**, erring safe.
- **Recommendation:** No action needed. Class is covered.

---

### F6 — [LOW–INFO] Untracked-specific: what is invisible to git, and its ONLY recovery path

Enumeration of gitignored-but-precious assets (git recovery does **not** apply — these never enter git history):

| Asset | Size | Gitignored by | ONLY recovery path |
|---|---|---|---|
| The dashboard blob `reports/timelapse/2026-05-25-overnight/index.html` | 769 KB | `.gitignore:46 /reports/` | **Tracked backup** `context/markdowns/dashboard-source-backup/…html` (committed `52991a8`) + `dist/` copy. **Well-covered.** |
| Entire `reports/` tree | **1.9 GB** | `.gitignore:46` | **None** beyond filesystem. Includes `phase-a-evidence` (13M), `evium-baseline` (4.1M), `audit-findings` (2.3M), `conversation-extracts` (1.4M), `g2-signoff` (52K). Ephemeral by design, but some (baselines, signoff evidence) are arguably durable. |
| `context/Evium-Charging-Google-Drive-AllAvailable-Data/` | **2.0 GB** | `.gitignore:66` | Re-downloadable from client Drive (the original source). Recoverable-in-principle, not local-recoverable. |
| `context/codebases/` | **3.9 GB** | `.gitignore:65` | Re-clonable reference material. Recoverable-in-principle. |
| `public/billboard.html`, dashboard symlinks | small | `.gitignore:79,85-92` | Regenerable (`render-billboard-page.py`, `ln -s`). Safe. |
| Agent-inbox `*.md` | small | `.gitignore:99-103` | Ephemeral by design (drained→archived). Acceptable. |

- **Key insight:** Of the gitignored set, only **the blob** was genuinely irreplaceable, and it has now been rescued into tracked storage. The two multi-GB trees are *reconstructable from external sources* (client Drive, upstream repos), so their loss is cost/time, not irreversibility. `reports/` is the gray area — mostly ephemeral, but baseline/signoff evidence inside it has no backup. **103 top-level ignored entries** total are invisible to git.
- **Gap:** No catalog distinguishes "ephemeral reports/ subdir (fine to lose)" from "durable evidence subdir (back this up)." If durable evidence accumulates under `reports/`, it silently has no safety net.
- **Recommendation (PROPOSE):** (a) Decide per-subtree which `reports/` children are durable; either promote those to a tracked location (like the blob was) or include in the F1(b) bundle/rsync mirror. (b) The blob's own durable fix (regenerable generator) makes its tracked backup moot — then drop the backup to avoid a stale-copy hazard (see F7).

---

### F7 — [INFO] Secondary hazards noticed during the audit

- **Stale-backup hazard (latent):** The tracked backup `operational-dashboard-2026-05-29.html` is currently **byte-identical** to the live blob (md5 verified `adf4e2d2…` for both, plus the `dist/` copy). But if the live blob is later hand-edited (it's hand-authored source — edits are expected), the backup goes stale and silently diverges. There's no sync/verify between them. The backup README acknowledges this is interim. **Recommendation:** until the regenerable-generator fix lands, add a cheap Stop-hook md5-compare that WARNs when live-blob ≠ tracked-backup (so a divergence surfaces instead of silently rotting). PROPOSE only.
- **`git branch -d/-D change-*` and `calibration-*` appear in BOTH allow and deny:** Claude Code precedence is **deny-wins**, so these are effectively blocked despite the allow entries (lines 449–450 allow; 542–545 deny). This means the documented dual-worktree/multi-change cleanup flow that *expects* to delete `change-*` branches will be **refused** by the agent. Not a data-loss risk (safe-side), but a **workflow friction / dead-allow-rule** worth reconciling. **Recommendation:** user decides whether change-branch deletes should be agent-allowed; if yes, remove the broad `git branch -D*`/`git branch -d*` deny and rely on the pattern-specific allows. Flag, don't change.
- **No `git gc` / `git reflog expire` / `git prune` in allow OR deny:** They hit default-ask (won't auto-run, won't auto-block). Acceptable — an agent can't silently prune the reflog that protects the 37 commits. Noting for completeness; could add to deny for defense-in-depth.
- **14 worktrees exist** (incl. 2 detached-HEAD `/tmp` worktrees at old commits `ad246df`, `7122425`, and 11 `.claude/worktrees/*`). Each detached worktree HEAD **pins** its commit against gc — a minor *positive* for recoverability, but the `.claude/worktrees/` count (gitignored) is disk pressure and a confounder if someone greps for "copies of generate.py" (10 stale copies showed up). Not destructive; informational.

---

## CROSS-CHECK: existing protections inventory (what's already wired)

**settings.json deny list (lines 513–579)** — comprehensive for git + bulk-fs destruction. Confirmed blocking: `rm -rf/-r/-f`, `rm /*`, `rm ~/*`, `rm ../*`, all history-rewriting git (`push/pull/fetch/merge/rebase/cherry-pick/revert/reset --hard/checkout --/restore/switch/rm`), `git clean`, `git stash drop/clear`, `git branch -d/-D/-m/-M/-c/-C`, `git tag -d`, `git update-ref -d`, `git worktree prune` + dangerous remove targets, `find * -delete/-exec rm/mv`, `xargs rm/mv`, `rsync --delete`, `*--force*`, `*--no-verify*`, adb push/install/rm, simctl erase/delete, npm publish/uninstall.

**PreToolUse hooks wired** (settings.json `PreToolUse`):
- `git-checkout-safety.sh` (line 159) — blocks checkout/restore on staged files. **Wired** (despite its own stale "not wired yet" comment).
- `worktree-remove-safety.sh` (line 151) — restricts worktree removes to calibration/change patterns.
- `settings-edit-guard.sh` (line 204) — blocks Write/Edit to settings.json (closes the Bash-deny-doesn't-cover-Write gap).
- `src-edit-guard.sh` (line 208) — WARN-mode gate on src/** edits (A66 high-risk list item #1).
- `snapshot-drift-pre.sh` (147), `snapshot-baseline-staleness-check.sh` (163) — snapshot-regen guards.

**A66 high-risk-always-gated list** (from memory `feedback_decision-handling-discipline`): src/ edits, settings.json, remote push, snapshot regen, LaunchAgent, mass deletes, chamber bypass, settings hooks deploy, memory-rule deletion. The git destructive class (F2) maps onto "mass deletes" + is independently deny-listed. **The gap A66 doesn't name: plain-filesystem overwrite of gitignored-precious paths (F3) and tooling-regen-destroys-handwork (F4a).**

**No Trash-grace / safe-rm wrapper** exists (`scripts/` has `bg-dispatch-wrapper.sh`, no safe-delete wrapper). The "Trash grace" mentioned in the quality-discipline skill refers to worktree-removal architecture, not a general rm-to-Trash shim.

---

## ALL DESTRUCTIVE-OP CLASSES — assessment matrix (exit-criterion table)

| # | Class | Destroys 37 commits? | Destroys uncommitted/staged? | Destroys untracked blob? | Current protection | Gap | Rank |
|---|---|---|---|---|---|---|---|
| F2 | `git reset --hard` | No (reflog 30–90d) | **Yes** if run | No (doesn't touch untracked) | Deny + hooks | none material | LOW ✅ |
| F2 | `git clean -fd` | No | No | **Yes** | Deny (`git clean*` + `*--force*`) | none | LOW ✅ |
| F2 | `git checkout/restore -- .` | No | **Yes** if run | No | Deny + `git-checkout-safety.sh` | stale comment only | LOW ✅ |
| F2 | `git stash drop/clear` | No | stashed work | No | Deny | none | LOW ✅ |
| F2 | `git branch -D` / `update-ref -d` | recoverable (reflog) | n/a | No | Deny | dead allow-rules (F7) | LOW ✅ |
| F1 | disk loss / no off-machine copy | **YES, total** | **YES** | **YES** | **none** (push denied; reflog local) | the big one | **HIGH** |
| F3 | `>` truncate blob | No | No | **Yes** (but 3 copies) | none (multi-copy incidental) | no path-guard | MED |
| F3 | `cp`/`mv` over blob | No | No | **Yes** (but 3 copies) | none (allowed ops) | no path-guard | MED |
| F3 | single-file `rm reports/x` | No | No | **Yes** (but 3 copies) | none (`rm reports/*` allowed) | no path-guard | MED |
| F4a | re-run `generate.py` | No | No | **destroys 6 hand tabs** | tracked backup | foot-gun live | MED |
| F4b | `--update-snapshots` | No | No | n/a (tracked PNGs) | tracked + drift-pre hook | silent poison | LOW–MED |
| F5 | `find -delete`/`xargs rm` | No | No | **Yes** if run | Deny (thorough) | negligible | LOW ✅ |
| F6 | untracked invisibility | n/a | n/a | inherent | per-asset (table above) | reports/ durable subset | LOW–INFO |

---

## TOP 3 RECOMMENDATIONS (all PROPOSE — none installed; settings.json + hooks are high-risk-gated)

1. **Close the off-machine gap (F1).** User pushes the 37 commits to `origin` (agent can't — `git push` denied). Add a Stop/SessionStart **reminder** that counts `origin/main..HEAD` and nags past a threshold. Add a bundle/rsync mirror plan for the gitignored multi-GB precious set. *This is the only class with total blast radius and zero local mitigation.*

2. **Make the operational dashboard regenerable (F3b/F4a/F6/F7 all collapse).** The single durable fix that retires the "irreplaceable blob" status, the stale-backup hazard, AND the generate.py foot-gun. Named by the backup README as A78 wave 2. Highest leverage structural fix.

3. **Propose a precious-path write-guard hook (F3).** PreToolUse(Bash) WARN→BLOCK on `>`/`cp`/`mv`/`rm` targeting a small protected-glob list (dashboard blob, dashboard-source-backup/). Mirrors `src-edit-guard.sh`'s dry-run pattern. Turns the blob's *incidental* multi-copy safety into a *structural* guard for the next irreplaceable artifact too.

**Minor flags (read-only — not fixed here):** stale "not wired yet" comment in `git-checkout-safety.sh` (it IS wired); dead/contradicted `git branch -d/-D change-*` allow-rules (deny wins); add `git reset --keep/--merge` + `git gc`/`reflog expire` to deny for symmetry/defense-in-depth.

BG-COMPLETE-SENTINEL
