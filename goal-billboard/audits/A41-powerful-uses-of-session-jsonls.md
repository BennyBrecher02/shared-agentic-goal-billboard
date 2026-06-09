---
audit_id: A41
title: Powerful uses of session JSONLs — brainstorm 20+ ways to mine ~/.claude/projects/<project>/<session>.jsonl
status: in_progress
catalogued: 2026-05-27T03:47:54Z
priority_when_run: P1
estimated_effort: medium (exploratory brainstorm + prioritization; specific builds spawn per-use)
trigger: |-
  2026-05-27T03:47Z — user explicit: *"brainstorm more insanely powerful uses of ~/.claude/projects/<project>/<session>.jsonl."* Companion to A40 (gap-detector — the FIRST powerful use); this audit catalogs THE OTHER ones.
deferral_reason: NONE — brainstorm fits foreground; per-use builds queue
related_goals: [G14, G4]
related_plans: [context/markdowns/plans/automation/session-jsonl-mining-plan.md]
serves_northern_star: G2
belongs_to_goal: G11
serves_guiding_light: G14
related_refs:
  - reports/conversation-extracts/ALL-PROMPTS.md (A38 substrate)
  - scripts/extract-user-prompts.py (the extractor pattern; per-use scripts follow)
  - A38 + A39 + A40 (the first 3 uses; A41 catalogs the rest)
findings: []
---

# A41 — Powerful uses of session JSONLs

## The substrate (recap)

`~/.claude/projects/<PROJECT_SLUG>/*.jsonl` — 6 sessions, 469MB, complete conversation history (user prompts + agent responses + tool calls + tool results + hook outputs). 296 user-typed prompts isolated to `reports/conversation-extracts/ALL-PROMPTS.md`. **This is one of our richest data sources and we'd been ignoring it.**

## The 25-item brainstorm

Ranked into TIERS by ROI × ease:

### TIER A — Wire NOW (high ROI, low effort, immediate user value)

**A1. Idea-gap detector** (A40 — SEPARATE AUDIT this turn)
- Find user prompts that didn't get serviced
- Recurring sweep + SessionStart surfacing
- THE user's named fear-killer

**A2. Slam-meter retrospective** (A39 — already queued BG)
- Classify every historical prompt by slam tier
- Calibrate the meter against ground truth

**A3. Voice analysis for skill-maker** (A38 — already queued BG)
- Look-through-Ben's-eyes ref
- Pre-flight prediction layer

**A4. Token-spend attribution per prompt**
- Each prompt costs N tokens to respond to
- Which prompts are highest cost / lowest value?
- Ratio: artifacts-produced / tokens-spent = per-prompt ROI

**A5. Recurring topic detector**
- Find topics that keep coming up across sessions despite "we addressed it" claims
- The build-vs-use catcher at conversation-pattern level (A18-A29 cousin)

### TIER B — High-leverage exploratory

**A6. Cross-session decision-finality tracker**
- Find decisions the user explicitly made
- Verify they were honored in subsequent sessions
- Flag "we already decided this — why is it being re-litigated?"

**A7. Frustration timeline**
- When does Ben get frustrated? (timestamp markers + frustration phrases)
- Predict trigger conditions: long session length? Repeated mistakes? Specific topic classes?
- Inform: when to NOT add more BGs / when to take a breath

**A8. Anti-pattern detector ("things Ben calls mid")**
- Extract instances where user said "no", "wrong", "actually", "you're missing"
- Each is a calibration sample for what NOT to do

**A9. Master-prompt-template extractor**
- User's hallmark prompt shapes (e.g., "build X with Y constraints because Z")
- Extract templates for re-use when proposing future tasks

**A10. Calibration trainer (predicts user reactions)**
- Train: prompt → agent-response → user-next-prompt-sentiment
- Use to pre-flight major responses

### TIER C — Sophisticated mining

**A11. Auto-skill-spawning trigger**
- When 5+ similar asks happen across sessions → auto-propose skill creation
- Eliminates "should this be a skill?" decisions

**A12. Auto-memory-rule spawner**
- When user repeats correction 3+ times → auto-propose memory rule
- Closes the user-shouldn't-have-to-repeat-this loop

**A13. Master changelog generator**
- Per-session: extract user prompts + spawned artifacts + steering decisions
- Compact changelog for retrospection

**A14. Cross-session continuity check**
- Does THIS session continue cleanly from where LAST one left off?
- Identify dropped threads automatically

**A15. Question-answer-link tracker**
- Find unanswered direct questions in history
- The Q-paired-with-A check

### TIER D — Speculative but interesting

**A16. Emotional state timeline**
- Wave plot of session-level enthusiasm vs frustration
- Cross-correlate with productivity (artifacts per prompt)

**A17. Knowledge-graph extractor**
- Entities (audits, goals, files) + decisions + cross-refs per prompt
- Build a graph; query "what touches X?"

**A18. Cross-project bleed detector**
- User mentions concepts from other projects → cross-project insight catcher

**A19. The "ask Ben" simulator**
- Trained on voice model; predict reactions to drafts BEFORE sending response
- Pre-flight check for major prompts

**A20. Auto-FAQ extractor**
- Questions user has asked multiple times → FAQ for skill refs / docs
- Reduces re-explanation

### TIER E — Substrate-level (enabling)

**A21. Session-bundle priming**
- Use JSONL pre-extraction to skip the parse step in `analyzer-themes.py`
- Faster bundle analysis

**A22. Test case generator**
- Historical decisions → unit test cases for behavioral skills
- E.g., "this was a decision the user made; future behavior should respect it"

**A23. Conversational shape miner**
- What conversational SHAPES lead to best outcomes? (e.g., user asks question → agent answers → user clarifies → agent proposes plan = high-success shape)
- Optimize conversational flow

**A24. Trigger-phrase ML calibrator**
- Auto-discover NEW trigger phrases the user uses
- A39's trigger list expands itself

**A25. Cross-session artifact-citation tracker**
- Find artifacts that NEVER get cited again after creation
- Identifies dead code in the artifact graph (skill refs with 0 inbound refs, etc.)

## Phases

### Phase 1 — Prioritize + scaffold top 5 (foreground; this turn → next session)

1. A40 idea-gap-detector (separate audit this turn)
2. A2 slam-meter retrospective (A39 BG queued)
3. A3 voice analysis (A38 BG queued)
4. A4 token-spend attribution — scaffold ONLY (script template + planned outputs)
5. A5 recurring topic detector — scaffold ONLY

### Phase 2 — Build out top 10 in Tier A + B (next 2-3 sessions)

After Phase 1 lands, pick the 5 from Tier B that demonstrated highest leverage in Phase 1 outputs.

### Phase 3 — Tier C + D opt-in (user picks; not pushed)

Some Tier C/D items might never get built. They live in this audit as a catalog of POSSIBILITIES, not commitments.

### Phase 4 — Tier E (substrate) — wired when corresponding bundle/session work needs them

Tier E items are enablers — built only when the consumer is ready.

## The compound effect

**The substrate enables COMPOUND mining**. Once we have:
- (A38) voice model
- (A39) trigger detection
- (A40) gap detector
- (A4) token attribution
- (A11) auto-skill-spawning

...then the system can run a single comprehensive sweep that produces:
- "Here's what you asked about across all sessions"
- "Here's what got serviced vs slipped"
- "Here's what cost the most"
- "Here's what wants to be a new skill"
- "Here's what wants to be a new memory rule"

That's a FULL self-reflection layer the user has been asking for since A24 (meta-loop-eval).

## Cross-references

- A38 — voice substrate (Phase 1 BG queued)
- A39 — trigger detection + slam-meter (Phase 1 BG queued)
- A40 — first powerful use (gap-detector); separate audit this turn
- A24 — meta-loop-eval (this is its substrate)
- G14 — master system audit (consumes A41's outputs)
- A18-A29 family — A41's Tier-A items together complete the cross-session shape catcher

## Status

IN PROGRESS — brainstorm complete (25 items, tiered, this turn); Phase 1 scaffolds queue; Phase 2-4 user-driven prioritization.

