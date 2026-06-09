---
audit_id: A35
title: "Rabbit holes research framework + insanely-wide agentic-workflow survey (top-starred GitHub + non-GitHub + academic + industry)"
status: in_progress
catalogued: 2026-05-27T02:59:11Z
priority_when_run: P1
estimated_effort: very large (multi-week research sweep across ≥10 rabbit holes; output drives downstream audit/plan/skill/hook proposals)
trigger: 2026-05-27 03:25Z — user vision *"evaluate all of our previous runs data from the entire start to end day at work to analyze any skill and or hook ideas and also give me new out of the box ideas, do research for similar agentic workflows that leverage HUGE wins with simple to replicate stuff we just simply havent thought of, look at all the highest github starred agentic systems/harnesses/workflows/skillsets and more are doing and make plans and goals to implement them here, and dont stop by just github, make this an insanely wide research rabbit hole, in fact this is a chance to upgrade the research folder, lets make a new thin called rabbit holes we can have many of them as research subfolders and they should be well integrated in our overall system and also rabbitholes will need top teir research skills so look those up first to make sure you build right from the get go."*
deferral_reason: NONE — running immediately. Skill/hook ideation has been INTERNAL-perspective all session; this opens the field-of-view to external best practices.
related_goals: [G15, G4, G14]  # G15 NEW (rabbit-holes infra); G4 (skill maintenance owns); G14 (master audit consumes rabbit hole synthesis)
related_plans: [context/markdowns/plans/automation/rabbit-hole-research-framework-plan.md]
serves_northern_star: G2
belongs_to_goal: G15
serves_guiding_light: G15
related_refs:
  - context/markdowns/research/methodology/top-tier-research-discipline.md (TO BE WRITTEN BY BG THIS TURN)
  - context/markdowns/research/rabbit-holes/README.md (NEW; landed this turn)
  - .claude/skills/agentic-quality-discipline/references/research-folder-discipline.md (v2 from #109; A35 extends)
  - A15 + A31 Front 5 (research folder evolution)
findings: []
---

# A35 — Rabbit holes research framework + agentic survey

## The vision (user framing)

> "do research for similar agentic workflows that leverage HUGE wins with simple to replicate stuff we just simply havent thought of, look at all the highest github starred agentic systems/harnesses/workflows/skillsets and more are doing and make plans and goals to implement them here, and dont stop by just github, make this an insanely wide research rabbit hole."

Two distinct asks:

**Ask 1 — Internal evaluation:** analyze TODAY's session data for skill/hook IDEAS we missed
**Ask 2 — External survey:** wide rabbit-hole research across top-starred agentic systems, academic literature, non-GitHub sources

These complement: internal = "what would have helped today?"; external = "what does the field know that we don't?"

## Why this matters

We've been BUILDING heavily today (**36 audits, 13 active goals, 87 skill refs, 56 plans** — verified counts as of 2026-05-27T03:17Z per A37 Phase 1; original wording cited stale counts "33/14/95+/33+") but ALL from internal perspective. The risk: re-inventing wheels, missing established patterns, blind to recent breakthroughs.

A35 opens field-of-view to:
- Top-starred GitHub agentic systems (LangGraph, AutoGen, CrewAI, swe-agent, OpenHands, etc.)
- Anthropic blog posts + Claude Code docs
- Academic agent papers (2024-2025)
- Industry workflow patterns (Cognition's Devin, Cursor, Continue)
- Local-LLM ecosystem (overlap with A33 — different angle)
- Cost-discipline patterns from production agent deployments
- Multi-agent orchestration (academic + industry)
- Hook / event-driven agent architectures
- Agent memory architectures (vector DB vs symbolic vs hybrid)

## The rabbit-hole framework (NEW system)

User explicit: *"lets make a new thin called rabbit holes we can have many of them as research subfolders and they should be well integrated in our overall system."*

### Structure
- `context/markdowns/research/rabbit-holes/<topic-slug>/` per rabbit hole
- Each: README.md (scope + methodology + key findings) + landscape.md (broad survey) + deep-dive-*.md (per-subtopic) + synthesis.md (cross-cutting insights + implementation proposals)
- Synthesis.md drives downstream artifacts: audit / plan / skill ref / hook / goal

### Integration with research v2 (just landed via #109)
- `scripts/render-research-index.py` extended to include "Rabbit Holes" section
- SessionStart surface (per A31 Front 5) shows recent rabbit-hole entries
- Lint script validates rabbit-hole frontmatter
- Cross-link harvester picks up `findings_drove:` cross-refs

### Methodology requirement (THE prerequisite per user)
User explicit: *"rabbitholes will need top teir research skills so look those up first to make sure you build right from the get go."*

Phase 0 BG launching this turn: **research the research methodology itself**. Output: `research/methodology/top-tier-research-discipline.md`. Topics:
- Systematic literature review methodology (PRISMA framework etc.)
- Source-quality criteria (primary vs secondary; peer-reviewed vs blog)
- Citation hygiene (when to cite, how to deduplicate, primary-source preference)
- Scope-creep avoidance vs rabbit-hole-warranted depth
- Comparison-table vs narrative-review decision matrix
- Reproducibility (sources need to be verifiable)
- Output structure (intro / method / findings / discussion / implications-for-us)
- AI-specific research challenges (rapid landscape change; benchmark gaming; reproducibility crisis)

This ref becomes load-bearing for ALL future rabbit holes.

## 10 candidate rabbit holes (initial catalog)

Ordered roughly by expected ROI / impact for our system:

| # | Topic | Why this one | Expected output |
|---|---|---|---|
| **RH-001** | `top-starred-agentic-github` | The biggest external knowledge pool; user explicit | Audit + plan for borrowed patterns |
| **RH-002** | `non-github-agentic-systems` | Cognition Devin, Cursor, Continue, Anthropic blog patterns | Audit + skill ref proposals |
| **RH-003** | `multi-agent-orchestration-patterns` | We're building multi-agent; survey academic + industry | Plan(s) for orchestration improvements |
| **RH-004** | `self-improving-agent-academic` | Recent papers (2024-2025) on agents that learn | Audit candidates for our meta-monitoring layer |
| **RH-005** | `agent-memory-architectures` | Vector DB vs symbolic vs hybrid; we use symbolic — survey alternatives | Plan + memory-architecture audit |
| **RH-006** | `tool-use-at-scale` | How do agents at scale handle 100+ tools? Routing / caching / cost-discipline | Plan + skill ref |
| **RH-007** | `local-llm-ecosystems-extended` | Extends A33's landscape; deeper dive on MLX-vLLM-Ollama production patterns | Extends A33 plans; G12 acceleration |
| **RH-008** | `cost-discipline-token-economy` | How do production agent deployments manage token budgets? | Extends A14 + G9 cost framework |
| **RH-009** | `hook-event-driven-agent-architectures` | Are there better hook patterns than ours? Event-bus alternatives? | Refines A31 Front 2 + A26 hook-chain-composition |
| **RH-010** | `cluster-distributed-agent-systems` | Multi-machine agent orchestration patterns | Extends G7 + G13 |

Each rabbit hole = future BG (or multi-BG for big ones). Per A21 ceiling: 1-3 launches per session over multiple sessions.

## Internal evaluation track (Ask 1)

Parallel to the rabbit holes, internal session evaluation:
- BG #105's master writeup already covered themes + structural fixes
- BG #109's BG taxation showed cost lanes
- Bundle analyzers (G11 Phase 1+2) provide per-session retrospection
- A35's specific extension: review today's audits/plans/skill refs for "skill/hook ideas we MISSED in foreground"

Output: `research/methodology/today-skill-hook-idea-evaluation.md` — gap analysis from external lens.

This is one of the rabbit-hole spawns: an evaluation methodology applied to our own work.

## Phasing

- **Phase 0 (this turn, foreground + 1 BG)**: A35 + plan + G15 + memory rule + rabbit-holes/ subfolder created + 1 BG = research methodology
- **Phase 1**: RH-001 + RH-002 + RH-003 (3 BGs when ceiling opens)
- **Phase 2**: RH-004 through RH-007 (4 BGs)
- **Phase 3**: RH-008 + RH-009 + RH-010 (3 BGs)
- **Phase 4**: Synthesis cross-cutting — what spans multiple rabbit holes? What patterns emerge?
- **Phase 5**: Implementation BGs based on rabbit-hole synthesis (could be many; spawn audits/plans/skills as warranted)

## Cost discipline

Per A14 + G9:
- Each rabbit hole BG: <500k tokens (cite sources; don't reproduce content)
- 10 rabbit holes × 500k = 5M tokens upper bound across multiple sessions
- Synthesis BG: <300k each
- Phase 5 implementation: per-spawn cost gates

Sustainable across 5-10 sessions with cache amortization.

## Cross-references

- G15 — rabbit-holes infrastructure owner
- G14 (master audit) — Phase 4 cross-cutting synthesis feeds into recurring master audit
- A33 — overlap with local AI; RH-007 + RH-008 extend
- A24 — meta-loop-eval monthly synthesis includes rabbit-hole digest
- A31 Front 5 — research v2; A35 extends with /rabbit-holes/ subfolder
- A28 — skill health (rabbit-hole synthesis may spawn skill refresh recommendations)
- G7 + G12/G13 — RH-010 cluster + RH-007 local AI extensions

## Lessons (preliminary)

- We've been internal-perspective-only all session. Risk: re-invention. A35 opens field-of-view.
- "Rabbit hole" as a NAMED structure is the right pattern: explicit scope expansion, methodology-disciplined, output-structured.
- User's "build right from the get go" instinct (methodology first) is correct — sloppy rabbit holes produce noise.
- Research v2 subfolder structure (#109) is already the substrate; adding /rabbit-holes/ + /methodology/ is the minimal extension.
- Each rabbit hole's synthesis.md is the ARTIFACT that justifies the rabbit hole — if no concrete proposals emerge, the rabbit hole was scope-creep.

## Status

IN PROGRESS — Phase 0 ✅ landed 2026-05-27 (methodology ref); Phase 1 🚧 RH-001 + RH-002 BGs in flight + Ask-1 internal eval BG in flight; RH-003 → RH-009 swap pending user decision.

## Phase 0 landing summary (2026-05-27)

Methodology ref written: `context/markdowns/research/methodology/top-tier-research-discipline.md`
- 31,994 bytes / 691 lines / 15 sources cited
- 5 most-important principles: tier-1→5 source-quality system, stopping criteria declared at scoping time, synthesis.md is THE artifact, AI-specific challenges baked in (6 named with mitigation rules), cost gate <500k per RH with budget table
- Sources include PRISMA 2020, Wohlin snowballing, Saunders saturation, fake-stars arXiv:2412.13459 (GitHub-star skepticism), LLM-judge agreeableness arXiv:2510.11822, COPE, COS reproducibility, Anthropic eval challenges
- Confidence labels: settled (PRISMA, snowballing, primary/secondary hierarchy, URL decay mitigation, COPE) | contested (tier-4 arXiv vs tier-1 peer-review weighting in fast-moving fields) | open (LLM-judge contamination, §6e)
- Plan-change recommendation: swap RH-003 → RH-009 for current-sprint fit (RH-009 extends A26 + A31 Front 2 directly). Deferred to user input; not unilaterally changing plan.
