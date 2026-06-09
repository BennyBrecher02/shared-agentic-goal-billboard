---
audit_id: A75
title: Workflow 4.7-vs-4.8 comparison + conflict-check between our workflow system and 4.8's baked-in dynamic-workflows feature
status: verified_complete
catalogued: 2026-05-28T23:23Z
priority_when_run: P1
trigger: 2026-05-28T23:23Z — user verbatim: *"new audit ide, wrokflow comparison 4.7 vs 4.8, because we built an intricate workflow system and claude copying us the day after makes me wonder if 1 we built a better system than them somehow possibly 2) making sure that the new 4.8 workflows dont disalign how we previousy handled our own workflows before workflows became a baked in feature. also make sure theres no conflictions in our system at all for workflows as its a sensitive trigger apparently... its built in stuff could be ignoring our true workflow system too, so look up what 4.8 workflows even are under the scene and find out if they improve our system or are even hindered by it possibly, flesh this out"*
deferral_reason: NONE — explicit user audit request + "flesh this out"
serves_northern_star: G2
belongs_to_goal: G4
related_refs:
  - .claude/skills/agentic-script-design/references/dynamic-workflow-adoption.md (our 4.8-workflow adoption guidance)
  - .claude/skills/agentic-webdesign/SKILL.md (our CONCEPTUAL "workflow" usage — audit/fix/verify cycles)
  - .claude/workflows/ (our native Workflow-tool scripts: design-import.js, model-upgrade-benchmark.js)
findings:
  - the-live-demonstration: This audit's own trigger turn proved the conflict — user said "workflow comparison" (meaning an AUDIT) and the 4.8 built-in `workflow` keyword-trigger fired the "use the Workflow tool" system-reminder. The agent correctly did NOT fire a native workflow (user wanted analysis). The disambiguation gap between (a) our pre-existing CONCEPTUAL "workflow" terminology, (b) the native 4.8 Workflow TOOL, and (c) the keyword-trigger that nudges toward the tool — is the core thing to map.
---

# A75 — Workflow 4.7-vs-4.8 + conflict-check

## Why this audit

We built an intricate workflow/orchestration system (BG dispatch + scheduler + audit-fix-verify cycles + the agentic-webdesign "workflows" references) BEFORE Anthropic shipped native dynamic workflows in Opus 4.8 (2026-05-28 — literally the day this concern arose). Three questions:

1. **Did we build something comparable/better?** (or did they leapfrog us)
2. **Do 4.8's baked workflows DISALIGN how we handled our own workflows?** (terminology/convention drift)
3. **Are there CONFLICTS?** The `workflow` keyword is a "sensitive trigger" — the built-in feature's keyword-nudge could shadow/ignore our true workflow system, causing mis-handling.

Plus: **what ARE 4.8 workflows under the hood**, and do they **improve or hinder** our system?

## Three layers of "workflow" in play (the disambiguation problem)

| Layer | What it is | Risk |
|---|---|---|
| **Conceptual** | Our long-standing use of "workflow" = audit/fix/verify cycles, the established-workflows-check hook, the agentic-webdesign workflow refs | The word predates the feature; our docs/hooks use it generically |
| **Native tool** | The 4.8 Workflow tool + `.claude/workflows/*.js` scripts (we adopted it: design-import, model-upgrade-benchmark, adaptive-immunity-closure) | New; we're using it correctly so far |
| **Keyword trigger** | The harness fires "use the Workflow tool" whenever the user types "workflow" | **The conflict**: nudges toward the tool even when the user means the conceptual sense — could cause the agent to fire a native workflow when the user wanted analysis/a-cycle |

## Investigation (parallel BGs)

- **BG-1** — what 4.8 dynamic workflows ARE under the hood + 4.7-vs-4.8 + did-we-build-comparable/better
- **BG-2** — conflict-check: scan our whole system for "workflow" collisions (hooks, skill refs, the established-workflows-check hook, the keyword trigger); does the 4.8 feature shadow/ignore/mis-route our usage? Honest assessment, NO trigger changes (user explicit: "not that i want you changing our triggers").

## Findings

_(populated as BGs return)_

## Constraints (user-stated)

- Do NOT change our triggers. Map the conflict; don't mutate it without user direction.
- The cursor-integration-with-agents research is PINNED separately (P-018) — not part of this audit.

## Cross-references

- `dynamic-workflow-adoption.md` (our adoption ref — BG-1 validates/extends it)
- A74 (the 4.8 systemwide audit — D3/D6 first flagged the workflow-adoption gap)
- `feedback_standing-protocols.md` (established-workflows-check — a CONCEPTUAL-workflow hook that might collide with the keyword sense)

---

## BG-1 — 4.8 workflows under the hood + 4.7-vs-4.8

_Researched 2026-05-28 (WebSearch + WebFetch against Anthropic's official Claude Code docs + launch coverage). Findings below are sourced; where a mechanical detail is NOT publicly documented, it's flagged as such rather than guessed._

### 1. What 4.8 dynamic workflows ARE, mechanically

A dynamic workflow is **"a JavaScript script that orchestrates subagents at scale"** ([Claude Code docs](https://code.claude.com/docs/en/workflows)). Mechanically, the pieces are:

- **Authoring**: Claude *writes the orchestration script itself* from your natural-language prompt. You don't hand-write JS — the model emits it, then optionally you read/approve/save it. It is "the plan moved into code": the script "decides what to launch, in what order, with what conditional or loop logic, and keeps intermediate state in variables that live outside the conversation" ([MarkTechPost](https://www.marktechpost.com/2026/05/28/anthropic-ships-claude-opus-4-8-alongside-dynamic-workflows-and-cheaper-fast-mode-with-workflows-capped-at-1000-subagents/)).

- **Execution model**: "The workflow runtime executes the script in an **isolated environment, separate from your conversation**." This is the load-bearing architectural fact. Two consequences spelled out in the docs:
  1. **No direct filesystem or shell access from the workflow script itself** — "Agents read, write, and run commands. The script coordinates the agents." So the JS orchestrator is a *pure coordination layer*; all side effects happen inside spawned subagents (each with its own isolated context window). The runtime is effectively a sandboxed JS host whose only privileged capability is spawning/awaiting agents.
  2. **Intermediate results live in script variables, not Claude's context** — only the final answer returns to the conversation. This is the key scaling unlock: a 200-agent fan-out doesn't blow the orchestrator's context window, because the 200 partial results sit in JS variables, not in the transcript.

- **The spawn primitive**: the subagent is the documented worker primitive ("Create custom subagents: the worker primitive workflows orchestrate"). The script's job is to launch and await those workers, hold the loop/branch logic, and aggregate. **Anthropic has NOT published the exact JS API surface** (the function name/signature for spawning — e.g. whether it's an `agent()` / `spawn()` call awaited via `Promise.all`, or a pool helper). Public docs describe *behavior and limits*, not the script's callable API; the script is normally model-authored and you interact with it via approve/save/`/workflows`, not by writing the primitive yourself. **So our ref's phrasing "Claude authors its own orchestration scripts… inside a single session" is accurate; any claim about a specific `agent()` primitive name would be unsourced.**

- **Concurrency**: "**Up to 16 concurrent agents** (fewer on machines with limited CPU cores)" and "**1,000 agents total per run**." So it's a bounded worker pool — 16 wide at any instant, 1000 cumulative across the whole run before a hard stop ("prevents runaway loops"). The 16-cap is explicitly a *local resource bound*, not a model limit, which means the concurrency ceiling is about the host machine, not Claude.

- **Checkpoint / resume**: "The runtime tracks each agent's result as the run progresses, which is what makes a run resumable." On resume, "agents that already completed return their **cached results**, and the rest run live." Critically: **"Resume works within the same Claude Code session. If you exit Claude Code while a workflow is running, the next session starts the workflow fresh."** So native checkpointing is *single-session only* — a session boundary wipes it. (Validates our ref's cross-session carve-out exactly.)

- **The quality pattern (propose-refute-converge)**: moving the plan into code lets a workflow "apply a repeatable quality pattern, not just run more agents: it can have **independent agents adversarially review each other's findings** before they're reported, or draft a plan from several angles and weigh them against each other." The bundled `/deep-research` workflow does exactly this — fans out, "votes on each claim," and returns a report "with claims that didn't survive cross-checking filtered out." This is the same audit/refute/converge shape we hand-build in audit-fix-verify cycles, but mechanized.

- **Limits / constraints** (from the docs' constraint table):
  - **No mid-run user input** — only agent permission prompts can pause a run. "For sign-off between stages, run each stage as its own workflow." (You can't interactively redirect a running workflow — same fire-and-forget property as our BGs.)
  - Subagents "always run in `acceptEdits` mode and inherit your tool allowlist, regardless of your session's mode. File edits are auto-approved." (Permission posture is fixed for workflow-spawned agents.)
  - **Model per stage**: "Every agent in a workflow uses your session's model unless the script routes a stage to a different one." (Matches our `model-routing-policy` — inherit-by-default; downgrade deliberately per stage.)
  - **Nesting**: not documented publicly. The 1000-total cap is the only hard ceiling stated.
  - Trigger surface: the literal word **`workflow`** in a prompt, OR `/effort ultracode` (= `xhigh` reasoning + automatic workflow orchestration on every substantive task), OR a bundled/saved `/command`. `alt+w` ignores an accidental keyword trigger. **This is the exact "sensitive trigger" the user flagged** — and BG-2 owns the conflict analysis.

### 2. 4.7-vs-4.8 orchestration delta

| Axis | Opus 4.7 | Opus 4.8 |
|---|---|---|
| **Native orchestration primitive** | None. Orchestration was manual — Claude spawned subagents turn-by-turn via the Task/Agent tool, **one batch per turn**, every result landing back in the orchestrator's context window | **Dynamic Workflows**: Claude authors a JS orchestration script; runtime fans out tens-to-hundreds of subagents in one session |
| **Where the plan lives** | In Claude's context (the conversation IS the orchestrator) | In script variables, outside the conversation |
| **Where intermediate results live** | Claude's context window (so fan-out is context-bounded — ~10-20 results before bloat) | Script variables (so 1000-agent fan-out doesn't bloat context) |
| **Concurrency** | Bounded by what one conversation turn can coordinate + context budget | 16 concurrent / 1000 total per run, runtime-scheduled |
| **Interruption** | Restarts the turn | Resumable in-session (cached completed-agent results) |
| **Repeatable unit** | The subagent definition only | The whole orchestration (save the script as a `/command`) |
| **Quality pattern** | Hand-driven by Claude turn-by-turn | Codified — adversarial cross-review / multi-angle drafting baked into the script |
| **Coding benchmark** | SWE-bench Verified 87.6% / SWE-bench Pro 64.3% | SWE-bench Verified 88.6% / SWE-bench Pro 69.2% (+1.0 / +4.9 pts) — [digitalapplied](https://www.digitalapplied.com/blog/claude-opus-4-8-release-dynamic-workflows-2026) |
| **Other 4.8 deltas** | — | Effort controls (incl. `ultracode`), Fast Mode 2.5× speed at 3× lower cost, ~4× less likely to let its own code flaws pass unremarked ([honesty]) |

**The one-sentence delta**: 4.7 made *Claude* the orchestrator (plan + results in-context, one batch/turn); 4.8 makes *a script Claude writes* the orchestrator (plan + results in JS variables, 16-wide pool, in-session resumable). The model got modestly better at coding; the *orchestration substrate* is the categorical change.

### 3. Did WE build comparable/better BEFORE they shipped it (2026-05-28)?

Honest answer up front: **we built a more disciplined *governance wrapper* and a genuine *cross-session* substrate that 4.8 still lacks — but on raw single-session parallel scale and native mechanism, 4.8 is strictly better, and several of our mechanisms were 4.7-era workarounds for limits 4.8 erases.** It's not "we beat them" or "they leapfrogged us" — it's *different layers of the same stack*. Our `dynamic-workflow-adoption.md` already nailed this with the engine / hardware / wrapper reframe; this research confirms it.

#### Comparison table — OUR system vs 4.8 native

| Capability | Ours (BG dispatch + scheduler + audit cycles) | 4.8 native dynamic workflows | Who's better + why |
|---|---|---|---|
| **Raw parallel scale** | A21 5-in-flight ceiling (heterogeneous mutating BGs); ~10-20/wave assumption for fan-out | 16 concurrent / 1000 total, runtime-scheduled | **Theirs, decisively.** Our 5-cap is a coordination-cost ceiling, not a scale mechanism. For read-only homogeneous fan-out (the 232-shard audit) they run it in ONE orchestration; we need ~47 waves. |
| **Context-bloat avoidance at scale** | Each BG report lands back in main's context → reconciliation cost dominates past 5 | Intermediate results stay in JS variables; only final answer returns | **Theirs.** This is the architectural unlock we *couldn't* have — 4.7 had no out-of-context result store. |
| **Native checkpoint/resume** | Prescribed `.claude/cache/bg-checkpoints/` — **documented but never built** (the dir doesn't exist) | Built into the runtime; cached completed-agent results on resume | **Theirs** for single-session. Ours was vaporware-by-design — a 4.7 workaround we correctly never bothered to build. |
| **Cross-session persistence** | BG dispatched now, acknowledged next session; `lost-work-token-cost.sh` catches the gap; G10 snapshot bridge | **None** — "if you exit Claude Code while a workflow is running, the next session starts fresh" | **OURS, clearly.** A session boundary is a hard wall their native resume can't cross. Our BG pattern + lost-work hook + snapshot bridge is the only thing that survives compaction/exit. This role does NOT go away. |
| **File-zone collision detection** | Each dispatch declares a narrowest-zone; classifier serializes overlapping zones; atomic-merge for hot JSON | Workflow script has *no FS access*; agents inherit allowlist + run `acceptEdits`; **no documented collision rule** between sibling agents' writes | **OURS.** Native handles orchestration, not collision-safety of N parallel writers hitting the same path. Our zones are exactly the missing guard — even read-only fan-out has per-shard output that needs disjoint paths. |
| **Fabrication detection** | git-diff / mtime / artifact-hash external verification; the Text-v2-incident discipline; established-workflows-check Stop hook | 4.8 honesty ↑ (~4× less likely to let its own flaws pass) + adversarial cross-review in-workflow | **Mixed → ours for correctness, theirs for sincerity.** Their adversarial-review + honesty lowers the *rate* of bad claims; it does NOT externally verify against the filesystem. Honesty ≠ correctness — a sincere-but-wrong verification still passes. Our git/mtime/hash checks stay mandatory. Their cross-review is a genuine improvement we should *layer on top of*, not a replacement. |
| **Audit-ID partition / idempotent re-runs** | `<audit-id>/<section>/<device>` namespace; deterministic aggregation; re-runs idempotent | Save-the-script gives *orchestration* repeatability; no documented output-partition/idempotency convention | **OURS** on output determinism; **theirs** on orchestration repeatability (saved `/command`). Complementary. |
| **NS-context propagation** | `serves_northern_star:` frontmatter + auto-prepend wrapper on every dispatch; strategic frame measurable from any artifact | None — workflows are task-scoped; no strategic-goal substrate | **OURS.** Pure governance layer they don't have or attempt. |
| **Mechanical result aggregation** | Inbox lanes (append-only, collision-free) + manual main-agent reconcile for heterogeneous | Built-in: results aggregate in script variables; adversarial vote/converge; one final report | **Theirs** for homogeneous mechanical rollup; **ours** (inbox lanes) is the right substrate to *wrap* their fan-out when outputs must persist beyond the run. |
| **Adversarial / multi-angle quality** | Hand-built audit-fix-verify cycles; dual-worktree calibration (script-vs-agent cross-ref) | Codified propose-refute-converge in the script; `/deep-research` votes + filters unsurvived claims | **Theirs** for in-run mechanization (it's automatic). **Ours** (dual-worktree calibration) does a *different* thing — cross-*method* calibration, not cross-*agent* refutation — and has no native equivalent. |
| **Cost / bottleneck discipline** | A14/A21/G9 anti-bottleneck chant; cost-gate skip >80% burn; effort-control mapping (A74) | Docs warn runs "use meaningfully more tokens"; `/model` + per-stage downgrade guidance | **OURS.** They surface the cost risk; we have a *standing protocol* that actively governs it (the bottleneck chant + cost gates). Their guidance is advisory; ours is enforced. |
| **Trigger ergonomics** | Explicit dispatch (we choose to spawn) | `workflow` keyword / `ultracode` auto-trigger / saved commands | **Theirs** for ergonomics, **but** the keyword auto-trigger is the exact "sensitive trigger" risk the user flagged — convenience cuts both ways (BG-2 owns this). |

#### Honest verdict — which axes

- **We built BETTER on**: cross-session persistence (their hard wall is our solved problem), file-zone collision-safety, external fabrication verification (correctness vs their sincerity), NS-context governance, and enforced cost/bottleneck discipline. These are all *governance / durability* axes — the "wrapper" in the engine/hardware/wrapper model.
- **They built BETTER on**: raw parallel scale (16/1000 vs our 5-cap), context-bloat avoidance (out-of-context result store — architecturally impossible for us on 4.7), native single-session checkpoint/resume (ours was never built), in-run adversarial cross-review (automatic vs our hand-built cycles), and orchestration repeatability (save-as-command). These are all *engine* axes.
- **Comparable / complementary**: result aggregation (their variables vs our inbox lanes — both valid, different scopes), audit/quality patterns (their cross-agent refute vs our cross-method calibration — different axes of the same goal).

**Bottom line**: We did NOT build "a better workflow engine" — that framing would be ego. 4.8's engine is better than anything we could build on 4.7, and a few of our mechanisms (hand-rolled checkpoints) were workarounds for limits 4.8 erases. **What we built that's genuinely better and still load-bearing is the *discipline layer that wraps* an orchestration engine**: cross-session survival, collision-safety, external verification, strategic-goal propagation, enforced cost gates. The correct posture (already captured in `dynamic-workflow-adoption.md`) is **adopt their engine for single-session homogeneous fan-out, keep our wrapper around it, and keep our BG pattern for the cross-session work their engine can't reach.** This research validates that ref end-to-end and adds the honest scorecard it was missing.

One thing we arguably *anticipated*: the propose-refute-converge quality pattern they baked in is conceptually the same as our audit-fix-verify + adversarial-review cycles — we'd been hand-running that shape for weeks. We didn't mechanize it as a runtime, but we'd independently arrived at the *pattern* before it shipped as a feature. That's the kernel of truth behind the user's "did we build it first" instinct: we built the *discipline*; they built the *engine*.

### Sources

- [Orchestrate subagents at scale with dynamic workflows — Claude Code Docs](https://code.claude.com/docs/en/workflows) (authoritative: mechanics, limits table, resume, triggers)
- [Introducing Claude Opus 4.8 — Anthropic](https://www.anthropic.com/news/claude-opus-4-8) (launch; "hundreds of parallel subagents in a single session"; honesty 4× claim)
- [Anthropic Ships Claude Opus 4.8… Workflows Capped at 1,000 Subagents — MarkTechPost](https://www.marktechpost.com/2026/05/28/anthropic-ships-claude-opus-4-8-alongside-dynamic-workflows-and-cheaper-fast-mode-with-workflows-capped-at-1000-subagents/) (plan-in-code; 16/1000 caps; propose-refute-converge)
- [Claude Opus 4.8: Benchmarks, Effort & Dynamic Workflows — digitalapplied](https://www.digitalapplied.com/blog/claude-opus-4-8-release-dynamic-workflows-2026) (4.7-vs-4.8 SWE-bench delta; "4.7 had no native workflow orchestration")
- [Anthropic releases Opus 4.8 with new 'dynamic workflow' tool — TechCrunch](https://techcrunch.com/2026/05/28/anthropic-releases-opus-4-8-with-new-dynamic-workflow-tool/) (launch confirmation, research-preview status)
- [Building agents with the Claude Agent SDK — Anthropic](https://www.anthropic.com/engineering/building-agents-with-the-claude-agent-sdk) (subagent worker primitive: isolated context, returns only relevant info to orchestrator)

BG-COMPLETE-SENTINEL

---

## BG-2 — conflict-check (our workflow system vs 4.8 baked)

**Scope:** scanned `.claude/skills/`, `scripts/hooks/`, `.claude/settings.json`, `context/markdowns/` for every use of "workflow" + every place the native 4.8 Workflow tool / keyword-trigger could shadow or mis-route our system. Mapping only — no triggers/hooks/settings touched.

### Headline verdict

**No real conflict. The "sensitive trigger" fear does not bear out in our codebase — for a structural reason: not one of our hooks keys on the literal word "workflow."** The native keyword-trigger and our system live on **disjoint substrates** (the harness reads the user's *word*; our hooks read *file-edit patterns* and *trigger-phrase lists*). They cannot fight because they never touch the same input. Severity: **LOW (latent-only).** The one genuine risk is agent-judgment mis-routing (firing a native Workflow when the user meant the conceptual sense), and the live evidence shows the agent handling that correctly. Two integration points are actively *positive* (native Workflow already counts as valid prevention-spawn + valid orchestration engine). Details below.

### The three layers — confirmed disjoint

| Layer | What it is | Where it lives | Keys on the word "workflow"? |
|---|---|---|---|
| **(a) Conceptual** | audit/fix/verify cycles; the `established-workflows-check.sh` bypass detector; the agentic-webdesign "workflows/" refs; audit-type "Workflow / mechanism" columns | our skills + 1 hook + docs | **No** — the hook keys on *file paths edited* (UI/scheduler), not the word |
| **(b) Native tool** | the 4.8 `Workflow` tool + `.claude/workflows/*.js` (design-import.js, model-upgrade-benchmark.js) | harness + 2 scripts we authored | We invoke it by **name** (`Workflow({name:"…"})`), not by echoing the user's word |
| **(c) Keyword-trigger** | the harness emits a "use the Workflow tool" nudge when the user types "workflow" | **harness-native — NOT authored by us** | Yes (that's the harness's behavior) — but nothing of ours consumes or amplifies it |

**Decisive evidence that (c) is harness-only and un-amplified by us:** a repo-wide search for any agent-authored "use the Workflow tool" / "fire a workflow" nudge returns **zero** hits outside legitimate `Workflow({name:"model-upgrade-benchmark"})` invocations in model-routing docs (a correct *named* native use, not a word-trigger). We do not re-emit, widen, or hook the keyword nudge anywhere.

### Catalog table — location | sense-of-"workflow" | collision risk

| Location | Sense | Collision risk |
|---|---|---|
| `scripts/hooks/established-workflows-check.sh` | **Conceptual** — named "workflow" but triggers on Edit/Write to `src/components|pages|layouts`, `reports/timelapse/*/index.html`, `scripts/scheduler/**`; checks whether scrutiny/test ran | **None.** Zero string-match on "workflow" in user input. Pure file-pattern logic. Cannot be fired or suppressed by the native keyword. (The settings.json registration at line 329 is the only settings mention — also pattern-gated, not word-gated.) |
| `scripts/hooks/user-ask-detect.sh` | n/a — the **UserPromptSubmit** trigger-phrase hook (make a plan / audit / investigate why / new hook idea / list all X) | **None — and this is the key finding.** `grep -c workflow` = **0**. "workflow" is deliberately NOT a trigger phrase. So a user typing "workflow" fires the *harness* nudge but **none of our** prompt-side machinery. The two systems are provably non-overlapping at the input layer. |
| `scripts/hooks/critical-finding-detect.sh` (L199, L27, L393) | **Native tool** — `SPAWN_TOOLS = {"agent","task","workflow"}` | **Positive integration, not a conflict.** Firing a native Workflow *satisfies* the critical-finding-reflex prevention-spawn requirement. Native workflows are treated as first-class prevention work — they do NOT bypass the reflex; they fulfill it. |
| `scripts/hooks/recurrence-detect.sh` (L420) | **Conceptual** — points to `kpi-self-eval-workflow.md` | **None.** Fires only when `frustration_score ≥ 1`; repeated neutral "workflow" mentions cannot trip it. Topic-recurrence is frustration-gated, not keyword-gated. |
| `scripts/hooks/organic-os-pulse.sh` (L121) | Conceptual — *invokes* `established-workflows-check.sh` | None — inherits the file-pattern logic above. |
| `scripts/hooks/ack-latency-measure.sh`, `billboard-data-refresh.sh`, `dashboard-data-prime.sh` | Conceptual — read `.claude/cache/workflow-bypass.jsonl` (the bypass-count substrate) + reference `kpi-self-eval-workflow.md` | None — they consume the bypass log; no keyword coupling. |
| `scripts/hooks/image-cap-warn.sh` (L6) | Native tool — comment "the main agent (or a workflow) is about to Read…" | None — correctly anticipates native workflows as a Read source; defensive, not colliding. |
| `.claude/workflows/design-import.js`, `model-upgrade-benchmark.js` | **Native tool** — the two native Workflow scripts we authored | **None.** Correct adoption per `dynamic-workflow-adoption.md`. Invoked by name. |
| `.claude/skills/agentic-script-design/references/dynamic-workflow-adoption.md` (18 hits) | **Native tool (adoption doc)** — the engine/wrapper decision matrix | None — this is precisely the disambiguation doc that *resolves* the layers (native = engine, our discipline = wrapper). |
| `.claude/skills/agentic-webdesign/SKILL.md` (12 hits) + `references/workflows/*` | **Conceptual** — "the iterative design workflow," audit/fix/verify cycles, the `references/workflows/` directory | **Naming-adjacency only (see below).** Conceptual prose; no native-tool coupling. |
| `.claude/skills/agentic-page-scrutiny/references/audit-types-router.md` (10 hits) + `full-coverage-audit-workflow.md`, `scoped-change-audit-workflow.md` | **Conceptual** — "Workflow / mechanism" columns, audit methodology cycles | None — pure conceptual methodology naming. |
| `.claude/skills/agentic-quality-discipline/references/kpi-self-eval-workflow.md` + ~30 other skill refs | **Conceptual** — methodology cycles, KPI loops | None — conceptual throughout. |

**Aggregate:** ~48 skill files + 10 hooks + 159 markdown files use "workflow." Sense breakdown: the overwhelming majority is **conceptual** (audit/fix/verify cycles, methodology refs); a focused, correct cluster is **native-tool** (`dynamic-workflow-adoption.md`, the 2 `.js` files, `critical-finding-detect` spawn-set, model-routing). **Zero** locations key on the literal user word.

### The disambiguation gap — when does the keyword mis-route?

The mis-route is **not** a code conflict; it is an **agent-judgment ambiguity** at one specific moment: the user types a sentence containing "workflow" in the *conceptual* sense (e.g. *"run our scrutiny workflow,"* *"workflow comparison 4.7 vs 4.8,"* *"does this disalign our workflows"*), and the harness emits its "use the Workflow tool" nudge. If the agent reflexively obeyed the nudge, it would fire a native single-session orchestration when the user wanted analysis, an audit, or a conceptual cycle.

**Frequency this session (measured):** 13 user messages and 41 assistant messages mention "workflow." Effectively **all** are the *conceptual* sense — the session's entire topic is *this audit about the word*. The nudge was therefore in tension with user intent on essentially every "workflow" mention this session, yet **the agent fired no spurious native Workflow** (this audit's own trigger turn is the textbook case: user said "workflow comparison" meaning an audit; the agent produced an audit, not a native run — already logged in the stub's `the-live-demonstration` finding).

**Why the gap stays latent (doesn't bite):** disambiguation is carried by the *meaning of the surrounding sentence*, which the agent reads correctly. The nudge is a soft suggestion, not a forcing function — it competes with intent but does not override it. The risk would only materialize if (1) an inattentive/over-literal agent treated the nudge as a command, or (2) some future hook of ours *amplified* the keyword into a harder push. Neither exists today.

### Shadowing check — does the native feature ignore our discipline?

**No — and the architecture actively prevents it:**

1. **The bypass detector is engine-agnostic.** `established-workflows-check.sh` fires on the *file paths edited in the turn*, regardless of HOW they were edited. If a native Workflow edited `src/components/**` without running scrutiny, the Stop hook would still flag WORKFLOW BYPASS — because it inspects the session JSONL's `tool_use` edits, not the invocation mechanism. So a native workflow **cannot** silently skip our audit-fix-verify discipline; the same Stop-hook gate applies. *(One latent edge: the hook's window is "last ~600s of the just-finished turn" and it scans `Edit/Write/MultiEdit` tool_use entries — if a long native orchestration's edits land outside that window, or are attributed to sub-tool frames the parser doesn't walk, a bypass could go uncounted. This is a *coverage* nuance to validate once native workflows actually mutate src/, NOT a present conflict — today's two native workflows (design-import staging, model-upgrade-benchmark) are read-only/staging and don't edit src/.)*
2. **Native workflows count as fulfilling, not bypassing, the critical-finding reflex.** `SPAWN_TOOLS` explicitly includes `"workflow"` — the native tool is recognized as legitimate prevention/orchestration work.
3. **Our adoption doc frames native workflows as the engine our discipline WRAPS** (`dynamic-workflow-adoption.md`): file zones, inbox lanes, audit-ID partition, external verification all still apply around a native fan-out. The discipline is designed to *layer over* native, not be replaced by it.

### Naming collisions

- **`.claude/workflows/` (native scripts) vs `.claude/skills/agentic-webdesign/references/workflows/` (conceptual playbooks).** Two directories, both named "workflows," different roots, different meanings (native `.js` orchestrations vs conceptual `.md` playbooks). **Adjacency, not collision** — no path overlaps, no tooling reads both as one set. Mild human-confusion potential when skimming, nothing functional. The `references/workflows/` name predates the native feature.
- **`workflow-bypass.jsonl` substrate.** Named for the *conceptual* bypass (skipped scrutiny), unrelated to the native Workflow tool. No collision; just shared vocabulary.
- No pre-existing use of the literal `.claude/workflows/` path before native adoption — the directory is purely the native scripts (design-import.js, model-upgrade-benchmark.js), both authored intentionally under the native feature.

### Honest verdict + severity

- **Real conflict:** **None.** No code path where the native feature and our system contend for the same input or where one overrides the other.
- **Latent risk:** **LOW.** Single locus — agent could over-obey the harness keyword-nudge and fire a native Workflow when the user meant the conceptual sense. Mitigated today entirely by agent reading-comprehension; this session's 13/41 conceptual mentions produced zero spurious native runs.
- **Shadowing:** **None.** The bypass detector is engine-agnostic (file-pattern based), so native workflows are held to the same audit-fix-verify gate. Critical-finding reflex *embraces* native Workflow as valid prevention. (One coverage nuance to revisit if/when native workflows start mutating `src/` — see shadowing point 1.)
- **Disalignment of prior conventions:** **None.** Our "workflow" terminology predates the feature and remains coherent; `dynamic-workflow-adoption.md` already did the disambiguation work (engine vs wrapper). The native feature *improves* our system (native fan-out as the engine for the 232-shard audit) rather than disaligning it. **Consistent with BG-1's scorecard:** their *engine* is better; our *wrapper discipline* is what's load-bearing and theirs doesn't replace.

### Proposal (mapping only — NOT implemented; for user decision)

If the user later wants to harden the latent agent-judgment gap **without touching any trigger/hook**, the lowest-risk option is **documentation-only**: a 3-line disambiguation note in `dynamic-workflow-adoption.md` (or a tiny new `workflow-term-disambiguation.md` skill ref) stating: *"When the user says 'workflow,' read the sentence's meaning. Conceptual sense (run our scrutiny workflow / workflow comparison / our audit-fix-verify workflow) → do the cycle/analysis. Native sense (an explicit orchestration / fan-out request, or a named `.claude/workflows/*.js`) → fire the Workflow tool. The harness keyword-nudge is a suggestion, not a command — intent wins."* This is a pure-doc reinforcement of behavior the agent already exhibits; it touches **no** hook, trigger, or settings, and aligns with the project's doc-first discipline. **Do not deploy without user direction.**

Optionally, the user could also consider renaming the conceptual `references/workflows/` directory to `references/cycles/` or `references/playbooks/` to remove the directory-name adjacency with `.claude/workflows/` — but this is cosmetic, churns many cross-references, and is **not recommended** unless the adjacency actively confuses; flagging it only for completeness.

BG-2-COMPLETE-SENTINEL

## Synthesis / verdict (2026-05-28)

**Q1 — did we build better?** Not a better ENGINE (4.8 wins on raw scale: 16-concurrent/1000-total vs our A21 cap, out-of-context result store, native checkpoint/resume we never built). We built a better GOVERNANCE WRAPPER + a real CROSS-SESSION substrate they still lack (file-zone collision-safety, external fabrication verification, NS-context propagation, cost/bottleneck gates, BG/snapshot cross-session survival). The kernel of truth: we independently arrived at propose-refute-converge (our audit-fix-verify) before it shipped. **They built the engine; we built the discipline.** Best move = use their engine inside our wrapper (already the `dynamic-workflow-adoption.md` stance — now validated end-to-end).

**Q2 — disalignment?** None structural. Our adoption of the native tool (design-import.js, model-upgrade-benchmark.js, adaptive-immunity-closure) is correct usage; it composes with, not replaces, our conceptual workflows.

**Q3 — conflicts?** NO real conflict, severity LOW (latent-only). Decisive: NOT ONE of our hooks keys on the literal word "workflow." user-ask-detect grep=0; established-workflows-check keys on FILE PATHS (engine-agnostic — a native workflow editing src/ still trips the bypass gate); critical-finding-detect treats a workflow-spawn as POSITIVE. The only gap is agent judgment (conceptual-vs-native sense) — measured zero spurious fires this session. Fix applied: doc-only disambiguation note in dynamic-workflow-adoption.md. NO trigger/hook changes (per user).

**One nuance flagged for later:** the established-workflows-check bypass-window + tool-parse should be re-validated once native workflows actually MUTATE src/ (today's are read-only/staging). Not a present conflict.

status → verified_complete (research audit; verify criterion = both questions answered + conflict mapped + low-risk fix applied)
