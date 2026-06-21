---
goal_id: G1
title: Multi-change scheduler MVP
status: achieved
archived: "2026-06-21T21:01Z — ARCHIVED→achieved per user-approved goal-grooming triage (goal-grooming-proposal-2026-06-21.md §4-B). MVP exit criteria met + battle-tested: Phase A.1–A.5 all [x], Phase B partial + Phase D verified, used in anger on G2 (4 real Phase 5 fixes shipped via scheduler). Further scheduler work (PR-mode, M2/SSH, cluster) is design-parked in linked_plans and spins off as its own scoped goal rather than holding G1 active. Moved active/ → archived/achieved/."
track: infrastructure
phase: Phase D ✅ + A11 ✅ + G2 launch ✅ (4 real Phase 5 fixes verified via scheduler)
priority: P0
created: 2026-05-25T21:16Z
updated: 2026-06-21T21:01Z
linked_plans:
  - context/markdowns/plans/multi-change-scheduler-plan.md
  - context/markdowns/plans/multi-change-scheduler-implementation-plan.md
  - context/markdowns/plans/multi-change-scheduler-phase-A3-A5-plan.md
  - context/markdowns/plans/multi-change-scheduler-phase-D-plan.md
  - context/markdowns/plans/automation/m2-ssh-scheduler-integration.md
  - context/markdowns/plans/automation/m2-mac-test-farm.md
  - context/markdowns/plans/automation/cluster-readiness.md
  - context/markdowns/plans/automation/git-hygiene-hooks-plan.md
  - context/markdowns/plans/automation/live-agent-message-queue-plan.md
  - context/markdowns/plans/automation/scheduler-pr-mode-plan.md
linked_refs:
  - .claude/skills/agentic-quality-discipline/SKILL.md
  - .claude/skills/agentic-quality-discipline/references/git-hygiene.md
  - .claude/skills/agentic-script-design/SKILL.md
linked_bugs: []
linked_changes:
  - context/markdowns/multi-change-queue/scheduled/  (P0-02 to be queued)
serves_northern_star: G2  # migrated 2026-05-26 - goal_id=G1 (not NS) → GL; NS defaults to G2
serves_guiding_light: G1
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G1 — Multi-change scheduler MVP

## Why it exists

The Phase 5 site overhaul has 14+ audited defects that need to be fixed, verified across the 12-project Playwright matrix, and merged without per-change human verification overhead. Manual per-bug matrix runs eat hours and don't scale.

The scheduler is the orchestration layer: accept a `change.md`, apply the diff to a worktree, run the relevant matrix shards, verify via 3 guards (artifact hash + timing sanity + billboard cross-ref), and mark the change verified. Multi-change concurrency in Phase B.

## Current state

- **Phase A.1 complete** (2026-05-25T22:00Z): 9 modules + 69 unit tests GREEN + CLI + bash glue. 44 min wall-clock.
- **Phase A.2 attempt blocked** (2026-05-25T22:45Z) by working-tree-vs-HEAD drift. Repo had single-commit HEAD + 73 modified + 26 untracked.
- **Drift resolved** (2026-05-26 AM): bulk commit 02359cd brings HEAD ≈ working tree. Phase A.2 now unblockable.

## Blockers

- None as of 2026-05-26 AM post-commit. (Previous blocker resolved.)

## Next action

1. Add `--3way` flag to [scripts/scheduler/scheduler.py:59-65](../../../../scripts/scheduler/scheduler.py) (`git apply` → `git apply --3way`) for resilience to small main movement
2. Run unit tests; verify GREEN
3. Generate fresh P0-02 diff against new HEAD (CSS fix for `/blog` §06 subscribe form overflow)
4. Write `change.md` with frontmatter + diff body; drop in `context/markdowns/multi-change-queue/scheduled/`
5. Run `bash scripts/multi-change-tick.sh`
6. Watch token cost + wall-clock + verdict; report

## Exit criteria

- [x] Phase A.1: scaffolding green
- [x] **Phase A.2: one real change applied + matrix-verified end-to-end via scheduler** ← 2026-05-26T09:16Z (12.3s wall-clock; chromium-desktop; verdict=verified)
- [x] Phase A.3 partial: --3way default in scheduler.py + node_modules symlink + advisory parser fixes
- [x] **Phase A.3 full: metrics CSV + idempotent create_worktree + path-shape safety + auto-cleanup wiring** (2026-05-26 PM)
- [x] **Phase A.4: second change (different file, different defect) verified first-try in 15.2s** ← 2026-05-26T12:38Z. contact.astro tablet-portrait gutter fix. Auto-cleanup fired post-verify (worktree + branch gone). Zero new scheduler code needed — A.3 hardening generalized cleanly.
- [x] **Phase A.5: 5 verified changes total + baseline metrics + LESSONS** ← 2026-05-26T12:50Z. 3 more changes (multifamily, highway, services) all verified first-try in 14.4-15.5s. Pipeline production-stable. Baseline mean 14.9s wall-clock per single-shard tick. See `notes/scheduler-baselines.md` + `notes/scheduler-phase-a-lessons.md`.
- [x] **Phase B partial: ThreadPoolExecutor multi-shard dispatch + shared pre-started dev server + post-verify cleanup.** 3-shard parallel verified clean (39.5s wall, 2.76× speedup, ~92% parallel efficiency). 12-shard surfaced concurrency ceiling — catalogued as A11. max_workers=4 conservative cap until A11 diagnoses root cause.
- [ ] **Phase B+ (deferred via A11):** safely raise concurrency ceiling beyond 4. Likely fix once root cause identified (probably browser-emulation memory pressure OR dev-server concurrent-request ceiling).
- [ ] Phase C (optional): Lighthouse + a11y as shard types
- [x] **Phase D / M-3: iOS Sim + Android Emu shards + Semaphore(1) resources — VERIFIED 2026-05-26T13:41Z.** 3 runtime types (Playwright + iOS sim + Android emu) coordinated in parallel in 24.3s. Cold-boot caveat for Android emu — wait_for_boot_completed needs work; warm-emu works first-try. Documented in phase-D-plan.md.

## Matrix integration milestones (post-A.5)

This goal's MVP exits at Phase A.5. The **full matrix-resource integration** the scheduler was designed for is phased AFTER MVP — keeping it visible so it doesn't slip through:

- [ ] **Phase B (post-A.5):** add all 12 Playwright shards (`chromium-{desktop,laptop,android,android-narrow,android-tablet}`, `webkit-{desktop,iphone,iphone-large,iphone-small,ipad,ipad-mini}`, `firefox-desktop`)
- [ ] **Phase D / M-3:** add `iOSSimulator = Semaphore(1)` + `AndroidEmulator = Semaphore(1)` to `scripts/scheduler/resources/` so changes can declare iOS/Android shards in their `required_coverage.matrix`
- [ ] **Phase C (optional):** Lighthouse + a11y as new shard types

**Status of matrix layers right now (independent of scheduler integration):**

| Layer | State |
|---|---|
| Layer 1 (build verification) | active |
| Layer 2 (Playwright 12-project matrix) | configured + per-project green; full end-to-end run not benchmarked recently |
| Layer 3 (iOS Simulator) | **ACTIVE** since 2026-05-24 — iPhone 17, iPhone 17 Pro Max, iPhone SE, iPad Pro 11 validated |
| Layer 4 (Android Emulator) | **ACTIVE** since 2026-05-24 — Pixel_9_API_35, Pixel_9a_API_34, Pixel_Tablet_API_35 validated |
| Layer 5 (real devices) | deferred (not panic-mode urgent) |

The scheduler hasn't absorbed Layers 3-4 yet (still single-shard chromium-desktop). They'll come in Phase D / M-3. Documented here to prevent execution-splintering.
- [ ] Phase A.4: 2nd change through scheduler (different priority)
- [ ] Phase A.5: at least 5 changes verified through scheduler with measured token + time stats
- [ ] Phase B start: parallel-mode unlock (post-Phase-A.5 stability)

## History

- 2026-05-25T17:00Z — Implementation plan locked
- 2026-05-25T21:16Z — Phase A kickoff (GO command)
- 2026-05-25T22:00Z — Phase A.1 complete; 72 unit tests GREEN
- 2026-05-25T22:30Z — apply_strategy decision (hybrid B+A default, bro deferred)
- 2026-05-25T22:45Z — Phase A.2 attempt blocked on working-tree drift
- 2026-05-26 AM — drift resolved via bulk commit; --3way fix queued

## Notes

- Worktree capacity = 2 (4-layer safety: capacity + per-mode target + advisory gate + effective = min)
- WATCH MODE for tokens (no gates yet; collect baseline)
- The "Bro" semantic-apply strategy is Phase B+ deliverable
