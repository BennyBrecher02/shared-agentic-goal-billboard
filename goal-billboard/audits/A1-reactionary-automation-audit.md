---
audit_id: A1
title: Reactionary-automation audit (gray-area decisions)
status: available
catalogued: 2026-05-26T09:50:00Z
priority_when_run: P1
estimated_effort: medium
trigger: 20+ entries in steering-decisions-log.md OR 10+ "agentic vs scripted" decisions accumulated OR a user-flagged sense of "we keep making this call inconsistently"
deferral_reason: Need sample data to be meaningful — too few decision points right now (~30 steering log entries, mostly bootstrap)
related_goals: [G1, G4]
related_plans: []
related_refs:
  - .claude/skills/agentic-script-design/SKILL.md
  - context/markdowns/notes/steering-decisions-log.md
  - context/markdowns/notes/agentic-vs-scripted-gray-area.md
serves_northern_star: G2  # migrated 2026-05-26 - body keyword 'cluster' -> GL G7; NS defaults to G2
belongs_to_goal: G4
serves_guiding_light: G7
---
# A1 — Reactionary-automation audit

## Why this audit matters

Most decisions about "should this be agentic or scripted" are gray-area judgement calls made on a per-step basis. After enough of them accumulate, patterns emerge — same shape decision being made *differently* by the same agent across sessions is a calibration signal. This audit captures that signal.

## What it would look at

- All `steering-decisions-log.md` entries tagged with agentic-vs-scripted shape
- Per-decision: outcome (did the choice pay off — fewer subagent dispatches OR cleaner outputs OR matched user intent)
- Clusters: are there decision shapes where we consistently go one direction when the other was correct?
- Trends: did we drift over time toward over- or under-automation?

## Expected outputs

- A note in `context/markdowns/notes/reactionary-automation-audit-{date}.md`
- Calibration recommendations (e.g., "next time you're in shape X, default to script-first")
- Updated skill ref text if a pattern is clear (e.g., new section in `agentic-script-design/references/`)
- Maybe new steering log entries documenting the recalibration

## How to trigger

Whichever fires first:
- `steering-decisions-log.md` crosses **50 total entries** (currently ~30; 20 more from this baseline)
- 10+ entries that read as "switch to script" or "keep agentic" decisions
- User says "I've been making the same call differently lately — audit"

## Notes

This audit was a deferred concept since 2026-05-25. The original framing was a SKILL ("reactionary-automation-audit" as a reactionary skill) but it really lives at the audit-queue layer — fires when concern arises, not when a run group completes.
