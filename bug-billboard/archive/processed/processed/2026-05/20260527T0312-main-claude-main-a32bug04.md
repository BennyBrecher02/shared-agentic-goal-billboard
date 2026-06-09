# detect-script-candidates silent exit-1 on empty .jsonl glob

- **id:** 20260527T0312-main-claude-main-a32bug04
- **discovered:** 2026-05-27T03:12:00Z
- **agent:** claude-main
- **worktree:** main
- **route:** N/A (hook)
- **section:** infrastructure
- **viewport-bucket:** all
- **severity:** low
- **status:** open
- **defect-type:** infrastructure
- **dedupe-key:** hook|detect-script-candidates|empty-glob|all

## Description

`scripts/hooks/detect-script-candidates.sh` exits with status 1 silently when its `.jsonl` input glob matches nothing. No error message; the SessionStart digest just shows fewer recommendations and the user can't tell something failed.

## Repro

Run hook with no JSONL files present in expected source directory. Hook exits 1 without writing output.

## Source

A32 Phase 2 hook test suite — discovered by `tests/hooks/test_detect_script_candidates.py`.

## Proposed fix

Use `shopt -s nullglob` or check the glob result before processing. Exit 0 with empty output if no files. Optionally log to stderr that no files were found.

## Cross-references

- A32 Phase 2 BG report (this turn)
- `tests/hooks/test_detect_script_candidates.py`
