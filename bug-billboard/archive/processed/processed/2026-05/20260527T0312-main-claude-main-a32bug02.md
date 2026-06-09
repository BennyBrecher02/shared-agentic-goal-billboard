# goal-billboard-status hook crashes when paused/ or archived/achieved/ missing

- **id:** 20260527T0312-main-claude-main-a32bug02
- **discovered:** 2026-05-27T03:12:00Z
- **agent:** claude-main
- **worktree:** main
- **route:** N/A (hook)
- **section:** infrastructure
- **viewport-bucket:** all
- **severity:** medium
- **status:** open
- **defect-type:** infrastructure
- **dedupe-key:** hook|goal-billboard-status|missing-dirs|all

## Description

`scripts/hooks/goal-billboard-status.sh` reads from `paused/` and `archived/achieved/` subdirs without checking existence. With pipefail set, missing dirs cause crash mid-hook → no SessionStart digest output.

## Repro

Rename or delete `context/markdowns/goal-billboard/paused/` or `archived/achieved/`, then trigger SessionStart. Hook crashes.

## Source

A32 Phase 2 hook test suite — discovered by `tests/hooks/test_goal_billboard_status.py`.

## Proposed fix

Wrap directory reads with `[[ -d "$DIR" ]] && ...` guards. Or `mkdir -p` the expected dirs at hook entry.

## Cross-references

- A32 Phase 2 BG report (this turn)
- `tests/hooks/test_goal_billboard_status.py`
