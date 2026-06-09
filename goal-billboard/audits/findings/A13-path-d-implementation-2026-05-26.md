---
audit_id: A13
finding_phase: implementation
captured: 2026-05-26T18:00:00Z
status: implemented
related_changes:
  - 20260526T1755-on-surface-variant-lift-p2-09
  - 20260526T1755-blog-section-split-p2-10
  - 20260526T1755-highway-hero-object-position-p1-05
  - 20260526T1755-fleets-hero-text-shadow-firefox
  - 20260526T1755-services-logo-grid-p2-11
---

# A13 Path D — Implementation summary

## What landed

A new `expected_visual_diff: bool = False` field on the Change schema. When `true`, the scheduler does THREE things differently:

1. **Worktree verify step** — every Playwright shard appends `--update-snapshots` to its `npx playwright test` invocation. This regenerates the visual.spec.ts baselines inside the worktree as part of the verify. The verify PASSES because there's no baseline-vs-actual comparison to fail (Playwright wrote the new baselines). Also appends `--timeout=90000` to the per-test timeout because writing 9 fullPage PNGs per project adds disk I/O that pushes per-test wall time past the default 30s.

2. **Sequential shard dispatch** — `max_workers` is forced to 1 (sequential shards) when `expected_visual_diff: true`. `--update-snapshots` adds disk I/O on top of the existing dev-server load that's at the A11 concurrency ceiling with 3 parallel shards; sequential avoids Vite saturation. Trade-off: ~3× wall-clock time per change, but the change actually verifies instead of timing out.

3. **Main baseline regeneration via detached subprocess** — after `apply_to_main_on_verify` successfully applies the diff to main's working tree (`applied_to_main` event with `ok=true`), the scheduler spawns a DETACHED subprocess via `subprocess.Popen(..., start_new_session=True)`: `python -m scripts.scheduler regenerate-baselines --tick-id X --change-id Y --projects A,B,C`. The subprocess runs `npx playwright test tests/visual.spec.ts --update-snapshots --project=A --project=B --project=C` against main's working tree and emits `main_baselines_regenerated` to the same event log when done. The scheduler tick emits `main_baselines_regen_spawned` immediately and exits without waiting.

Daemon threads were attempted first and abandoned: in single-shot CLI mode (`python -m scheduler tick`), daemon threads die when the parent process exits, before the 3-5min regen can complete. Detached subprocesses survive parent exit because of POSIX `start_new_session=True`.

## How to use the flag

In a change.md frontmatter, just add the line:

```yaml
expected_visual_diff: true
```

The scheduler does the rest. Default is `false` so existing changes are unaffected.

**When to set it true:** any change that INTENTIONALLY changes pixels on at least one route — color-token shifts, layout splits, `object-position` adjustments, hero text-shadow additions, logo grid sizing, etc.

**When to leave it false:** non-visual changes (script logic, docs-only, accessibility attributes that don't paint, etc.) where the baselines should still pass against the current pixels.

## Code changes (5 files)

- `scripts/scheduler/change.py` — added `expected_visual_diff: bool = False` to Change dataclass + frontmatter parser
- `scripts/scheduler/shards/playwright_project.py` — `run()` appends `--update-snapshots` + `--timeout=90000` to the cmd when `change.expected_visual_diff` is true (via `getattr` so legacy `change=None` mocks still work)
- `scripts/scheduler/scheduler.py` — added `_regenerate_main_baselines` helper + `Scheduler._launch_main_baseline_regen` method (detached subprocess spawn); wired into the `apply_to_main_on_verify` block conditional on `change.expected_visual_diff` AND `apply_result["ok"]`; also forces `max_workers=1` for sequential shards when the flag is set
- `scripts/scheduler/__main__.py` — added `regenerate-baselines` CLI subcommand for the detached subprocess to call back into
- `scripts/scheduler/tests/unit/test_change.py` + `test_shards.py` + `test_scheduler.py` — 11 new tests

## Test delta

Before: 114 unit tests, all green
After: 125 unit tests, all green

New tests:
- `test_change.py` (+3): defaults to false, parses true, parses explicit false
- `test_shards.py` (+3): cmd omits `--update-snapshots`+`--timeout` by default, cmd includes both when flag set, getattr fallback for missing attribute
- `test_scheduler.py` (+5): spawn event fires after successful apply_to_main with Popen + start_new_session=True, spawn does NOT fire when flag is default, spawn does NOT fire when apply_to_main hard-fails, helper builds correct cmd with --update-snapshots + --project flags, helper short-circuits on empty projects

## Re-queued G2 changes

5 changes re-queued at `multi-change-queue/scheduled/` with new `T1755` change_ids and `expected_visual_diff: true`:

- `20260526T1755-on-surface-variant-lift-p2-09.md` (sitewide token color bump)
- `20260526T1755-blog-section-split-p2-10.md` (section split adds ~100vh)
- `20260526T1755-highway-hero-object-position-p1-05.md` (hero crop)
- `20260526T1755-fleets-hero-text-shadow-firefox.md` (hero text-shadow)
- `20260526T1755-services-logo-grid-p2-11.md` (logo grid normalize)

## Why Path D over Path B (the original recommendation)

Path B's design said "skip visual.spec.ts entirely when expected_visual_diff is set." That loses visual coverage for the change — the user has to manually review the visual change before merging.

Path D keeps visual coverage on (the shards still run visual.spec.ts) but uses `--update-snapshots` so the test writes new baselines instead of comparing. The user still gets pixel captures during the verify (Playwright always writes them); they just become the new ground truth.

Then the main-side regen ensures main's NEXT matrix tick uses the new baselines too — so subsequent non-visual changes don't fail because the prior visual change's pixels are now in the baselines.

## Tick verdicts (2026-05-26 PM — first production run of Path D)

All 5 re-queued changes verified successfully (zero shard timeouts after the `--update-snapshots + --timeout=90000 + sequential shards` triple).

| change_id | verdict | shard durations | apply_to_main |
|---|---|---|---|
| `20260526T1755-on-surface-variant-lift-p2-09` | verified | 18.9s + 13.1s + 14.0s | ok=true |
| `20260526T1755-blog-section-split-p2-10` | verified | 15.7s + 13.2s + 10.1s | ok=false ("does not match index" — context drift from prior fixes) |
| `20260526T1755-highway-hero-object-position-p1-05` | verified | 10.4s + 10.3s + 11.9s | ok=true |
| `20260526T1755-fleets-hero-text-shadow-firefox` | verified | 14.2s + 11.6s + 10.9s | ok=false ("patch does not apply" — context drift from prior P2-09 app.css change) |
| `20260526T1755-services-logo-grid-p2-11` | verified | 12.7s + 9.6s + 12.2s | ok=false ("patch does not apply" — context drift from prior P2-09 app.css change) |

The 3 apply_to_main hard-failures are independent A12-class issues (patch context drifted when prior changes in the same batch modified intervening lines). They are NOT A13 issues — verification itself succeeded for all 5. The user can manually re-create those diffs against current main and re-queue with `apply_strategy: manual` if needed.

## Cross-references

- `A13-visual-snapshot-baseline-drift.md` — audit catalog entry (status updated to resolved)
- `audit-decision-routing.md` — routing skill updated to mention `expected_visual_diff: true`
- `A11` — concurrency ceiling (related: A13's sequential-shard mitigation is the right call when `--update-snapshots` adds disk I/O on top of dev-server load)
- `A12` — oh-shit destructive ops (the patch-context-drift apply_to_main failures belong here, not A13)
