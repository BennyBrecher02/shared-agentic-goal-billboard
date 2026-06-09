---
goal_id: G21
title: Portable multi-tool agentic-OS — portable core + per-tool adapters (the ongoing successor to G20)
status: active
track: infrastructure
priority: |-
  P1 (high leverage — the long-term build that lets the whole agentic-OS ride on any tool/repo/machine; compounds across every future project, not just Evium)
created: 2026-06-08T13:00Z
updated: 2026-06-08T13:00Z
serves_northern_star: G21  # IS the Northern Star itself now (was G2, demoted 2026-06-03; R1 re-anchor 2026-06-09 — the OS is the end-point, the site is test-data)
guiding_light: true  # long-term build that compounds across future projects too
northern_star: true  # SET 2026-06-09 (R1 fix): the OS (G21) is the Northern Star — the user's stated end-point. G2 demoted; BLS is test-data, not a goal. (Confirm G8-vs-G21 if you meant the deep-integration goal instead.)
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

## Current state (2026-06-08)

- **Design corpus complete** for the first slice: convo-torch protocol drafted; orchestrator + config
  architecture decided; the Cursor research consolidated (3 surviving docs + the wield-Cursor map);
  brain-lifecycle plan (unified memory-stream + handoff lineage) staged.
- **Mirrors partially real today** inside Claude Code: the memory mirror (canonical auto-memory →
  `.claude/memory-mirror/`, one-way sync) and AGENTS.md as the harness-agnostic convention bridge already
  exist; the skill + guardrail mirrors and the cross-tool generalization are the build-out.
- **Not yet built:** the standalone `agent-dev-env-kit` package, the `_core/torch` runtime, the live Cursor
  adapter. These are G21's work.

## Next action

- Lift the existing Claude-Code-internal mirrors (memory + AGENTS.md bridge) into a tool-neutral
  `agent-dev-env-kit` skeleton; pin the `_core/torch` payload schema from the protocol doc; stand up the
  Cursor adapter as the first per-tool proof.

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
