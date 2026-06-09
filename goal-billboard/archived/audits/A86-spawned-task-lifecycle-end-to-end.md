---
id: A86
title: Spawned-task lifecycle end-to-end (safe-to-start / report-back / self-clean)
type: audit
status: completed
created: 2026-05-30T23:40:00Z
author: claude
severity: HIGH
disposition: read-only — recommend only; NOTHING applied
gated_items: 3 (settings.json SessionStart wire + Stop wire + a single worktree-prune allow-rule)
ungated_items: 6 (all buildable by Claude without touching settings.json/src)
cross_refs:
  - context/markdowns/research/systems/concurrency-cost-and-suggested-task-safety.md  (Part B — the SST contract)
  - context/markdowns/MASTER-REALIGNMENT-MANIFESTO.md §6  (the repeated ask + its 3 requirements)
  - .claude/memory/feedback_bg-lifecycle-discipline.md  (A47 — sentinel / ghost-reap / dispatch-bg wrapper)
  - .claude/memory/feedback_decision-handling-discipline.md  (A66 — high-risk-always-gated: mass-delete / worktree-remove)
  - scripts/hooks/bg-auto-stop.sh  (the dry-run-first ghost-reap pattern this audit reuses)
  - context/markdowns/goal-billboard/audits/A85-settings-hook-hygiene-audit.md (F4 — Python git-worktree-remove bypasses the Bash deny-list/hook)
---

# A86 — Spawned-task lifecycle, end-to-end

> **One-line verdict.** The three requirements the user has asked for 30+ times are **each ~60–70 % built and ~0 % wired.** Every load-bearing script EXISTS (harvest, the full Safe-Suggested-Task Contract, the removal guards) but **NOT ONE is connected to a hook**, and **NOTHING auto-prunes a spawned worktree** — by design Claude is walled off from every prune path. The result is exactly the lived symptom: 15 accumulated worktrees, 6 with genuinely-stranded work, zero auto-surfacing, zero auto-clean. This is the textbook **"build-but-don't-surface / async-decoupling"** disease the manifesto §6 names.

---

## 0. What the user asked for (the recurring, frustrated ask)

A clicked suggested-task chip opens a **new Claude session + a new git worktree**. The user wants that spawn to be **truly end-to-end self-managing, every time, zero babysitting**:

1. **SAFE-TO-START-ANYTIME** — clicking the popup is 100 % safe even days later, after the repo moved on.
2. **REPORT-BACK / RECOVER** — its output reliably gets back to the main session; never silently stranded.
3. **SELF-CLEAN** — spawned worktrees self-prune; no piling-up duplicates (~15 accumulated).

Source of the ask: `MASTER-REALIGNMENT-MANIFESTO.md` §6 (lines 105–112), STATUS still **PENDING**. The trigger incident: an overnight 5hr-limit halt cancelled a "Start locally" task while the user slept.

---

## 1. HARNESS REALITY (what can vs. cannot be controlled from inside this system)

Measured this session — **this is the ground truth the whole plan must respect:**

- **15 spawned worktrees** live at `.claude/worktrees/<random-name>/` (e.g. `amazing-moore-b01e16`), each on a branch named **`claude/<name>`**. These are created by the **Claude Code harness itself** when a chip is clicked — NOT by our scheduler.
- The scheduler's positive-ownership registry (`.claude/cache/scheduler-owned-worktrees.json`) currently has **0 entries**. The spawned worktrees are **completely invisible** to the scheduler's ownership model.
- A clicked chip spawns a **separate session in a separate worktree with NO automatic callback** to the main session. The main session is never notified the spawn finished; output lands **uncommitted** in the worktree and stays there.
- **What we CAN control from inside:** anything that runs in OUR session's hooks (Stop / SessionStart / PreToolUse) and any script we invoke. We can *scan* the worktrees dir, *classify* what's stranded, *recover* files into main, and *prune* worktrees **if permissions allow**.
- **What we CANNOT control:** the spawned session's own behavior (it doesn't run our Stop hook against the main repo's substrate unless it shares the same `.claude/settings.json` — it does, but it acts on its own worktree), and we cannot make the harness emit a completion callback. **Recovery must therefore be PULL-based from the main session, not PUSH-based from the spawn.**

> **Design consequence:** the only reliable, structural surface we own is **the main session's SessionStart + Stop hooks**. Everything self-managing must hang off those two events (plus the optional launchd daemon already present for research-furthering, which could host a poll). A spawn cannot be trusted to report itself — the main session must *go look*.

---

## 2. STAGE-BY-STAGE ASSESSMENT (brutally honest: built-but-not-wired vs. genuinely-missing)

### Stage 1 — SAFE-TO-START-ANYTIME  → **BUILT, NOT WIRED**

The Safe-Suggested-Task Contract (SST-1..SST-5) is **fully implemented in scripts:**

| Piece | File | State |
|---|---|---|
| State engine (8-state machine, disk-artifact-is-truth, dismissed-is-final, at-most-once claim) | `scripts/suggested-task-state.py` (27 KB) | **BUILT** ✓ |
| Click-time precondition revalidation (HEAD-drift + window + own-artifact + sibling-in-flight; refuses cleanly) | `scripts/suggested-task-revalidate.sh` | **BUILT** ✓ |
| Verdict classifier (`suggested-pending` / `dismissed` / `completed` w/ the popup≠progress guard) | `scripts/suggested-task-verdict.sh` | **BUILT** ✓ |

**The defect:** `grep -nE 'suggested-task|revalidate|verdict' .claude/settings.json` returns **NOTHING**. None of these run from any hook. Concretely:
- **No PreToolUse / Stop hook records a `suggested` prelaunch row** when a chip is surfaced (SST-5 says "write the row at suggest-time"). So a never-clicked task isn't even in the substrate.
- **No hook runs `suggested-task-revalidate.sh` at click-time.** The spawned session does not call it before doing work. The safety is therefore **theoretical** — the gate exists but nothing forces a clicked task through it.
- **No poll runs `suggested-task-verdict.sh`** to keep the `suggested-pending` vs `dismissed` distinction live (SST-5 polling).

**Verdict:** the safety is **not live.** A days-later click today does NOT pass through SST-1 — it just runs in a fresh worktree against whatever HEAD the harness checked out, with no precondition revalidation and no claim marker. (The contract's *design* makes the halt a non-event — but only once the gate is actually in the click path.) See F1.

### Stage 2 — REPORT-BACK / RECOVER  → **BUILT, NOT WIRED (100 % manual today)**

`scripts/harvest-spawned-tasks.sh` (12 KB) is a **complete, correct, read-only** classifier. Run live this session it produced:

```
scanned 15 worktree(s): 6 with real work · 8 noise-only · 1 clean
⚠ GENUINE STRANDED WORK (recover before pruning):
  ▸ confident-lamport-bb6058  scripts/hooks/sessionstart-services-digest.sh, scripts/daemons/reload-all.sh
  ▸ cool-rhodes-d0d97c        scripts/hooks/sessionstart-services-digest.sh
  ▸ elated-ramanujan-f4d7c3   scripts/compute-matrix-stats.sh
  ▸ epic-zhukovsky-d43d09     scripts/fix-frontmatter-yaml.py, scripts/lint-frontmatter-yaml.py
  ▸ nice-grothendieck-c84212  .gitignore, scripts/post-audit-timelapse.sh, scripts/setup-dashboard-symlinks.sh
  ▸ priceless-dhawan-35f3dc   scripts/bg-dispatch-log-gc.sh + plan
```

It correctly filters auto-gen churn (script-candidates.md, matrix-runs.md, goal-billboard/*, memory-mirror/*, __pycache__, *.pyc, .DS_Store) and is fail-safe (unrecognized paths default to REAL WORK, never silently hidden). It has `--json` and `--quiet` modes purpose-built for a hook.

**The defect:** `grep harvest .claude/settings.json` matches only `research-cross-link-harvest.sh` — **`harvest-spawned-tasks.sh` is NOT in any SessionStart entry.** A staged patch sits unused at `scripts/hooks/settings-patches/harvest-spawned-tasks-wire.json`. **Nothing auto-surfaces stranded spawned-task output. It is 100 % manual** — the user (or Claude) must remember to run the script by hand, which is exactly the babysitting the user is frustrated by. **6 real-work worktrees are stranded right now and nothing is telling anyone.** See F2.

### Stage 3 — SELF-CLEAN (auto-prune)  → **GENUINELY MISSING (and Claude is walled off from building it ungated)**

There is **NO mechanism that auto-prunes a spawned worktree** once its work is recovered or deemed noise. Confirmed from two angles:

**(a) The existing removal guards cannot see spawned worktrees.** Both guards key on the *scheduler's* naming pattern, which spawned worktrees do not match:
- `scripts/scheduler/worktree.py::assert_safe_to_remove` — regex `_OWNED_WT_PATH_RE = .*/EviumOverhaul-(change|calibration)-<id>$`. A path like `.claude/worktrees/amazing-moore-b01e16` **fails the regex** → `UnsafeWorktreeRemoval`. It also checks the registry (`is_owned`), which has 0 entries → fails again. The scheduler's `cleanup_expired_worktrees` iterates **only the registry**, so it structurally never touches a spawned worktree.
- `scripts/hooks/worktree-remove-safety.sh` (wired at settings.json:151) — same regex `^\.\./${_PROJECT_NAME}-(calibration-…|change-…)$`. A `git worktree remove .claude/worktrees/...` command **fails the allow-pattern → exit 2 (BLOCK).** So the wired hook actively *refuses* to remove a spawned worktree.

  > These guards are doing their job for the scheduler — they are not "broken." They simply have **no jurisdiction over the spawned-worktree namespace.** A spawned-worktree pruner is a *separate* tool with its own (stricter) safety, not an extension of these.

**(b) Permissions wall Claude off from EVERY prune path** (`permissions.deny` in settings.json):
- `Bash(git worktree prune*)` — **denied.**
- `Bash(*--force*)` — **denied** → `git worktree remove --force .claude/worktrees/...` blocked.
- `Bash(git rm*)`, `Bash(rm -rf*)`, `Bash(rm -r*)`, `Bash(rm -f*)` — **all denied** → can't `rm` the dir.
- `permissions.allow` contains **only** `git worktree remove ../EviumOverhaul-(change|calibration)-*` — there is **no allow rule for `.claude/worktrees/*`.**

**Why prune is gated (this is correct, not an oversight):** worktree-removal / mass-delete is on the **A66 high-risk-always-gated list** (`feedback_decision-handling-discipline.md`): src / settings / push / launchd / mass-delete are USER-gated by policy. A85 F4 separately showed Python's `subprocess` `git worktree remove` *bypasses* the Bash deny-list + hook, which is exactly why the in-Python `assert_safe_to_remove` guard was added — so a pruner must NOT become a deny-list-bypass hole. **Auto-prune of spawned worktrees is genuinely missing AND cannot be fully shipped ungated** — the actual `git worktree remove` step needs either a user-approved allow-rule or a user-run apply. See F3.

### Stage 4 — The missing PIPELINE that ties 1→2→3  → **GENUINELY MISSING**

Even with all three stages wired individually, there is **no orchestration** that runs `completion → harvest → recover-or-discard → prune` as one structural flow. Today each piece is a standalone script a human invokes. The self-managing promise requires a **single driver** (hook- or daemon-fired) that: detects a spawn reached a terminal state (artifact+sentinel, or stale/dismissed) → harvests its real work (or confirms noise) → recovers real files into a staging area for review → marks the worktree prune-eligible → (gated) prunes. See F4.

---

## 3. FINDINGS (severity · evidence · GATED-vs-ungated fix)

### F1 — SST contract is built but NOT in the click path → safe-to-start is theoretical · **SEV: HIGH**
**Evidence.** `suggested-task-state.py` / `-revalidate.sh` / `-verdict.sh` all exist and pass their own guards, but `grep -E 'suggested-task|revalidate|verdict' .claude/settings.json` → **empty.** No hook writes a `suggested` prelaunch row at suggest-time; no hook forces a clicked task through `revalidate` before it mutates; no poll keeps the verdict vocabulary live.
**Fix — UNGATED (Claude builds):**
- `scripts/hooks/suggested-task-prelaunch.sh` — a `PreToolUse` hook matching the spawn/`spawn_task` tool that calls `suggested-task-state.py suggest --task-id <toolUseId>` to write the prelaunch row. (Writes substrate only — no settings edit.)
- A self-contained **revalidation preamble** that spawned-session prompts can include verbatim: `bash scripts/suggested-task-revalidate.sh --task-id <id> --expect-head <sha> --artifact <path> || exit` — drop it into the `spawn_task` prompt template (we own prompt text).
**Fix — GATED (user approves the wire):** add the prelaunch hook + a verdict poll to settings.json (exact lines in §5).

### F2 — Harvest is built but NOT wired → stranded output is 100 % manual · **SEV: HIGH**
**Evidence.** `harvest-spawned-tasks.sh` works (shown: 6 stranded worktrees this session) and a wire patch is staged at `scripts/hooks/settings-patches/harvest-spawned-tasks-wire.json`, but it is **not present** in settings.json SessionStart. Nothing tells the main session that 6 worktrees hold real, unrecovered work.
**Fix — UNGATED (Claude builds):** nothing more to build — the script + `--quiet` mode + the staged patch already exist. (Optionally add a `--surface-md` mode that writes the report to a `reports/` file the dashboard can read, so surfacing doesn't depend solely on a hook line.)
**Fix — GATED (user approves the wire):** merge the one-line SessionStart entry (§5). This is the single highest-leverage flip in the whole audit.

### F3 — No auto-prune for spawned worktrees + Claude walled off from every prune path · **SEV: HIGH**
**Evidence.** `assert_safe_to_remove` regex + the wired `worktree-remove-safety.sh` regex both reject `.claude/worktrees/*`; registry has 0 entries; `git worktree prune*`, `*--force*`, `git rm*`, `rm -r*` all in `permissions.deny`; allow-list has no `.claude/worktrees/*` rule. 15 worktrees, 8 noise-only, oldest 5 days old, none pruned.
**Fix — UNGATED (Claude builds):** `scripts/prune-spawned-worktrees.sh` — a **dry-run-first, recovered-or-noise-gated** pruner modeled exactly on `bg-auto-stop.sh`'s Phase-1 discipline:
  - eligible **only** if: (a) the worktree is `noise-only` or `clean` per harvest, **OR** its real work has a recorded `recovered` sentinel (see F4); **and** (b) path matches `^.../.claude/worktrees/[a-z]+-[a-z]+-[0-9a-f]{6}$` (the harness naming shape — a NEW, separate safety regex, not the scheduler's); **and** (c) older than a TTL (e.g. 24 h since last commit).
  - **Default mode = DRY-RUN:** writes `reports/spawned-worktree-prune-dry-run.jsonl` (which worktrees it *would* prune + why) and surfaces a count. **Kills nothing.** This is fully buildable + safe + structural — it gives the user the exact prune list to approve.
**Fix — GATED (user approves — pick ONE):**
  - **Option A (preferred, minimal):** add allow-rule `Bash(git worktree remove .claude/worktrees/*)` to `permissions.allow` AND a companion deny-guard hook so removal only fires on noise/recovered+TTL worktrees. Then `prune-spawned-worktrees.sh --apply` can run from a hook.
  - **Option B (no settings change):** keep prune fully manual — the dry-run JSONL becomes a copy-paste list the **user** runs. Self-surfacing, not self-cleaning.
  - Either way the `--apply` path must route through an in-Python/in-script `assert_safe_to_remove`-style guard so it can NEVER become an A85-F4-style deny-list bypass.

### F4 — No completion→harvest→recover→prune PIPELINE → pieces don't compose into "self-managing" · **SEV: MEDIUM**
**Evidence.** Each stage is a standalone manual script; nothing fires them as one flow; "recovered" has no recorded marker so prune can't know a worktree is safe to drop.
**Fix — UNGATED (Claude builds):**
- `scripts/spawned-task-recover.sh` — given a worktree with real work, `cp` each real file into a **staging dir** (`.claude/cache/spawned-recovered/<wt>/...`) for the user to review-and-apply (NOT auto-applied to src — A66/src is gated), then append a `recovered` sentinel to `reports/spawned-task-recovery.jsonl` keyed by worktree+commit. (Read-from-worktree + write-to-cache only; touches no tracked file.)
- `scripts/spawned-task-pipeline.sh` — the orchestrator: `harvest --json` → for each `has_real_work` call `recover` → for each `noise_only`/`clean`/recovered call `prune-spawned-worktrees.sh` (dry-run by default). One command = the whole lifecycle.
**Fix — GATED:** wiring the pipeline to fire automatically (SessionStart for surface+recover-to-staging; the prune `--apply` step inherits F3's gate).

### F5 — Spawned worktrees absent from the scheduler registry → ownership model has a blind namespace · **SEV: LOW (informational)**
**Evidence.** Registry has 0 entries; all 15 spawned branches are `claude/*`. The scheduler is correct to ignore them (it didn't create them), but it means **no existing inventory tracks spawned worktrees** — the harvest scan IS that inventory, which is another reason F2's wire matters.
**Fix — UNGATED:** have `harvest-spawned-tasks.sh --json` double as the spawned-worktree inventory the dashboard reads. No code change needed beyond F2.

### F6 — Noise-only worktrees re-classify as noise every session → churn the user can't see shrinking · **SEV: LOW**
**Evidence.** 8 noise-only worktrees, the oldest 5 days old, persist because nothing prunes them; each session re-reports them. Without F3's pruner the "safe-to-prune candidate" list only ever grows.
**Fix:** F3's dry-run pruner closes this directly once the user approves Option A, or via the manual Option-B list.

---

## 4. THE BUILD-READY PLAN (end-to-end self-managing system)

Goal: the lifecycle fires **every time via hooks, not by discipline.** Split exactly per the user's instruction.

### 4A. UNGATED — Claude can build now (no settings.json / src / git-rm / push / worktree-remove)

| # | Artifact | What it does | Reuses |
|---|---|---|---|
| U1 | `scripts/hooks/suggested-task-prelaunch.sh` | PreToolUse on spawn → `state.py suggest` writes the prelaunch row (SST-5). | suggested-task-state.py |
| U2 | Revalidation preamble snippet for the `spawn_task` prompt template | Forces a clicked spawn through SST-1 before it mutates (we own prompt text). | suggested-task-revalidate.sh |
| U3 | `scripts/spawned-task-recover.sh` | Copies real stranded files → `.claude/cache/spawned-recovered/<wt>/` staging + writes a `recovered` sentinel to `reports/spawned-task-recovery.jsonl`. Does NOT touch src. | harvest noise filter |
| U4 | `scripts/prune-spawned-worktrees.sh` (**DRY-RUN default**) | Classifies prune-eligibility (noise/clean OR recovered + harness-name-regex + TTL); writes `reports/spawned-worktree-prune-dry-run.jsonl`; kills nothing. | bg-auto-stop.sh Phase-1 pattern |
| U5 | `scripts/spawned-task-pipeline.sh` | Orchestrator: harvest → recover (real) → prune-dry-run (noise/recovered). One command = full lifecycle, all non-destructive. | U3, U4, harvest |
| U6 | `harvest-spawned-tasks.sh --surface-md` (small add) | Optional: write the report to `reports/spawned-task-harvest.md` so the dashboard surfaces it independent of the hook line. | harvest |
| U7 | Tests | `scripts/scheduler/tests/` style: prune-eligibility regex (rejects scheduler paths AND `.`/`..`), dry-run kills nothing, recover writes to cache only, fail-safe on git error. | agentic-testing-discipline (affected-tests-only) |

All of U1–U7 honor: read-only w.r.t. tracked files, dry-run-first for anything destructive, fail-safe (uncertain → don't act), and the BG-COMPLETE-SENTINEL contract.

### 4B. GATED — needs the user to approve settings.json wiring (EXACT lines)

> The settings-edit-guard PreToolUse blocks agent Write/Edit on `.claude/settings.json` (override `EVIUM_SETTINGS_EDIT_OK=1`, reserved for the user). The user merges these.

**G1 — Wire harvest auto-surface (F2 — highest leverage).** Add to `hooks.SessionStart[*].hooks`:
```json
{ "type": "command",
  "command": "bash -c '[ -z \"$EVIUM_HARVEST_OFF\" ] && bash scripts/harvest-spawned-tasks.sh --quiet || true'" }
```
(Identical to the staged patch `scripts/hooks/settings-patches/harvest-spawned-tasks-wire.json`. Prints ONE ⚠ line only when real work is stranded; exits 0 always.)

**G2 — Wire the SST prelaunch row + verdict poll (F1).** Add to `hooks.PreToolUse` a matcher on the spawn tool:
```json
{ "type": "command",
  "command": "bash -c '[ -z \"$EVIUM_SST_PRELAUNCH_OFF\" ] && bash scripts/hooks/suggested-task-prelaunch.sh || true'" }
```
and (optionally) a SessionStart verdict surface:
```json
{ "type": "command",
  "command": "bash -c '[ -z \"$EVIUM_SST_VERDICT_OFF\" ] && bash scripts/suggested-task-verdict.sh | head -20 || true'" }
```

**G3 — Wire the pipeline's surface+recover-to-staging at SessionStart (F4).** Add:
```json
{ "type": "command",
  "command": "bash -c '[ -z \"$EVIUM_SPAWN_PIPELINE_OFF\" ] && bash scripts/spawned-task-pipeline.sh --surface-and-stage || true'" }
```
(`--surface-and-stage` runs harvest + recover-to-cache + prune-DRY-RUN only — still destroys nothing.)

**G4 — (only if the user wants TRUE auto-clean, not just auto-surface) approve ONE prune path.** Choose:
- **Option A** — add to `permissions.allow`:
  ```json
  "Bash(git worktree remove .claude/worktrees/*)"
  ```
  AND add a deny-guard hook `scripts/hooks/spawned-worktree-remove-safety.sh` (PreToolUse) that BLOCKS unless the target is noise-only/recovered + matches the harness regex + passes TTL — so the allow-rule can't be abused. Then `prune-spawned-worktrees.sh --apply` (fired from G3 or a daemon) actually cleans.
  - Note: `Bash(*--force*)` is denied, so the pruner must call `git worktree remove` **without `--force`** and pre-verify the worktree is clean/recovered itself (it does).
- **Option B (no allow-rule)** — leave prune manual: the dry-run JSONL is the user's copy-paste list. Self-surfacing, not self-cleaning.

> **Recommendation:** ship **all of 4A now** + flip **G1 immediately** (closes the worst symptom — silent stranding — with one safe line). Then **G2/G3**. Treat **G4-Option-A** as the final, deliberate "I want it to delete worktrees on its own" decision, since it's the one step that crosses the A66 mass-delete line — keep it dry-run until the user has watched `reports/spawned-worktree-prune-dry-run.jsonl` for a cycle (the same 24 h-observation discipline `bg-auto-stop.sh` uses before any Phase-2 flip).

### 4C. How it becomes STRUCTURAL (fires every time, not discipline-dependent)

- **Suggest-time:** G2 PreToolUse writes the prelaunch row automatically on every chip surfaced → no never-tracked tasks.
- **Click-time:** U2 preamble baked into the spawn prompt → every clicked spawn self-revalidates (SST-1) before mutating → safe-to-start-anytime is enforced by the prompt, not by memory.
- **Session-start of the MAIN session:** G1 surfaces stranded work; G3 recovers it to staging + emits the prune dry-run → the main session always "goes and looks" (PULL-based, the only reliable model per §1).
- **Prune:** G4-A (gated) lets the dry-run flip to apply behind a dedicated safety hook; until then the dry-run list is the structural surface.
- **Kill-switches** (`EVIUM_*_OFF`) mirror every other env-gated hook, so any stage can be silenced per-shell without editing settings.

---

## 5. EXACT SETTINGS LINES THE USER MUST APPROVE (copy-paste summary)

```jsonc
// hooks.SessionStart[*].hooks  — G1 (do this first; safest, highest leverage)
{ "type":"command","command":"bash -c '[ -z \"$EVIUM_HARVEST_OFF\" ] && bash scripts/harvest-spawned-tasks.sh --quiet || true'" }

// hooks.PreToolUse  (matcher on the spawn tool) — G2 prelaunch row
{ "type":"command","command":"bash -c '[ -z \"$EVIUM_SST_PRELAUNCH_OFF\" ] && bash scripts/hooks/suggested-task-prelaunch.sh || true'" }

// hooks.SessionStart[*].hooks — G3 surface + recover-to-staging + prune-DRY-RUN
{ "type":"command","command":"bash -c '[ -z \"$EVIUM_SPAWN_PIPELINE_OFF\" ] && bash scripts/spawned-task-pipeline.sh --surface-and-stage || true'" }

// permissions.allow — G4 Option A ONLY (crosses A66 mass-delete line — last, deliberate)
"Bash(git worktree remove .claude/worktrees/*)"
```

Everything else (U1–U7) requires **no approval** — Claude can build and test it immediately, and it will be fully functional in dry-run/surface/stage form the moment it lands; only the literal worktree-deletion step waits on G4.

---

## 6. Bottom line

- **Safe-to-start:** scripts BUILT, **NOT WIRED** (F1). Not live until G2 + U2.
- **Report-back:** script BUILT, **NOT WIRED** (F2). 100 % manual today; 6 worktrees stranded right now. One line (G1) fixes the worst of it.
- **Self-clean:** **GENUINELY MISSING** (F3/F6) and Claude is **deliberately walled off** from every prune path (deny-list + A66). Buildable ungated as a **dry-run surfacer**; true auto-delete needs G4-Option-A.
- **Pipeline:** **MISSING** (F4) — the glue that makes it one self-managing flow.

The honest summary the user has been missing: **this isn't "not built" — it's "built and left disconnected."** The fix is mostly *wiring* (3 gated lines) + a thin *orchestration/prune* layer (6 ungated scripts), reusing the A47 ghost-reap discipline wholesale. Flip G1 today.

*Read-only audit. Nothing applied: no settings.json edit, no src edit, no git rm/commit/push, no worktree removed. This file is the only artifact written.*
