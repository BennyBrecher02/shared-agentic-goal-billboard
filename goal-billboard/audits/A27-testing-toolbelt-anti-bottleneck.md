---
audit_id: A27
title: "Testing toolbelt + anti-bottleneck discipline (user stressed 25× — never a time/token bottleneck)"
status: in_progress
catalogued: 2026-05-27T00:18:00Z
priority_when_run: P0
estimated_effort: large (multi-week; cluster-integrated phased rollout)
trigger: 2026-05-26 23:18Z — user repeated 25× *"i dont ever want testing becoming a time/tokencost bottleneck."* Build a TDD workflow + full testing toolbelt iteratively over the cluster/sharding/scheduler/opsys push. Pull from diverse testing mindsets beyond TDD. Use cluster substrate (G7) to distribute test load.
deferral_reason: NONE — running immediately. The user's stress level is the highest registered today. The cluster work that just landed (BG #80 Phase 1 foundation) is the right substrate to integrate testing with.
related_goals: [G9, G7, G2]  # G9 being proposed in this turn
related_plans: [context/markdowns/plans/automation/testing-toolbelt-plan.md]
serves_northern_star: G2
belongs_to_goal: G9
serves_guiding_light: G9
related_refs:
  - .claude/skills/agentic-testing-discipline/SKILL.md (NEW this turn)
  - .claude/skills/agentic-testing-discipline/references/testing-toolbelt-taxonomy.md (NEW)
  - .claude/skills/agentic-testing-discipline/references/anti-bottleneck-strategies.md (NEW)
  - .claude/skills/agentic-device-testing/SKILL.md (existing; narrow scope = device matrix; complements but doesn't subsume)
findings: []
---

# A27 — Testing toolbelt + anti-bottleneck discipline

## Why this audit matters

User explicit (repeated 25×): *"i dont ever want testing becoming a time/tokencost bottleneck."* The repetition IS the spec. Every design decision in this audit and its plan must serve that constraint above all else.

Current testing reality:
- `tests/` directory: 5 Playwright `.spec.ts` files (a11y, rail, responsive, visual, visual-quality-capture)
- npm scripts: `test:e2e`, `test:vq:capture`, `test:lh`, `test:a11y`, `test:e2e:report`
- Scheduler matrix: 12-project Playwright run (per `cluster-readiness.md` math)
- **A11 catch**: full matrix is 15+ min wall-clock on single Mac
- **No unit tests** for scripts/ Python or shell logic (~30 hooks, ~10 scheduler files, ~15 utility scripts — all untested)
- **No type checks** beyond what TypeScript / `astro check` provides
- **No mutation testing** (we don't verify tests would catch bugs)
- **No property-based / fuzz testing** (we don't generate inputs)
- **No contract testing** (subsystem interfaces unvalidated)

The gap: substantial logic (hooks, scheduler, heartbeat, cluster, render scripts) ships without test coverage. Adding coverage WITHOUT discipline = bottleneck.

## The 14-type testing toolbelt taxonomy

Each type with cost rating (TIME, TOKEN) on a fast/medium/slow scale:

| Type | Description | TIME | TOKEN (when run via BG) | Best for |
|---|---|---|---|---|
| **Static analysis** | Type-check, lint, shellcheck | fast | fast | All scripts; CI-gate |
| **Unit tests (TDD)** | Single function/component, mocked deps | fast | fast | Pure logic (Python utilities, parsers) |
| **Snapshot tests** | Captured state vs current state | fast-medium | fast | Visual regression (Playwright), JSON shape stability |
| **Property-based** | Generated inputs vs invariants (Hypothesis) | medium | fast (local) | Edge case discovery in pure functions |
| **Mutation testing** | Mutate code; verify tests catch it (mutmut, Stryker) | slow | low (offline) | Coverage quality |
| **Contract tests** | Interface stability between components | fast-medium | fast | Hook chain composition (input/output schema) |
| **Integration tests** | Multi-component flows | medium | medium | Scheduler + heartbeat coordinator |
| **E2E tests** | Full user journey | slow | medium | The Playwright matrix |
| **Fuzzing** | Random inputs, find crashes | slow (offline) | low | Bash parsers, JSON ingest |
| **Chaos engineering** | Inject failures | slow | medium | BG dispatch failure recovery, lost-work, network drops |
| **Smoke tests** | Quick pre-deploy sanity | fast | fast | Hook health check, server up |
| **Load/perf tests** | Throughput, latency under load | slow | medium | Heartbeat tick performance, scheduler concurrency |
| **Affected-tests-only** | Diff-aware: only tests touching changed files | matches subset | matches subset | Every change; the velocity unlock |
| **Cached test results** | Skip if inputs unchanged | matches cache hit | matches cache hit | All deterministic types |

## 10 Anti-bottleneck strategies

1. **Affected-tests-only mode** — diff-aware test selection. Build `scripts/test-affected.sh` that maps file paths → tests. Don't run tests that don't touch changed code.
2. **Test result caching** — input-hash → result cache. If inputs unchanged since last green, skip. Cache schema `{input_hash, test_id, verdict, ts}` in `.claude/cache/test-results/`.
3. **Tier system** (P0/P1/P2/nightly):
   - **P0 fast (run every change)**: static analysis, unit tests, smoke. <30s total.
   - **P1 medium (run per PR or batch)**: contract, integration, snapshot. <5min.
   - **P2 slow (run nightly or on-demand)**: full E2E matrix, mutation, fuzz, chaos.
4. **Cluster-distributed execution** — once cluster Phase 2 lands, fan out P1+P2 to remote nodes. Each Mac contributes ~3 shards (per A11); cluster of 3 nodes ≈ 9 shards parallel.
5. **Lazy fixture loading** — don't spin up Astro dev server unless test needs it.
6. **Failure-budget halt** — if >N tests fail, halt rest. Don't waste tokens on cascading failures.
7. **Smart retry** — flaky-test detection + retry-once policy (Playwright already has this; codify).
8. **Cost-cap per BG dispatch** — testing-BG token budget capped (`MAX_TESTING_TOKENS` env var); abort if exceeded.
9. **Local-first** — prefer local execution over BG dispatch for tests <30s. Save BG for slow stuff.
10. **Token-cost-aware test selection** — annotate tests with `estimated_tokens` frontmatter; aggregate before run; abort if budget exceeded.

## Cluster integration (G7 ↔ G9)

The cluster (G7) provides the substrate. Testing (G9) is its first big consumer:
- Cluster Phase 1 (just landed) — `scripts/cluster/dispatch.sh` placeholder works locally
- Cluster Phase 2 (hardware-gated) — SSH dispatch ready
- Testing Phase 5 — uses cluster dispatch to fan out test shards across nodes
- **Result**: full Playwright matrix in ~5min wall-clock (vs 15+ min single-Mac) once cluster is real

Testing doesn't WAIT for cluster Phase 2 — Phases 1-4 land first on single-machine + cache + tier system. Phase 5 unlocks when cluster is real.

## Mindsets beyond TDD (per user request)

User said: *"pull from other testing mindsets other than tdd because thats just a start."* Captured in the toolbelt taxonomy above. Brief framing:

- **TDD** — Red/Green/Refactor; tests-first discipline
- **BDD** — Given/When/Then; user-facing scenarios (Playwright e2e already does this implicitly)
- **Property-based** — Invariants over generated inputs (Hypothesis-style for our pure Python)
- **Contract** — Interface stability across components (hook chain composition is full of contracts)
- **Mutation** — Verify tests catch real bugs (codifies "test quality")
- **Fuzz** — Random inputs find crashes (bash parsers + JSON ingest)
- **Chaos** — Inject failures (BG dispatch survives network drops? Lost-work recovery actually fires?)
- **Visual regression** — Snapshot UI (already used via Playwright `toHaveScreenshot`)

Different mindsets catch different bugs. The toolbelt assembles them with cost-discipline.

## Phasing (see plan)

See `plans/automation/testing-toolbelt-plan.md`. 5 phases over multiple weeks; each gate has cost ceiling.

- **Phase 1 (this turn):** Foundation — A27 audit + plan + skill scaffold + G9 goal + memory rule + 3 BGs (substrate baseline, affected-tests, test caching)
- **Phase 2:** Tier system + scheduler integration
- **Phase 3:** Property-based + contract + mutation tests added
- **Phase 4:** Chaos + fuzz + load tests added
- **Phase 5:** Cluster-distributed execution (unblocks at G7 Phase 2)

## Cost gates (CRITICAL — user 25× emphasis)

Before each phase advances:
1. Total tokens spent on testing infrastructure in current week < X (TBD; baseline from BG A)
2. Median P0 test suite time <30s; P1 <5min
3. No single test > 60s wall-clock (else: refactor or move to nightly tier)
4. Cache hit rate ≥50% after warmup
5. Affected-tests-only correctness: ≥95% of changes only run the right subset

If any gate breached: PAUSE that phase + reassess. Testing infra ITSELF doesn't get to be the bottleneck.

## Verification

After Phase 1 (this turn):
1. G9 goal lives in active/
2. Skill ref exists with frontmatter
3. testing-toolbelt-plan.md exists
4. 3 BGs report back: (A) cost baseline; (B) affected-tests scaffold; (C) cache schema scaffold
5. Memory rule mentions 25× stress + cost gates

After Phase 5 (months out):
6. Full matrix runs in <5min wall-clock via cluster
7. Average change runs ≤10% of tests (affected-only working)
8. Cache hit rate >70% in steady state
9. 14-type toolbelt has at least one test per type (proof of concept; not full coverage of all code)

## Status

IN PROGRESS — Phase 1 landing this turn. Phase 1 BGs launching in parallel.

## Cross-references

- A11 — single-machine concurrency ceiling (testing's biggest single-machine bottleneck)
- A14 — token-burn analysis (testing must factor in here)
- A21 — parallel BG ceiling (testing-BG count also subject to this)
- A22 — cluster + NS integration (G7 is testing's substrate)
- G7 — cluster goal (testing depends on it eventually)
- G9 — new testing goal (this audit's owner)
- `agentic-device-testing/SKILL.md` — narrow (device matrix); complements; doesn't replace
- `agentic-script-design/references/opsys-scheduling-patterns.md` — tier system patterns
