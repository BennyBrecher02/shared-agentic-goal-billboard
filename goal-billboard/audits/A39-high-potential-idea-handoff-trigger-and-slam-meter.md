---
audit_id: A39
title: High-potential idea handoff — trigger + workflow + slam-meter (measure how hard we're actually slamming)
status: in_progress
catalogued: 2026-05-27T03:30:00Z
priority_when_run: P0
estimated_effort: large
trigger: 2026-05-27 user explicit — *"we need some new kinda trigger that has a full workflow for when i hand you an idea with high potential and i the tell you to flesh it out, because im handing you slam dunk after slam dunk and idk how hard we're truly slamming so far."* Sibling of A38 (skill-maker); the two together form the IDEA-HANDOFF-QUALITY layer.
deferral_reason: NONE — running immediately. Trigger/workflow definition is foreground; meter implementation queues for BG.
related_goals: [G14, G9]
related_plans: [context/markdowns/plans/automation/slam-dunk-handoff-workflow-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - .claude/skills/agentic-quality-discipline/references/slam-dunk-handoff.md (NEW; this audit creates it via Phase 1 BG)
  - A18 trigger-phrase rule (existing; A39 extends with new phrases + structured response)
  - A38 (sibling; user-voice empirical training)
findings: []
---

# A39 — Slam-dunk idea handoff workflow + meter

## The trigger event

User 2026-05-27 03:30Z: *"we need some new kinda trigger that has a full workflow for when i hand you an idea with high potential and i the tell you to flesh it out, because im handing you slam dunk after slam dunk and idk how hard we're truly slamming so far."*

Two problems user is naming:
1. **No structured workflow** — when user hands a big idea + says "flesh it out", my response shape is ad-hoc. Sometimes I land 5 artifacts. Sometimes I land 1. Sometimes I land nothing and just talk. There's no PROTOCOL.
2. **No slam-meter** — user can't tell how hard each handoff is actually getting slammed. Was that prompt a slam dunk? Or did it produce mid-effort? **The feedback loop is broken.**

## The trigger detection

A handoff qualifies as slam-dunk-grade when the prompt contains ANY of these signal classes:

### Explicit-flesh markers (HIGHEST CONFIDENCE)
- "flesh this out"
- "flesh it out"
- "lets flesh this out"
- "stew on this"
- "stew on it"
- "let me cook"

### Big-impact markers
- "this should have massive effects"
- "this one prompt should have massive effects"
- "this is the biggest stress"
- "major prompt with big implications"
- "this should be a chance to upgrade"
- "track it and handle it wisely"
- "lock in on this"

### Idea-handoff markers
- "what if we"
- "i have an idea"
- "new idea:"
- "brainstorm based on"
- "consider X and how we can"
- "how can we"
- "build a [new thing]"
- "make a new"

### Frustration-flesh-out markers (high signal — user already tried before)
- "we absolutely NEED to nail fixing this"
- "i'm pulling the trigger for an emergency"
- "this should never happen again"
- ALL-CAPS sections in long prompts

### Stylistic markers (sharp Ben-voice)
- "go ape" / "ape mode" / "ham" / "ham mode" / "max mode"
- "be ape" / "go full throttle"
- "dawg" (intimate frustration; demands action)
- "lock in" / "locked in"
- "real talk"

## The workflow (executed in this order; no shortcuts)

When the trigger fires, the agent MUST execute this 8-step workflow before its response ends:

### 1. CAPTURE (verbatim)
Quote the user's exact words in the audit/plan trigger field. **DO NOT paraphrase the slam-dunk seed.** The seed phrasing is what makes it slam-dunk-shaped.

### 2. CONTEXT (state read)
Note what surrounded the prompt: user's emotional state, prior thread, time-of-session, any frustration markers in the previous 3 prompts. This goes into the audit's `## Context` section.

### 3. DECOMPOSE (sub-asks)
Identify EVERY sub-ask within the prompt. Slam-dunk prompts often pack 3-7 distinct asks. Missing one = leaving slam on the table.

### 4. STEW (explicit pause)
Internal reasoning section: what are the multiple angles? What's the SECOND-order effect? What's the THIRD-order? Don't jump to surface-level response.

### 5. GENERATE (multi-angle options)
Generate ≥3 candidate approaches. Document trade-offs. Don't just take the first idea.

### 6. CHOOSE (with rationale)
Pick the highest-leverage option. Document WHY in steering log entry. **Trade-off rationale is part of the artifact.**

### 7. FLESH (concrete artifacts — the meter inputs)
For a SLAM DUNK, this MUST produce ≥4 artifacts:
- 1 audit (A<N>)
- 1 companion plan (`plans/automation/<name>-plan.md`)
- ≥1 memory rule update (mirror + auto-loaded)
- ≥1 steering log entry
- Cross-references to ≥3 existing audits/goals/plans (compounding into the system)
- Audit catalog README updated
- Tasks created/updated

### 8. MEASURE (the slam-meter)
At the end of the response, emit a SLAM TIER + the count.

## The slam-meter

```
🏀💥 SLAM DUNK     — ≥5 artifacts, all cross-linked, ≥3 cross-refs, user expressed enthusiasm next prompt
🏀  GOOD          — 3-4 artifacts, 1-2 cross-refs, response landed
🏀↓ MID            — 1-2 artifacts OR pure conversation despite seed-shaped-like-slam
🤷 FUMBLE         — 0 artifacts despite explicit-flesh marker; missed the opportunity
🚨 OVER-SLAM      — >10 artifacts on a sub-slam-grade prompt (over-investment; cost discipline violation)
```

The meter is reported in the response: e.g., "🏀💥 SLAM DUNK — A38 + A39 + 2 plans + 2 memory rules + extract script = 7 artifacts, 5 cross-refs."

### Calibration data (from this conversation, retrospective)

A38 estimates the historical slam-meter for major handoffs (to be filled by Phase 1 BG):

| Prompt | Trigger marker | Artifacts | Cross-refs | Tier |
|---|---|---|---|---|
| "build the scheduler heavily thorough" (May 25) | "heavily thorough" | scheduler plan + 4 sub-plans + 3 skill refs + memory + dashboard | 12+ | 🏀💥 |
| "lets make a new thin called rabbit holes" (May 26) | "lets make a new" | A35 + G15 + plan + methodology BG + research/rabbit-holes/ + memory | 6+ | 🏀💥 |
| "im annoyed why do you go ham" (May 26) | frustration | A29 + plan + memory + 4-layer prevention | 4+ | 🏀 |
| "extract every prompt + skill-maker" (now, this turn) | "lets flesh this out" + "literally" | A38 + A39 + 2 plans + extract script + 1 memory + steering log | 7+ | 🏀💥 |

## Phase 1 — Build the skill ref + retrospective slam-rating BG (queue for capacity)

A single BG that:
1. Reads `reports/conversation-extracts/ALL-PROMPTS.md` (the A38 substrate)
2. For each prompt, applies the trigger detection above → categorize as trigger-firing or not
3. For each trigger-firing prompt, maps to the artifacts it spawned (cross-reference to A-IDs, G-IDs, plans landed within next 5 prompts)
4. Computes slam-tier for each
5. Outputs `reports/audit-findings/A39-slam-meter-retrospective.md` — the calibration table
6. Outputs `.claude/skills/agentic-quality-discipline/references/slam-dunk-handoff.md` — THE skill ref with:
   - Trigger detection rules
   - 8-step workflow
   - Slam-meter rubric
   - Worked examples from the retrospective

Token budget: <500k. Wall-clock: <45min.

## Phase 2 — Wire as standing protocol + hook

- Add memory rule: "trigger-firing prompts MUST execute the 8-step workflow + emit slam-meter at end of response"
- Optional hook: `slam-meter-check.sh` on Stop — verifies emitted slam-meter line for trigger-firing prompts
- If no slam-meter emitted on a trigger-firing prompt: flag as FUMBLE candidate in next-response state-batch

## Phase 3 — Cross-session compounding

- After every graceful shutdown, the bundle analyzer's `analyzer-recurrence-detection.py` (G11 Phase 2) extends to include: trigger-firing-prompt count per session + average slam tier
- Master writeup (A34) gets a "Slam dunks landed this session: N (M🏀💥, K🏀, L🏀↓, P🤷)" line
- Cross-session trend: are we improving? Regressing?

## Anti-patterns (things that look like slam-dunks but aren't)

- **Bloat-slam** — N artifacts produced but they don't cross-link or compound. Volume ≠ slam. Mid.
- **Surface-slam** — artifact lands but doesn't actually address the deeper second-order ask. Mid.
- **Premature-slam** — agent jumps to FLESH without doing CONTEXT or STEW. Often produces wrong artifact.
- **Token-overspend-slam** — slam landed but cost more than the value delivered. Per A14 / G9, cost discipline matters.

## Why this matters

The user explicitly said *"idk how hard we're truly slamming so far."* That's a feedback-loop gap. Without the meter:
- User can't tell which prompts are getting slammed vs which are getting mid response
- Agent has no signal to calibrate effort
- Pattern of "build it / not use it" repeats (the A18-A29 family)

The slam-meter closes the loop.

## Cross-references

- A38 — sibling; supplies the empirical voice training (what makes a slam-dunk LOOK like one)
- A18 — trigger-phrase rule predecessor (A39 extends with new phrases + structured 8-step response)
- A29 — lazy regression catch (slam-meter is the cousin pattern for output-quality regression)
- A28 — skill health (slam-dunk-handoff skill ref refreshes per cadence)
- G14 — master system audit owner (slam-meter feeds the master audit)
- G11 / A34 — session bundle (Phase 3 cross-session trending)

## The user-visible promise

After Phase 1 lands:
- Every response to a trigger-firing prompt ends with a slam-meter line
- The skill ref is loaded; predictions can be made before the response
- Retrospective shows which historical prompts WERE slam dunks vs which the agent fumbled

After Phase 3 lands:
- Cross-session "slam-rate" KPI in the master writeup
- Agent self-improves: predicts → checks → calibrates

## Status

IN PROGRESS — workflow defined (this audit + companion plan); skill-ref creation BG queues for capacity. Phase 1 BG depends on A38 substrate (already extracted).

## Lessons (preliminary)

- The current prompt FIRES the trigger (explicit-flesh + idea-handoff + big-impact markers all present). This response is itself a test case.
- A38 + A39 are paired — substrate (voice training) + protocol (response shape). Neither alone is sufficient.
- The historical retrospective will be embarrassing. Some prompts the user thought were slam-dunks probably produced mid-effort responses. The retrospective surfaces those for honest calibration.
