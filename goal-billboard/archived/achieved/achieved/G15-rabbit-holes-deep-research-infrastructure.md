---
goal_id: G15
title: Rabbit holes — deep research infrastructure (methodology-disciplined wide surveys driving system upgrades)
status: achieved
archived: "2026-06-21T21:01Z — ARCHIVED→achieved (v1) per user-approved goal-grooming triage (goal-grooming-proposal-2026-06-21.md §4-B). Two-part v1 'done' bar MET: (1) framework shipped — methodology ref + rabbit-holes/ folder discipline + catalog + research/methodology/ corpus all landed; (2) spawn demonstrated — RH-015 → G19 (the exit criterion explicitly cites this as already-satisfying). 2026-05-31 rewrite confirmed present in body (scope paragraph + two-part exit-criteria + History). Annual-refresh cadence stands as the follow-on. Moved active/ → archived/achieved/."
track: quality
northern_star: false
guiding_light: true
priority: P1
created: 2026-05-27T02:59:02Z
updated: 2026-06-21T21:01Z
serves_northern_star: G2
linked_plans:
  - context/markdowns/plans/automation/rabbit-hole-research-framework-plan.md
linked_audits:
  - A35 — Rabbit holes research framework + agentic-workflow survey (origin)
linked_skills:
  - .claude/skills/agentic-quality-discipline/references/research-folder-discipline.md (v2; A35 extends with rabbit-holes/)
linked_bugs: []
linked_changes: []
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G15 — Rabbit holes deep research infrastructure

## Why it exists

User vision 2026-05-27 03:25Z: *"insanely wide research rabbit hole... many of them as research subfolders... well integrated in our overall system... rabbitholes will need top teir research skills so look those up first to make sure you build right from the get go."*

This goal owns the **rabbit-holes/ subfolder + methodology-first discipline + 10-domain catalog + synthesis + implementation spawn pattern**.

**Scope (own the FRAMEWORK, not the content) — 2026-05-31:** G15 owns the rabbit-hole framework
+ the rabbit-holes/ folder discipline — the methodology-first ordering, the synthesis→spawn
pattern, the catalog, and the research-method refs. Individual rabbit-hole **content attaches to
its topical goal, not to G15** (RH-014 throughput → G17; RH-017 local-AI → G12; token-cheap-out →
G9; top-starred-GitHub → G4; RH-015 blog → G19; reference-design → G2). What lands under G15 is the
framework itself + the shared `research/methodology/` corpus.

Rabbit holes are EXPLORATORY wide surveys distinct from per-domain research files. They drive multiple downstream artifacts (audits, plans, skill refs, hooks, goals) through synthesis.md outputs.

## Why it's load-bearing

We've built heavily today (**36 audits, 13 active goals, 87 skill refs, 56 plans** — verified counts as of 2026-05-27T03:17Z per A37 Phase 1; original wording cited stale counts "33/15/95+/33+") but ALL from INTERNAL perspective. Risk: re-inventing wheels, missing established patterns, blind to recent breakthroughs.

G15 opens field-of-view to:
- Top-starred GitHub agentic systems
- Non-GitHub commercial / blog-documented systems
- Academic papers (2024-2025)
- Multi-agent / cluster / memory architecture patterns
- Cost-discipline + tool-use patterns

Each rabbit hole's synthesis produces concrete proposals → spawns audits/plans/etc.

## Current state

- Phase 0 ✅ LANDED 2026-05-27: methodology ref at `research/methodology/top-tier-research-discipline.md` (31,994 bytes / 691 lines / 15 sources cited / PRISMA + snowballing + COPE adapted)
- Phase 1 ✅ LANDED 2026-05-27: RH-001 (top-starred GitHub agentic; 9 files, 28 sources, 12 candidates) + RH-002 (non-GitHub agentic; 9 files, 22 sources, 7 candidates)
- **Phase 2 user-driven re-route (2026-05-27T03:42Z)**: instead of the originally-planned RH-003 + RH-004-7, user explicit ask for "another dual rabbithole" launches RH-011 (Pass 2 deep on token-cheap-out — Pass 1 prompt-cache finding goes DEEPER) + RH-012 (local AI cluster swarms — linked to A33/G12/G13). Originally-planned RH-003 + RH-009 swap question STILL pending.
- Phase 2 ✅ LANDED 2026-05-27: RH-011 token cheap-out Pass 2 (9 artifacts, 26 sources, **15-40% weekly token savings potential**, Batch API + cache stacking = 5% of standard pricing on eligible work) + RH-012 local AI cluster swarms (8 artifacts, 25 sources, **EXO substrate identified as right fit, 1.9× speedup on 4 nodes**, hardware routing matrix M4=vision/M2=code/RPi=triage)
- Phase 2 EXT ✅ LANDED 2026-05-27T05:11Z: RH-013 reference-site design pattern extraction (8 files / 1821 lines / 14.8min wall-clock / sentinel-emitted; 8 skill refs proposed under `agentic-webdesign/references/design-system/` ranked by ROI×effort; top finding = "premium feel = high effort-to-visible-area ratio × disciplined restraint"; brand-asserts-through-chrome not loudness). Paired P1-07b brainstorm BG landed 05:06Z (12.3min; 5 redesign options; top = "The Calculator" with outcome-first hierarchy + zero new JS/images, ~2.5-3hr implementation).
- Phases 3-5: queued per master plan

## Methodology BG's plan-change recommendation (PENDING USER)

The Phase 0 methodology BG recommended RH-009 (hook/event-driven architectures) instead of RH-003 (multi-agent orchestration) as the third Phase 1 rabbit hole. Rationale: RH-009 directly extends our existing A26 (script design) + A31 Front 2 (event bus) investment. Plan-change deferred to user — not changing unilaterally.

## Blockers

- Phase 1 completion (third RH): pending user decision on RH-003 vs RH-009 swap
- Phases 2-3: depend on Phase 1 completion
- Phases 4-5: synthesis depends on all rabbit holes

## Next action

- Await RH-001 + RH-002 BG completion (token budget ≤500k each; <30 min wall-clock each)
- Surface RH-003 vs RH-009 decision to user
- Phases 2-3 follow per plan

## Exit criteria

"Done" for v1 is two-part — own the framework, and prove it spawns downstream work:

1. **Framework shipped:** the methodology ref + the rabbit-holes/ folder discipline + the catalog
   are in place (the shared `research/methodology/` corpus lands under G15).
2. **Spawn demonstrated:** ≥1 rabbit hole has demonstrably spawned a downstream audit/plan/goal
   (RH-015 → G19 already satisfies this).

When both hold, **G15 → `achieved` for v1**, with an **annual-refresh cadence** as the standing
follow-on.

## History

- 2026-05-31: Goal-definition rewrite applied per user's 2026-05-31 chamber decision (goal-rewrite-drafts.md G15): added the "own the FRAMEWORK, not the content" scope paragraph to "Why it exists" (framework + folder discipline + methodology corpus stay under G15; RH content attaches to its topical goal — RH-014→G17, RH-017→G12, token-cheap-out→G9, top-starred-GitHub→G4, RH-015→G19, reference-design→G2); restated Exit-criteria as the two-part v1 "done" (framework shipped + ≥1 RH spawned a downstream artifact, RH-015→G19 already satisfies) with annual-refresh cadence as the standing follow-on. No link changes (the linkage pass already routed RH content to topical goals). Links + history otherwise untouched.
- 2026-05-27 03:30Z: G15 created. A35 audit + 5-phase plan + memory rule + research/rabbit-holes/ subfolder + research/methodology/ subfolder landed. Phase 0 methodology BG launching this turn. 10 initial domain rabbit holes catalogued and queued.
- 2026-05-27 ~04:30Z: Phase 0 methodology BG LANDED — `research/methodology/top-tier-research-discipline.md` (31,994 bytes, 691 lines, 15 sources cited, PRISMA + snowballing + COPE-adapted, Tier-1→5 source-quality system, 6 named AI-specific challenges with mitigation rules). Phase 1 launched: RH-001 (top-starred GitHub agentic) + RH-002 (non-GitHub agentic) BGs parallel. RH-003 → RH-009 swap recommendation from methodology BG deferred to user input. Ask-1 internal session eval BG also launched (parallel to RHs per A35 plan; analyzes today's artifacts for missed skill/hook ideas from external-lens perspective).
- 2026-05-27T03:42:10Z: Phase 1 BGs landed (RH-001 + RH-002 synthesized; 9+9 files; top adoption candidate prompt-prefix-caching with ~31× cache-read amortization). User explicit redirect for Phase 2: "remember when we did research on top rated agentic codebases? how did they maximize token cost saves because my weekly limit is something i wanna consider without altering our workflows tho. initiating pass 2 agentic codebase/harness reaserch rabbithole deepdive... lets do another dual rabithole for safe token cheap out maximization and also for our local ais lets aim for cluster swarms." → **RH-011 (token cheap-out Pass 2; 7 deep-dive tracks)** + **RH-012 (local AI cluster swarms; 7 deep-dive tracks, hardware-grounded for M4+M2+RPi)** launched parallel. Constraint: workflows unchanged. Both BGs cite RH-001/002 + A33 Front A/B + A14/A37 cost work as substrate.

## Notes

- G15 SERVES G2 indirectly — better external knowledge integration = better internal decisions
- The methodology-first ordering is non-negotiable (user explicit "build right from the get go")
- Per-rabbit-hole synthesis.md drives downstream artifact spawns; the rabbit hole isn't done until synthesis cross-references concrete proposals
- Synthesis cross-cutting (Phase 4) is where the BIGGEST wins emerge — patterns that span multiple systems
- After Phase 5, the cycle repeats annually with refreshed rabbit holes (field changes fast)
- A34/G14 master system audit can include rabbit-hole digest as a Module
