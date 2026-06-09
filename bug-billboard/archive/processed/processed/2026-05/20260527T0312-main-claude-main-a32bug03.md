# goal-billboard-status hook: PLAN_PATHS unbound on bash 3.2 with no plans

- **id:** 20260527T0312-main-claude-main-a32bug03
- **discovered:** 2026-05-27T03:12:00Z
- **agent:** claude-main
- **worktree:** main
- **route:** N/A (hook)
- **section:** infrastructure
- **viewport-bucket:** all
- **severity:** medium
- **status:** open
- **defect-type:** infrastructure
- **dedupe-key:** hook|goal-billboard-status|bash-3.2-array-empty|all

## Description

`scripts/hooks/goal-billboard-status.sh` references `${PLAN_PATHS[@]}` without the bash-3.2-safe `${PLAN_PATHS[@]+"${PLAN_PATHS[@]}"}` pattern when the array is empty. macOS ships bash 3.2 by default → unbound variable error.

## Repro

On macOS default bash, with no plans listed for a goal, run the hook. Get `PLAN_PATHS[@]: unbound variable` error.

## Source

A32 Phase 2 hook test suite — discovered by `tests/hooks/test_goal_billboard_status.py`.

## Proposed fix

Either `set +u` locally around the array expansion, or use the bash-3.2-safe pattern: `${PLAN_PATHS[@]+"${PLAN_PATHS[@]}"}`.

## Cross-references

- A32 Phase 2 BG report (this turn)
- `tests/hooks/test_goal_billboard_status.py`
