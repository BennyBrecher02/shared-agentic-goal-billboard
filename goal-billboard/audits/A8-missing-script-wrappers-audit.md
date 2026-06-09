---
audit_id: A8
title: Missing script-wrapper audit
status: available
catalogued: 2026-05-26T10:55:00Z
priority_when_run: P2
estimated_effort: small
trigger: When an agent finds themselves re-running the same multi-line shell incantation 3+ times in different sessions OR when adding a new skill ref that describes a workflow with no script citation OR before any major skill consolidation pass
deferral_reason: Most of the originally-proposed missing wrappers (capture-region.sh, computed-styles.sh, console-log-scrape.sh) have been functionally replaced by the Chrome plugin (Mode 4 computed-style, Mode 8 live CSS, Mode 6 network). The residual question is small enough to be a 30-min audit, not a plan.
related_goals: [G4]
related_plans: []
related_refs:
  - .claude/skills/agentic-webdesign/SKILL.md
  - .claude/skills/agentic-page-scrutiny/SKILL.md
  - .claude/skills/agentic-chrome-tools/SKILL.md
  - .claude/skills/reactionary/references/reactionary-skill-refresh.md
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'skill ref' -> GL G4; NS defaults to G2
belongs_to_goal: G4
serves_guiding_light: G4
---
# A8 — Missing script-wrapper audit

## Why this audit matters

The skill bodies describe workflows in prose ("run the matrix at this viewport, screenshot the section, read getComputedStyle"). Some of those workflows have an `scripts/*.sh` wrapper that an agent could invoke (`scripts/record-matrix-timing.sh`, `scripts/android-emulator-capture.sh`). Others are still prose-only, meaning each agent run re-invents the exact incantation.

The original plan (`skill-scripts-integration.md`, deleted 2026-05-26) framed this as a 3-step plan. Most of its scope has been absorbed by `reactionary-skill-refresh` (event-triggered consolidation) and by the Chrome plugin (which made several proposed wrapper scripts unnecessary).

This audit captures the residual question: **are there any wrapper scripts we'd still genuinely benefit from?**

## What it would look at

- Walk `.claude/skills/*/SKILL.md` and `references/*.md`
- For each described workflow that involves a multi-step shell incantation, ask:
  - Is there an existing script in `scripts/` that wraps it? (grep + filesystem check)
  - If yes — is the script CITED in the skill? (if no, the citation is a 30-second fix; just edit inline)
  - If no — is the workflow used often enough to warrant a wrapper? (heuristic: appeared 3+ times in agent-recommendations or session traces)
  - If a wrapper would help — would it duplicate Chrome-plugin functionality? (Mode 4, Mode 8, etc.)

## Expected outputs

- `notes/missing-script-wrappers-{date}.md` with findings
- Categorized table:
  - **Wrap it** — workflow is shell-only + reused + not covered by Chrome plugin
  - **Cite existing** — script exists, just needs the SKILL.md citation
  - **Drop the prose** — the Chrome plugin replaced this workflow; the prose should reference the plugin instead
  - **Leave alone** — one-off workflow; wrapping wastes effort
- Direct edits where applicable (cite-existing is mechanical)

## How to trigger

Any of:
- Agent re-runs the same multi-line shell incantation 3+ times across sessions
- Adding a new skill ref that describes a shell workflow with no citation
- Before any major skill consolidation pass
- Right before a `reactionary-skill-refresh` run

## What this replaces

The deleted `plans/automation/skill-scripts-integration.md` (2026-05-24 draft). Its core question lives here; its mechanical "cite existing scripts" sub-step happens inline as discovered.

## Notes

If at any point the missing-wrapper count grows past ~5, this audit upgrades to a goal: "G_N — Skill-script wrapper buildout." For now, the catalog entry is the right grain.
