---
audit_id: A33
title: "Local AI mass-wielding on Macs — dual-track research initiative (single-Mac + cluster swarms; token-free heavy lifting; Claude reserved for orchestration)"
status: in_progress
catalogued: 2026-05-27T01:00:00Z
priority_when_run: P1
estimated_effort: very large (multi-month research + multi-phase architecture build; landscape changes the token economy paradigm)
trigger: 2026-05-27 00:55Z — user vision *"if we theoretically utilized this system at maxmode for website matrix joyrides if we didnt care about tokens then we could be a truly unstoppable force, so what if we made a new guiding light background path for research on local ai training+mass-wielding on macs and a completely separate similar second research for macs with clusters like mine, and try to find some way where we can orchestrate insanely powerful swarms all local no token bottleneck at all for the heavy work while we save all our token usage for you... this one prompt should have massive effects, track it and handle it wisely."*
deferral_reason: NONE — running immediately. The architectural insight (token-free local compute layer + Claude as orchestration) is foundational; deferring compounds opportunity cost.
related_goals: [G12, G13, G7, G9]  # G12 + G13 NEW; G7 cluster sibling; G9 testing toolbelt downstream consumer
related_plans:
  - context/markdowns/plans/automation/local-ai-single-mac-mass-wielding-plan.md (G12 owner — NEW)
  - context/markdowns/plans/automation/local-ai-cluster-swarms-plan.md (G13 owner — NEW)
serves_northern_star: G2
belongs_to_goal: G12
serves_guiding_light: G12
related_refs:
  - .claude/skills/agentic-script-design/references/distributed-systems-patterns.md (cluster patterns; G13 substrate)
  - .claude/skills/agentic-script-design/references/bg-dispatch-architecture.md (BG ceiling discipline applies)
  - .claude/skills/agentic-testing-discipline/SKILL.md (cost-discipline shapes when local vs Claude)
  - context/markdowns/plans/automation/cross-tool-event-bus-plan.md (Front 2 of A31 — substrate for local-worker invocation)
  - context/markdowns/research/local-ai/  (NEW subfolder — research target; #109 will create or this audit pre-seeds)
findings: []
---

# A33 — Local AI mass-wielding research initiative

## The vision (user framing)

> "if we theoretically utilized this system at maxmode for website matrix joyrides if we didnt care about tokens then we could be a truly unstoppable force... orchestrate insanely powerful swarms all local no token bottleneck at all for the heavy work while we save all our token usage for you."

The architectural pivot: **Claude becomes the orchestration layer; local AI becomes the worker layer.** Token-free heavy compute via user's Mac hardware (single + clustered); Claude API tokens reserved for what only Claude can do (irreplaceable reasoning, novel design, strategic calls).

## Why this is massive

Today's binding constraints:
- Claude 5hr-window token ceiling (per A14 observed ~30M; today exceeded at 51M via cache amortization)
- BG ceiling at 5 in-flight (A21)
- Per-BG token cost ~100-500k

If most BGs become local AI workers instead:
- Token ceiling stops binding for high-volume mechanical work
- BG ceiling becomes a scheduler concern, not an economic one
- Per-task cost shifts from $ (tokens) to compute (electricity + time)
- User's hardware (M2/M3/M4 unified memory) is the new compute substrate

This isn't an optimization — it's a paradigm shift.

## The two tracks (user explicit split)

### Track 1 — G12: Local AI mass-wielding (single Mac)

**Scope:** what can ONE Mac do well today?

**Achievable now:**
- 7B-13B parameter LLMs via MLX (Apple Silicon optimized) or Ollama
- Vision models (Qwen2-VL, LLaVA) for visual quality audit per cell
- Code-specialized models (Qwen2.5-coder, deepseek-coder) for diff review
- Small fast models for triage / classification / parsing

**Use cases:**
- Audit each section across 12 devices (visual model on each capture)
- Generate property-based test inputs (per G9 + BG #88)
- Mass content synthesis / rewrites (Stream D-style work)
- Bug billboard triage + clustering
- Log analysis / pattern detection in JSONL
- Test result interpretation

**Constraints:**
- One model loaded at a time (RAM-bound)
- ~10-50 tok/s inference on M4 for 13B model
- Quality threshold: when does local-output need Claude verification?

**Owner:** G12 goal; plan = `local-ai-single-mac-mass-wielding-plan.md`

### Track 2 — G13: Local AI cluster swarms (multi-Mac)

**Scope:** what can N Macs orchestrated together do?

**Depends on G7 Phase 2** (cluster hardware coming online). Research can land independently of hardware.

**Multipliers:**
- N nodes × 3 shards/node = 3N parallel inference channels
- Specialist-per-node pattern: each Mac runs a different model
  - Node 1: visual audit model
  - Node 2: code review model
  - Node 3: test gen model
  - Node 4: bug triage model
- Or replicated for throughput: all nodes run same model, fan out load

**Use cases (cluster-specific):**
- Full 12-project Playwright matrix WITH per-cell vision evaluation in parallel
- Mass code review across N changes simultaneously
- Cross-cluster experimentation (try 3 fix strategies on 3 different nodes; pick winner)
- Continuous integration: cluster runs the matrix while developers iterate

**Constraints:**
- Network latency between nodes (~ms range; acceptable for inference dispatch)
- Coordination overhead (per `agentic-script-design` distributed patterns)
- Model synchronization (which node has which model loaded)

**Owner:** G13 goal; plan = `local-ai-cluster-swarms-plan.md`

## Research scope (what BGs will investigate)

Once A21 ceiling opens (5 BGs in flight currently):

### Research Front A — Model landscape (single-Mac)
Output: `context/markdowns/research/local-ai/single-mac-landscape-20260527.md`
- Survey: MLX, Ollama, llama.cpp, vllm — tradeoffs
- Survey: best 7B/13B/30B models for each subtask class (code, vision, general)
- Benchmarks: tok/s on M-series chips
- RAM footprint per model
- Quality vs Claude API for our specific use cases (audit, code review, test gen)

### Research Front B — Architecture integration design
Output: `context/markdowns/research/local-ai/integration-architecture-20260527.md`
- How local AI workers receive tasks (event-bus from A31 Front 2)
- How outputs return to Claude orchestration
- Cost-discipline: when use local vs Claude (decision matrix)
- Failure modes (local model wrong; how does Claude catch it?)
- Quality-gate: confidence thresholds

### Research Front C — Use-case-to-model mapping
Output: `context/markdowns/research/local-ai/use-case-model-mapping-20260527.md`
- For each existing BG type (visual audit, code review, test gen, etc.): which local model best fits?
- Cost vs quality tradeoff per mapping
- Pilot priorities

### Research Front D — Cluster swarm patterns (G13 scope)
Output: `context/markdowns/research/local-ai/cluster-swarm-patterns-20260527.md`
- Specialist-per-node vs replicated patterns
- Load balancing strategies
- Model warm-loading on cluster startup
- Cross-node coordination protocols

### Research Front E — Cost analysis (token vs compute)
Output: `context/markdowns/research/local-ai/cost-analysis-20260527.md`
- Token cost of recent BGs (per BG taxation analysis from #109)
- Estimated compute cost equivalent on local (electricity, wall-time)
- ROI threshold: when does local win?

## Research folder upgrade trigger

Per user's explicit phrasing *"this should be a chance to totally upgrade the research folder too"*:

This audit serves as the **first major real-content test** of the v2 research folder structure (BG #109 currently delivering). The local-ai/ subfolder will hold ≥5 research files (Fronts A-E above). If the v2 taxonomy doesn't support this scale of content, A31 Front 5 v2 needs further refinement.

**Specific extension to BG #109's scope** (queued for when delivered): ensure `research/local-ai/` subfolder is created with proper README + auto-index inclusion.

## Architecture sketch (preliminary)

```
┌─────────────────────────────────────────────────────────────┐
│ Claude (orchestration; token-aware)                         │
│  - Strategic decisions                                       │
│  - Novel design                                              │
│  - User dialogue                                             │
│  - Audit + plan creation                                     │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼  (event-bus per A31 Front 2)
┌─────────────────────────────────────────────────────────────┐
│ .claude/cache/event-bus.jsonl                               │
│  Events: "local-ai:dispatch" / "local-ai:result"            │
└──────────────────────────┬──────────────────────────────────┘
                           │
        ┌──────────────────┴──────────────────┐
        ▼                                       ▼
┌────────────────────────┐         ┌────────────────────────┐
│ Single-Mac worker (G12)│         │ Cluster swarm (G13)    │
│ MLX/Ollama dispatcher  │         │ Per-node specialists   │
│ - Visual audit         │         │ - Node 1: vision       │
│ - Code review          │         │ - Node 2: code         │
│ - Test gen             │         │ - Node 3: test         │
│ - Triage               │         │ - Node 4: triage       │
└────────────────────────┘         └────────────────────────┘
        │                                       │
        ▼                                       ▼
┌─────────────────────────────────────────────────────────────┐
│ Aggregator → cache outputs → return event "local-ai:result" │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│ Claude reads result; orchestrates next decision             │
└─────────────────────────────────────────────────────────────┘
```

## Cost-discipline framework (preliminary)

When the architecture lands, agent (me) decides per-task:

| Task class | Default routing | Why |
|---|---|---|
| Visual audit per cell | Local (vision model) | High-volume; pattern recognition fits LLM well |
| Code review on diff | Local (coder model) | Mechanical; quality threshold known |
| Test case generation | Local (general model) | Property-based; high volume |
| Bug triage / clustering | Local (small fast model) | Classification; low stakes |
| Log pattern detection | Local (small fast model) | High volume; well-suited |
| Audit + plan creation | Claude (me) | Novel reasoning; orchestration | 
| Strategic call (NS pivot) | Claude (me) | Irreplaceable judgment |
| Memory rule changes | Claude (me) | Cross-cutting; needs context |
| Recurrence detection | Local + Claude hybrid | Local detects; Claude analyzes |

This matrix will refine via Front E cost analysis.

## Cost gates

Per G9 anti-bottleneck + this audit's nature:

| Phase | Token budget (Claude) | Compute budget (local) |
|---|---|---|
| Research Fronts A-E (~5 BGs) | <500k each (<2.5M total) | n/a (research only) |
| Pilot impl (G12 Phase 1) | <500k (one model integration) | TBD per model |
| Production G12 | <100k per session (orchestration only) | unlimited |
| G13 wait on G7 Phase 2 | gated by hardware | gated by hardware |

**The key insight:** post-A33, Claude tokens become predictable + low (orchestration only). Local compute becomes the variable.

## Phases (see plans)

See `local-ai-single-mac-mass-wielding-plan.md` (G12) + `local-ai-cluster-swarms-plan.md` (G13).

- **Phase 0 (this turn, foreground):** Audit + 2 plans + 2 goals + memory rule + research folder extension directive
- **Phase 1 (queued; BG when ceiling opens):** Research Front A — landscape survey for single-Mac (G12 enabler)
- **Phase 2:** Research Front B — integration architecture design
- **Phase 3:** Research Front C + D — use-case mapping + cluster swarm patterns
- **Phase 4:** Research Front E — cost analysis with real numbers
- **Phase 5 (G12 impl pilot):** First real local AI integration (recommend: visual audit per cell, since it's high-value high-volume)
- **Phase 6 (G12 production):** Scheduler/event-bus integration; multiple use cases active
- **Phase 7 (G13 — gated by G7 Phase 2):** Cluster swarm impl

## Verification (after Phases 1-5)

1. 5 research files in `research/local-ai/` covering Fronts A-E
2. Cost-discipline framework refined with real benchmarks
3. G12 pilot integration runs at least once with measurable token-savings
4. Architecture sketch validated against pilot
5. G13 readiness: when G7 Phase 2 lands, G13 Phase 7 can start immediately

## Status

IN PROGRESS — foreground artifacts landing this turn. Research BGs queue for when A21 ceiling opens (5 in flight from A31 prep wave).

## Cross-references

- G7 (cluster) — G13 depends on Phase 2 hardware
- G9 (testing toolbelt) — downstream consumer of local AI workers (test gen + visual audit)
- G11 (session bundle) — local AI workers can run analyzer sub-tasks
- A14 (stats correlation) — token economy fundamentally reshapes A14's projection model
- A21 (parallel capacity) — local AI ceiling becomes separate from Claude BG ceiling
- A24 (meta-loop-eval) — local AI workers run analyzer cycles
- A26 (script-design) — `distributed-systems-patterns.md` extended by G13 needs
- A31 Front 2 (event-bus) — substrate for local AI worker invocation
- A31 Front 5 (research folder v2) — first major real-content test

## Lessons (preliminary, pre-research)

- Token economy = the original A14 constraint. A33 fundamentally reshapes it.
- "Unstoppable force" framing is the user's stress signal. Per A29 INVERSE pattern: respond with EXECUTION-heavy (BG-rich) work, NOT diagnostic-heavy text.
- Two tracks (single + cluster) is the right granularity — different timelines, different constraints, same vision.
- Research folder v2 (BG #109) is THE substrate for this — its taxonomy will hold these research files.
- The architecture sketch is preliminary; first 2-3 research fronts will refine.
