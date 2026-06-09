---
title: "Safe fresh-agent startup convention + fearless auto-mode (AGENTS.md + sandbox)"
status: proposed (researched 2026-05-31)
priority: parked behind G20 boss handoff
related: prime-orchestrator-swap-capture-and-handoff-protocol.md (Part 7 boot sequence)
---

# Safe fresh-agent startup + fearless auto-mode

## The question
A root-level "startup folder" that gives any fresh agent a consistent, safest-possible boot — enough to
run it on **auto mode with no worries.** Is it smart? What do top systems do?

## What the ecosystem actually does (researched)
Two **separate** layers, often conflated:

### Layer 1 — the startup CONVENTION (consistency): `AGENTS.md`
- The emerging **cross-tool open standard** — a "README for agents" at repo root. Adopted by **20,000+
  repos**, originated by OpenAI, now governed by the **Linux Foundation's Agentic AI Foundation**, read
  **natively** by Codex, GitHub Copilot, Cursor, Windsurf, Amp, Devin (and Claude via CLAUDE.md).
- Concise (<300 lines), structured, imperative, operational. **Nested** like `.gitignore` (nearest file wins).
- ⇒ This is the "consistent startup" answer — but it's **instructions, not enforcement.** A confused agent
  can still ignore it. It makes onboarding consistent; it does NOT by itself make auto-mode worry-free.

### AGENTS.md + CLAUDE.md together (the cross-tool split — for widening to Cursor)
- **CLAUDE.md** is read ONLY by Claude Code and sits at the TOP of its authority hierarchy (Claude-specific
  overrides win). **AGENTS.md** is the UNIVERSAL baseline — read natively by Cursor, Copilot, Codex, OpenCode.
  When both exist, Claude Code merges them (CLAUDE.md wins for Claude-specifics).
- **Three sync strategies:** (1) `@AGENTS.md` IMPORT inside CLAUDE.md (Claude reads it inline); (2) symlink
  `ln -sf AGENTS.md CLAUDE.md`; (3) pointer ("READ AGENTS.md FIRST").
- **Recommendation for US** (we carry heavy Claude-Code-specific machinery — hooks, deny-list, memory slug):
  use the **IMPORT** approach, not a symlink. **AGENTS.md = the single canonical source** (boot sequence +
  canon pointers + cross-tool rules → portable; Cursor reads it). **CLAUDE.md = `@AGENTS.md` + ONLY the
  Claude-Code deltas** (settings.json deny-list mechanics, hooks, the slug). A symlink can't hold those
  deltas; the import can.
- Widening to Cursor: AGENTS.md carries the contract; Cursor correctly ignores CLAUDE.md's Claude-specifics
  (it has its own `.cursor/rules` + permission model). **Full stack: AGENTS.md (universal) ← CLAUDE.md
  (Claude deltas via @import) · .cursor/rules (Cursor deltas)** — one canonical brain, tool-specific edges
  isolated. Mirrors the portability discipline we already built.

### Layer 2 — the SANDBOX (fearlessness): what actually enables "no worries auto"
- The industry verdict on YOLO/auto on your own machine: **risky** (an agent can touch unintended files,
  run destructive commands, or be prompt-injected). The fix is **isolation**, not instructions.
- **The recommended pattern: YOLO/auto + sandbox = "no approval prompts, but bounded execution."**
- Mechanisms: DevContainers / Docker microVM sandboxes; **Claude Code's built-in OS-level sandbox —
  Seatbelt on macOS, bubblewrap on Linux** — isolating filesystem + network. Cursor reports sandboxed
  agents stop **40% less often.** This is the literal "auto mode with no worries" unlock.

## So — is the folder smart? Yes, but it's only half
- **Do `AGENTS.md`** (or CLAUDE.md) as the root entry — it's the *standard*, cross-tool, and it generalizes
  our **Part-7 boot sequence** (POST → ingest canon → resume) into the file *every* agent auto-reads.
  Prefer it over a bespoke `boot/` folder: a custom folder isn't auto-discovered by other tools; AGENTS.md
  is. Keep it thin and have it *point into* the canon dirs (latest digest, guardrail summary, deny-list).
- **For no-worries auto, add the SANDBOX** — that's the missing layer. On macOS (this machine) Claude Code's
  Seatbelt sandbox is built-in and low-effort.

## Tradeoffs
| Layer | Gives | Cost | We have it? |
|---|---|---|---|
| `AGENTS.md` boot convention | consistency, cross-tool onboarding | cheap | partially (handoff prompt + canon-digest — not yet a root AGENTS.md) |
| Deny-list (settings.json) | blocks catastrophic ops (push/force/rm-rf/mv~) on host | have it | ✅ |
| **Sandbox (Seatbelt/devcontainer)** | **true isolation — agent can't escape the project or reach the network** | moderate setup; **restricts cross-project + network** | ❌ (the missing piece) |

## Brainstorm — the nuance that matters for US
- The sandbox's restriction *aligns* with our existing rule: a network-isolated agent **literally cannot
  push** — which is exactly "agent never runs git mutations." Sandbox = that rule, enforced by the OS, not
  by discipline. Belt-and-suspenders with the deny-list.
- **But:** a sandbox confines the agent to its mounted project. **This very handoff is a cross-repo special
  case** — the agent must reach `~/Claude/Design/EviumOverhaul` for zach's tree. A strict sandbox would
  block that. So: **keep per-approval + deny-list for the cross-repo handoff** (what we're doing — correct),
  and reserve **sandbox + auto** for *future single-project* fresh-agent work (where it's ideal).
- Net stack for fearless single-project auto: **AGENTS.md (boot) + deny-list (floor) + Seatbelt sandbox
  (isolation).** Three layers; we have one, should add the other two.

## Recommendation
1. Add a root **`AGENTS.md`** that = the standing boot sequence (points to latest canon-digest + guardrails).
2. Adopt **Claude Code's macOS Seatbelt sandbox** for future single-project auto-runs → fearless auto there.
3. Keep the current **per-approval + deny-list** for the cross-repo handoff (sandbox would over-restrict it).
PROPOSED — parked behind the handoff.
