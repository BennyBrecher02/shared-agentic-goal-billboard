---
goal_id: G16
title: Multi-orchestrator slam-dunk wave — 5 SLAM-tier integrations enabled by M2 going live
status: paused
superseded_by: G17
paused_reason: collapsed into G17 umbrella (user decision 2026-05-29). The 5 SLAM-tier slam-dunks stay P0 WITHIN G17 (also P0) + detailed in m2-ownership-tradeoff-analysis.md; this stops redundant goal-level double-tracking. Reversible.
track: infrastructure
priority: P0 (elevated per user 2026-05-28T16:10Z: "slam dunks sounds great, make sure theyve been moved upin priority on goal bilboard")
created: 2026-05-28T16:12Z
updated: 2026-05-29
serves_northern_star: G2  # all 5 slam-dunks accelerate Evium Phase 5+ throughput
guiding_light: false
umbrella_parent: G17  # 2026-05-28 A74-D2: G17 declares absorbs_goals:[G16]. Relationship documented; pause/merge of THIS goal deferred to user (G16 was explicitly P0-elevated 2026-05-28T16:10Z — state change is user's call, not auto-executed from audit recommendation).
linked_plans:
  - context/markdowns/plans/multi-orchestrator-new-subscription-plan.md (parent)
  - context/markdowns/plans/m2-ownership-tradeoff-analysis.md (Model C ownership)
  - context/markdowns/research/rabbit-holes/RH-018-m2-multi-orchestrator-connectivity-slam-dunks/README.md (catalog)
  - context/markdowns/plans/dashboard-archive-full-upgrade-plan.md (slam-dunk #2 detail)
  - context/markdowns/plans/TODO-after-weekly-limit-resets.md (adjacent throughput plan)
linked_audits:
  - A72 — Anthropic cost comparison + money-tracking (slam-dunk #4 implements A72 Tier 1+2)
  - A73 — Emulator coverage justification (slam-dunk #1 matrix shard partition depends on this)
linked_refs:
  - .claude/skills/agentic-quality-discipline/references/multi-orchestrator-inbox-conventions.md
  - .claude/skills/agentic-quality-discipline/references/rollback-discipline.md
  - feedback_idle-capacity-furthering (A63 — slam-dunk #3 maps directly)
linked_bugs: []
phase: design-complete; waiting on Anthropic subscription + M2 setup (Phase 0 of parent plan); 5 SLAM-tier slam-dunks ready to fire in priority order once M2 live
shrink_back_path: true  # ALL slam-dunks must have a "fall back to M4-only" recovery path per user explicit constraint
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G16 — Multi-orchestrator slam-dunk wave

## Why P0 elevated

User explicit (2026-05-28T16:10Z): *"slam dunks sounds great, make sure theyve been moved upin priority on goal bilboard"*.

The 5 SLAM-tier slam-dunks from RH-018 each have leverage ≥6.0 and each directly accelerates G2 (Evium Phase 5+ throughput). Together they form a coherent "M2 productivity stack."

## The 5 SLAM-tier slam-dunks (priority order)

| # | Slam-dunk | Leverage | What M2 does | Parent ref |
|---|---|---:|---|---|
| 1 | **Matrix shard runner** | 9.0 | Partition Playwright projects M4↔M2; ~50% wall-clock cut per UI change | `feedback_device-matrix-discipline` |
| 2 | **Dashboard UI builder (Full path W3)** | 8.5 | Plow dashboard upgrade chunks 2+4+5 while main does Evium | `plans/dashboard-archive-full-upgrade-plan.md` |
| 3 | **Background research BG farm** | 7.5 | Long-running research dispatches per A63 idle-cap on M2's pot | `feedback_idle-capacity-furthering` |
| 4 | **Cost-tracker daemon host** | 6.5 | A72 Tier 1+2 implementation: pricing.json + cost-tracker.py + widget | `audits/A72-...` |
| 5 | **Audit-evaluator BG farm** | 6.0 | Visual eval pipelines for sweep captures; vision-tokens on M2 | RH-018 |

## Phased plan (when M2 goes live)

| Phase | Slam-dunks | Pre-reqs | Effort |
|---|---|---|---|
| **G16-P0** | M2 connectivity (SSH + git + inbox routing) | Anthropic account + M2 setup (user gestures) | parent plan Phase 0-1 |
| **G16-P1** | Slam-dunk #1 (matrix shard runner) | M2 has npm install + Playwright installed | ~2 hr |
| **G16-P2** | Slam-dunk #4 (cost-tracker daemon) | A72 Tier 1 spec finalized; pricing.json populated | ~2 hr |
| **G16-P3** | Slam-dunk #2 (dashboard UI builder) | Chunks 1+3 done by main; M2 owns 2+4+5 | ~5-6 hr M2-time |
| **G16-P4** | Slam-dunk #3 (research BG farm) | Research queue file convention established | ~1 hr setup |
| **G16-P5** | Slam-dunk #5 (audit-evaluator BG farm) | Visual-quality-capture pipeline wired to M2 | ~3 hr |

**Sequencing rationale** (expanded per user 2026-05-28T16:18Z: *"this was a bit breif of an explanation but sounds like overall the right move so sure"*):

The order isn't just "cheapest first" — it's structured by **what each phase unblocks for the next**, plus risk-front-loading:

1. **P1 Matrix shard runner FIRST** because (a) it's the most-tested workload (Playwright already runs everywhere; just partitioning), (b) it's the cheapest validation that M4↔M2 coordination actually works — if shard partition fails, we learn it on a low-stakes workload before anything depends on M2, (c) it produces an immediate visible win (matrix wall-clock cut) the user can see and validate, (d) it requires zero NEW substrate beyond what RH-018 already designed.

2. **P2 Cost-tracker daemon SECOND** because (a) once we're spending double ($400/mo for 2× Max 20x), we need real-time visibility into burn-rate per account BEFORE building anything that could blow the budget, (b) A72 Tier 1/2 designs are already done; this is implementation only, (c) the cost-widget integrates with P3's dashboard work; landing it before P3 means the dashboard chunk lands a complete artifact, (d) failure mode is observable not destructive — if cost-tracker breaks, we still have Claude Code; we just lack visibility.

3. **P3 Dashboard UI builder THIRD** because (a) the dashboard hookup is the workflow the user explicitly chose Full path on — it's a real promise to deliver, (b) main agent does chunks 1+3 (vq-capture patches + _index.json hookup) in any short window; M2 takes chunks 2+4+5 (gallery + lightbox + cost widget) which together = ~5-6 hr of M2-time, (c) per-commit linkage unlocks the rollback workflow user described in #3 of the previous turn — high real-value, (d) doing this BEFORE the research/eval farms means the dashboard can SHOW their outputs visually.

4. **P4 Background research BG farm FOURTH** because (a) it needs the inbox lane discipline (already in place) plus a "research-only" predicate to filter what M2 picks up — small new logic, (b) lower urgency than P1-P3 because main can do research today on its own pot, just less efficiently, (c) by P4 we have measured M2's actual concurrent-workload capacity (from P1+P2 running together) so we know if 3-cap is right.

5. **P5 Audit-evaluator BG farm LAST** because (a) it's the highest-risk workload — visual eval is one of the largest token burns, vision-prompting needs care, results need cross-validation, (b) we want M2 to have established its discipline track record (1 week of clean P1-P4 operation) before trusting it with the most-token-heavy job, (c) by P5, the cost-tracker (P2) gives us real numbers to validate the savings claim — we can SEE whether moving visual eval to M2 actually freed main's pot, (d) if anything goes wrong with M2's visual eval (subtle wrong calls, hallucinated defects), we want a robust shrink-back fallback already proven.

**Anti-pattern guard**: don't reorder P1 to last "because matrix is boring." The matrix shard is the **calibration shot** — it proves multi-orchestrator works before anything depends on it. Skipping it = building on unverified foundation.

**Anti-pattern guard 2**: don't bundle phases (e.g. "do P1+P2 together"). Each phase gets ≥1 deliverable cycle to expose issues before next phase compounds them.

## Shrink-back path (user explicit requirement)

User (2026-05-28T16:10Z): *"we also may want to shrink back to just one main m4 orchestrator too so this all needs a lot of care and research"*

Every slam-dunk MUST have a clean shrink-back-to-M4-only fallback:

| Slam-dunk | Shrink-back trigger | Recovery action |
|---|---|---|
| #1 Matrix shard | M2 offline / unreachable | Main runs full matrix on M4 (current state); no data loss; wall-clock back to baseline |
| #2 Dashboard builder | M2 offline | Main picks up remaining chunks; staged feature branches still valid via git |
| #3 Research BG farm | M2 offline | Main runs research BGs on M4's pot (current state); slower but functional |
| #4 Cost-tracker daemon | M2 offline | Cost-tracker is cron-able on M4 too; ledger location is git-tracked; daemon swap is mechanical |
| #5 Audit-evaluator | M2 offline | Main runs visual eval (current state); slower but functional |

**Core principle**: M2 is an ACCELERANT, never a load-bearing dependency. If M2 vanishes, system degrades to M4-only without breakage. This is the architectural constraint that maps to **Liskov Substitution** + **Dependency Inversion** (M2 implements an interface; M4 is the default implementation).

## SOLID alignment (architectural skills validation)

Per user requirement: "conforms to our architecural skills like solid etc when it makes sense to so we can expand cleanly in the future if need be."

| Principle | How G16 honors it |
|---|---|
| **S** — Single Responsibility | Each slam-dunk has ONE deliverable + ONE exit criterion (per A47 BG lifecycle Rule 3) |
| **O** — Open/Closed | New slam-dunks can be added without modifying existing ones (additive design) |
| **L** — Liskov Substitution | M2 ↔ M4 interface (per cluster Resource.host abstraction); either node runs the workload |
| **I** — Interface Segregation | Slam-dunks 1-5 are independent — adopting one doesn't force adopting others |
| **D** — Dependency Inversion | Workloads depend on the SCHEDULER abstraction, not on specific hardware; M4-only is the default implementation |

Cross-ref: `agentic-script-design` skill SOLID-for-scripts ref + cluster-readiness.md "SOLID architecture requirements" section.

## Acceptance criteria for "G16 complete"

- All 5 slam-dunks operational on M2
- All 5 have verified shrink-back-to-M4 paths (each tested at least once)
- A62 acid-test pass: hardware + software + biology + VERIFY for the multi-orchestrator substrate
- Sustained 1-week run with no main-vs-M2 conflicts in git or inbox
- Token-pot savings measurable (cost-tracker shows main's burn drops vs pre-G16 baseline)

## Risks (rolled up from RH-018 anti-patterns)

| Risk | Likelihood | Mitigation |
|---|---|---|
| M2 overcommits — RAM exhaustion | Medium | Concurrent-workload cap at 3 (per RH-018 anti-pattern guard) |
| Git conflicts between M4 + M2 on `main` | Low | M2 NEVER pushes to `main`; only feature branches; main reviews |
| Cost-tracker becomes the bottleneck | Low | Cron interval ≥15min; never per-tool-call |
| Slam-dunks couple unexpectedly | Low | Designed independent (ISP); test each in isolation |
| User decides to shrink back mid-G16 | Documented above | Each slam-dunk has explicit fallback |

## Open decisions inherited from prior turns

(See "What needs deciding" section in the response to user 2026-05-28T16:10Z)

## Cross-references

- See linked_plans + linked_audits + linked_refs frontmatter above
