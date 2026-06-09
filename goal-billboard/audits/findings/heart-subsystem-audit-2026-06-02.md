---
title: "HEART subsystem audit — closing the gray area the defib workflow's HEART auditor left"
date: 2026-06-02
type: findings
audit_kind: system-infrastructure (heart / heartbeat)
trigger: "user 2026-06-02 — the defib workflow's HEART auditor failed to emit structured output; 'if the heart audit didnt report that sounds like a major gray area to just never look into ever again??' — correct. This is the dedicated HEART audit done properly, verifying the now-live state."
---

# HEART subsystem audit (2026-06-02)

## Why this exists
The `organic-os-defibrillation` workflow's HEART auditor (`parallel[0]`) failed to return structured
output. The orchestrator initially waved it off as "covered by the other auditors" — **lazy**: a failed
audit is an *unfilled* gray area, not a closed one. This is the dedicated re-audit, verifying the LIVE
state after the user installed the heart plist.

## The HEART has TWO independent layers (the thing the gray area hid)
| Layer | Script | launchd job | Cadence | Writes | Status |
|---|---|---|---|---|---|
| **Beacon / SPOF-backstop** (A74-D1) | `scripts/hooks/heartbeat-independent-tick.sh` | `com.evium.heartbeat-independent` | 600s, **DRY_RUN** | `heartbeat/last-independent-tick.json` (+ event-bus cursor-advance & queue-keep in `--apply`) | ✅ **INSTALLED + BEATING** (user installed 2026-06-02; beacon fresh; pulse HEART ✓) |
| **Tiered dispatcher** | `scripts/heartbeat/tick.sh` | `com.evium.heartbeat` (via `scripts/heartbeat/install-launchd.sh`) | tier-high 60s / med 300 / low 900 / hourly 3600 | runs the 4 tier-checks: `check-inbox`, `check-goal-queue`, `check-token-budget`, `check-scheduler-health` | ❌ **NOT loaded** |

## VERDICT (verified, not inferred)
- **The pulse's HEART is genuinely healthy.** The installed `com.evium.heartbeat-independent.plist` runs
  the CORRECT script (`heartbeat-independent-tick.sh` — I verified the ProgramArguments, not assumed),
  is `launchctl`-loaded, and the beacon `last-independent-tick.json` refreshes every ~10 min (dry-run
  still writes JOB 3, by design). `organic-os-pulse.sh` reads that beacon → **HEART ✓**. The A74-D1 SPOF
  fix (a self-contained tick that needs nothing but its own launchd timer) is now live. **"Heart back to
  100%" is achieved.**
- **The tiered `tick.sh` heart (`com.evium.heartbeat`) is uninstalled.** Its 4 tier-checks therefore do
  NOT fire *between* sessions. **This is a CHOICE, not a defect:**
  - The independent-tick is the intended SPOF-proof CORE (it keeps the beacon + event-bus alive even
    when the scheduler/agent-action path is dead — the exact failure A74-D1 was built for).
  - The tier-checks (inbox urgency, goal staleness, token budget, scheduler health) are **largely
    re-surfaced at SessionStart** (the digest + pulse) — so for an interactively-driven system their
    between-session value is "proactive surfacing while you're away," not core liveness.
  - Installing it adds a **60s-cadence background job** — weigh against the bottleneck-chant + the
    autonomic-system caution about background activity/heat.

## RECOMMENDATION
**Leave the tiered heart uninstalled** unless the user specifically wants between-session tier-surfacing
(inbox/goal/token alerts while away). The HEART-✓ goal is met by the independent-tick alone. If the user
DOES want it: `bash scripts/heartbeat/install-launchd.sh` (gated — launchctl) installs `com.evium.heartbeat`.

## Process lesson (banked)
A subagent that fails to emit its StructuredOutput = an **unfilled audit cell**, never "covered by a
neighbor." The orchestrator must re-run it or do it directly — surfacing the gap, not absorbing it.
(Reinforces verify-don't-claim + the never-miss-an-idea closure discipline.)
