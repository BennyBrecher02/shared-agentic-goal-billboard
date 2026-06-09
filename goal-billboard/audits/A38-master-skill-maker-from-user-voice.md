---
audit_id: A38
title: Master skill-maker skill — analyze every user prompt + build a meta-skill that looks through user's eyes
status: in_progress
catalogued: 2026-05-27T03:30:00Z
priority_when_run: P0
estimated_effort: large
trigger: 2026-05-27 user explicit — *"can you analyze LITERALLY EVERY prompt ive sent you in this conversation... extract the convo logs in a file form... want to use that to analyze how i ask my sharppest hitting highest result net prompts plans and just general ideas, and then utilize that info to make a master skill maker skill that literally looks through MY eyes, lets flesh this out."* This is itself a slam-dunk-grade prompt-handoff with "lets flesh this out" trigger — A39 (sibling) catches the meta-pattern.
deferral_reason: NONE — running immediately. Extraction is foundational; analysis BG queues for capacity.
related_goals: [G14, G4]
related_plans: [context/markdowns/plans/automation/master-skill-maker-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - reports/conversation-extracts/ALL-PROMPTS.md (296 prompts extracted; substrate for analysis)
  - reports/conversation-extracts/INDEX.md (per-session metadata)
  - scripts/extract-user-prompts.py (the extractor; reproducible)
  - A28 (skill health) — skill-maker is the inverse: instead of refreshing existing skills, BIRTH new ones
findings: []
---

# A38 — Master skill-maker skill (look-through-user's-eyes)

## The trigger event

User 2026-05-27 03:30Z (verbatim): *"can you analyze LITERALLY EVERY prompt ive sent you in this conversation? i know that your context window gets compacted constantly but does claude have some feature where we can extract the convo logs in a file form? do research on that because i want to use that to analyze how i ask my sharppest hitting highest result net prompts plans and just general ideas, and then utilize that info to make a master skill maker skill that literally looks through MY eyes, lets flesh this out."*

## What this is NOT

- NOT a one-off prompt analysis report
- NOT a meta-discussion of "what makes Ben tick"
- NOT a list of patterns observed

## What this IS

**A new skill** at `.claude/skills/look-through-bens-eyes/` (or similar) — a meta-skill that the agent invokes BEFORE major foreground responses. Its job: simulate the user's reaction in advance.

The skill is built from EMPIRICAL DATA — the 296+ prompts extracted to `reports/conversation-extracts/ALL-PROMPTS.md` — not from speculation.

### The skill's purposes

1. **Pre-flight check** — before I respond to a major prompt, the skill ref asks: "would Ben call this mid? Would he be frustrated? Would this be a slam dunk by his metrics?"
2. **Slam-dunk detection** — recognize the language patterns that indicate user is handing a slam-dunk-grade idea (separate from A39, but feeds it)
3. **Style matching** — when I generate skill refs / audits / plans, they should match the user's documented voice (terseness, jokes, naming conventions, abbreviations)
4. **Calibration** — the skill ref includes WORKED EXAMPLES of past prompts + the artifacts that scored well vs poorly
5. **Anti-patterns** — explicit "things Ben hates" list (corporate phrasings, "click when ready," fake completion claims, over-conservative BG launches, etc.)

## The substrate

`reports/conversation-extracts/ALL-PROMPTS.md` — 296 prompts across 6 sessions, chronological. Per-session breakdowns also available.

This is the EMPIRICAL TRAINING SET. The skill-maker analysis must cite specific prompts (by #NNNN reference) for every claim it makes about the user.

## Phase 1 — Analysis BG (queue for capacity)

A single large BG that:
1. Reads `reports/conversation-extracts/ALL-PROMPTS.md`
2. Categorizes EVERY prompt into types:
   - **Slam-dunk handoff** (big idea + "flesh it out" / "stew on this" / "massive effects" markers)
   - **Course-correction** (calling out a specific mistake or hallucination)
   - **Status check** (asking where things stand)
   - **Decision** (picking between options)
   - **Tangent** (related observation, not main thread)
   - **Affirmation** (approval / "yes proceed" / "go")
   - **Frustration** (repeated requests, "annoying", emphasized caps)
3. Maps each prompt to the artifacts that spawned from it (cross-reference to A-IDs, G-IDs, plans)
4. Computes per-prompt impact: # downstream artifacts × cross-references × persistence-across-sessions
5. Identifies the TOP 20 highest-impact prompts (slam dunks)
6. Identifies the BOTTOM 20 lowest-impact prompts (effort spent for little return)
7. Extracts patterns:
   - Linguistic hallmarks (vocab, sentence structure, abbreviation style)
   - Idea-handoff patterns (how user frames a big ask)
   - Emotional signals (frustration markers, enthusiasm markers)
   - Naming conventions (how user names artifacts vs how I name them)
8. Output: `reports/audit-findings/A38-user-voice-analysis.md` (corpus analysis)
9. Output: `.claude/skills/agentic-quality-discipline/references/look-through-bens-eyes.md` (THE skill ref)

Token budget: <800k (large substrate). Wall-clock: <60min.

## Phase 2 — Skill installation + first-use

- Skill ref lives under existing skill family (don't proliferate)
- Decision: which skill? Probably `agentic-quality-discipline` (it's about quality of my responses)
- First-use experiment: before next major user prompt, the agent explicitly invokes look-through-bens-eyes prediction; logs prediction + actual reaction; compares
- After 5+ uses, refine ref based on prediction accuracy

## Phase 3 — Recursive refinement

The skill ref gets refreshed (per A28 skill-health framework) when:
- New session JSONLs accumulate ≥50 new prompts
- A clear pattern shift is detected (user adopts new abbreviation, drops old one)
- A prediction is significantly wrong

The extractor `scripts/extract-user-prompts.py` runs as a SessionStart hook (cheap; 1s wall-clock) appending only new prompts to the ALL-PROMPTS.md.

## Cross-references

- A39 — sibling audit (high-potential-idea trigger + slam-meter); A38 supplies the empirical voice, A39 supplies the structured workflow
- A28 — skill health framework (refresh cadence)
- G4 — skill maintenance owner
- A35 / Ask-1 — internal eval predecessor (this is the next layer of self-reflection)
- A24 — meta-loop-eval (skill cross-utilization scanner; A38 feeds it)

## What's hard about this

- **Avoiding sycophancy** — the skill can't just be "say what Ben wants to hear." It must SIMULATE his reaction accurately, including the times he calls things mid.
- **Avoiding caricature** — capture style without becoming parody. The user's voice is sharp and specific; the skill must be calibrated, not exaggerated.
- **Keeping it small** — 296+ prompts × verbose analysis would balloon. Output skill ref must be <8KB, with worked examples that compress signal.

## Status

IN PROGRESS — substrate extracted (296 prompts at `reports/conversation-extracts/ALL-PROMPTS.md`). Analysis BG queues for capacity.

## Lessons (preliminary)

- The extraction took 1 second. The substrate was sitting in `~/.claude/projects/` the whole time, unused. **The data was always there; we just hadn't looked.**
- 469MB of conversation history × 6 sessions × multiple compactions × no one had asked to mine it. Pattern: substrate-built-but-not-used (same theme as A18-A29).
- This audit is itself a slam-dunk-handoff per A39's emerging framework — it spawns: A38 audit + A39 sibling + extraction script + plan + memory rule + future BG analysis + future skill ref + recursive refresh cycle. **Compounding 7+ artifacts from one prompt.**
