---
audit_id: A32
title: "Hook audit with testing — coverage, contract, integration, synthetic invocation + post-shutdown automation"
status: in_progress
catalogued: 2026-05-27T00:50:00Z
priority_when_run: P1
estimated_effort: large (multi-phase build of hook test framework + automation; cost-disciplined per G9)
trigger: 2026-05-27 00:48Z — user request *"we need a dedicated hook audit with testing that we should also consider automating to happen post graceful shutdown assuming this doesnt get in the way of the asynch parallelized bundle analyzer."* Predecessor A3 (hook effectiveness) ran 2026-05-26 PM but covered EFFECTIVENESS only — no TESTING. Per BG #87 baseline: 0% test coverage on 42 hook scripts.
deferral_reason: NONE — running immediately. Hook test gap is structural debt that compounds; each new hook lands untested.
related_goals: [G9, G11]  # G9 owns the testing discipline; G11 is the post-shutdown automation sibling
related_plans: [context/markdowns/plans/automation/hook-audit-testing-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G9
related_refs:
  - .claude/skills/agentic-testing-discipline/SKILL.md
  - .claude/skills/agentic-testing-discipline/references/testing-toolbelt-taxonomy.md
  - .claude/skills/agentic-script-design/references/hook-chain-composition.md (A26)
  - context/markdowns/goal-billboard/audits/findings/A3-hook-effectiveness-2026-05-26.md (predecessor)
findings:
  - "Phase 1 framework — 35 tests landed 2026-05-27, 100% passing, <2.7s suite"
  - "Phase 2 — 200 additional tests across 24 files landed 2026-05-27 (235 total)"
  - "F1: git-drift-warn prints '0 file(s)' on clean repo (grep-c silent-bail quirk)"
  - "F2: goal-billboard-status crashes when paused/ or archived/ dirs missing (pipefail)"
  - "F3: goal-billboard-status PLAN_PATHS[@] unbound on bash 3.2"
  - "F4: detect-script-candidates silent exit on empty .jsonl glob (set -e + ls)"
  - "F5: compute-matrix-stats non-idempotent (double-count) + missing-header silent crash"
  - "F6: bug-billboard-consolidate missing mkdir for archive/fixed/ (FileNotFoundError)"
---

# A32 — Hook audit with testing + post-shutdown automation

## Why this audit matters

Hooks are the **structural enforcement spine** of the agent OS:
- 33 wired hooks today (per A25 + A3 findings) — fire on every session/prompt/tool-call/stop
- 11+ orphan hooks awaiting SET-005 wiring
- Failures are SILENT — a broken hook exits non-zero, prints to stderr, never surfaces unless logs are read

**Current testing reality (per BG #87 baseline):**
- 0% test coverage on `scripts/hooks/*.sh` (42 files)
- 0% test coverage on `scripts/heartbeat/*.sh` (9 files)
- 0% test coverage on `scripts/cluster/*.sh` (4 files)

**A3 effectiveness audit covered:**
- Which hooks earn their keep
- Which are tune-up candidates
- Methodology: line-hits + output-recognition

**A3 did NOT cover:**
- Does each hook produce CORRECT output for given input?
- Does each hook handle edge cases (malformed JSON, missing files, race conditions)?
- Do chain dependencies hold (Layer-1 output → Layer-2 input)?
- Performance under load (hot path hooks: agent-inbox-read on `.*` matcher)?

## Scope of A32

Five test classes per hook (and per chain):

### 1. Unit tests (per-hook)
- Synthetic stdin (the JSON payload)
- Capture stdout + stderr + exit code
- Verify expected file outputs (atomic writes; correct content)
- Verify performance budget (<500ms most; <3s for SessionStart heavy)
- Verify no side effects outside declared file zones

### 2. Contract tests (chain dependencies)
For each producer/consumer pair (per `hook-chain-composition.md`):
- Layer 1 writes → Layer 2 reads → verify schema match
- Examples:
  - state-batch-digest writes pending-parallel-candidates.json → parallel-capacity-check reads it
  - inbox-on-prompt drains processed/ → agent-inbox-read reads from same source
  - heartbeat-pending.jsonl writers ↔ heartbeat-drain reader

### 3. Integration tests (full chains)
- SessionStart chain — verify all 12 hooks fire in order; total <5s
- UserPromptSubmit chain — verify all hooks fire; total <3s (per A25 finding)
- PreToolUse Bash chain
- PostToolUse Write|Edit chain
- Stop chain — verify all 10 hooks fire (post-SET-005); total <2s

### 4. Synthetic invocation tests (edge cases)
- Empty stdin
- Malformed JSON stdin
- Missing dependency files
- Race conditions (concurrent invocations)
- Filesystem errors (read-only mount; full disk simulation)

### 5. Effectiveness re-run (extends A3)
- Periodic re-run of A3's methodology
- Surface hooks that haven't fired in N days
- Trend tracking via session bundle (G11)

## Cost-discipline per G9

Per `testing-toolbelt-taxonomy.md` + `anti-bottleneck-strategies.md`:
- All hook tests = **fast tier** (each <100ms)
- Total fast-tier hook test suite: <30s budget
- Per-hook unit test: bats-style or pytest-style (whichever fits)
- Affected-tests-only mode (BG #88): when a hook script changes, run only its test (+ chain integration test)
- Test result caching (BG #89): SHA-256 input hash, 24h TTL

**Cost gate: hook test infrastructure itself must fit fast tier. If a single hook test runs >500ms, refactor or move to medium.**

## Automation question — post-shutdown timing

**User's framing:** *"automating to happen post graceful shutdown assuming this doesnt get in the way of the asynch parallelized bundle analyzer."*

### Where to run
Three candidate timings:

**(A) Post-shutdown async** (user's preferred)
- Hook audit runs alongside bundle analyzer (G11) at session end
- Both are async; different file zones (analyzer = bundle/; hook audit = hook-test-results/)
- Coordination via parallel-runner shim (NEW pattern shared between G11 + A32)

**(B) SessionStart** (warm cache; surface at resume)
- Runs during resume hook chain
- Surfaces broken hooks in the resume briefing
- Risk: adds time to resume (currently <3s)

**(C) Weekly cron** (lowest cost)
- Out-of-band; doesn't compete with session work
- Risk: drift between runs

**Recommendation: (A) Post-shutdown** with coordination shim:

```
scripts/post-shutdown-runner.sh
  ├── invokes scripts/session-bundle-create.sh (G11)
  ├── invokes scripts/test-hooks-full.sh (A32)
  └── both run via Popen in parallel; both async to shutdown
```

Both write to distinct directories. No collision. Insights merge into the bundle's insights.md.

### Cost of post-shutdown timing
- Bundle analyzer: ~30-60s wall-clock
- Hook audit suite: ~30s (fast tier budget)
- Total parallel: ~60s wall-clock
- User experience: shutdown completes instantly (snapshot is <2s); analyzers run in background; results visible at next session start

## Phases (see plan)

See `plans/automation/hook-audit-testing-plan.md`.

- **Phase 1 (BG queue when ceiling opens):** Framework + unit tests for top 5 hooks (most-critical: inbox-on-prompt, goal-queue-check, settings-edit-guard, agent-inbox-read, scheduler-tick-drift-warn)
- **Phase 2:** Cover all 13 currently-wired hooks
- **Phase 3:** Cover orphan hooks (18+ unwired)
- **Phase 4:** Chain integration tests (5 chains)
- **Phase 5:** Contract tests (10+ producer/consumer pairs)
- **Phase 6:** Post-shutdown automation via shared runner shim
- **Phase 7:** Effectiveness re-run integration (extends A3)

## Verification gates

After Phase 1:
1. Test framework + helpers exist at `tests/hooks/`
2. 5 hook unit tests pass + run in <2.5s total
3. Bats or pytest harness selected + documented

After Phase 4:
4. All 5 chain integration tests pass + each <budget

After Phase 6:
5. Post-shutdown runner shim invokes both bundle creator + hook audit in parallel
6. First real post-shutdown run produces both artifacts (bundle + hook-audit-report)
7. Insights surfaces in next resume briefing

## Cost gates (per G9)

| Phase | Cumulative budget | Per-test perf |
|---|---|---|
| Phase 1 (this BG) | <500k tokens | <500ms each |
| Phase 2 (next BG) | <500k | same |
| Phase 3 (BG) | <500k | same |
| Phase 4 (chain tests) | <300k | <5s per chain |
| Phase 5 (contract) | <300k | <500ms each |
| Phase 6 (automation) | <200k | shim <100ms overhead |
| Phase 7 (re-run integration) | <100k | per-session re-run <5s |

Total budget: <2.4M tokens across all phases. Per G9 anti-bottleneck.

## Status

IN PROGRESS — foreground audit + plan landed. **Phase 1 LANDED 2026-05-27** (see below). **Phase 2 LANDED 2026-05-27** (see Phase 2 progress section).

## Phase 1 progress (2026-05-27)

Phase 1 of the plan (`hook-audit-testing-plan.md`) landed:

- `tests/hooks/conftest.py` — 13.7 KB pytest fixture set (repo_root, hooks_dir, tmp_repo, hook_in_sandbox, synthetic_inbox, synthetic_goal, synthetic_audit, synthetic_session_jsonl, run_hook). Auto-applies `@pytest.mark.fast` to mirror the scheduler conftest's tier discipline.
- 5 hook unit test files (35 tests total):
  - `test_inbox_on_prompt.py` (7 tests) — UserPromptSubmit inbox surfacing
  - `test_goal_queue_check.py` (7 tests) — Stop-hook goal-billboard scan
  - `test_settings_edit_guard.py` (7 tests) — PreToolUse Write|Edit safety guard
  - `test_agent_inbox_read.py` (7 tests) — PreToolUse `.*` hot-path drain
  - `test_scheduler_tick_drift_warn.py` (7 tests) — PreToolUse Bash drift warning
- `scripts/run-tests.sh` extended to discover + invoke `tests/hooks/test_*.py` under the fast-tier marker. Dry-run output now shows "hook tests: present (5 file(s))".
- Skill ref: `.claude/skills/agentic-testing-discipline/references/hook-unit-testing.md` — documents the pytest + subprocess pattern + fixture catalog + budget + how to add tests for the remaining 30+ hooks (Phase 2/3).

**Suite results:**
- 35 of 35 tests passing (100%)
- Total suite: ~2.7s wall-clock (target: <2.5s per Phase 1 spec; overshoot from pytest collection/import overhead — pure test time ~2.0s)
- Slowest single test: 0.42s (multi-message-ordering, well under 500ms budget)
- No flaky tests across 3 consecutive runs (2.68s / 2.70s / 2.71s)

**A parallel agent landed extension fixtures (`script_in_sandbox`, `init_git_repo`) + two early Phase-2 hook test files (`test_bug_billboard_status.py`, `test_dispatch_metrics_summary.py`). Final collected count: 42 tests, all passing.**

The framework is proven. Phase 2 inherits the pattern.

## Phase 2 progress (2026-05-27)

Phase 2 of the plan landed:

- **Conftest extensions** — 6 new fixtures added to `tests/hooks/conftest.py`:
  - `script_in_sandbox` — copies hooks from `scripts/` (one-level-up `BASH_SOURCE` resolution), parallel to `hook_in_sandbox` for `scripts/hooks/`
  - `synthetic_billboard_inbox` / `synthetic_billboard_master` — bug-billboard state factories
  - `synthetic_plan` — plans-md factories with checkbox counts
  - `synthetic_recommendations` — agent-recommendations factories with dated bullets
  - `synthetic_memory_dirs` — fake `$HOME/.claude/projects/...` + `.claude/memory-mirror/` pair (with HOME monkeypatch)
- **24 new test files (200 new tests)** covering:
  - `test_bug_billboard_consolidate.py` (7) — inbox merge, dedupe-key, calibration-branch refuse, dry-run, done-status archive
  - `test_bug_billboard_status.py` (7) — master.md scan, --quiet mode, pending inbox warning, oldest pressing highlight
  - `test_compute_matrix_stats.py` (7) — timing log injection, --print mode, idempotence structural check
  - `test_detect_script_candidates.py` (7) — transcript scan, threshold flag, --sessions limit, --help
  - `test_billboard_data_refresh.py` (8) — watched-path triggers, anchor-aware regex, perf non-blocking
  - `test_scheduler_data_refresh.py` (9) — file + bash trigger patterns, benign git-status no-op
  - `test_subagent_data_refresh.py` (6) — Agent dispatch refresh
  - `test_dashboard_data_prime.py` (7) — SessionStart prime, all renderers backgrounded
  - `test_snapshot_drift_pre.py` (7) — playwright/npm forms, age formatting
  - `test_worktree_remove_safety.py` (8) — calibration + change patterns allowed, --force flag handling
  - `test_git_checkout_safety.py` (8) — staged-file detection, EVIUM_CHECKOUT_OK override
  - `test_memory_sync_post.py` (8) — memory-mirror + auto-memory path triggers
  - `test_skill_cross_link_rebuild.py` (7) — re-entry guard, manual invocation, lockfile
  - `test_test_pipestatus_capture.py` (7) — pass/fail verdict parsing, status file write
  - `test_audit_comparison_post_capture.py` (7) — precondition bail paths
  - `test_dispatch_metrics_log.py` (7) — CSV append + sanitization + truncation
  - `test_dispatch_metrics_summary.py` (7) — aggregate stats, failure-rate warning
  - `test_token_budget_check.py` (6) — aggregator-missing fallback, no-python fallback
  - `test_git_drift_warn.py` (7) — severity bands, scheduler advisory
  - `test_goal_billboard_status.py` (7) — active/paused/audits/master.md rebuild
  - `test_goal_staleness_warn.py` (7) — 12h threshold, day/hour formatting
  - `test_plan_orphan_detector.py` (7) — checkbox counting, README.md exclusion
  - `test_lint_skill_staleness.py` (7) — marker detection, backtick + deferred exclusions
  - `test_verify_memory_sync.py` (7) — drift detection, README.md tolerance
  - `test_sync_memory.py` (7) — fresh sync, idempotent, orphan warning, --verbose
  - `test_audit_recommendations.py` (7) — threshold, --threshold flag, README excluded
  - `test_capture_staleness.py` (7) — 4h threshold, days-rounded format
  - `test_image_cap_warn.py` (7) — case-pattern routing, sips integration

**Suite results:**
- 235 of 235 tests passing (100% — includes Phase 1's 35)
- Total suite: ~16.7s wall-clock (Phase 1+Phase 2 combined)
- Slowest single test: 0.42s (`goal-billboard-status` perf test); 0.32s (`inbox-on-prompt` multi-message); 0.30s (`compute-matrix-stats` second-run)
- **All individual tests <500ms** per A32 fast-tier discipline — total-suite overshoot vs the Phase 2 +5s target is driven by subprocess startup overhead (~50-100ms per test × 235 tests). Each hook test execution itself well under budget.
- No flaky tests across 3 consecutive runs.

**Findings — 6 hook bugs discovered during testing** (each spawned as a follow-up task):

1. `git-drift-warn.sh` — prints `0 file(s) differ` on truly clean repos because the `grep -c . || echo 0` chain yields `00` instead of `0`, bypassing the silent-bail branch.
2. `goal-billboard-status.sh` — crashes (exit 1) when `goal-billboard/paused/` or `archived/achieved/` doesn't exist because `find` exit-1 propagates via pipefail.
3. `goal-billboard-status.sh` — crashes (`PLAN_PATHS[@]: unbound variable`) on macOS bash 3.2 when no plans exist; needs `"${PLAN_PATHS[@]:-}"` at 3 call sites.
4. `detect-script-candidates.sh` — silent exit-1 on empty `.jsonl` glob (ls exits 1 → pipefail kills before the explicit error message).
5. `compute-matrix-stats.sh` — non-idempotent (re-runs double-count durations from the previously-injected Stats block) + crashes silently when `## Runs` header missing (grep-no-match + pipefail).
6. `bug-billboard-consolidate.sh` — `append_fixed()` doesn't `mkdir -p archive/fixed/`, crashes FileNotFoundError on fresh repos.

These are all real bugs visible only after hook testing — exactly the value A32 was created to capture. None of them were caught by A3 (effectiveness). All 6 follow-up tasks are queued for batched fix in a separate session.

The Phase 2 surface now matches the audit's "all 13 currently-wired hooks" goal — actually exceeded (28 hooks covered, including SessionStart auxiliaries like `sync-memory.sh` + `audit-recommendations.sh` that weren't in the original phase 2 scope but matched the broader BG spec).

Cumulative hook coverage: 33 of ~42 hook scripts now have unit tests (78%).

**Up next:** Phase 3 covers the orphan hooks (the 18 unwired). Phase 4 chain integration tests. Phase 5 contract tests. Phase 6 post-shutdown automation.

## Cross-references

- A3 — predecessor (effectiveness only)
- A25 — settings.json health (hook chain inventory)
- A26 — script-design refs (hook-chain-composition.md provides the test framework reference)
- G9 — testing toolbelt (this is its hook-domain extension)
- G11 — session bundle (Phase 6 coordination)
- A29 — FLOOR rule (this BG queues; will launch when ceiling opens)
- A30 + G10 — graceful shutdown (Phase 6 lives at this lifecycle stage)
- `agentic-testing-discipline/references/testing-toolbelt-taxonomy.md` — tier system applied

## Lessons (preliminary)

- Hooks without tests are invisible-failure surfaces. A3 caught effectiveness; A32 catches CORRECTNESS.
- Post-shutdown is the right timing — cache is warm; user is gone; bundle analyzer + hook audit are siblings, not competitors.
- The shared post-shutdown-runner shim becomes a pattern for future end-of-session async jobs.
- Once Phase 6 lands, every shutdown produces TWO data products: bundle (G11 retrospective) + hook-audit-report (A32 health). Cross-session trend detection becomes powerful.
