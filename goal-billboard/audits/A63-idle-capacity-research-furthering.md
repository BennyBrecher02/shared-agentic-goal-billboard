---
audit_id: A63
title: Idle-capacity research-furthering — close the wait-window parallelism gap
status: in_progress
catalogued: 2026-05-27T16:50:00Z
phase_1_landed_at: 2026-05-27T16:50:00Z
priority_when_run: P1
estimated_effort: small (skill ref + memory rule + audit this turn; integration with Heart tier-medium deferred)
trigger: 2026-05-27 user explicit — *"i see a parallelism logic flaw, many times you'll start multiple processes and then while waiting for them to finish the number of running processes dwindles as you wait so you should maybe have a new thing that would trigger research furthering on idle agent count on wait for many things, is this something we can safely implement under certain circumstances in our system anywhere?"*
deferral_reason: NONE — concept + discipline + skill ref + memory rule land this turn; Heart tier-medium "idle slots available" signal integration deferred to A58 Phase 3.
related_goals: [G2]
related_plans: []
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - .claude/skills/agentic-quality-discipline/references/idle-capacity-research-furthering.md (NEW; the discipline)
  - .claude/memory-mirror/feedback_idle-capacity-furthering.md (NEW; standing protocol; both auto-load + mirror)
  - context/markdowns/research/rabbit-holes/RH-014-async-throughput-and-token-saves/findings/parallelization-safe-escalation-criteria.md (RH-014 baseline; this audit extends with refill discipline)
  - A21 audit (parallel-capacity ceiling — this discipline operates WITHIN that cap)
  - A47 audit (BG lifecycle — dispatch discipline this layer extends)
  - A58 audit (Heart tier-medium fires "idle slots" signal when integrated)
findings:
  - measured-this-turn-instance: |-
      A61 dispatched 4 BGs at T+0. BG3 landed T+5min; BG2 landed T+10min; BG4 landed T+10min; BG1 errored at T+4min + retry T+8min + landed T+15min. **Peak idle slots: 4 (for ~10 min cumulative).** This audit captures the missed dispatch window.
  - existing-discipline-gap: A21 cap is 5; A47 enforces per-BG discipline; RH-014 Track 4 named pipelining; NONE of these address the wait-window refill pattern. A63 closes it.
  - safety-boundaries-required: refill is safe IF (research-only / file-zones disjoint / within A21 cap / within token budget / not against user pivot signal). Anything else risks race or budget breach.
---

# A63 — Idle-capacity research-furthering

## What this audit names

A parallelism optimization the user identified in real-time during the A61 4-BG dispatch:

> *"i see a parallelism logic flaw, many times you'll start multiple processes and then while waiting for them to finish the number of running processes dwindles as you wait so you should maybe have a new thing that would trigger research furthering on idle agent count on wait for many things."*

The flaw: I dispatch N BGs in parallel → as they complete one-by-one, idle slots grow → I sit on idle capacity waiting. Each idle slot is an unused unit of parallel compute that could be doing research-furthering work.

## Quantification from this turn

A61 dispatch timeline (epoch start `1779899440`):

| Time | Running | Idle (cap=5) | What was happening |
|------|---------|--------------|---------------------|
| T+0 | 4 (BG1/2/3/4) | 1 | All 4 dispatched |
| T+4 | 3 (BG2/3/4) | 2 | BG1 errored at 4min |
| T+5 | 3 (BG2/3/4) | 2 | (foreground BG1 retry-prep work) |
| T+8 | 4 (BG1-retry/2/3/4) | 1 | BG1 retry dispatched |
| T+5 | 3 (BG1-retry/2/4) | 2 | BG3 landed |
| T+10 | 1 (BG1-retry) | 4 | BG2 + BG4 landed |
| T+15 | 0 | 5 | BG1-retry landed |

**Cumulative idle slot-minutes: ~30** (sum across all time-windows × idle slots). Most of that is the T+10→T+15 window where 4 slots sat empty for ~5 min — that's 20 slot-minutes of research-furthering capacity wasted.

## The discipline (the rule)

**When N BGs are in flight AND idle slots exist within the A21 cap AND token budget allows AND the parent task has natural research-furthering candidates, dispatch those candidates to fill the slots.**

## Safety boundaries (must all be true to refill)

| Boundary | Why |
|----------|-----|
| **Research-only candidate** (no code edits) | Eliminates race risk with in-flight work |
| **File-zone disjoint from in-flight BGs** | RH-014 safety prereq |
| **Within A21 cap of 5 total concurrent** | Existing parallel-capacity limit |
| **Token budget < 80% session ceiling** | Conservation guard |
| **User has NOT signaled pivot/stop** | Respect user intent |
| **Refill candidate is on-topic for parent task** | Don't random-spawn |
| **Each refill is logged via prelaunch row** (per A47 wrapper discipline) | Visibility |

If any boundary fails → DON'T refill. Sit on the idle slot.

## When this fires (the trigger)

The discipline fires when EITHER:

1. **Active mode** — agent dispatches an initial batch of N BGs; as completions arrive, agent surveys research-furthering candidates and refills.
2. **Heart tier-medium signal** (when A58 Phase 3 ships) — heartbeat detects `running_bgs < cap AND has_queued_research`; emits an "idle-slots-available" event; agent considers refill.

The active-mode version can land NOW (it's agent discipline + skill ref). The Heart-tier-medium version waits for A58 Phase 3.

## Research-furthering candidates (what to dispatch in idle slots)

Examples by parent-task shape:

| Parent task | Idle-slot research candidates |
|-------------|-------------------------------|
| Multi-BG audit (like A61) | Per-BG deep-dive on a single layer; cross-reference with sibling open-source projects; historical-similar-class survey across our own audits |
| RH-XXX research rabbit hole | One additional track refinement; literature review on a specific finding |
| Plow batch with multiple changes | Speculative matrix run on likely next batch; per-change CoT chain prep |
| Skill-ref drafting | Sibling skill ref candidates that pair; cross-link mapping |

Always: research-only, no code edits.

## Anti-patterns

| Anti-pattern | Why bad | Fix |
|--------------|---------|-----|
| Refill with code-changing BG | Race risk + may invalidate in-flight work | Research-only rule |
| Refill with off-topic work | Defeats the purpose; user wanted focused parallel | On-topic constraint |
| Refill that overlaps file-zones | RH-014 safety violated | Disjoint-zone rule |
| Refill that pushes token spend past 80% | Budget overshoot | Conservation guard |
| Refill against user pivot signal | Disrespects intent | Pivot-signal rule |

## Integration with the Organic OS

- **Brain** (CoT): each refill writes a SIDETRACK-INTENT entry (the parent BG batch is the primary PLOWING; refills are explicit sidetracks)
- **Heart** (heartbeat): tier-medium "idle slots available" signal triggers refill consideration when A58 Phase 3 ships
- **Immune** (innate + adaptive): the safety boundaries above are the adaptive layer's mutation — they prevent the failure mode of "filled an idle slot with conflicting work"
- **Organs**: liver-flow regulation (capacity-aware throughput) per A58 Phase 2.5 organs framing

## Retroactive applicability

When this discipline could've already applied:

- A61 4-BG dispatch (this turn) — captured above
- The earlier 3-BG B-K7-{a,b,c} dispatch (Phase A plow) — short batch (8-15 min total) so idle window was small
- Any future multi-BG audit-level work
- Any RH-XXX research that spawns multiple parallel deep-dives

## Status

PHASE 1 LANDED 2026-05-27T16:50Z. Discipline + skill ref + memory rule + this audit. Phase 2 (Heart tier-medium signal integration) deferred to A58 Phase 3 ship.

## Lessons

- **The user's eye for parallelism flaws is sharper than my own utilization discipline.** Watching real-time wait-windows is the right diagnostic that I wasn't doing.
- **Idle capacity is a resource that decays.** A slot you don't use this minute is a slot you didn't use. Unlike memory or disk, idle compute doesn't accumulate; it vanishes.
- **Safety boundaries are the lever.** Without them, opportunistic refill is dangerous. With them, it's pure win.

## Cross-references

- `idle-capacity-research-furthering.md` skill ref (the discipline)
- `feedback_idle-capacity-furthering.md` memory rule (binding)
- A21 audit (parallel cap baseline)
- A47 audit (BG dispatch wrapper discipline)
- A58 audit (Heart tier-medium signal — future integration)
- A62 audit (adaptive immunity — the safety boundaries ARE the adaptive mutation against refill-race failure)
- RH-014 Track 4 (pipelining was named; this is its formalization)
- RH-014 Track 5 (parallelization escalation criteria — this discipline operates WITHIN the cap, not raising it)

