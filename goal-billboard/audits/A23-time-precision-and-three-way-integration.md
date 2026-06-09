---
audit_id: A23
title: "Time precision verification + scheduler/heartbeat/time three-way integration"
status: in_progress
catalogued: 2026-05-26T23:05:00Z
priority_when_run: P0
estimated_effort: medium
trigger: |-
  2026-05-26 22:58Z — user audit *"launch another stats audit that sees if we use true system backed unix type verified proved time or just hokey pokey slop time. also in this audit check how we can better use these with our scheduler and heartbeat system and plan how we can more powerfully wield all 3 to upgrade our agentic operating system."* Two gaps: (1) timestamps written across hooks/scripts/JSONLs may not all use the same authoritative time source — drift between them = false correlations + bad analytics; (2) scheduler + heartbeat + time precision are independently designed but not synergistically wielded — they could compound but don't.
deferral_reason: NONE — running immediately. A14 already flagged 5hr-reset-file is 79min stale (uses unverified hand-set time). A21 + A19 + A20 hooks all timestamp events. If those timestamps are slop, the conversation-pattern KPI + recurrence detection + ack-latency measurements are all built on sand.
related_goals: [G4, G7]
related_plans: [context/markdowns/plans/automation/time-precision-and-three-way-integration-plan.md]
serves_northern_star: G2
belongs_to_goal: G1
serves_guiding_light: G7
related_refs:
  - .claude/skills/agentic-quality-discipline/references/timestamp-convention.md
  - .claude/skills/agentic-quality-discipline/references/logging-discipline.md
  - scripts/utils/now-iso.sh
  - scripts/heartbeat/lib-common.sh
  - scripts/scheduler/scheduler.py
findings: []
---

# A23 — Time precision + 3-way integration

## Why this audit matters

Today's KPIs (ack-latency, recurrence detection, conversation-pattern, stats correlation, parallel capacity) all build on timestamps. **If timestamps are inconsistent or unverified, the KPIs lie.** User framing: "system backed unix type verified proved time or just hokey pokey slop time."

Plus: scheduler + heartbeat + time-precision are 3 powerful primitives we already have, but they operate in isolation. Coordinating them is the meta-OS upgrade the user is asking for.

## Two gaps

### Gap 1: time-source drift potential

The project has multiple time-source patterns:
- `scripts/utils/now-iso.sh` — canonical UTC ISO 8601 (per timestamp-convention skill ref)
- `date -u +%Y-%m-%dT%H:%M:%SZ` — inline bash (most hooks)
- `python3 -c "import time; print(time.time())"` — epoch seconds (python utilities)
- `Date.now()` / `new Date().toISOString()` — browser JS in dashboard
- Hand-set `.claude/cache/5hr-window-reset.txt` (user enters this; can be stale — A14 finding)
- File mtimes via `stat`, `find -newermt`
- Git commit timestamps

**Risks not yet audited:**
- Clock drift between systems (single Mac today — minimal; cluster — real)
- TZ confusion (UTC vs local) in user-facing strings
- Monotonic vs wall-clock (heartbeat tick intervals should use monotonic; current uses `date +%s`)
- File mtime precision (HFS+ vs APFS — different resolutions)
- NTP sync status (verified system time? or drifting?)
- Browser JS clock can differ from server (timezone discrepancy in stat displays)

### Gap 2: scheduler + heartbeat + time-precision not synergized

Each operates well in isolation:
- **Scheduler** — fires changes, captures events, has its own tick-id space
- **Heartbeat** — polls inbox/goal/token/scheduler-health on 60s/5min/15min/60min tiers (when installed)
- **Time precision** — `scripts/utils/now-iso.sh`, hooks timestamp consistently

But they don't compound:
- Scheduler tick completion doesn't fire heartbeat re-check off-cycle (so a slow tick eats waiting time before heartbeat notices completion)
- Heartbeat doesn't read scheduler events to know what's "in progress" — only detects stuck-ticks
- Time precision isn't verified before any tick (a clock-drifted machine would silently corrupt the event log)
- 5hr-window-reset isn't computed FROM measured session-start-time; it's hand-entered (A14 flag)
- The state-batch-digest hook reads timestamps but doesn't verify they're well-formed UTC ISO

## Root causes

1. **No central time-source validator** — scripts call `date` independently; no health check confirms "we agree about now."
2. **5hr-window-reset is manual** — can be wrong; A14 caught it stale by 79min.
3. **No NTP-sync verification** — we trust the OS clock without checking.
4. **Scheduler events have `tick_start`/`tick_end` but no integration with heartbeat tier-X** — should fire a heartbeat re-check on tick_end.
5. **Heartbeat tick.sh logic uses `date +%s`** — wall-clock, not monotonic. Reasonable for interval tracking but drift-vulnerable if clock leaps.
6. **No `time-precision-check.sh` hook** — could verify all script-emitted timestamps agree with UTC + reasonable bound (e.g. ±2s from `ntpdate -q` if NTP available).

## Fix (designed + scaffolded same turn — implementation BG launching)

See `plans/automation/time-precision-and-three-way-integration-plan.md`. Three layers:

### Layer 1 — Time precision audit + verification CLI
- `scripts/run-time-precision-audit.py` — scans all timestamp emit points (hooks + scripts + JSONLs); reports format inconsistencies, TZ issues, drift between same-event timestamps
- `scripts/utils/verify-clock.sh` — checks system clock against NTP if available; warns if drifted > 2s
- Output: `context/markdowns/research/time-precision-audit-YYYYMMDD.md` (research-folder; per A15 convention)

### Layer 2 — 3-way coordinator hook
- `scripts/hooks/scheduler-heartbeat-coordinator.sh` — PostToolUse Bash on scheduler ticks: when scheduler emits `tick_end`, fires `bash scripts/heartbeat/tick.sh --force-tier medium` to refresh state immediately
- Reverse direction: heartbeat tier-medium scheduler-health check now reads `tick_end` timestamps + correlates with scheduler-events JSONL
- `state-batch-digest.sh` extended to validate timestamps in its summary line (drop or warn on malformed)

### Layer 3 — 5hr-window auto-detection (where possible)
- Anthropic doesn't expose reset API — manual stays primary
- BUT we can detect reset-occurrence: if cumulative token spend reset between adjacent sessions, that's the reset moment
- `scripts/detect-5hr-reset.py` — scans session JSONLs; reports plausible reset times; allows user to confirm
- Heartbeat tier-low gains "5hr-reset stale >5min?" check; surfaces if true

## Verification

After implementation:
1. `python3 scripts/run-time-precision-audit.py` reports zero TZ/format inconsistencies
2. `bash scripts/utils/verify-clock.sh` returns "ok" or "drift_warning"
3. Synthetic scheduler tick → confirm heartbeat fires off-cycle within 1s
4. `scripts/detect-5hr-reset.py` plausibly identifies last 1-2 reset moments
5. state-batch-digest line includes "time-source: verified" indicator

## Status

IN PROGRESS. Implementation BG launching this turn. Cluster Phase 1 BG (#80) is the right substrate to integrate this with — once cluster nodes can run, time-precision verification must span nodes.

## Cross-references

- A14 — stats correlation (relies on accurate timestamps; this audit validates A14's data quality)
- A17 — 5hr-reset operator alerting (Phase 3 of this plan extends A17)
- A21 — parallel-capacity logging (relies on accurate Stop-time vs start-time math)
- A19 Layer 1 state-batch — should integrate timestamp validation
- `timestamp-convention.md` skill ref — canonical; this audit verifies adherence
- Cluster (G7) — multi-node integration MUST verify clocks agree

