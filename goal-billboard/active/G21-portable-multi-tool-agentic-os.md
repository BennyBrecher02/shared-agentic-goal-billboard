---
goal_id: G21
title: Portable multi-tool agentic-OS — portable core + per-tool adapters (the ongoing successor to G20)
status: active
track: infrastructure
priority: |-
  P1 (high leverage — the long-term build that lets the whole agentic-OS ride on any tool/repo/machine; compounds across every future project, not just Evium)
phase: >
  ACTIVATION landed — N=3 adapters (Cursor+Claude+Codex) + the kit now SELF-ACTIVATES on deploy:
  directive-skills (imperative USE-WHEN triggers + a SessionStart disciplines-primer) make the disciplines
  FIRE, not just sit; repo-bootstrap (deploy-kit.sh) brings the OS alive on clone; config-generator keeps
  every agent's native config in lockstep with --check drift; the activation-conformance detector (14/14)
  guards against passive reversion (passive-tree → RED, real-tree → GREEN). BLS field-harvest folded back
  (4 memories + 6 skill-refs + the post-project-harvest workflow). tiered-QA (SOLO default / MATRIX opt-in)
  + os-tools MCP (the ONE MCP — work-substrate retired 2026-06-21; 10 tools incl. cross-tool cot_state) staged.
  VERIFIED end-to-end on a fresh per-repo deploy (conformance 14/14, all 6 levers live). Remaining: wire the MCP
  (gated) + the cross-TOOL parallel-brains handoff (Claude×Cursor×Codex, ONE machine — the 2nd-Anthropic-sub /
  cross-node idea is RETIRED) + close the agnostic-% tail.
created: 2026-06-08T13:00Z
updated: 2026-06-21T21:01Z  # activation chapter (verified on disk): directive-skills + repo-bootstrap (deploy-kit.sh) + 4 BLS-mined memories + 6 skill-refs + post-project-harvest workflow + tiered-QA (SOLO/MATRIX) + os-tools MCP design + activation-conformance detector (14/14) + config-generator (--check drift). The OS now ACTIVATES, not just deploys. (prior: 2026-06-09 all-night plow — N=3 adapters Cursor+Claude+Codex, ~92% file-level agnostic, census 6->18 mechanisms.)
serves_northern_star: G21  # IS the Northern Star itself now (was G2, demoted 2026-06-03; R1 re-anchor 2026-06-09 — the OS is the end-point, the site is test-data)
guiding_light: true  # long-term build that compounds across future projects too
northern_star: true  # NS LOCKED to G21 2026-06-21 (user OK on the goal-grooming triage resolved the G8-vs-G21 question: G21 — the portable agentic-OS — IS the Northern Star, the user's stated end-point; G8 = deep-integration links UNDER G21, see linked_goals). Set 2026-06-09 (R1); G2 demoted, BLS is test-data not a goal.
succeeds_goal: G20  # G20 (RED-ALERT repo-decouple) is ACHIEVED; this is the ongoing portable-OS successor it seeded
leverage: high
linked_plans:
  - context/markdowns/plans/automation/brain-lifecycle-plan.md
linked_research:
  - context/markdowns/research/systems/convo-torch-protocol-2026-06-08.md
  - context/markdowns/research/external/agent-orchestrator-architecture-2026-06-07.md
  - context/markdowns/research/external/config-handling-architecture-2026-06-07.md
  - context/markdowns/research/external/leveraging-cursor-2026-06-03.md
  - context/markdowns/research/external/cursor-skill-registration-2026-06-03.md
  - context/markdowns/research/rabbit-holes/RH-020-cursor-integration/part-2-feature-wins.md
  - context/markdowns/research/rabbit-holes/RH-020-cursor-integration/part-3-cursor-does-better-slam-dunks.md
linked_refs:
  - AGENTS.md
  - .claude/skills/agentic-context-paging/SKILL.md
linked_audits: []
linked_bugs: []
linked_changes: []
linked_goals:
  # Sub-goals that sit UNDER G21 (the Northern Star) per the 2026-06-21 NS-lock decision.
  - G8 — Northern Star deep integration (the deep-integration arm of the portable OS; its remaining exit
    criteria #4 per-NS token attribution + #5 clears-path badge are dashboard surfaces under G18). Links UNDER G21.

# Chain of Thought integration (A60 / A58 Brain — schema parity with the other goal files)
cot_chain_id: "G21-portable-multi-tool-os"
last_cot_state: null
last_cot_ts: null
last_cot_summary: null
project: OS-core  # stamped by migrate-billboard-to-shared (shared store)
---
# G21 — Portable multi-tool agentic-OS (portable core + per-tool adapters)

## Why it exists

G20 (the RED-ALERT repo-decouple + ownership emergency) is **achieved** — it was the one-time forced
extraction of the agent OS out from under the boss's repo. But that emergency surfaced the real long-term
need it could only gesture at: the agentic-OS should not be welded to *one* repo, *one* tool (Claude Code),
or *one* machine. It should be a **portable core** that any AI coding tool can pick up, in any project, on
any machine — with thin, disposable per-tool adapters bridging the differences.

G21 is the **ongoing successor** that homes all of today's C→A→B portability work — the work that was being
parked under achieved-G20 only for lineage. G20 records the emergency that's done; **G21 is the active goal
to actually build the portable OS.**

## What G21 owns (the deliverables)

1. **The portable `agent-dev-env-kit`** — a tool-/project-/machine-agnostic core (the neutral portable layer
   from the canonical architecture: neutral portable core + tool-specific machinery + disposable adapters).
   This is the thing you drop into a fresh repo on a fresh machine and the OS comes alive.
2. **The 3 cross-env mirrors** — the portability spine that keeps the OS coherent across tools:
   - **memory mirror** — the canonical auto-memory mirrored to a tool-neutral store (today: `~/.claude/...`
     → `.claude/memory-mirror/`; generalized so Cursor/Composer read the same canonical conventions).
   - **skill mirror** — skills made available across harnesses (Claude Code skills ↔ `~/.cursor/skills/`,
     etc.) so the same disciplines fire regardless of tool.
   - **guardrail mirror** — the gated-op / deny-list / safety reflexes mirrored so a non-Claude-Code harness
     inherits the same guardrails (the high-risk-always-gated list travels with the OS).
3. **The convo-torch protocol** — the per-**environment** handoff: a `_core/torch` PAYLOAD schema +
   verify→render→inject pipeline that passes live context (decisions, DEAD list, open threads, in-flight
   state) from one environment/session to the next without a lossy re-derive. Design:
   `research/systems/convo-torch-protocol-2026-06-08.md`.
4. **The Cursor adapter** — the first concrete per-tool adapter: how to wield the SAME core from Cursor /
   Composer (skills registration, rules, AGENTS.md bridge, capability wins). Design:
   `research/external/leveraging-cursor-2026-06-03.md` + `cursor-skill-registration-2026-06-03.md` +
   RH-020 parts 2–3.

### VERIFY tethers (the portability promises made testable — adaptive-immunity discipline)

Two VERIFY tethers turn G21's *assumed* promises into *tested* invariants (run both:
`bash _core/contract-tests/run-g21-tethers.sh`; both also wired into the `run-tests.sh` fast tier):

- **torch round-trip selftest** — `_core/contract-tests/torch-roundtrip.sh` (+ `_core/torch/lib-torch.sh` verbs
  + `_core/torch/schema/torch-1.json`). The convo-torch is design-only, so this is the minimal selftest
  harness against the documented protocol (§7 P1): proves a same-env round-trip preserves the DURABLE layer
  (E/S/P/I + goals) byte-stable and that Verify fails CLOSED on a corrupt / truncated / forged / policy-stripped
  torch. Makes exit-criterion #3 testable BEFORE the first real adapter exists. 10/10 green.
- **adapter conformance contract** — `portable-kit/tests/adapter-conformance.sh` (+ spec
  `portable-kit/ADAPTER-CONTRACT.md`). The 5-verb contract every adapter must satisfy
  (deploy / resolve-memory / load-guardrails / report-health / render-torch), pinned at N=1 against the Cursor
  adapter — incl. a guardrail ATTESTATION (block-dangerous.sh actually DENIES 4 gated-op probes + ALLOWS a safe
  one). Makes the "thin adapter" promise real: a new adapter is "implement 5 verbs, pass the suite." 23/23 green
  (1 skip = the P2 full per-env render, not built yet).

## Why it's load-bearing (the leverage)

- **Compounds across every future project**, not just Evium — once the OS is portable, every new client
  site / app starts with the full agentic-OS already wired, on whatever tool the user reaches for.
- **De-risks tool lock-in** — if Claude Code, Cursor, or any single tool changes terms or capability, the
  core moves; only a thin adapter is rewritten.
- **De-risks repo/machine moves** — the same failure that made G20 an emergency (OS welded to one repo,
  the macOS-slug memory time-bomb) is structurally closed when the core is portable by design.

## Architecture basis (the LOCKED canonical shape)

Neutral **portable core** (the kit + the 3 mirrors + the torch) · **tool-specific machinery** (Claude Code
hooks/settings, Cursor rules/Composer) · **disposable adapters** (raw copies into a tool's expected location,
e.g. `~/.cursor/skills/`, regenerated freely). See the orchestrator-architecture decision
(`research/external/agent-orchestrator-architecture-2026-06-07.md`: per-env OS **A** vs one master OS **B**)
and the adapters config layer (`research/external/config-handling-architecture-2026-06-07.md`).

## Current state (2026-06-21) — agnostic + self-activating

- **N=3 adapters are real** — Cursor + Claude + Codex, with conformance suites green. The kit is
  **~92% file-level agnostic** (per the 2026-06-09 all-night plow).
- **The activation leap (the chapter that just landed):** the kit no longer merely *deploys* mechanisms —
  it **activates** them. **directive-skills** (imperative USE-WHEN triggers) + a SessionStart
  **disciplines-primer** make the disciplines FIRE rather than sit dormant; **repo-bootstrap**
  (`portable-kit/deploy-kit.sh`) brings the OS alive on clone (detect port, wire hooks, inject directives);
  the **config-generator** (`portable-kit/generate-agent-configs.sh`, `--check` drift) keeps every agent's
  native config in lockstep from one source; and the **activation-conformance detector**
  (`scripts/tests/test-activation-conformance.sh`, 14/14) proves the three activation mechanisms actually
  fire — a passive tree goes RED, a real tree goes GREEN — guarding against passive reversion.
- **BLS field-harvest folded back** — the first real field-test of the **post-project-harvest** workflow
  (`scripts/post-project-harvest.sh` + `.claude/workflows/post-project-harvest.js`): **4 memories** +
  **6 skill-refs** mined from ~2 weeks of Cursor/BLS usage now live in the OS.
- **tiered-QA** (SOLO default vs MATRIX opt-in) is staged in the kit AGENTS templates + cursor-rules.
- **os-tools MCP is designed** (thin-shell, 9 tools, stdio-local) as the STATE-carrying surface, with a
  working server (`scripts/mcp/os_tools_server.py`); the disciplines themselves stay in directive SKILL.md
  + hooks. (Design: `context/markdowns/research/systems/os-tools-mcp-design-2026-06-21.md`.)
- **Mirrors:** memory mirror (canonical auto-memory → `.claude/memory-mirror/`, one-way sync) + AGENTS.md
  harness-agnostic bridge are live; the skill + guardrail mirrors travel via the kit's adapter contract.

## Next action

- Close the gap-to-100%: stand up the **live torch render** (the P2 full per-env render is the one skip in
  the adapter-conformance suite), **extract the matrix-config** so MATRIX-mode QA is one-command on any
  deploy, and **close the agnostic-% tail** (the residual hardcoded-to-Claude-Code pieces) toward a fully
  tool-neutral core.

## Exit criteria

**The USER owns "done"** (per the manifesto rule — Claude never marks it solved). G21 reaches `achieved`
only when the user signs off that:
- ☐ The `agent-dev-env-kit` portable core drops into a fresh repo/machine and the OS comes alive.
- ☐ All 3 mirrors (memory / skill / guardrail) are real and cross-tool, not Claude-Code-only.
- ☐ The convo-torch protocol passes live context across at least one real environment handoff without a
  lossy re-derive.
- ☐ The Cursor adapter lets the SAME core be wielded from Cursor / Composer.

## History

- 2026-06-08 — G21 created at the user's request ("sure" to a new goal) as the ongoing successor to the
  achieved G20. Homes today's C→A→B portability work: the 8 portable-OS/torch/Cursor design docs re-mapped
  onto it (resolving the proposal's "Decision 1 — does a NEW portable-multi-tool-OS goal exist?" → YES).
  Spawned by the B4 goal-remapping pass (`audits/findings/goal-remapping-proposal-2026-06-08.md`).

## Cross-references

- **G20** (archived/achieved) — the RED-ALERT repo-decouple emergency that seeded this; G21 is its
  ongoing successor.
- **G2** — the Northern Star this portability accelerates (and every future project after it).
- **G4** (skill system maintenance) — the skill/memory/guardrail substrate G21's mirrors generalize across
  tools.
- **G17** (peak-throughput infra) — the cross-MACHINE story (cluster/multi-orchestrator); G21 is the
  cross-TOOL / cross-REPO story. Siblings, not duplicates.
