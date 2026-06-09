---
pin_id: P-018
title: Cursor-WITH-agents integration — how to wield our system in Cursor using Cursor's agents (not just the terminal CLI)
created: 2026-05-28T23:23Z
why_paused: user explicit "im not gonna do the cli just yet so put a pin in cursor stuff for a bit"
resume_criteria: when benchmarking/ shows Composer is worth integrating DEEPER than the decoupled handoff, OR the user explicitly unpins. (2026-06-03: user affirmed keep-parked even as the decouple directive landed — the benchmark is the next step, not this deep-integration research.)
estimated_cost: ~60-100k tokens (deep research + design)
leverage: TBD — the user believes there ARE positives I under-explored
state: pinned
related: context/markdowns/research/rabbit-holes/RH-020-cursor-integration/ (RH-020 parts 1+2)
---

# P-018 — Cursor-WITH-agents integration research (PINNED)

## What I got wrong (the correction that spawned this pin)

RH-020 part 1 concluded "run Claude Code CLI in Cursor's terminal; **never invoke Cursor's own AI agent**." That answered a NARROWER question (how to avoid per-token billing) than the user asked. The user pushed back (2026-05-28T23:23Z):

> *"if my boss wants me in cursor ill work in cursor so i was also asking about researching how we can theoretically use cursor agents WITH our system too, and also just in general what are some upgrades we can add to our system if we would be working in cursor, and also how can we maybe wield this system in cursor with their agents, you made it sound like there were no possible positives but i think more research is needed"*

**The framing error**: I treated Cursor's agent as pure competitor to avoid. But the user's real question is the harder, more interesting one: **can Cursor's agents and our system COMPOSE?** There are plausible positives I dismissed too fast.

## What this pin should research (when resumed)

1. **Cursor agents + our system composition** — can Cursor's agent operate on a repo where our Claude-Code substrate (hooks, skills, scheduler, audits) also lives? Do they coexist, complement, or collide? Can Cursor's agent CALL our scripts/workflows? Can our agent hand off to Cursor's agent for specific tasks (e.g. Cursor's strong multi-file tab-edit)?
2. **System upgrades unlocked BY working in Cursor** — does Cursor's codebase index / diff-UI / Composer enable new capabilities for our system we can't do in a bare terminal?
3. **Wielding our system IN Cursor with their agents** — the hybrid: our orchestration discipline + Cursor's agent execution. Is there a division of labor (our agent plans + audits + verifies; Cursor's agent does fast multi-file edits)?
4. **The billing reality check** — using Cursor's agents DOES bill per-token to the boss's plan. So the research must weigh: is the capability gain worth the per-token cost vs running our Claude-Code CLI on the Max sub? When is each right?

## Why pinned (not done now)

User: "im not gonna do the cli just yet so put a pin in cursor stuff for a bit." They're not setting up Cursor yet. Researching deep integration now is premature — the boss directive hasn't fully landed + the user wants to settle other things first.

**2026-06-03 update:** the decouple directive has now landed, but the user explicitly affirmed **keep-parked** — the decoupled `benchmarking/` handoff is the chosen next step. The resume trigger above is re-pointed accordingly (resumes only if the benchmark shows Composer is worth integrating *deeper* than a hand-off), so it stops surfacing on the bare "directive landed" condition.

## NOT in this pin (already done / separate)

- RH-020 part 1 (billing — Claude Code in terminal) ✓ done
- RH-020 part 2 (feature comparison KEEP/ADOPT/AVOID) ✓ done
- "What Cursor does better → slam-dunks to build into OUR system" — this is a SEPARATE do-now question (RH-020 part 3), NOT pinned, because it's capability-inspiration for our own system, independent of Cursor integration.

## Cross-references

- RH-020 parts 1+2 (the prior cursor research)
- RH-020 part 3 (cursor-does-better slam-dunks — the do-now sibling)
