---
idea_id: I-017
title: Graceful mid-plow rate-limit handling — a resumable checkpoint on abrupt limit-hit
lifecycle: captured
serves_northern_star: G10   # agent lifecycle graceful start/stop; advisory
source: floated-ideas-audit-2026-06-07
source_ref: context/markdowns/goal-billboard/audits/findings/floated-ideas-audit-2026-06-07.md
leverage: low
related: scripts/hooks/token-budget-check.sh (the existing pre-flight; this is the abrupt-hit complement)
created: 2026-06-07T00:00Z
updated: 2026-06-07T00:00Z
---

# I-017 — Graceful mid-plow rate-limit handling (resumable checkpoint)

**What it is.** A Stop-hook checkpoint-write so that if we get abruptly cut off by a rate limit mid-plow, the
session always leaves behind a resumable chain-of-thought marker (what was in flight, the next concrete
step) — turning an abrupt limit-hit into a clean resume point instead of a cold restart.

**Why it matters.** The user floated it (23f93bfd, 2026-05-26): *"how do we potentially eventually handle
randomly getting hit by rate limit when we're plowing…"* — it was answered in conversation only, with no hook
built beyond the existing token-budget pre-flight check (`scripts/hooks/token-budget-check.sh`). Filesystem
state already survives a hard stop, so this is a **nicety**, not a gap — it just makes the resume explicit
rather than reconstructed.

**Concrete first step.** Add a small Stop-hook that writes a resumable CoT marker (in-flight files +
verified-vs-not + exact next step — the compaction-survival invariants) to a known checkpoint path, so any
abrupt cutoff leaves a deterministic resume note.

**Leverage: LOW.** Awaiting greenlight (surface-only by contract).
