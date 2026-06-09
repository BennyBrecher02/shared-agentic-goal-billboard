---
idea_id: I-011
title: Master skill-maker — the GENERATIVE skill that authors new skills in Ben's voice (A38 Phase-1 payoff)
lifecycle: captured
serves_northern_star: G4   # skill-system maintenance; advisory
belongs_to_goal: G4
source: floated-ideas-audit-2026-06-07
source_ref: context/markdowns/goal-billboard/audits/findings/floated-ideas-audit-2026-06-07.md
leverage: high
related: A38 (master-skill-maker audit) · plans/automation/master-skill-maker-plan.md (Phase 0 done, Phase 1+ unrun)
created: 2026-06-07T00:00Z
updated: 2026-06-07T00:00Z
---

# I-011 — Master skill-maker: the generative "looks through MY eyes" skill (greenlight-ready)

> **The user's words (floated 3 sessions — 0d739f65, 23f93bfd, 23492bcf):** *"…utilize that info to make a
> master skill maker skill that literally looks through MY eyes, lets flesh this out."* This is the
> unrealized **payoff half** of a slam-dunk the user personally flagged: the analysis substrate shipped, the
> generative skill it was for never got built.

## What it is

A **generative skill** — not an analysis report — that AUTHORS new skills in Ben's own voice and style. It
mines his prompt corpus to learn how he writes his sharpest, highest-net prompts (structure, register,
slam-dunk markers, what he front-loads, how he scopes), then encodes those patterns into a skill-ref the
agent loads **before** drafting any new skill, so every skill we create reads like Ben wrote it. The
deliverable is the **skill-maker mechanism**, not a one-time voice study.

This is distinct from the generic `anthropic-skills:skill-creator` plugin (which knows skill *mechanics* —
SKILL.md shape, eval harness) — I-011 supplies the **voice + judgment layer** that plugin lacks: it's the
bespoke "write like Ben" lens, composable on top of skill-creator.

## Why it matters

- The user explicitly flagged it a slam-dunk and floated it across **3 separate sessions** — it kept
  surfacing and kept not getting built.
- **The substrate is already paid for.** `scripts/extract-user-prompts.py` runs and 296 prompts sit in
  `reports/conversation-extracts/ALL-PROMPTS.md`. The expensive extraction is done; only the
  pattern-synthesis + skill-ref authoring remain.
- **A38 is still `status: in_progress`** (`context/markdowns/goal-billboard/audits/A38-master-skill-maker-from-user-voice.md`),
  frozen at "Phase 1 analysis BG queues for capacity." The plan
  (`context/markdowns/plans/automation/master-skill-maker-plan.md`) has Phase 0 complete and Phase 1+ unrun.
  The `look-through-bens-eyes.md` ref the plan calls for **does not exist on disk** (confirmed absent).
- Without it, every future skill we draft re-derives "how would Ben phrase this" from scratch — leaving net
  quality on the table on a corpus we already mined.

## Scope (what's IN / what's OUT)

- **IN:** (a) a voice/pattern analysis pass over `ALL-PROMPTS.md` → a structured pattern set (slam-dunk
  markers, scoping habits, register, front-loading, the "flesh this out / lets talk it out" musing cues);
  (b) `look-through-bens-eyes.md` — a loadable skill-ref encoding those patterns as authoring guidance;
  (c) a thin generative wrapper/skill that, given a raw skill concept, drafts the SKILL.md applying the ref,
  composing with `anthropic-skills:skill-creator` for the mechanical shell.
- **OUT (defer):** any always-on hook that auto-rewrites prompts; auto-publishing skills without review;
  re-running the corpus extraction (already done). Keep it surface-and-review, not auto-apply.

## Concrete first step (the exact next action — greenlight-ready)

Run **A38 Phase 1**: analyze `reports/conversation-extracts/ALL-PROMPTS.md` for the slam-dunk + voice
patterns and emit `look-through-bens-eyes.md` as a skill-ref (the artifact the frozen plan names). That one
ref is the keystone — the generative wrapper (Phase 2) builds directly on it. This unblocks A38 from its
"queued for capacity" freeze with a single, scoped, already-resourced deliverable.

**Leverage: HIGH** — substrate already paid for; this is the unrealized payoff of a user-flagged slam-dunk.
**Awaiting greenlight** (the ideas lane is surface-only by contract; the user owns graduation to a build).
