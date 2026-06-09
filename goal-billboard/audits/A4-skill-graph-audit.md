---
audit_id: A4
title: Skill graph audit (cross-link + discovery health)
status: available
catalogued: 2026-05-26T09:50:00Z
priority_when_run: P2
estimated_effort: small
trigger: After any major skill restructure (e.g., agentic-script-design → agentic-architecture rename) OR after >5 new skill refs land OR every quarter
deferral_reason: Skill system is recently restructured (2026-05-25 split + 2026-05-26 git-hygiene addition). Want to let it stabilize before reflecting on graph health.
related_goals: [G4]
related_plans: []
related_refs:
  - .claude/skills/  (all 7 skills)
serves_northern_star: G2  # migrated 2026-05-26 - path 'skill' -> GL G4; NS defaults to G2
belongs_to_goal: G4
serves_guiding_light: G4
---
# A4 — Skill graph audit

## Why this audit matters

The skill system is the long-term memory of how to do work. As skills accumulate refs, splits happen, and renames occur, the graph can develop:
- Dead cross-links (`[[name]]` pointing to renamed/deleted refs)
- Orphan refs (live but never linked from any SKILL.md)
- Description rot (SKILL.md `description:` field describes capabilities the skill no longer covers)
- Overlap (two skills' descriptions both claim the same triggering language)

The `skill-cross-link-rebuild.sh` PostToolUse hook handles the AUTO-SIBLINGS block but doesn't catch these other rot patterns.

## What it would look at

- Walk `.claude/skills/*/SKILL.md` and `.claude/skills/*/references/*.md`
- Check every `[[link]]` and `references/foo.md` path mention resolves
- Check every ref under references/ is mentioned somewhere in its parent SKILL.md
- Compare each SKILL.md's description against the actual ref content — does it overstate / understate?
- Detect description triggering-language overlap (e.g., two skills both surface on "test the matrix")

## Expected outputs

- `notes/skill-graph-audit-{date}.md` with findings
- A patch to any stale SKILL.md descriptions
- A patch to fix dead cross-links
- A recommendation if a skill needs to split / merge / rename

## How to trigger

Whichever fires first:
- Major skill restructure happens (the deferred `agentic-script-design` → `agentic-architecture` rename would be one)
- 5+ new skill refs land since last audit
- A trigger-language collision causes wrong skill to surface

## Notes

Could be partially automated (link resolution is mechanical). Description-overlap detection probably needs human judgement.
