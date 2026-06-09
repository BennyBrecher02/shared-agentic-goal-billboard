---
audit_id: A77
title: Idea-conflict-check capability — make A76-style "does this conflict with current + stockpiled-future?" a built-in, reusable system gate
status: verified_complete
catalogued: 2026-05-29T00:00Z
priority_when_run: P1
trigger: 2026-05-28 — user: *"the same type of audit workflow i just asked you to make for this big cursor type implementation [A76] that would make this kinda question built into our system etc flesh this out as itll make sure we arent adding conflicting ideas as we have been stockpiling ideas all week"*
deferral_reason: NONE — explicit "flesh this out" + a real governance gap (a week of stockpiled ideas with no conflict gate)
serves_northern_star: G2
belongs_to_goal: G4
related:
  - context/markdowns/goal-billboard/audits/A76-semantic-index-adoption-downside-analysis.md (the one-off instance to generalize)
  - .claude/skills/agentic-quality-discipline/references/impact-domino-analysis-discipline.md (the existing triage layer this extends)
  - .claude/workflows/model-upgrade-benchmark.js + design-import.js (the reusable-workflow precedent)
findings:
  - the-gap: A76 was a one-off (hand-built downside analysis for the semantic-index idea). We have NO reusable mechanism to ask "does idea X conflict with (a) the current system or (b) the stockpile of designed-but-unbuilt ideas?" before adopting. With ~76 audits / 100+ plans / 15 goals / 4 pins / multiple RHs accumulated this week, conflict/duplication risk is real + growing.
  - relationship-to-impact-domino: the impact-domino-analysis discipline triages NEW ideas by leverage. It does NOT check for CONFLICTS against the existing stockpile. A77 is the missing conflict/dedup gate — it composes WITH impact-domino (domino = "is this worth it?"; conflict-check = "does this fight something we already have/plan?").
---

# A77 — Idea-conflict-check capability

## What the user wants

Generalize the A76 pattern (which was a bespoke downside/conflict analysis for ONE idea — the semantic index) into a **reusable, built-in gate**: before adopting any new idea, automatically check it against (a) the current live system and (b) the stockpile of designed-but-not-yet-built ideas. The motivation: "we have been stockpiling ideas all week" — and nothing currently catches when a new idea conflicts with, duplicates, or contradicts one already in the pile.

## Three deliverables

1. **The reusable capability** — a Workflow script `.claude/workflows/idea-conflict-check.js` (given a new idea, fans out: idea-vs-current-system + idea-vs-each-stockpile-cluster → conflict/dup/contradiction report) + a skill ref documenting the process. (BG-2)
2. **A first-pass scan of the EXISTING stockpile** — apply the methodology now to the week's accumulated ideas (plans/pins/audits/goals that are designed-but-unbuilt) → surface any ALREADY-conflicting/duplicate/contradictory ideas. Directly addresses the user's worry. (BG-3 → findings/A77-stockpile-conflict-scan.md)
3. **Integration** — where this gate fires: composes with impact-domino (triage), goal-billboard (proposal gate), and the "feature/idea persistence" standing protocol. Documented in the skill ref.

## Why it composes (not duplicates) existing discipline

| Existing layer | What it does | Gap A77 fills |
|---|---|---|
| impact-domino-analysis | scores a new idea's leverage | doesn't check conflicts vs stockpile |
| goal-billboard proposal gate | "ask before adding a goal" | doesn't check the idea against existing goals/plans for contradiction |
| A76 (one-off) | downside analysis for ONE idea | not reusable |

A77 = the reusable conflict/dedup gate that runs BEFORE an idea graduates from "stockpiled" to "building."

## Findings

_(populated as BG-2 build + BG-3 stockpile-scan return)_

## Cross-references

- A76 (the instance generalized here)
- `impact-domino-analysis-discipline.md` (sibling triage layer)
- `feedback_standing-protocols.md` (feature/idea persistence section)
- `dynamic-workflow-adoption.md` (the conflict-check is a read-only fan-out — a native-workflow fit)

## Findings + capstone (2026-05-29)

### The capability — BUILT
`.claude/workflows/idea-conflict-check.js` + `idea-conflict-check-discipline.md` skill ref landed + registered. 8 skeptic clusters across 2 axes (vs-current / vs-stockpiled-future). Composes with impact-domino (worth it?) → conflict-check (does it fight what we have/plan?) → goal-billboard (user yes?). Generalizes the A76 one-off into a reusable gate.

### The first run (stockpile scan) — pile is RECONCILABLE, not broken
11 collisions, all doc/decision-fixable, none block G2. ALL real collisions cluster in the 2026-05-26→28 multi-orchestrator/cluster/local-AI research burst — the signature of fan-out that out-ran its consolidation pass.

### Reconciliation status (this turn)
| Collision | Severity | Resolution | Status |
|---|---|---|---|
| BG cap 5-vs-8 (skill ref + dispatch-bg.sh lagged at 5; memory said 8) | CRITICAL | Updated bg-dispatch-architecture.md + dispatch-bg.sh → 8 (matching the user's decided truth + memory) | ✅ FIXED (agent — reconciling docs to a decided fact) |
| RH-020 #1 semantic-index vs G12/RH-017 RAG stack | HIGH | Supersession stamp on RH-020 part-3 (subsumed into G12) | ✅ FIXED |
| G7/G16/G17 triple-count (G17 absorbs but all 3 active = 188% goals) | HIGH | **USER DECISION** — pause G7+G16 under G17? (G7 is guiding-light, G16 was P0-elevated — agent won't unilaterally move them) | ⏳ SURFACED |
| daemon/RAG/node-assignment knot (jobs in both RH-018+RH-019; daemons homed 3 places) | HIGH | **USER DECISION** — needs one authoritative node-assignment table under G17 | ⏳ SURFACED |
| phantom pin count (index narrates 16, disk has 5) | LOW | likely table-rows-vs-detail-files by design; verify, don't force | ⏳ NOTED |
| ~6 MED/LOW fragmentation items | MED-LOW | documented in the scan; resolve as each goal is next touched | ⏳ NOTED |

### The prime convergence (a dedup WIN, not a conflict)
**coverage-audit.js = RH-021's #1 swarm app = RH-018's slam-dunk #1 = G17's matrix-shard win.** Three names, ONE artifact. Build once, serves all three. This is the conflict-check catching duplication BEFORE three separate builds.

### Meta-lesson
A 3-day research burst generated huge value but out-ran consolidation — the exact failure A77 now gates. Going forward: non-trivial ideas run through idea-conflict-check before graduating stockpiled→building.

status → verified_complete (capability built + first-run done + reconciliation acted/surfaced)
