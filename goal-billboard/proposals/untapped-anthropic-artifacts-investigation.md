---
title: "Untapped Anthropic session artifacts — investigated + utilization plan"
status: proposed (investigated 2026-05-31)
priority: parked behind G20 boss handoff
related: prime-orchestrator-swap-capture-and-handoff-protocol.md, canon-digests/
---

# Untapped Anthropic session artifacts — investigated

## TL;DR — a verify-don't-claim correction
An earlier pass listed `telemetry/` + `session-env/` + `file-history/` as untapped "goldmines."
Direct investigation (2026-05-31) **corrects that**: the **60MB JSONL transcript remains THE goldmine**;
the rest are supplementary, and **two are empty**. Honest inventory below — this is the kind of overstated
claim the completeness audit + investigation exist to catch.

## What's actually on disk (`~/.claude/`, this install)
| Artifact | Size | Reality | Mineable value |
|---|---|---|---|
| `projects/<slug>/*.jsonl` | ~60MB/session | **THE goldmine** | full conversation + per-turn token usage (the 5 analyzers + completeness audit) |
| `telemetry/` | 204K | **PARTIAL** — `1p_failed_events.*.json` (failed-to-upload buffer only) | per-session model (`claude-opus-4-7[1m]`) + active betas + env fingerprint + event names + timestamps |
| `sessions/` | 16K | **REAL** — `<pid>.json` metadata | `sessionId → cwd → startedAt → version → entrypoint` = ORCHESTRATOR LINEAGE substrate |
| `shell-snapshots/` | 12K | minor | zsh env snapshots (reproducibility) |
| `session-env/` | **0B** | **EMPTY** (124 dirs, no files) | none |
| `file-history/` | **0B** | **EMPTY** | none |
| `todos/` | — | not per-session mineable for `666b8e4c` | none here |

## The real opportunity (modest, but useful)
1. **Hard provenance in the digest.** The canon-digest's "Identity" is currently hand-estimated. Pull
   the *measured* truth from `sessions/<pid>.json` + `telemetry/*.<session-id>.*.json`: exact model,
   active betas, cwd, entrypoint, start/end timestamps. Measured > guessed.
2. **Orchestrator lineage from `sessions/`.** The `<pid>.json` registry maps every session → where/when/how
   it started. This is the verified backbone for the canon-digest lineage chain (which session booted which
   home, when, on what version).
3. **The JSONL stays primary** — `telemetry/` holds only *failed* events; never treat it as a complete
   usage log.

## Plan (parked behind the boss handoff)
- Add a tiny `session-provenance.sh` to the shutdown ritual's CAPTURE step: read `sessions/<pid>.json` +
  the session's telemetry events → emit a measured provenance block into the digest's Identity.
- Drop the `session-env` / `file-history` claims everywhere (empty).
- Core stays: JSONL + 5 analyzers + completeness audit.

## Status
PROPOSED — investigated, honest, corrected. Parked behind the boss handoff. Knowledge is now canon.
