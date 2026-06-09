---
pin_id: P-022
title: Pinned threads — do NOT drop (2026-06-07)
state: active
created: 2026-06-07
---

# Pinned threads — do NOT drop

> Durable pin so these survive compaction + the audit blind spots. Surface each at its trigger.

## PARKED — resume on trigger
- **System-prompt A/B** — PARKED per user "pin it for now." Research DONE (`research/systems/system-prompt-value-and-metrics-2026-06-07.md`: scalpel-not-slab; small `--append-system-prompt` for 3-4 system-tier directives, A/B-proven). Resume = run the small A/B when the user un-parks.
- **A/B/C + ALL handoff artifacts** — resume AFTER the skill work (cross-env consistency + safety cluster + all-35 mining + skill gaps/audits/creations) is resolved. **REMIND the user** at that point. (A = Cursor bridge-kit + the guided handoff; B = journal pipeline [done]; C = extraction [one-script ready: `cursor-composer-handoff/extract-kit-do-it.sh`].)

## MUST-BUILD — don't forget
- **Global Claude setup** — build POST-extraction. `deploy claude` → `~/.claude/` global (skills/CLAUDE.md/settings/rules go global; memory stays per-project, the one boundary). Doc: `research/systems/global-claude-setup-2026-06-07.md`. **We are NOT globally set up today** — it's a future build.

## METHODOLOGY HOLE — the root cause of the misses
- The request-audits catch EXPLICIT requests but **MISS FLOATED IDEAS** — the skill-mirror + global-setup slipped exactly this way (recovered only when the user re-surfaced them). Fix: the audits + alignment-watchdog must scan for ideas-floated-in-passing and cross-ref vs what's tracked. (The skill-mining now hunts these; the watchdog still needs hardening — open.)
