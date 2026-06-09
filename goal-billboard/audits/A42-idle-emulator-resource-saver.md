---
audit_id: A42
title: Idle emulator resource-saver — situational-awareness shutdown + dynamic threshold + intense back-up logic
status: in_progress
catalogued: 2026-05-27T03:53:00Z
priority_when_run: P0
estimated_effort: medium
trigger: |-
  2026-05-27T03:53Z — user explicit: *"can we add some resource saving hook to close any emulator not used in a crazy amount of time when theres no way it could randomly be used by our system after some given time that we'd experiment on trying to find a sweet spot but only based on situational awareness and we should have some intense logic to not only back this up but also dynamically readjust the sweet spot time and this should be factored into our overall stats system which should get a major upgrade again."*
deferral_reason: NONE — concrete resource-saving build; pairs with A43 stats upgrade
related_goals: [G14, G9]
related_plans: [context/markdowns/plans/automation/idle-emulator-resource-saver-plan.md]
serves_northern_star: G2
belongs_to_goal: G9
serves_guiding_light: G14
related_refs:
  - .claude/skills/agentic-device-testing/ (iOS Sim + Android Emu skill family)
  - scripts/ios-simulator-capture.sh + scripts/android-emulator-capture.sh (emulator owners)
  - scripts/scheduler/ (scheduler awareness)
  - A14 — stats audit (predecessor; A42 feeds new resource metrics into A43 upgrade)
findings: []
---

# A42 — Idle emulator resource-saver

## The user's exact ask (the 4 requirements)

1. **Close emulators not used in a crazy amount of time** — kill iOS Sim / Android Emu processes when idle
2. **Situational awareness ONLY** — *"no way it could randomly be used"* — not a blind timeout
3. **Intense logic to back this up** — robust safety; never kill something actually needed
4. **Dynamically readjust the sweet spot** — adaptive threshold based on observed reactivation patterns

Plus: feed metrics into A43 stats upgrade.

## Why idle emulators matter

- iOS Simulator: 300-800MB RAM each; runs background processes; sustained CPU even when idle (animation loops, location services)
- Android Emulator: 1-3GB RAM each; HEAVY (qemu); spinning GPU even when "paused"
- 9-project Playwright matrix can spin up 5+ emulators per run
- Idle emulators left running consume resources for HOURS even if next test run won't touch them
- Cold-start cost on respawn: iOS Sim ~5-15s; Android Emu ~30-60s

## Situational-awareness signal layers (the "intense logic")

The hook reads MULTIPLE substrates before any kill decision. Kill ONLY if ALL of these are quiet:

### Signal layer 1: Scheduler queue
- Read `reports/scheduler/queue.json` (or whatever scheduler-state path)
- If ANY scheduled change has `target_devices` matching this emulator's project → DO NOT KILL
- Even if scheduler is paused — pending intent matters

### Signal layer 2: Recent activity
- Read `reports/dispatch-log.jsonl` (newly added by A36 Phase 2)
- Recent matrix-run captures in `reports/vq-captures/*/` 
- Last touch on `tests/**` files (recent test edits → likely re-run incoming)
- If any signal within last X minutes → DO NOT KILL (X = current dynamic threshold)

### Signal layer 3: Foreground session state
- Read heartbeat state — is current Claude session actively running?
- If user just sent a prompt < 30s ago → grace period, DO NOT KILL
- If `npm run test:*` is running in any worktree (check process list) → DO NOT KILL

### Signal layer 4: Time-of-day patterns
- Read `reports/emulator-usage-history.jsonl` (NEW — emit on every emulator spin-up/down)
- Compute: probability of usage at THIS hour-of-day-of-week
- If probability > threshold → DO NOT KILL (e.g., "every weekday 9-11am we run iOS tests")
- This is the LEARNED pattern — initially defaults to "always assume usage"; tightens as history accumulates

### Signal layer 5: Recent spin-up grace
- If emulator was started < 5 minutes ago → DO NOT KILL (cold-start protection)
- Prevents thrashing: kill → rebirth → kill → rebirth

### Signal layer 6: Active connection
- Check for active SSH/USB/network connections to the emulator
- ADB devices list (Android)
- xcrun simctl active sessions (iOS)
- If anything attached → DO NOT KILL

### Signal layer 7: Manual hold
- Read `.claude/cache/emulator-holds.txt` — if any emulator slug is in there → DO NOT KILL
- CLI: `emulator-hold add ios-iphone-15` / `emulator-hold remove ios-iphone-15`
- Belt-and-suspenders: user can explicitly pin

**ALL 7 signal layers must say "quiet" for the kill to proceed.**

## Dynamic threshold ("sweet spot")

### Initial defaults (experiment seeds)
- iOS Simulator: 45 minutes idle threshold
- Android Emulator: 30 minutes (heavier; bigger savings; harder cold start so worth less time idle)

### Adaptation algorithm
Track per-emulator:
- `last_kill_time`
- `time_to_next_use_after_kill` (how long until next-needed-rebirth)
- `cold_start_cost_seconds` (rebirth wall-clock)
- `idle_resource_cost_per_min` (RAM × time)

After each kill→reborn cycle:
```
if time_to_next_use_after_kill < 10 minutes:
    threshold += 5 min   # too aggressive; back off
elif time_to_next_use_after_kill > 2 hours:
    threshold -= 5 min   # too conservative; tighten
```

Bounds: `[15 min, 4 hours]`. Per-emulator-class threshold.

### The empirical sweet spot
After ~10 kill/rebirth cycles per emulator, threshold should stabilize. The goal:
`minimize(cold_start_cost_incurred + idle_resource_waste)` over a rolling 7-day window.

Output to `reports/emulator-threshold-tuning.jsonl` — append-only audit trail of every adjustment.

## The hook architecture

### `scripts/hooks/emulator-idle-killer.sh` (scheduled / cron-like)

Runs every 5 minutes via:
- Option A: cron entry (user-installed; opt-in)
- Option B: SessionStart hook (cheap; runs once per session start; user-active sessions = less aggressive killing)
- Option C: PostToolUse on any Stop event (frequent enough to catch idle states)

**Recommendation: Option B + Option C** — runs on session boundaries (cheap, gives most kills); won't run mid-active-work.

Flow:
1. Enumerate running emulators (xcrun simctl list / adb devices)
2. For each, check the 7 signal layers
3. Compute idle time (last-activity-timestamp from per-emulator activity log)
4. If idle_time > current_threshold AND all 7 layers quiet → kill
5. Log to `reports/emulator-kills.jsonl`: `(timestamp, emulator, idle_time, threshold, signal_layers_state, kill_rationale)`
6. Update threshold per adaptation algorithm

### `scripts/hooks/emulator-activity-tracker.sh` (PostToolUse on capture / test runs)

Hooks into test-runs and capture scripts. Every time an emulator is USED, append to `reports/emulator-activity.jsonl`:
- `(timestamp, emulator, activity_type, duration_ms)`

This is the "last-activity-timestamp" source.

## Safety: "intense back-up logic"

Per user explicit *"intense logic to not only back this up"*:

1. **Dry-run mode by default for first 7 days** — kill decisions logged but NOT executed; user reviews the "would have killed" log
2. **Re-spawn protection** — if killed emulator is re-spawned within 5 min, log as MISCALL; adjust threshold UP
3. **Threshold floor + ceiling** — never below 15 min (too aggressive); never above 4 hours (defeats purpose)
4. **Per-emulator-class isolation** — iOS bug ≠ Android bug; threshold drift is per-class
5. **Graceful kill** — `xcrun simctl shutdown <UDID>` (not `kill -9`); `adb emu kill` (not `pkill`)
6. **Manual override CLI** — `emulator-hold <slug>` immediately pins; `emulator-killer pause` halts all killing
7. **Watchdog** — if 3+ MISCALL events in a 24h window, hook auto-disables itself + surfaces alert

## Integration with stats (the bridge to A43)

A42 emits these metrics (consumed by A43 stats upgrade):

| Metric | Source | A43 surface |
|---|---|---|
| `emulator_uptime_minutes_total` (per emulator class) | activity-tracker + kill-log | per-session bar |
| `emulator_kills_count` (per session) | emulator-kills.jsonl | sparkline + total |
| `emulator_miscall_count` (rebirths within 5 min) | re-spawn protection | inverse-quality KPI |
| `emulator_cold_start_seconds` (rolling avg) | activity-tracker spin-up time | latency dimension |
| `current_threshold_minutes` (per emulator class) | threshold-tuning.jsonl | current-state display |
| `idle_resource_waste_mb_min` (RAM × idle time saved) | calculated | savings KPI |
| `cold_start_cost_incurred_minutes` (rebirth wall-clock) | tracked | cost KPI |

A43 ingests these + cross-correlates with token-usage, session-length, BG count, etc.

## Phases

### Phase 1 — Foreground design + dry-run mode (this turn + BG)
1. Foreground: emulator-activity-tracker + emulator-idle-killer scripts (skeleton)
2. BG: full implementation of the 7-signal-layer check + adaptation algorithm + dry-run mode + initial threshold defaults
3. BG: settings.json delta queued at `scripts/hooks/settings-patches/A42-emulator-killer-wire.json`

### Phase 2 — 7-day dry-run + tuning
- Real activity tracked; kill decisions LOGGED ONLY
- User reviews `reports/emulator-kills.jsonl` (would-have-killed log)
- After 7 days, switch from dry-run to live

### Phase 3 — Live mode + adaptation loop
- Real kills happen
- Threshold adapts based on miscall rate
- A43 dashboards show savings + costs

### Phase 4 — Cross-cluster (when G7 Phase 2 hardware lands)
- M2 secondary machine has its own emulators
- Per-machine kill thresholds
- Cluster-wide resource view

## Cost gates

Phase 1 BG: <300k tokens. Phase 2-4 runtime overhead negligible (hook is <100ms per invocation).

## Cross-references

- A14 — stats audit predecessor
- A43 — stats system upgrade #2 (this turn sibling; consumes A42's metrics)
- G9 — testing toolbelt (emulators are testing infrastructure)
- G7 — cluster (Phase 4 cross-machine extension)
- A37 — calculation audit (verify A42's adaptation math)
- A36 — true-time (timestamps in activity-tracker must be epoch-sourced)

## Status

IN PROGRESS — design landed this turn; Phase 1 BG queues for capacity.

## Lessons (preliminary)

- The simplest version of this hook ("kill after 30 min idle") would be DANGEROUS — would thrash with scheduler unpredictability. The intense back-up logic is non-negotiable.
- Per-emulator-class threshold adaptation matters: Android is heavier than iOS (more savings on kill, more painful on cold start). One-size-fits-all is wrong.
- The 7-day dry-run discipline matches the A30 graceful-shutdown round-trip test pattern — never deploy automation without observing it in shadow first.

