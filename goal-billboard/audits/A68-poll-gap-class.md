---
audit_id: A68
title: Poll-gap class — agent doesn't surface inbox during text-only/BG-wait turns
status: in_progress
catalogued: 2026-05-27T18:11:00Z
priority_when_run: P0
estimated_effort: small (design landed; 3-shape deploy in flight)
trigger: 2026-05-27T18:05Z user — "i answered the banana backlog again and you had no clue, thats a red flag triggering a negative signal" + 3× session recurrence
related_plans: [context/markdowns/plans/poll-gap-class-fix-design.md]
serves_northern_star: G2
belongs_to_goal: G4
related_refs:
  - scripts/hooks/agent-inbox-drain-stop.sh (Shape B — deployed; user adds settings.json line)
  - scripts/hooks/agent-inbox-drain-notification.sh (Shape A ext — deployed; user adds settings.json line)
  - scripts/event-bus/consumers/consumer-heartbeat-tier-low-daemon10.sh (Shape C — deployed; 3 deploy-mode options)
findings:
  - poll-gap-narrower-than-diagnosed: agent-inbox-read.sh fires on PreToolUse `.*`, so tool-call responses DO drain. Real gap = (a) text-only responses, (b) parent silence during BG-subagent execution.
  - 9-substrates-with-class: agent-inbox + chamber data.json + scheduler-events + lost-work + 5 others (per A68 plan §2); 4 are mid-turn-actionable.
  - 3-shapes-deployed-this-session: A (Notification hook), B (Stop hook), C (Heart tier-low Daemon #10); all 3 scripts written + executable + dry-run-default; user settings.json gates remain.
---

# A68 — Poll-gap class fix

## What this audit names

Failure mode that fired 3× in 2026-05-27 session: user posts to inbox between/mid agent turn; agent doesn't see until next chat (which may be much later). User explicitly: *"red flag triggering a negative signal."*

## Resolution

Plan landed at `context/markdowns/plans/poll-gap-class-fix-design.md` (244 lines). Triple-tether deployment per A62:

| Tier | Mechanism | Script | User-action gate |
|------|-----------|--------|-------------------|
| Hardware | Notification hook (Shape A) | `scripts/hooks/agent-inbox-drain-notification.sh` | Add to `.claude/settings.json` `hooks.Notification` |
| Hardware | Stop hook (Shape B) | `scripts/hooks/agent-inbox-drain-stop.sh` | Add to `.claude/settings.json` `hooks.Stop` |
| Hardware | Heart tier-low Daemon #10 (Shape C) | `scripts/event-bus/consumers/consumer-heartbeat-tier-low-daemon10.sh` | Pick 1 of 3 deploy modes (scheduler tick / LaunchAgent / cron) |
| Software | Inline self-poll discipline (Shape D) | (no script — discipline) | n/a |
| Biology | `feedback_decision-handling-discipline.md` anti-pattern entry | Memory rule | ✅ deployed |

## A62 acid test

| Tier | Status |
|------|--------|
| Hardware | ⚠ 3 scripts written + dry-run-default; user adds settings.json lines to activate |
| Software | ✅ inline self-poll discipline encoded |
| Biology | ✅ memory rule + this audit + steering log |
| VERIFY | ⚠ 7 assertions specified in plan §7 (CLASS-1 regression test simulates 18:05Z incident); tests pending wire-up |

3/4 deployed + VERIFY pending wire-up. Best-scored adaptive-immunity response in our catalog when fully gated.

## Cross-references

- A48 (sync auto-fire — parallel class; sync wired but doesn't auto-fire on decision-arrival)
- A45 (server-death class — parallel)
- A62 (adaptive immunity — A68 follows the 5-step loop)
- A65 (autonomic system — Daemon #10 catalogued)
- A66 (decision-handling — Notification/Stop deploys NOT high-risk-gated; agent wrote scripts)
- A67 (domino analysis — A68 is the SD#3 derivative)

## Status

PHASE 1 LANDED 2026-05-27T18:20Z. Phase 2 (user adds settings.json lines + picks Daemon #10 deploy mode + VERIFY tests wire-up) pending user authorization.
