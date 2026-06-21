---
goal_id: G17
title: Peak development throughput infrastructure — multi-orchestrator + heterogeneous cluster + slam-dunk wave (umbrella goal)
status: active
track: infrastructure
priority: P0  # user explicit 2026-05-28T16:25Z: "make a new overarching plan/goal in the bilboard so we can truly focus on using our system to get this all done the best way possible" (malformed `priority: |-` block-scalar fixed to a plain P0 per goal-grooming-proposal-2026-06-21.md §4-D)
created: 2026-05-28T16:32Z
updated: 2026-06-21T21:01Z
status_tag: awaiting-user
gated_on: "USER gesture — (1) a 2nd Anthropic Max subscription on a new account (~$200/mo; the ONLY cost — A72 has the analysis) and (2) setup of the ALREADY-OWNED hardware (M2 old Mac + Pi #1 + Pi #2, all owned/flashed — zero capex). Effectively dormant-by-design: ZERO agent work is possible until the user acts (no sub = no M2 agent; un-wired Pis = no cluster). Any 'buy hardware' framing is WRONG. (Surfaced per goal-grooming-proposal-2026-06-21.md §4-D.)"
serves_northern_star: G2  # the WHOLE infrastructure exists to accelerate G2 Evium delivery
guiding_light: true  # this is the long-term build that compounds across future projects too
northern_star: false  # G2 still ships first
absorbs_goals: [G7 cluster, G16 multi-orchestrator slam-dunk wave]
absorbs_audits: [A72 subscription cost + money tracking, A73 emulator coverage]
linked_plans:
  - context/markdowns/plans/multi-orchestrator-new-subscription-plan.md
  - context/markdowns/plans/m2-ownership-tradeoff-analysis.md
  - context/markdowns/plans/dashboard-archive-full-upgrade-plan.md
  - context/markdowns/plans/icloud-narrow-uses-analysis.md
  - context/markdowns/plans/TODO-after-weekly-limit-resets.md
  - context/markdowns/plans/automation/cluster-readiness.md
  - context/markdowns/plans/automation/cluster-kickoff-plan.md
  - context/markdowns/plans/automation/m2-ssh-scheduler-integration.md
linked_research:
  - context/markdowns/research/rabbit-holes/RH-018-m2-multi-orchestrator-connectivity-slam-dunks/README.md
  - context/markdowns/research/rabbit-holes/RH-019-pi-sub-cluster-heterogeneous-compute/README.md
  - context/markdowns/research/rabbit-holes/RH-014-async-throughput-and-token-saves/README.md
linked_refs:
  - .claude/skills/agentic-quality-discipline/references/multi-orchestrator-inbox-conventions.md
  - .claude/skills/agentic-quality-discipline/references/rollback-discipline.md
  - .claude/skills/agentic-script-design/references/heterogeneous-cluster-design.md (NEW)
  - .claude/skills/agentic-script-design/references/distributed-systems-patterns.md
linked_audits:
  - A11 — concurrency-ceiling (single-machine bottleneck)
  - A21 — parallel capacity (single-machine BG ceiling)
  - A22 — cluster + NS integration gap (origin)
  - A65 — autonomic system framework (8-daemon catalog)
  - A72 — Anthropic subscription cost + money tracking
  - A73 — emulator coverage justification
phase: phase-0 — absorbed G7+G16 (2026-05-29; both paused/ under this umbrella); pending only the 2nd Anthropic Max sub + setup of ALREADY-OWNED hardware
absorbed_executed: 2026-05-29 — G7 (cluster guiding-light) + G16 (slam-dunk wave) moved to paused/ with superseded_by:G17. This is now the SINGLE active goal for all cluster/multi-orchestrator/throughput work. G16's 5 slam-dunks stay P0 within (see P2); G7's cluster vision is the heterogeneous-cluster scope.
hardware_inventory: ALL OWNED — zero capex. M4 (24GB, in use) + M2 (old Mac, 8GB, owned) + Pi #1 + Pi #2 (both already flashed). The ONLY money item is one 2nd Anthropic Max sub (~$200/mo) for M2's agent; Pis are script-only (no sub). See plans/node-assignment-table.md for the authoritative node→job map. Any "buy M2/Pis" framing is WRONG.
phases:
  - G17-P0 — 2nd Anthropic Max subscription on a new account (USER GESTURE; the only cost; A72 has the cost analysis) — NOT a hardware purchase
  - G17-P1 — M2 setup: it's the owned old Mac — Claude Code install + new-account login + git clone + npm (USER GESTURE; 1-2 hr)
  - G17-P2 — slam-dunk wave Phases 1-5, M2 active (~13-17 hr; the absorbed-G16 work, still P0)
  - G17-P3 — Pi #1 (already flashed) SSH-wire + autonomic daemon migration (slam-dunk S1; ~8-10 hr) — no purchase, just setup
  - G17-P4 — Pi #2 (already flashed) SSH-wire + heartbeat redundancy (S3) + Chromium shard farm split with Pi #1 (S2)
  - G17-P5 — Lighthouse runner + snapshot-drift detection + bug-billboard groomer + image-shrink pipeline (S4-S8)
  - G17-COMPLETE — sustained 1-week run with all 4 owned nodes contributing; cost-tracker proves savings; G2 throughput measurably accelerated
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G17 — Peak development throughput infrastructure

## Why this exists

User explicit (2026-05-28T16:25Z): *"make a new overarching plan/goal in the bilboard so we can truly focus on using our system to get this all done the best way possible"*

The 4 prior turns have generated:
- 1 audit (A72) — Anthropic subscription cost research
- 1 audit (A73) — Emulator coverage justification
- 2 research rabbit holes (RH-018 M2 connectivity + RH-019 Pi sub-cluster)
- 4 plans (multi-orchestrator + M2 ownership + dashboard upgrade + iCloud)
- 1 sibling goal (G16 slam-dunk wave)
- 1 new skill ref (heterogeneous-cluster-design)
- Multiple updates to existing memory rules + audit catalog

G17 is the **umbrella** that ties all of this together as ONE coherent initiative: build the infrastructure that lets G2 (and every future project) ship at peak throughput by using all our compute.

## What G17 absorbs

| Item | Why under G17 |
|---|---|
| G7 (cluster + true parallelization) | G17 is the broader form; G7's original 4-node math is correct but framing was narrower |
| G16 (multi-orchestrator slam-dunk wave) | G16 = the M2-portion of G17; M2 is one of 4 nodes |
| A72 (subscription cost + money tracking) | G17-P0 prereq |
| A73 (emulator coverage justification) | Component of "use compute wisely"; tighter matrix → faster sweeps → more headroom for cluster work |

G7 status → continues; G16 status → continues. Both fold under G17 as sub-goals, not abandoned.

## The end-state vision (G17-COMPLETE)

When G17 is complete, the system has:

1. **4 cooperating nodes**: M4 (primary orchestrator) + M2 (worker agent) + Pi #1 (autonomic daemon host) + Pi #2 (heartbeat + Chromium shard farm)
2. **2 Anthropic accounts** with $400/mo combined giving 40× Pro-equivalent independent pots (per A72 recommendation)
3. **Cost-tracker live** showing per-account burn vs cap in dashboard widget
4. **Matrix wall-clock cut to ~1/3** via 4-node Playwright shard farm (per RH-019 capability-asymmetric split)
5. **8 autonomic daemons running 24/7** on Pi #1 (memory-consolidator, cost-tracker, bug-billboard-groomer, etc — per A65 catalog)
6. **Heartbeat ticks 24/7** via Pi #2 redundancy (M4 + M2 can sleep without losing tick — closes A45 server-death class structurally)
7. **Per-commit screenshot linkage** in dashboard (Full path complete from prior turn's choice)
8. **Cleanup**: emulator profile cap (2-per-OS) enforced; redundant sweep cycles eliminated

## Acceptance criteria for G17-COMPLETE

- ☐ A72 verification facts confirmed with Anthropic billing (subscription path)
- ☐ Second Anthropic account active on M2
- ☐ Both Macs successfully running their respective Claude Code agents simultaneously with no git conflicts for 1 week
- ☐ Both Pis powered on, SSH-reachable, running their assigned daemons
- ☐ Matrix wall-clock measured at <2 min for full 12-project run (vs ~3 min M4-only baseline)
- ☐ Cost-tracker shows both accounts' burn vs cap in dashboard
- ☐ A62 acid-test passes for the multi-orchestrator substrate (hardware + software + biology + VERIFY)
- ☐ At least 3 G2 Phase 5+ design batches landed using the new infrastructure
- ☐ Shrink-back-to-M4 fallback tested at least once per slam-dunk (planned exercise)
- ☐ Dashboard data.json refresh rate ≤30s with all 4 nodes contributing telemetry

## Risks rolled up

(Each is detailed in its parent plan/RH; this is the high-level summary)

| Risk | Likelihood | Mitigation source |
|---|---|---|
| Anthropic policy change before purchase | Low | A72 — verify with billing first |
| M2 RAM saturation (8GB) | Medium | RH-018 — concurrent workload cap at 3 |
| Pi SD card corruption | Medium | RH-019 — daemon state in git; re-flash recovery |
| Multi-node git conflicts | Low | Role-stable file zones (heterogeneous-cluster-design ref) |
| iCloud-on-.git corruption | Low | icloud-narrow-uses-analysis — explicit DO-NOT list |
| Cost overrun beyond $400/mo | Low | Cost-tracker (A72 Tier 1/2) before doubling spend |
| Cross-arch binary incompatibility | Medium | heterogeneous-cluster-design — prefer interpreted scripts |
| User decision to shrink back | Documented | Every component has M4-only fallback path |

## What G17 does NOT do

- ❌ Local AI work (user explicit defer until after Evium)
- ❌ Replace G2 — G2 is still the Northern Star; G17 is the accelerant
- ❌ Mandate any specific component — every Pi/M2 piece is optional; M4-only fallback always exists
- ❌ Commit to a deadline — G17 is parallel infrastructure; G2 ships when G2 ships

## Strategic sequencing (the actual plow order)

**Don't read this as "we now plow G17 instead of G2."** G17 happens IN PARALLEL with G2 plowing:

```
G2 (Evium Phase 5+ delivery) ───────────────────────────────────────────►
                                                                          ↑
G17-P0 (subscription)    [user gesture] ────────                         │
G17-P1 (M2 setup)        [user gesture]      ──── ────                   │
G17-P2 (G16 slam-dunks)                          [M2 active] ─────────►  │
G17-P3 (Pi #1 daemons)                              [Pi #1 setup] ────► ▼
G17-P4 (Pi #2 + heartbeat + shard farm)                  ───────────►
G17-P5 (S4-S8 slam-dunks)                                      ──────►
```

Each G17 phase that goes live makes future G2 work faster. The first 2 phases are user gestures (account + setup); the rest is implementation that absorbs idle / backstop time per Model C (W3 backstop).

## Open decisions for user (rolled up from sibling artifacts)

| # | Decision | Reference | Status |
|---|---|---|---|
| 1 | Buy 2× Max 20x on separate accounts ($400/mo) | A72 | RESEARCH-COMPLETE; user authorizes purchase |
| 2 | M2 setup timeline (same day vs staged over week) | RH-018 + Model C plan | RECOMMENDED: staged Day 1/2/3+ |
| 3 | PR/merge gating policy (human gate vs auto-merge) | multi-orchestrator plan | RECOMMENDED: human gate first 2 weeks |
| 4 | M2 concurrent-workload cap | RH-018 anti-pattern | RECOMMENDED: 3 |
| 5 | Matrix shard partition specifics | RH-018 | RECOMMENDED: M4(8) + M2(4) for dual-Mac; revise to 4-node split when Pis online |
| 6 | Both Pis exist + boot today? Hardware verification needed | RH-019 Q1 | OPEN |
| 7 | Pi storage (SD card only vs USB SSD) | RH-019 Q2 | OPEN |
| 8 | Order: Pi #1 daemons FIRST vs shard farm FIRST | RH-019 Q3 | RECOMMENDED: daemons first (24/7 lower-stakes calibration) |
| 9 | Wake-on-LAN integration | RH-019 Q4 | OPEN |
| 10 | iCloud narrow-uses (adopt all 3 / pick subset / skip entirely) | icloud-narrow-uses-analysis | RECOMMENDED: skip; revisit later |

## Cross-references

- See linked_plans + linked_research + linked_refs + linked_audits in frontmatter above
- G2 — Northern Star (the goal G17 accelerates)
- G16 — M2 slam-dunk wave (subset of G17)
- G7 — cluster + true parallelization (broader form absorbed)

## Sentinel artifacts that prove G17 is shipping (rolling list)

Updated as work lands:

- 2026-05-28: G17 created; all design artifacts staged across 4 plans + 2 RHs + 2 audits + 2 skill refs
- (pending) Anthropic billing verification call → A72 status close
- (pending) New Anthropic account active
- (pending) M2 Claude Code logged in to new account
- (pending) First M2-originated PR merged into main
- (pending) Matrix shard split active; wall-clock measurement recorded
- (pending) Pi #1 daemons running 24h cleanly
- (pending) Pi #2 heartbeat tick observed during M4 sleep
- (pending) Cost-tracker widget on dashboard shows both account burn
- (pending) G17-COMPLETE acceptance criteria all checked

