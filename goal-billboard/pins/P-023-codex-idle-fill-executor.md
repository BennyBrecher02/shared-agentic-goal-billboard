---
pin_id: P-023
title: Codex idle-fill executor — AFTER standalone Codex proven
kind: paused-work
state: pinned
created: 2026-07-08
origin: Fable-orchestrated-Codex research (context/markdowns/research/codex-orchestration-slamdunks-2026-07-08.md §6)
leverage: high (frees Fable parallelism onto free Codex compute — but ONLY once proven)
---

# P-023 — Codex idle-fill executor (AFTER standalone Codex proven)

> **The reworked "drop Codex into an idle juggle-plow slot" idea — refined + pinned, NOT dropped.**

## What this is
Make Codex an **idle-fill EXECUTOR option** the scheduler *may* dispatch expensive busywork to when Fable
has spare parallelism — plugging into the existing idle-capacity research-furthering slot (A63/A81) as one
executor among several, behind the same 7 boundaries.

## Why it's PINNED, not active (the framing the user corrected twice)
The **scheduler / juggle-plow** system and **Codex** are **SEPARATE ideas**. Fusing them now is premature —
*"we have to use it first to know it before ingraining it."* So the standalone-Codex work (the 5 ranked
slam-dunks in the research doc) happens FIRST; this pin is the **bridge**, resumable only on proof.

## Refined future-phase spec
- Codex becomes an idle-fill executor **only for lanes already proven standalone** (matrix-farm first).
- **Manual graduation only** — never the default idle target until the user explicitly greenlights the
  scheduler dispatching to Codex.
- Reuses the existing rails: `task-inbox.sh --to codex` (hand-off) + `task-run-matrix.sh` / `codex exec`
  (execute) + `results/` + `complete` (collect) + `codex-block-dangerous.sh` (deny gate). No new system.
- Same idle-fill boundaries as A63/A81 research-furthering (genuine work only, never the critical path).

## Resume criteria (unpin when ALL met)
1. ≥1 slam-dunk lane from the research doc run **end-to-end** (recommended opener: #1 matrix-farm).
2. Codex reliability + latency + cost characterised from that real run.
3. **Explicit user greenlight** to let the scheduler dispatch idle-fill work to Codex.

## Cross-references
- Research + the 5 ranked slam-dunks: `context/markdowns/research/codex-orchestration-slamdunks-2026-07-08.md`
- Named candidate lanes: `project_codex-free-compute.md` (memory)
- N=3 Codex adapter (BUILT): `context/markdowns/research/external/codex-adapter-research-2026-06-09.md`
- Idle-capacity discipline this plugs into: `feedback_idle-capacity-furthering.md` (A63/A81)
