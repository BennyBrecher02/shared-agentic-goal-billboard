# compute-matrix-stats non-idempotent (double-counts) + missing-header silent crash

- **id:** 20260527T0312-main-claude-main-a32bug05
- **discovered:** 2026-05-27T03:12:00Z
- **agent:** claude-main
- **worktree:** main
- **route:** N/A (hook)
- **section:** infrastructure
- **viewport-bucket:** all
- **severity:** medium
- **status:** open
- **defect-type:** infrastructure
- **dedupe-key:** hook|compute-matrix-stats|idempotency-and-header|all

## Description

Two bugs in `scripts/hooks/compute-matrix-stats.sh`:
1. Non-idempotent — running twice double-counts entries in the output stats file.
2. Silent crash on missing header in source CSV/markdown.

## Repro

1. Run hook once → correct counts. Run again → counts doubled.
2. Hand-edit source to remove header line → hook crashes without clear error.

## Source

A32 Phase 2 hook test suite — discovered by `tests/hooks/test_compute_matrix_stats.py`.

## Proposed fix

1. Idempotency: read existing output, dedupe by entry-key before re-emitting. Or rewrite output from scratch each run (preferred).
2. Header check: validate header presence at start; if missing, write `[hook: skipped — missing header]` to stderr and exit 0.

## Cross-references

- A32 Phase 2 BG report (this turn)
- `tests/hooks/test_compute_matrix_stats.py`
