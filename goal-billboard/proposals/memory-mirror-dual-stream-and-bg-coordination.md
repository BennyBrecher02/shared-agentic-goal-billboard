---
title: "PROPOSAL — Memory-mirror dual-stream + BG-session coordination substrate"
status: proposed
flagged: 2026-05-31
flagged_by: user (explicit — "make sure this idea persists!!!! its a slam dunk to expand on later")
priority: parked behind G20 boss handoff
related:
  - feedback_memory-mirror-sync (the existing both-dirs discipline)
  - the BG / spawned-local-session chip-cascade problem
  - feedback_bug-billboard (the append-only async queue fix)
---

# Memory-mirror: dual-stream + BG coordination (user-flagged slam-dunk)

## The user's intent (2026-05-31)
1. **Keep the OLD memory-mirror bundle** from the EviumOverhaul-era session as a preserved **archive**.
2. **Start a NEW memory-mirror stream** for the NEW main orchestrator agent (in `agentic-organic-os`).
3. **Eventually expand the memory-mirror** to help **soothe the BG / spawned-local-session issues**.
   User flagged this explicitly as a slam-dunk to expand later. **MUST persist — do not lose.**

## Honest status — planned vs. new
- **De-facto half-built by the migration (a byproduct, not a deliberate design):** the memory MIGRATION
  copied the full brain to the new slug (the new live stream, seeded from old); the OLD slug + the OLD
  repo's `.claude/memory-mirror/` are intact (the archive); and `mv`-ing the repo carried its
  `.claude/memory-mirror/` into the new home. So a two-stream setup EXISTS today: **OLD** (EviumOverhaul
  slug + mirror = frozen archive) and **NEW** (agentic-organic-os slug + mirror = live, written-to forward).
- **NOT yet designed (NEW as of 2026-05-31):** (a) a DELIBERATE dual-stream model — explicit archive-vs-live
  separation, provenance/lineage, divergence rules; and (b) the big one — the memory-mirror as a **shared
  coordination substrate for BG / spawned sessions.**

## The slam-dunk — memory-mirror as BG-session coordination layer
Ties directly to the known BG pain: spawned-task chips spin off INDEPENDENT sessions the orchestrator
can't track; they freeze and spawn more chips. The fix already in motion = spawned agents APPEND to the
bug-billboard instead of spawning chips. The EXPANSION = a shared memory-mirror that every BG/spawned
session reads + appends to:
- each BG session writes its **state/progress/findings** → the orchestrator SEES them (no freeze-in-the-dark),
- the orchestrator reads the mirror to **consolidate** (no chip cascade),
- the shared brain is **readable by every BG session** (grounded, not blind).
This could generalize the bug-billboard-append fix into a full BG-coordination layer.

## Which stream the new orchestrator uses
The NEW main orchestrator (agentic-organic-os) reads/writes the **NEW** stream (new slug auto-load memory
+ the new repo's `.claude/memory-mirror/`), seeded with the full prior brain. The OLD stream stays frozen
as the archive. **Going forward = the new stream.**

## Next (when prioritized — AFTER the G20 boss handoff)
- Design the deliberate dual-stream model (archive vs. live; provenance).
- Design the BG-coordination mirror (append schema, orchestrator consolidation, freeze-avoidance).
- Reconcile with `feedback_memory-mirror-sync` + the bug-billboard.
- Add a one-line MEMORY.md index pointer to this proposal (so the brain carries it).

## Status
PROPOSED — user-flagged slam-dunk. Parked behind the boss handoff. **Do not lose this.**
