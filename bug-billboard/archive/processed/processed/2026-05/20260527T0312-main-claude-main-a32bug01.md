# git-drift-warn hook prints "00 file(s)" on clean repo

- **id:** 20260527T0312-main-claude-main-a32bug01
- **discovered:** 2026-05-27T03:12:00Z
- **agent:** claude-main
- **worktree:** main
- **route:** N/A (hook)
- **section:** infrastructure
- **viewport-bucket:** all
- **severity:** low
- **status:** open
- **defect-type:** infrastructure
- **dedupe-key:** hook|git-drift-warn|count-formula|all

## Description

`scripts/hooks/git-drift-warn.sh` uses `grep -c . || echo 0` pattern which yields `"00"` when there's nothing to count (output concatenates `0` from grep + `0` from echo). Cosmetic but confusing in the SessionStart digest.

## Repro

Run the hook on a clean repo with no drift. Observe "00 file(s)" in output.

## Source

A32 Phase 2 hook test suite — discovered by `tests/hooks/test_git_drift_warn.py`.

## Proposed fix

Replace `grep -c . || echo 0` with explicit conditional: `count=$(grep -c . 2>/dev/null || true); count=${count:-0}; [[ -z "$count" ]] && count=0`.

## Cross-references

- A32 Phase 2 BG report (this turn)
- `tests/hooks/test_git_drift_warn.py`
