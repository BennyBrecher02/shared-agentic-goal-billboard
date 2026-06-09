---
audit_id: A11
title: Phase B concurrency-ceiling investigation
status: completed
catalogued: 2026-05-26T13:10:00Z
completed: 2026-05-26T14:30:00Z
findings: audits/findings/phaseB-concurrency-ceiling-2026-05-26-phase1.md
outcome: empirical ceiling = 3 shards. Hard cap set in scheduler.py. Mitigation paths documented for A11.1 follow-up.
priority_when_run: P2
estimated_effort: medium
trigger: When Phase B's 4-shard cap becomes a real bottleneck (e.g., a real change needs full 12-shard matrix coverage AND 4-at-a-time is too slow for the workflow) OR when scheduler runs on M2 SSH (different machine; different ceiling)
deferral_reason: Currently 4-shard cap is the documented Phase B limit. 3-shard verified clean (Phase B initial verification on 2026-05-26 PM). 12-shard surfaced timeouts on M4 24GB. Investigating root cause is deferred — 4-shard is sufficient for current workflow.
related_goals: [G1]
related_plans:
  - context/markdowns/plans/multi-change-scheduler-phase-A3-A5-plan.md
related_refs:
  - .claude/skills/agentic-quality-discipline/references/git-hygiene.md
  - .claude/skills/agentic-script-design/references/opsys-scheduling-patterns.md
related_audits: [A6, A9, A10]
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'scheduler' -> GL G1; NS defaults to G2
belongs_to_goal: G1
serves_guiding_light: G1
---
# A11 — Phase B concurrency-ceiling

## Why this audit matters

Phase B verified parallel multi-shard execution at 3 shards (chromium-desktop + webkit-desktop + firefox-desktop) in 39.5s wall-clock with ~2.76× speedup. **At 12 shards, the same fix change failed: all shards timed out or hit per-test timeouts.** The cause is unclear without instrumentation. Conservative cap of `max_workers=4` set in scheduler.py until root cause is identified.

## What's known

- 3 desktop shards: ✓ verified, 39.5s, 92% parallel efficiency
- 12 shards (5 chromium + 6 webkit + 1 firefox, mixed desktop/mobile/tablet): ✗ all 12 shards failed
- First batch (10 in parallel due to ThreadPoolExecutor cap) hit 100-150s timeouts
- Queued tail (2 shards after pool freed): ran in 28s but also failed
- Dev server was started OK on port 4399 by scheduler (events confirm)

## Hypotheses to investigate

1. **Browser emulation memory pressure** — 10 concurrent Playwright invocations × multiple browser processes each may exceed M4 24GB available RAM during peak. Verify via `vm_stat` snapshots during a tick.
2. **Dev server concurrent-request ceiling** — Astro dev server may not handle 90+ concurrent page requests (12 projects × 9 routes) cleanly. Verify via dev server stdout (currently suppressed via DEVNULL).
3. **Playwright per-test timeout (30s default)** — individual tests may time out under concurrent load. Verify via per-test stderr.
4. **OS-level FD or socket limits** — 10+ Playwright procs × browser subprocess + dev server connections may hit ulimits. Verify via `ulimit -n`.
5. **Mobile emulation overhead** — first 3-shard test was all desktop; full matrix includes mobile/tablet which has more overhead. Test 4 desktop-only shards in parallel; if works, mobile emulation is the bottleneck.

## What it would look at

- Capture per-shard stderr in scheduler events (currently dropped)
- Add memory + CPU sampling during 12-shard runs
- Compare 4-desktop vs 4-mobile vs 4-mixed to isolate the variable
- Profile dev server response times under load
- Inspect Playwright trace files from failed runs

## Expected outputs

- `audits/findings/phaseB-concurrency-ceiling-<date>-phase1.md` with diagnostic findings
- Updated `max_workers` ceiling in scheduler.py if findings allow safely raising
- Update to `notes/scheduler-baselines.md` with concurrency-vs-throughput curve

## How to trigger

Whichever fires first:
- A real change needs full-matrix verification AND 4-at-a-time is too slow
- Scheduler runs on M2 SSH (different machine = different ceiling; may need re-tuning)
- A new shard type (Lighthouse, a11y) adds more concurrency pressure
- User says "let's diagnose the 12-shard limit"

## Current operational state

- Scheduler defaults to `max_workers=4`
- Single-shard changes: stable (~15s baseline)
- 2-4 shard parallel: verified working
- Beyond 4 shards: untested + flagged risky pending this audit

## Cross-references

- A10 — scheduler workaround audit (different concern: deny-list workarounds)
- Phase A.3-A.5 plan — captures Phase B scope
- LESSONS doc — `notes/scheduler-phase-a-lessons.md`
