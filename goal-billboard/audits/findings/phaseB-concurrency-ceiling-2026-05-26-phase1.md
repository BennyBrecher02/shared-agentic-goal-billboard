---
created: 2026-05-26T14:30:00Z
updated: 2026-05-26T14:30:00Z
audit_id: A11
phase: 1 — empirical-ceiling-via-binary-search
status: findings-locked
---

# A11 Phase 1 — Concurrency ceiling findings

## Experiment

Tested scheduler parallel-shard capacity via real ticks on M4 24GB. Each test used the same change.md content (an `@media (max-width: 280px)` rule that doesn't match any project viewport → snapshots unchanged → outcome depends entirely on pipeline reliability under load).

## Results

| Shard count | Wall-clock | Outcome | Per-shard duration |
|---:|---:|:---|---:|
| 1 | ~15s | ✅ verified (Phase A baseline) | ~13s |
| 3 | 39.5s | ✅ verified (Phase B verified, 92% parallel efficiency) | 35-38s each |
| 4 | 199s | ❌ all 4 failed | 167-199s each |
| 6 | 169s | ❌ all 6 failed | 135-162s each |
| 12 | 78s | ❌ 10 of 12 failed (first batch timeouts) | 100-150s each |

## Diagnosis

**The cliff is between 3 and 4.** Same scheduler code, same pre-started dev server, same change.md content — only the shard count varies.

### Likely root cause

**Astro dev server cannot sustain 4+ concurrent Playwright test workers.** Each Playwright invocation runs 9 routes per project (`tests/visual.spec.ts`). With 4 workers: 36 concurrent route fetches + concurrent network-idle wait + concurrent fonts.ready check + concurrent image-load polling.

Vite (Astro's dev server) handles requests asynchronously but has finite throughput. Under heavy concurrent load, individual request timeouts cascade into Playwright test timeouts.

### Per-shard duration analysis

At 4-shard: 167-199s per shard. Playwright's default per-test timeout is 30s. With 9 tests per shard, if 5-6 of them hit timeouts (30s × 5-6 = 150-180s), the totals match observed durations. Tests timing out, not whole-shard infrastructure failure.

## Verdict

- **Hard cap: `max_workers = 3`** for current setup (Astro dev server + Playwright matrix).
- 3-shard parallel is genuinely useful (39.5s for 3 projects vs ~45s sequential = ~12% speedup; gain is small for 3 but real).
- Full 12-shard matrix becomes 4 batches × 3 shards = ~160s total. Acceptable for occasional full-matrix verification.

## Open follow-ups (A11.1 candidates — future audit)

Three plausible mitigations to raise the ceiling later:

1. **Multiple dev server instances** — one per shard, on different ports. Requires playwright.config.ts to accept `process.env.PORT` dynamically. Each shard would have its own dev server (eliminates request contention).

2. **Run against built `dist/`** — `npm run build` then `npm run preview` (static file server). No HMR overhead. Much faster, much more concurrent-tolerant. Costs: build time per change (~20-30s).

3. **Reduce per-shard Playwright workers** — `--workers=1` flag forces single-worker per shard. Wouldn't fix the cross-shard race but would reduce per-shard pressure.

None of these are critical for G2 (single-shard or 3-shard works fine). Defer until G2 workflow demands faster full-matrix coverage.

## What this audit did NOT do

- Capture per-shard stderr (the on-failure stderr capture was added to scheduler.py but the failed runs at 167-199s exited gracefully via ShardResult, not crashing — so stderr is whatever Playwright wrote to stdout before its own timeout). Would be informative to inspect a stderr_tail event for a 4-shard failure; deferred.
- Profile dev server response times under load
- Test the 3 mitigation paths above

These are A11.1 work. For now: capped at 3. Move on to G2.

## What this audit DID land

- `max_workers = 3` (was 4 conservatively; now empirically grounded)
- On-failure `stderr_tail` + `stdout_tail` in shard_finished events (for future diagnosis)
- This findings doc
