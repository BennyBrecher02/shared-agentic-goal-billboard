---
audit_id: A17
title: "5hr-reset operator alerting — pre-emptive countdown vs after-the-fact"
status: in_progress
catalogued: 2026-05-26T22:00:00Z
priority_when_run: P1
estimated_effort: small (timing-source audit + alert ladder + heartbeat-tier alignment)
trigger: User pushback 2026-05-26T21:23Z — "how were you late for the 5pm reset? you reminded me at around 5:20 for a task that started long after 5, audit our timing stat handling again"
deferral_reason: NONE — running now. A14 covered the stats-correlation infrastructure (session age + burn rate + 5hr-window tracker) but did NOT cover OPERATOR-facing pre-emptive alerts. The mechanical infrastructure works; the alert timing doesn't.
related_goals: []
related_plans:
  - context/markdowns/plans/automation/anti-idle-infrastructure-plan.md
related_refs:
  - .claude/cache/5hr-window-reset.txt
  - scripts/claude-5hr-window.py
  - scripts/heartbeat/check-token-budget.sh
  - context/markdowns/goal-billboard/audits/A14-stats-time-correlation-audit.md
findings: []
owner: agent
serves_northern_star: G2  # migrated 2026-05-26 - no signal -> default to current NS (G2)
---
# A17 — 5hr-reset operator alerting audit

## Why this audit matters

The user reported being **alerted at 5:20pm local for a 5pm reset** — 20 minutes LATE — and even then only because the agent flagged it incidentally, not pre-emptively. The right shape is countdown alerts (T-30, T-15, T-5) so the user can pace before the reset, not after.

A14 (stats-time-correlation audit, 2026-05-26 PM) built the timing substrate:
- `.claude/cache/5hr-window-reset.txt` — the user-provided reset timestamp.
- `scripts/claude-5hr-window.py` — CLI to query / set.
- `scripts/heartbeat/check-token-budget.sh` — Low tier (15min) check that surfaces "5hr-window resets in N min" via heartbeat queue.

**But A14 only solved the data-availability problem**. The operator-facing alert timing was assumed to follow naturally from the heartbeat tier. It didn't:
- Heartbeat Low tier fires every 15 minutes. The first time post-reset that the heartbeat would catch a "T-15" window is up to 15min stale. By the time the agent surfaces it, it's already mid-reset.
- The agent's own pre-stop checklist doesn't include "did I countdown the user before the reset?" — so even if data exists, the agent doesn't act on it pre-emptively.

## Hypothesized root causes

### RC-A: Heartbeat Low tier interval is too coarse for fine-grained reset alerts

15-minute interval means worst-case 14m59s before a countdown fires. For T-5 alerts, this is unacceptable: the window can close before the heartbeat fires.

**Recommendation**: lower-tier-intervals OR add a dedicated "5hr-reset countdown" tier with 5min interval for the last hour before reset.

### RC-B: No "pre-emptive surface" — the agent waits for heartbeat instead of running the check itself

The agent only sees `check-token-budget.sh` output when the heartbeat queues it. The agent could query `scripts/claude-5hr-window.py status` itself at SessionStart and at every response that touches `token-budget-check.sh` output, but doesn't.

**Recommendation**: SessionStart hook + standing protocol says "if 5hr-reset within next 30min, surface in CURRENT response, not wait for heartbeat."

### RC-C: No clear "alert ladder" — only one threshold instead of T-30 / T-15 / T-5

Current code emits a single notification when threshold crossed. The user wants a **graduated** ladder: 30min advance heads-up, 15min "wrap up your current thing," 5min "stop or you'll get cut off."

**Recommendation**: explicit T-30 / T-15 / T-5 dedup keys with progressively higher severity.

### RC-D: Drift between when reset.txt was set + when reset actually happens

The user enters the reset timestamp manually (no Claude API for this). If they enter it imprecisely (e.g. "17:00 local" when reset is actually 16:55), the alerts fire at the wrong time.

**Recommendation**: drift detection — when a heartbeat sees a "T-0" alert that was scheduled for T-N min, log the actual time + delta. After 3 entries, suggest the user re-set the timestamp.

### RC-E: Standing protocol doesn't say "always surface 5hr-reset proactively"

`feedback_standing-protocols.md` doesn't mention 5hr-reset surfacing. So when the agent decides what to surface in a response, the reset isn't on its checklist. Without an explicit standing-protocol entry, it's optional.

**Recommendation**: add an explicit entry — "if 5hr-window reset within 30min and no alert in current response, surface it BEFORE other content."

## Cross-reference with heartbeat tier-low's 5hr-reset check

`scripts/heartbeat/check-token-budget.sh` Part 1 already implements the threshold-bucket pattern (line 36-50ish). The dedup buckets exist (5hr-reset-30min / 5hr-reset-15min / 5hr-reset-5min). The gap is **interval coarseness** — Low tier at 15min means a T-30 alert could be 15min late.

**Architecture decision**: rather than dropping the Low tier to 5-min globally (would spam other low-priority checks), introduce a **conditional finer tier** during the last hour before reset:
- Normal: Low tier 15min.
- Within 60min of reset (per `.claude/cache/5hr-window-reset.txt`): Low tier 5min for THIS check only.

This is implementable as a guard in `tick.sh` that checks reset.txt and downgrades the Low-tier interval when reset is near.

## What "fix" looks like

| Layer | Fix | Status |
|---|---|---|
| Data | reset.txt + claude-5hr-window.py + token-metrics.json | Done (A14) |
| Background | check-token-budget.sh Part 1 (heartbeat) | Done (A14) |
| Operator alert (Tier-low coarse) | 15min interval | Insufficient |
| Operator alert (fine-grained near reset) | 5min interval during last 60min | **To do — anti-idle plan §3** |
| In-response surfacing | SessionStart + per-response opportunistic check | **To do — anti-idle plan §3** |
| Standing protocol | Explicit "surface 5hr-reset within 30min" rule | **To do — memory update post-audit** |

## Expected outputs

1. This audit catalogued (done).
2. `anti-idle-infrastructure-plan.md` §3 covers the alert ladder + tier-conditional interval.
3. Standing-protocols memory entry added — "5hr-reset within 30min → surface in current response, don't wait for heartbeat."
4. One real session where T-30 / T-15 / T-5 alerts all fire correctly, before reset, with the user confirming receipt.

## How to trigger

User pushback 2026-05-26T21:23Z met. Re-trigger naturally if: another reset passes without ≥1 of the 3 countdown alerts being surfaced in chat before reset.

## Resolution criteria

- Tier-conditional interval implemented + verified by 1 reset cycle
- Standing-protocols memory entry added
- 3 alerts (T-30 / T-15 / T-5) surface in-chat (not just queued)
- User confirms the alerting feels timely

## Cross-references

- A14 (stats correlation) — built the data substrate; A17 builds the operator-facing layer on top
- `scripts/heartbeat/check-token-budget.sh` — current 5hr-reset code
- `context/markdowns/plans/automation/anti-idle-infrastructure-plan.md` — implementation
- `feedback_standing-protocols.md` — where the explicit-surface rule lands
