---
audit_id: A15
title: "Research folder underutilization — discipline isn't being followed"
status: completed
catalogued: 2026-05-26T19:30:00Z
priority_when_run: P1
estimated_effort: small (survey + backfill candidates + plan)
trigger: User observation 2026-05-26 PM — "i feel like you havent been utilizing the 'context/markdowns/research' folder at all, you havent been adding to it or organizing it or automating learning from it etc, this is an easy layup, make a plan+audit"
deferral_reason: NONE — run immediately. The standing-protocols memory entry explicitly mandates research → research/, and a same-session lapse means the rule isn't sticky.
related_goals: []
related_plans:
  - context/markdowns/plans/research-folder-automation-plan.md
related_refs:
  - context/markdowns/research/README.md
findings:
  - context/markdowns/goal-billboard/audits/findings/A15-research-utilization-2026-05-26-initial.md
phase_progress:
  phase_1_backfill: complete (5 candidates filed in research/ on 2026-05-26)
  phase_2_advisory_hook: complete (scripts/hooks/research-folder-advise.sh — 2026-05-26 PM)
  phase_3_cross_link_harvest: complete (scripts/hooks/research-cross-link-harvest.sh — 2026-05-26 PM)
  phase_4_synthesis_tool: complete (scripts/run-research-synthesis.py — 2026-05-26 PM; first real run produced reports/research-synthesis-2026-05.md, status closed 2026-05-26 23:45Z)
  skill_ref: complete (.claude/skills/agentic-quality-discipline/references/research-folder-discipline.md)
  settings_patch: complete (scripts/hooks/settings-patches/research-folder-advise.json — user wires manually)
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'skill ref' -> GL G4; NS defaults to G2
belongs_to_goal: G4
serves_guiding_light: G4
---
# A15 — Research folder utilization audit

## Why this audit matters

The `context/markdowns/research/` folder was created 2026-05-26 AM with a clear schema (sources + findings_drove frontmatter) and the standing-protocols memory entry says "When research is conducted → write to research/." But within the SAME session that created the convention, multiple research-shaped investigations landed in `notes/` instead — meaning the discipline isn't being followed even by the agent who designed it.

The cost: research insights don't compound across sessions. Each new session re-investigates topics rather than reading prior findings. The folder exists; the habit doesn't.

## What it would look at

- File count + freshness of research/
- Misrouted research-shaped content currently in notes/ or plans/
- Whether existing artifacts conform to the schema
- Whether downstream skill refs link back to research that informed them
- Whether any automation enforces the discipline (hooks / advisory checks)

## Expected outputs

1. Findings doc (initial) at `findings/A15-research-utilization-2026-05-26-initial.md`
2. Plan at `context/markdowns/plans/research-folder-automation-plan.md` documenting 4 phases
3. Memory upgrade strengthening the standing-protocols entry
4. Backfill candidate list — user approves moves; not blanket-moved

## How to trigger

User observation 2026-05-26 met. Run now. Re-trigger naturally if: another session passes without a research/ addition while research-shaped work was clearly done.

## Phase 5 update (2026-05-27) — v2 upgrade landing

User re-flagged research folder underutilization at 2026-05-27 00:25Z (3rd time in <24h): *"RESEARCH FOLDER UPGRADE NEEDED HEAVILY, we barely utilize it even though i asked many times."* Per A19 meta-recurrence rule, applied 6-step structural fix not band-aid. v2 upgrade landed as A31 Front 5:

- **Sub-folder taxonomy**: 8 sub-folders (architecture/, infrastructure/, agent-os/, evium-deliverable/, cluster/, testing/, sessions/, methodology/). 13 of 13 existing files migrated to matching homes per their `findings_drove:` + content keywords.
- **Auto-indexer**: `scripts/render-research-index.py` writes `reports/research-index.md`. Wired into `billboard-data-refresh.sh` (PostToolUse) + `dashboard-data-prime.sh` (SessionStart). 15 files indexed; 14 inbound refs detected on first run.
- **SessionStart surface**: `scripts/hooks/research-surface-on-resume.sh` + `scripts/_research-surface-parser.py` emit top-3 recent research files at session start. Tested working (jq-formatted `hookSpecificOutput.additionalContext`). Settings patch at `scripts/hooks/settings-patches/research-surface-on-resume.json`.
- **Lint extension**: `scripts/lint-research-frontmatter.py` validates `sources:` + `findings_drove:` + `domain:` + sub-folder match. `--strict` exits non-zero. `--fix-domain` patched 13 files in one run. 4 pre-existing files flagged for missing `sources:`/`findings_drove:` (legacy artifacts; can be patched in follow-up).
- **Skill ref v2**: `.claude/skills/agentic-quality-discipline/references/research-folder-discipline.md` upgraded to v2. 15.3K bytes (was 6.7K). New sub-folder decision matrix + v2 frontmatter schema + auto-index / SessionStart / lint sections.
- **Cross-reference updates**: 69 stale path references across 18 files auto-updated to new sub-folder paths (audits, plans, skills, scripts, reports).

Companion plan: `context/markdowns/plans/automation/research-folder-v2-upgrade-plan.md`.

Re-trigger logic for v3: if user flags underutilization a 4th time despite v2, the structural gap is something else (cadence, agent default-folder bias, etc.); v3 audit needed.
