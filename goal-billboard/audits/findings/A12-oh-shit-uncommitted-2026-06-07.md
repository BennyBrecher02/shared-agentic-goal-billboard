---
audit: A12
title: "Oh-shit destructive-op risk to uncommitted/staged work — re-run"
date: 2026-06-07T19:29Z
scope: scripts/ + scripts/hooks/ + live daemons + settings.proposed→settings.json paste path + auto-syncs
mode: READ-ONLY (no edits, no spawn_task)
prior: A12-destructive-ops-audit.md · A12-oh-shit-2026-05-26-initial.md
risks_found: 7
critical_count: 0
status: PASS-WITH-GAPS
---

# A12 — "Oh-shit" destructive-op audit (uncommitted/staged work) · 2026-06-07

## Working-tree state at audit time
8 dirty paths: `.claude/settings.json`, `.claude/settings.proposed.json`,
`.claude/memory-mirror/feedback_commit-authorship.md` (a mirror — should not be hand-edited),
4 auto-modified logs/trackers under `context/markdowns/`, and 3 untracked research `.md`. No stash.
None of this is at risk from any **executed** destructive op found below (see verdict).

## Headline verdict
**No live/autonomous code path can `git checkout`/`reset --hard`/`clean`/`stash drop`/`rm -rf`
uncommitted or staged work.** The one genuinely dangerous primitive (`git apply --3way` into the
**real main working tree**) is double-opt-in AND fronted by a correct dirty-guard (A12-F1) that
refuses on any staged/unstaged/untracked collision. The 6 currently-loaded launchd daemons are all
read/consolidate/groom jobs with zero git-mutation or `rm -rf`. The settings.proposed→settings.json
flow CANNOT auto-clobber the live file (the sync only writes the inert shadow; the paste is a manual
user action). Auto memory-sync is one-way into the mirror and cannot touch canonical memory.

The findings below are **residual gaps / latent risks**, not active bleeding. Severity uses:
CRITICAL = can silently destroy uncommitted work on an autonomous/default path; HIGH = can destroy on
an opt-in or agent-driven path with a weak/missing guard; MEDIUM = recoverable loss or guard-not-wired;
LOW = hardening.

---

## What was scanned (coverage)
- **Destructive git verbs** (`checkout`/`restore`/`reset --hard`/`clean`/`stash drop|clear|pop`) across all of `scripts/`.
- **`rm -rf` / `rm -r` / `rm -f`** across all of `scripts/` (≈70 hits triaged).
- **Forced overwrite** (`cp -f`, `mv -f`, `>` redirection onto tracked/repo files).
- **Diff application** (`git apply`, `--3way`, `--reject`, `patch`).
- **The two named guards** — `git-checkout-safety.sh`, `wrapper-only-write-guard.sh` — read in full + wiring verified.
- **The settings.proposed→settings.json path**, `settings-edit-guard.sh`, `sync-memory.sh`.
- **The 6 LIVE launchd daemons** + `heartbeat-independent-tick.sh` + `agent-inbox-server.py` (live HTTP) — scanned for destructive ops.
- **Worktree removers** — `worktree-remove-safety.sh`, `prune-spawned-worktrees.sh`, `calibration-cleanup.sh`, `dispose.sh`, scheduler `_remove_worktree`.

## Guards verified SOUND (the good news, so the gaps below are read in context)
| Guard | State | Assessment |
|---|---|---|
| `scheduler.py:_apply_diff_to_main` A12-F1 dirty-guard | code-present | **Correct.** Before `git apply --3way` to main, `git status --porcelain` (staged+unstaged+untracked) of every diff-touched path; refuses on any collision; **fail-safe-to-dirty** on git/parse error. `apply_to_main_on_verify` AND `apply_over_dirty` both default `false` (double opt-in; confirmed by `test_tick_no_apply_to_main_when_flag_default` + `test_tick_apply_to_main_refuses_when_main_path_dirty`). |
| `settings-proposed-sync.sh` | wired (SessionStart `--notify`) | **Correct.** Writes ONLY the inert `.proposed` shadow, rebased on the CURRENT live file each run; validates valid-JSON + strict-superset (refuses to drop a live command) + every referenced script exists. Cannot touch live `settings.json`. Paste-over is a manual one-shot user action. |
| `sync-memory.sh` | wired (Stop) | **Correct.** One-way SRC(canonical)→DST(mirror), atomic `tmp.$$`+rename. Never writes back to canonical. (It WILL overwrite a hand-edited mirror — but that is policy, the mirror is non-canonical.) |
| `worktree-remove-safety.sh` | wired (PreToolUse Bash) | **Correct.** Whitelists only `../<PROJECT>-(calibration-…|change-…)`; refuses main/personal worktree removal via the tool. |
| `prune-spawned-worktrees.sh` | manual/ghost-reap | **Correct.** DRY-RUN default; `--apply` inert unless `EVIUM_PRUNE_APPLY_OK=1`; `assert_safe_to_prune` (namespace-prefix + name-shape regex); TTL gate (fresh worktrees never eligible); `has_real_work` without a recovered-sentinel is HELD; never `--force`. |
| `calibration-cleanup.sh` | manual | **Correct.** Layer-6 Trash-grace: `cp -R` worktree to `~/.Trash` and **aborts removal if the copy fails** — so a removal can't proceed without a recoverable copy. |
| `dispose.sh` | manual | **Correct.** Move-not-delete into gitignored `disposal-bin/`; never `rm`/`git rm`; recoverable until user empties. |
| `scrub-rehearsal.sh` rm | manual | **Correct.** `rm -rf "$TMP_ROOT"` is whitelisted to `/tmp`/`/var/folders`/`$TMPDIR` only; refuses any other path. |
| 6 live launchd daemons + heartbeat + inbox-server | loaded | **Clean.** Zero `git checkout/reset/clean/stash`, zero `rm -rf` of repo content, zero settings.json write. (consumers = memory-consolidator, audit-catalog-scan, steering-log-compact, bug-billboard-groom, heartbeat-tier-low; inbox-server only `tmp.unlink` on its own temp.) |

---

## Findings (residual gaps)

### F1 — `git-checkout-safety.sh` protects STAGED work only; UNSTAGED edits are unguarded — HIGH
**Where:** `scripts/hooks/git-checkout-safety.sh` (lines 52–57).
The guard fetches `git diff --cached --name-only` and **exits 0 (allow) when nothing is staged**. So
`git checkout HEAD -- <file>` / `git restore <file>` on a file with **unstaged but uncommitted edits**
(the common case — edits not yet `git add`-ed) sails straight through and silently discards them.
The 2026-05-26 P0-03 incident it was built for happened to be *staged*; the far more frequent shape
(dirty-but-unstaged) is exactly what this misses. `git restore <file>` with no `--staged` operates on
the **working tree** — precisely the unstaged content the guard ignores.
**Severity rationale:** agent-driven path, default-active guard gives a false sense of safety against
the most common loss shape. Not CRITICAL only because it requires the agent to issue an explicit
checkout/restore (no autonomous path does).
**Guardrail fix:** widen the at-risk set from staged-only to **dirty** — union `git diff --cached
--name-only` (staged) with `git diff --name-only` (unstaged). For `git checkout … -- <p>` and bare
`git restore <p>`/`git restore --worktree <p>`, block if `<p>` is in the unstaged set too (these touch
the worktree). Keep `--staged`-only restores gated on the staged set. Same `EVIUM_CHECKOUT_OK=1`
override; same stash-first guidance. (One-line essence: replace the `STAGED=$(git diff --cached
--name-only)` source with a `DIRTY=$( … cached … ; … unstaged … )` union and match against it.)

### F2 — `wrapper-only-write-guard.sh` is NOT wired (live or proposed) AND is WARN-only by design — MEDIUM
**Where:** guard file exists; `grep` of both `.claude/settings.json` and `.claude/settings.proposed.json`
finds **neither** a wiring nor the script; its dry-run ledger `.claude/cache/wrapper-only-write-guard.jsonl`
**does not exist** (proof it has never fired). The task names this as one of the two guards meant to
prevent the clobber class — it currently provides **zero** runtime protection.
Note this guard targets data.json/inbox **content-integrity** (A57/A48/A61), not uncommitted-work
clobber per se; but as a named guardrail it is dormant. Even if wired, its DEFAULT mode is WARN
(exit 0 always) — so the BLOCK is doubly inactive.
**Guardrail fix (user-gated — settings.json is GATED):** add the kill-switch-guarded line under the
existing `Write|Edit` PreToolUse group (the header documents the exact form). Run the documented 24h
dry-run, review the ledger for false positives, THEN flip `EVIUM_WRAPPER_GUARD_MODE=BLOCK`. Until then,
do not cite it as an active protection. (This wire is a candidate for `settings-patches/pending-wires.json`
so it lands in `.proposed` for a one-shot paste.)

### F3 — `git apply --3way` writes conflict markers into CLEAN tracked files on a near-miss — MEDIUM
**Where:** `scripts/scheduler/scheduler.py:_apply_diff_to_main` (lines 344–355) and the worktree apply
at line 573. When `--3way` returns "applied with conflicts", the function reports `ok:True, conflict:True`
and the patch **HAS landed** — conflict markers are written into the file. The A12-F1 dirty-guard
protects files that are *already dirty*, but a **clean, committed** file that the diff touches can be
left with `<<<<<<<` markers in the working tree. This is git-recoverable (`git checkout -- <file>`),
so not destruction of uncommitted work — but it silently dirties a previously-clean file and a
downstream auto-commit (if any) could capture the markers.
**Severity rationale:** recoverable (committed baseline exists); only reachable on the opt-in
apply-to-main path; conflict (not clean) outcome only.
**Guardrail fix:** on `conflict:True` from apply-to-main, emit the existing loud event AND (a) record
the touched paths so the user can `git checkout -- <paths>` to revert, or (b) prefer `git apply
--3way --check` first and on any conflict refuse-and-report rather than landing markers. At minimum,
document in the change schema that `apply_to_main_on_verify:true` can leave conflict markers and the
user must inspect before committing.

### F4 — `screenshot-janitor.sh` sweeps UNTRACKED root images to Trash on every SessionStart+Stop — LOW
**Where:** `scripts/hooks/screenshot-janitor.sh` (line 32, `mv -f "$f" "$TRASH/"`), wired SessionStart+Stop.
It correctly **skips git-tracked files** (`git ls-files --error-unmatch`), but any **untracked**
`*.png/jpg/jpeg/gif` a user intentionally dropped at repo root (an uncommitted asset, a reference image)
is moved to `~/.Trash` automatically, twice per session boundary, with no per-file confirmation.
**Severity rationale:** LOW — destination is `~/.Trash` (30-day recoverable), only root-level, only
untracked, and there is a kill-switch (`EVIUM_SCREENSHOT_JANITOR_OFF=1`). Still a silent auto-move of
uncommitted user files.
**Guardrail fix:** none strictly required given Trash-grace. Optional hardening: skip files newer than
N minutes (an asset just saved this session), or skip files matching a user-allowlist glob, so a
freshly-dropped intentional image isn't whisked away mid-session.

### F5 — `migrate-memory-on-move.sh` does `rm -rf "$NEW_MEM"` (variable root) — LOW
**Where:** `scripts/migrate-memory-on-move.sh:144` — `rm -rf "$NEW_MEM" 2>/dev/null || true`.
The inline comment asserts it is "safe: we already gated non-empty without --force", and `$NEW_MEM`
is derived from the project-config memory dir (not a tracked working-tree path). It is a manual
migration script, not on any daemon/hook path.
**Severity rationale:** LOW — manual-only, not repo working-tree content, gated upstream. Flagged for
completeness because it is the one `rm -rf` of a *non-temp* variable root.
**Guardrail fix:** add a defensive prefix assertion before the `rm -rf` (refuse unless `$NEW_MEM`
matches `*/.claude/projects/*/memory` and is non-symlink), mirroring `scrub-rehearsal.sh`'s
whitelist-or-refuse pattern, so a mis-derived `$NEW_MEM` can never expand to something broad.

### F6 — `hook-command-lint.sh` deny-list (reset/clean/rm-rf/force/push) is NOT wired as a runtime gate — MEDIUM
**Where:** `scripts/hooks/hook-command-lint.sh:106` carries a strong `DENY_RE` (blocks `rm -rf`,
`git reset --hard`, `git clean`, `--force`, `git push`, `launchctl load/unload`, settings.json
redirection, etc.). `grep` of `.claude/settings.json` finds **no wiring** — it is not a PreToolUse(Bash)
gate. So the deny-list that *would* catch an agent typing `git reset --hard` / `git clean -fd` at the
Bash tool is not enforcing on the Bash tool path (it appears intended to lint hook *command strings*,
not live Bash). The net effect: there is **no broad deny-list** intercepting destructive git verbs
typed directly into Bash — only the narrow `git-checkout-safety` (checkout/restore) and
`worktree-remove-safety` (worktree remove) guards exist.
**Severity rationale:** MEDIUM — `reset --hard`/`clean -fd` from an agent Bash call is unguarded.
Mitigated in practice because the environment's own sandbox already prompts on commands containing
`--force` (observed this audit: a grep with `--force` in the pattern was denied), but that is the
harness sandbox, not a repo guard, and does not cover `git reset --hard`/`git clean` without `-f` in
the literal.
**Guardrail fix:** add a thin PreToolUse(Bash) guard (or reuse the `hook-command-lint` DENY_RE) that
refuses `git reset --hard`, `git checkout -f`, and `git clean -[fdx]` when the working tree is dirty,
with an `EVIUM_*_OK=1` override. User-gated (settings.json). Stage via `pending-wires.json` → `.proposed`.

### F7 — `settings-edit-guard.sh` is WARN-only AND now ALLOWS agent edits to settings.json for hook-wiring — LOW/INFO
**Where:** `scripts/hooks/settings-edit-guard.sh` (line 57, `MODE="${EVIUM_SETTINGS_GUARD_MODE:-WARN}"`).
Per its Mechanism-(3) rewrite, an agent CAN now Write/Edit `settings.json` for **non-permission**
(hook-wiring) changes; only **permission-keyword** edits would-block — and even those only in BLOCK
mode, which is NOT the default (ledger `.claude/cache/settings-edit-guard.jsonl` exists with 13 dry-run
rows, confirming WARN-active). This is by design (the 22-parked-patches bottleneck), and `settings.json`
is GATED by policy regardless. Noted because the audit asks whether the settings path can be clobbered:
an **agent** could overwrite `settings.json` content via Write in WARN mode (recoverable via git; the
file is tracked). No *automation* does this.
**Severity rationale:** LOW/INFO — recoverable (tracked), intentional design, policy-gated.
**Guardrail fix:** none beyond the documented user-gated flip to `EVIUM_SETTINGS_GUARD_MODE=BLOCK`
after dry-run review. Keep relying on the GATED-by-policy discipline + git history for recovery.

---

## Cross-check against the named concerns in the task
- **"pending settings.proposed→settings.json paste could overwrite uncommitted work"** → **No.** The
  sync writes only the inert shadow; the paste is a manual user action; the shadow is rebased on the
  CURRENT live file every run and validated strict-superset, so it cannot silently drop live content.
  The only "loss" vector is the user themselves pasting — outside automation scope. (Tangential: the
  live `settings.json` + `.proposed` are BOTH dirty right now; pasting `.proposed` over `settings.json`
  would discard the live file's uncommitted edits IF they aren't already reflected in `.proposed` —
  but the sync rebases on live, so a fresh `--notify`/regen captures them. Recommend regenerating
  `.proposed` immediately before any paste so it carries the current live edits.)
- **"any auto-sync could overwrite uncommitted work"** → memory-sync is one-way into the (non-canonical)
  mirror; proposed-sync writes only the shadow. Neither overwrites canonical/working content.
- **git-checkout-safety + wrapper-only-write-guard "present + sound"** → checkout-safety present + wired
  but **staged-only (F1)**; wrapper-guard **not wired + WARN-only (F2)**.

## Net assessment
The autonomous/daemon surface is clean and the highest-value clobber target (apply-to-main) is
well-guarded and double-opt-in. The real residual exposure is **agent-issued** destructive git on the
Bash tool: `git restore`/`checkout` against *unstaged* edits (F1, the one to fix first) and the absence
of a wired deny-list for `reset --hard`/`clean` (F6). Both are user-gated settings.json wires.

BG-COMPLETE-SENTINEL: (7, 0, PASS-WITH-GAPS)
