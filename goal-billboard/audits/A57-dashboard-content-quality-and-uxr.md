---
audit_id: A57
title: Dashboard content quality + UI/UX scrutiny (chamber post wall-of-text + render hygiene gap)
status: in_progress
catalogued: 2026-05-27T15:36:20Z
phase_1_landed_at: 2026-05-27T15:36:20Z
priority_when_run: P0
estimated_effort: medium (foreground standards + linter + chamber rewrite; BG for actual dashboard CSS overhaul)
trigger: |-
  2026-05-27T15:30Z — user screenshot of B-K7-c-FOLLOWUPS chamber post; user verbatim: *"this is absolute incoherent horrendous ui/ux. i cant tell whats going on in anything, two new enforcements must be made"*. The chamber post body was ~1500 words rendered as one unbroken paragraph; engineering jargon; literal `===` markdown delimiters in prose; no visual hierarchy between question, context, and option-card grid.
deferral_reason: NONE — landing this turn (foreground for standards + linter + chamber rewrite; BG for the dashboard CSS overhaul portion)
related_goals: [G2, G6]
related_plans: [context/markdowns/plans/dashboard-content-quality-and-uxr-overhaul-plan.md]
serves_northern_star: G2
belongs_to_goal: G18
serves_guiding_light: G14
related_refs:
  - .claude/skills/agentic-quality-discipline/references/dashboard-content-quality.md (NEW; the content rules)
  - .claude/skills/agentic-webdesign/references/dashboard-ui-ux-quality.md (NEW; the render rules)
  - .claude/memory-mirror/feedback_dashboard-content-quality.md (NEW; standing protocol)
  - scripts/dashboard-content-lint.py (NEW; pre-write linter)
  - scripts/post-to-chamber.py (NEW; mandatory wrapper for any chamber post)
  - A55 (sibling: dashboard tops scrutiny pass #2 — related but narrower)
  - A52 (sibling: bespoke lefter promoted to top of every dashboard page)
  - A53 (sibling: full matrix birdseye dashboard tab)
  - dashboard-centerpiece pattern memory (the original "no autoplay; meaningful not active" rule — A57 extends it)
findings:
  - dashboard-render-hierarchy: chamber renders question body as plain text with no markdown rendering, no paragraph breaks, no inline-code styling.
  - content-no-length-gate: I shipped 1500 words to a 5000-char input field designed for ~150-word questions; no linter caught it.
  - content-jargon: A47/file-zone/single-token-alpha-bump terminology with no user-context translation layer.
  - format-leakage: literal `===` heading delimiters survive into rendered prose as raw characters.
  - missing-wrapper: no `scripts/post-to-chamber.py` style gate that I MUST pipe through (parallel to `scripts/dispatch-bg.sh` for BG dispatch).
---

# A57 — Dashboard content quality + UI/UX scrutiny

## What triggered this audit

A user-facing chamber post (the `B-K7-c-FOLLOWUPS` decision I added at 2026-05-27T14:20Z) was rendered in the user's dashboard as a 1500-word wall of text. The user's full reaction:

> *"this is absolute incoherent horrendous ui/ux. i cant tell whats going on in anything, two new enforcements must be made 1) monkey type questions or any dashbord display will need better handling of language, vibe, length, formatting etc flesh this out and 2) the dashboard ui/ux needs further scrutinization and improvement before we can respond to the banana backlog we need to fulfill the completeion of these both hand in hand while also creating general useful skills/enforcements systemwide for the future, expound on this"*

Two distinct issues. Both must be fixed before the user engages further with the chamber.

## Failure breakdown (what the screenshot showed)

| # | Failure | Root cause |
|---|---------|------------|
| 1 | Question body ~1500 words | No length cap in agent's content-composition discipline. I treated the chamber input field's 5000-char ceiling as a target, not a worst-case bound. |
| 2 | `=== FOLLOWUP #1 — ...` literal delimiters in prose | Dashboard does not render markdown in the `question` field; I wrote in markdown-headed prose anyway. |
| 3 | Inline `rgba(26, 47, 58, 0.5)` and file paths in backticks running through prose | Dense code-in-prose is unreadable on a dashboard surface; should live in collapsible `context` or per-option `proposed_change.summary`. |
| 4 | Engineering jargon: "A47 discipline", "file-zone discipline", "single-token-alpha-bump" | User-facing surface should never assume agent-internal vocabulary. Translate every term that hasn't been canonized in a user-readable rule. |
| 5 | No visible hierarchy between question / context / options | Question pulls hierarchy from formatting; I provided none in either content or render. |
| 6 | No pre-write gate caught any of the above | Closest analogue (`scripts/dispatch-bg.sh` for BG prompts) doesn't extend to chamber posts. Gap. |

## What the standards must cover

Two distinct rule sets:

**Content quality (Track A — what the agent writes):**
- Length caps per field (title ≤14 words; question ≤150 words; context ≤150 words; option label ≤10 words; option summary ≤40 words)
- Voice/vibe (conversational; no engineering jargon; translate every term)
- Format constraints (no literal markdown delimiters; no inline backticks unless the surface renders them; one fact per sentence)
- Structural patterns (Q at top; context as "why this came up"; options as actionable verbs)
- Linter-enforceable (every constraint above is a regex or word-count check)

**Dashboard UI/UX (Track B — what the dashboard renders):**
- Markdown rendering for the `question` and `context` fields (paragraphs, lists, inline code)
- Visible hierarchy via typography scale (question = lead text; context = secondary; options = action grid)
- Density limits per surface (chamber decision card max 6 visible bullets above the fold; rest in "show more")
- Inline-truncation pattern with expand-on-click for over-budget content (so old long posts still readable)
- Cross-surface consistency: same hierarchy and density rules apply to billboard chips, asks panel, audit catalog summaries, scheduler queue items

## Recurrence diagnosis (sibling instances of the same class)

This is not a one-off. The same failure mode shows up across:

1. **Steering log entries** — multiple recent entries (e.g. P1-04 OTHER recovery; A47 BG lifecycle) are 600+ word run-on paragraphs. They're durable artifacts so the verbosity has a justification, but they're harder to scan than they should be.
2. **Plan summaries** — `g2-full-coverage-plow-plan.md` Current State section has paragraphs that re-state the same information in three forms.
3. **Audit titles in the catalog** — some are clear ("Reactionary automation audit"), others run-on ("Powerful uses of session JSONLs — 25-item brainstorm catalog").
4. **AskUserQuestion option labels** — I have a history of stuffing context into the label instead of using `description`; this turn's earlier "drift handling" question was an example the user explicitly flagged as unclear.

The chamber-post failure is the LOUDEST instance because the dashboard surface couldn't recover. But the same discipline gap exists across all user-facing content I generate.

## Track A — Content quality (this turn)

Deliverables landing in this audit's wake:

| Artifact | Path | Purpose |
|----------|------|---------|
| Skill ref | `.claude/skills/agentic-quality-discipline/references/dashboard-content-quality.md` | The rules + templates + examples |
| Memory rule | `.claude/memory-mirror/feedback_dashboard-content-quality.md` + auto-load mirror | Binding standing protocol |
| Linter | `scripts/dashboard-content-lint.py` | Pre-write validator; checks length, formatting, jargon |
| Wrapper | `scripts/post-to-chamber.py` | Mandatory gate for any chamber post (parallel to `scripts/dispatch-bg.sh` for BG dispatch) |
| Rewrite | `public/screenshot-monkey/data.json` — replace B-K7-c-FOLLOWUPS entry | Proof of new standards working |

## Track B — Dashboard UI/UX (this turn + BG)

Deliverables:

| Artifact | Path | Purpose |
|----------|------|---------|
| Skill ref | `.claude/skills/agentic-webdesign/references/dashboard-ui-ux-quality.md` | Visual/render rules for dashboard surfaces |
| BG dispatch | scrutinize current chamber/billboard rendering + apply CSS fixes | Markdown rendering, hierarchy, typography density on chamber decision cards + billboard chips + asks panel |

## The 5-pillar content quality model

The skill ref enumerates these in detail. Summary:

1. **Length** — every field has a hard cap; over-budget content goes into a `details` block that's hidden by default.
2. **Voice** — write to a 3-minute-attention-span human reading on a dashboard at 2pm. No jargon that hasn't appeared in a user-readable rule. Translate every term.
3. **Format** — match the rendering layer. If the surface doesn't render markdown, don't write markdown. If it does, use it sparingly.
4. **Hierarchy** — Question (what to decide) → Context (why now) → Options (what happens). Never bury the decision.
5. **Density** — one fact per sentence. No nested clauses. Lists for enumeration, prose for narrative.

## Hook proposal

PreToolUse hook `scripts/hooks/chamber-post-lint-pre.sh` runs `dashboard-content-lint.py` on any Write/Edit targeting `public/screenshot-monkey/data.json`. If lint fails, block the write. If lint warns, log + allow.

Same hook extends to:
- `context/markdowns/bug-billboard/inbox/*.md` (bug entries)
- `context/markdowns/goal-billboard/active/*.md` (goal active descriptions)
- `context/markdowns/plans/*.md` (plan TL;DRs and summaries)

## Cross-references

- A55: dashboard tops scrutiny pass #2 (sibling; narrower focus on top-of-page elements; A57 is broader content-quality + sitewide UX)
- A52: bespoke lefter promoted to top of every dashboard page (sibling)
- A53: full matrix birdseye dashboard tab (sibling)
- A18: trigger-phrase enforcement for user-prompt-driven artifact creation (parent class — user-facing artifact discipline)
- dashboard-centerpiece pattern memory (the "no autoplay; meaningful not active" rule; A57 extends to all chamber/billboard surfaces)

## Status

PHASE 1 LANDED 2026-05-27T15:36Z. Standards documented. Linter deployed. Chamber post rewritten under new rules. Dashboard CSS overhaul dispatched as BG.

## Lessons

- "5000-char input field" is a ceiling not a target. I should default to ≤200 words in any chamber field and only exceed with explicit justification.
- Engineering jargon that's clear inside the agent's head is opaque on a user dashboard. Translate every term that hasn't been canonized in a user-readable memory rule.
- The dashboard rendering layer is part of the content contract. Writing markdown to a non-markdown-rendering surface is the same defect class as writing HTML to a plain-text email.
- The fix for "I wrote a bad thing" is not "I'll try harder next time" — it's "I built a gate that catches the bad thing before it ships." Same logic as A47's dispatch-bg.sh wrapper.
- This is an A19-class meta-failure: the discipline gap was visible upstream (no length cap, no linter, no voice rule) and only became blocking when a single user-facing instance hit the dashboard. Sibling artifacts have the same gap; A57 closes it sitewide.

## Shared hardware barrier (2026-05-28 — adaptive-immunity retro-sweep)

The HARDWARE tether A57 was missing (a real pre-write gate on `data.json`; the linter was advisory-only) is now provided by a **shared barrier** built once and claimed by THREE instances — A57 (this) + A48 (sync-drift) + A61 (inbox-clog). Per the retro-sweep roadmap (`audits/findings/adaptive-immunity-retro-sweep-roadmap-2026-05-28.md`), this was the single highest-leverage move in the sweep: "block direct writes to `data.json` / inbox except via the sanctioned wrapper."

- **HW guard:** `scripts/hooks/wrapper-only-write-guard.sh` — PreToolUse `Write|Edit` hook. Detects a DIRECT agent write to `reports/screenshot-monkey/data.json` (and its `public/` symlink alias) and recommends the sanctioned `post-to-chamber.py` path. A wrapper-routed write never touches the Write/Edit tool, so the only thing the guard ever sees is the exact A57 failure path (a raw write bypassing the linter).
- **VERIFY:** `tests/wrapper-only-write-guard-test.sh` — A71-shaped, 5/5 passing. Assertion 2 reproduces the A57 wall-of-text scenario (raw `data.json` write) and asserts the barrier flags it; assertion 3 asserts the sanctioned wrapper passes silently. Fails if the guard is absent (verified).
- **Mode:** ships in **WARN/dry-run** (logs + warns, write proceeds; `exit 0`). Kill-switch `EVIUM_WRAPPER_GUARD_OFF=1`.
- **Live-flip is USER-GATED:** flipping WARN→BLOCK (`EVIUM_WRAPPER_GUARD_MODE=BLOCK`) + wiring into `settings.json` PreToolUse is an explicit USER action after a 24h dry-run review — NOT an agent turn (a stray match could block the user mid-edit). See the hook header for the exact flip steps.

