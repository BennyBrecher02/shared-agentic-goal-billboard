# bug-billboard-consolidate append_fixed() missing mkdir -p for archive dir

- **id:** 20260527T0312-main-claude-main-a32bug06
- **discovered:** 2026-05-27T03:12:00Z
- **agent:** claude-main
- **worktree:** main
- **route:** N/A (hook)
- **section:** infrastructure
- **viewport-bucket:** all
- **severity:** medium
- **status:** open
- **defect-type:** infrastructure
- **dedupe-key:** hook|bug-billboard-consolidate|missing-mkdir-archive|all

## Description

`scripts/hooks/bug-billboard-consolidate.sh`'s `append_fixed()` function attempts to write to the archive directory (`archive/fixed/{YYYY-MM}.md`) without ensuring the month subdirectory exists. First fix-archival of any new month fails silently or with cryptic error.

## Repro

In a fresh worktree (or month where no fixes have been archived), trigger the consolidate flow with a fixed bug. Observe archive write failure.

## Source

A32 Phase 2 hook test suite — discovered by `tests/hooks/test_bug_billboard_consolidate.py`.

## Proposed fix

Add `mkdir -p "$(dirname "$ARCHIVE_FILE")"` before each archive append in `append_fixed()`.

## Cross-references

- A32 Phase 2 BG report (this turn)
- `tests/hooks/test_bug_billboard_consolidate.py`
