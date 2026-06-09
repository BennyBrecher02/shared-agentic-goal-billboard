# Settings.json Health Audit — 2026-06-07

- **Type:** READ-ONLY audit (Ask 1a). No edits, no spawn_task.
- **Target:** `.claude/settings.json` (newly pasted by user)
- **Run (UTC):** 2026-06-07T22:54Z
- **Method:** `jq` structural parse + `grep` token sweep + cross-check against `scripts/` + `scripts/hooks/`.

## VERDICT: GREEN

All six checkpoints pass. Zero `EVIUM_` tokens, zero dangling wires, zero
mismatched/dead kill-switches, all result-loop + never-idle wires present at the right events,
`git revert` correctly in `allow`, valid JSON with no within-event duplicate blocks. Fix-list is empty.

---

## Checkpoint results

### (1) ZERO old `EVIUM_` tokens — GREEN
- `grep` of `.claude/settings.json` for `EVIUM_`: **NONE**.
- Every kill-switch / control token is `AOS_*`. 49 distinct `AOS_*` tokens present
  (48 `*_OFF` switches + `AOS_COT_CHECKPOINT_BLOCK` escalation + `AOS_NEVER_IDLE_QUIET` dry-run flag).
- Bonus sweeps: **0** `EVIUM_` occurrences anywhere under `scripts/` (wired or not);
  **0** in `.claude/*.json`.

### (2) No dangling wires — GREEN
- Extracted **91** unique wired script paths from all hook commands.
- All 91 **exist** and are **executable**. `missing=0 not_executable=0`.
- (A grep-logic "orphan basename" blip during cross-check was a false alarm — each flagged
  basename, e.g. `capture-staleness.sh`, `token-budget-check.sh`, was hand-confirmed present +
  executable under `scripts/hooks/`.)

### (3) De-shim consistency (no silently-dead kill-switches) — GREEN
- **48** guarded wires. For each, checked two enforcement paths:
  - **settings-level wrapper** `[ -z "$AOS_X_OFF" ] && bash … || true`, and
  - **script-level self-read** of the token.
- **All 48 carry the settings-level `[ -z ]` wrapper → every kill-switch is LIVE.**
  Setting any `AOS_X_OFF=1` short-circuits the `&&` and skips the script.
- The asked-for failure mode — settings gates `$AOS_X_OFF` but the script reads a leftover
  `$EVIUM_X_OFF` — has **ZERO** instances. No wired script references any `EVIUM_` token, so no
  rename-mismatch can silently kill a switch.
- 28 of 48 scripts do not *redundantly* re-read their token inside the script. **This is not a
  defect:** their gate is enforced by the settings wrapper (a valid architecture). The 20 that also
  self-read (e.g. `findings-life-thread-surface.sh`, `never-idle-refill-surface.sh`,
  `spawn-task-guard.sh`, `screenshot-janitor.sh`) belt-and-suspenders it. No `AOS_`/`EVIUM_` token mismatch in either group.
- Control tokens verified read where it matters: `AOS_COT_CHECKPOINT_BLOCK` is read by
  `cot-checkpoint-pre.sh` (escalation works); `AOS_NEVER_IDLE_QUIET` is read by
  `never-idle-refill-surface.sh` (dry-run works).

### (4) Result-loop + never-idle wires present at the right events — GREEN
- `findings-life-thread-surface` → wired at **SessionStart** + **Stop**.
- `spawned-task-result-surface` → wired at **SessionStart** + **Stop**.
- `never-idle-refill-surface` → wired at **UserPromptSubmit** + **Stop** (Stop variant carries
  `CLAUDE_HOOK_EVENT=Stop`; both default to `AOS_NEVER_IDLE_QUIET=1` 24h dry-run — un-QUIET both to go live).

### (5) `git revert` placement — GREEN
- `Bash(git revert*)` is in `permissions.allow`.
- **Not** in `permissions.deny`. (Note: deny does block adjacent destructive git ops —
  `reset --hard`, `checkout --`, `clean`, `switch`, `restore`, push/pull/fetch/merge/rebase — but `revert` is correctly exempt.)

### (6) Valid JSON, no duplicate-block issues — GREEN
- `jq empty` → **VALID JSON**.
- Cross-event command repeats are intentional, not duplicate blocks:
  - `verify-memory-sync.sh` → Stop + SessionEnd (1 each)
  - `findings-life-thread-surface.sh`, `spawned-task-result-surface.sh`,
    `screenshot-janitor.sh` → SessionStart + Stop (1 each)
  - `scheduler-data-refresh.sh` → PostToolUse twice, but under **different matchers**
    (`Write|Edit` and `Bash`) — correct, not a dup block.
- No within-single-event duplicate command. No `settings.local.json` override present.

---

## Fix-list
**EMPTY.** No remediation required. The de-shim from `EVIUM_*` → `AOS_*` is complete and consistent;
all wires resolve; all kill-switches are live.

## BG-COMPLETE-SENTINEL
- evium_in_settings: 0
- dangling_wires: 0
- mismatched_killswitches: 0
- verdict: GREEN
