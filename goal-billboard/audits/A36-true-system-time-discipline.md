---
audit_id: A36
title: True-system-time discipline — eliminate hallucinated time calculations everywhere
status: in_progress
catalogued: 2026-05-27T03:02:02Z
phase_2_completed_at: 2026-05-27T03:24:06Z
priority_when_run: P0
estimated_effort: large
trigger: 2026-05-27 — user screenshot showed BGs #80 (Cluster Phase 1) + #82 (G6 Phase 2) still "Running" at 231m / 229m wall-clock despite being marked completed. Artifacts landed but BG dispatcher never closed agents. Confirms broader untrustworthiness of agent-reported timing claims (13hr session, "200 min" BGs, etc.). User explicit — *"can we start course correcting our time estimates by getting the true real time a graceful startup begins and also when they end... ensure the rest of this run uses true time only, launch a true time only audit to make sure every part that cares about and uses time uses true unix type time mac provided system canon time and not halucinated calculations."*
deferral_reason: NONE — running immediately; this is foundational data integrity
related_goals: [G14, G11, G9]
related_plans: [context/markdowns/plans/automation/true-time-discipline-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - .claude/skills/agentic-quality-discipline/references/timestamp-convention.md (existing canonical convention — to verify compliance)
  - scripts/utils/now-iso.sh (existing canonical source — to verify usage)
findings: []
---

# A36 — True-system-time discipline

## The trigger event

User screenshot 2026-05-27 03:00Z: two BGs from earlier in session shown as still-Running at 231m 56s and 229m 14s — both already marked completed per task #80 + #82. Their artifacts DID land (cluster Phase 1 scripts exist + G6 Phase 2 wired) but the dispatcher never closed the agents.

This is a measurement-data-integrity failure. If I can hallucinate two BG completions while their wall-clock keeps ticking, then **every** time-attributed claim I've made this session needs verification. Including:
- "13-hour session" length
- "200 min" BG durations (per the #109 BG taxation writeup)
- Per-hook elapsed times in reports
- Scheduler tick timings
- Heartbeat ages
- 5hr-rolling-window calculations
- Memory-rule dates ("2026-05-25 PM", "2026-05-26 AM")

The user gate: *"we only fixed time midway through so we cant defend this runs time anyways, but we can ensure the rest of this run uses true time only."*

## The core principle

**Every script/hook/agent action that emits a timestamp or computes a duration MUST source time from the macOS Unix system clock, not from agent arithmetic on text.**

In practice:
- `date -u +%Y-%m-%dT%H:%M:%SZ` → canonical UTC ISO
- `date +%s` → epoch seconds for arithmetic
- `python3 -c "import time; print(time.time_ns())"` → nanosecond precision when needed
- `scripts/utils/now-iso.sh` → single source of truth (must be called, never bypassed)

**Forbidden patterns:**
- Hardcoded dates in agent-emitted text (e.g., the agent writing "2026-05-27 AM" in memory without sourcing system date)
- Duration arithmetic on natural-language text ("the session was about 13 hours")
- Inferring elapsed time from message counts or token counts
- Trusting BG-dispatcher displayed times when they may be stale (artifacts trump UI)

## Scope of audit

### Track 1: Script + hook compliance (foreground-doable)
1. Every script in `scripts/` that emits a timestamp — does it shell out to `date` / `now-iso.sh`?
2. Every Python file in `scripts/` that uses time — does it use `time.time()` / `datetime.now(timezone.utc)`?
3. Every hook in `.claude/settings.json` — when it computes "elapsed since X", does it source X from a file written with true time?
4. The scheduler — does it use real epoch for tick boundaries?
5. The heartbeat — does it use real epoch for "age" calculations?
6. The 5hr window — does it use real epoch for reset detection?

### Track 2: Agent claim verification (cross-cutting)
1. Memory rules with dates — sample N entries; verify dates align with file `stat` mtime
2. Steering log dates — same check
3. Audit `catalogued:` fields — same check
4. Goal `created:` / `updated:` — same check
5. Bundle manifests — verify start/end stamps against real epoch boundaries

### Track 3: BG dispatch instrumentation
1. The BG dispatcher — does it record launch-epoch + completion-epoch atomically?
2. The displayed "Running 231m" — what's the source? Wall-clock from agent launch? Or something else?
3. Why did BGs #80 + #82 never get closed by dispatcher? Stuck read? Tool-call timeout?
4. Add: on every BG completion notification, write a `(bg-id, launch-epoch, completion-epoch, duration-seconds)` row to a canonical log

### Track 4: Hook timing-budget reevaluation (post-shutdown research, deferred build)
After every graceful shutdown, re-evaluate each hook's actual measured elapsed time vs its budget. Surface recommendations: tighten / loosen / remove. **Build deferred until after NEXT graceful shutdown** because this run's bundle is timing-unreliable (we fixed mid-stream). See companion plan `hook-timing-reevaluation-plan.md`.

## Concrete deliverables (Phase 1 — BG this turn)

The BG will:
1. Walk `scripts/` + `.claude/settings.json` + `.claude/hooks/`
2. Identify every time-emission + time-computation site
3. Classify each: ✅ uses true system time / ❌ hallucinated / ⚠ ambiguous
4. For each ❌ + ⚠: propose concrete patch (which call to insert; what file to edit)
5. Cross-check sample memory + steering log dates against file mtimes (using `stat`)
6. Produce `reports/audit-findings/A36-time-discipline-pass-1.md` with categorized findings + patch queue
7. Land in <30 min wall-clock; <500k tokens

## Phase 2 — Apply patches (later)

Apply the patch queue. Test that no hook regresses. Re-measure baseline.

## Phase 3 — Build the post-shutdown hook-timing-reevaluation system

After next graceful shutdown lands clean bundle data, build the recurring analysis described in `hook-timing-reevaluation-plan.md`.

## Cross-references

- A14 — stats audit (cousin: A14 covered token + 5hr-window but assumed time was correct)
- A23 — time precision + 3-way integration (foundation; A36 extends with audit + verification)
- G11 / A34 — bundle infrastructure (consumer of true time)
- G9 — testing toolbelt (anti-bottleneck depends on accurate timing)
- A37 — calculation audit (parallel sibling; this turn)

## Lessons (preliminary)

- **The BG-dispatcher-never-closes-agent bug is the canary.** If the dispatcher had been emitting `(launch_epoch, completion_epoch)` rows, the discrepancy would have been caught the moment artifacts landed without a completion-epoch.
- **"Verify the time" must be a first-class discipline**, not a hidden assumption.
- **Mac-provided Unix time is canon.** Anything else is hallucination until proven from a `date` call.

## Status

Phase 2 PATCHES APPLIED — 3 patches; 244 hook tests pass (9 new); 1 settings.json delta queued at `scripts/hooks/settings-patches/A36-bg-stuck-warn-wire.json`.

- Phase 1 (sweep): complete — see `reports/audit-findings/A36-time-discipline-pass-1.md`
- Phase 2 (P0 patches): **complete 2026-05-27T03:24:06Z** — see `reports/audit-findings/A36-time-discipline-phase-2-apply.md`
  - P0-1 BG-dispatcher instrumentation (`scripts/hooks/dispatch-metrics-log.sh` patched)
  - P0-2 bg-stuck-warn Stop hook (`scripts/hooks/bg-stuck-warn.sh` new; settings wiring queued)
  - P0-3 5hr-window auto-refresh (`scripts/hooks/token-budget-check.sh` patched)
- Phase 2 P1-* patches: deferred to follow-on batch (timestamp UTC fixes in 3 sites, goal-updated-bump hook, claim-vs-mtime audit script, skill-ref note, state-batch-digest extension)
- Phase 3 (hook-timing-reevaluation system): still deferred — requires next graceful shutdown's clean bundle
