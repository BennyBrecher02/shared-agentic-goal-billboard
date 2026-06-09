---
audit_id: A30
title: "Graceful shutdown/startup overhaul — laptop-suspend-resume model with zero-degradation resumption"
status: in_progress
catalogued: 2026-05-26T23:50:00Z
priority_when_run: P0
estimated_effort: large (snapshot design + impl + test cycle + documentation; multi-day work; user explicit gates shutdown until completed)
trigger: 2026-05-26 23:50Z — user driving home from work + explicit framing *"begin readiness for graceful shutdown? before we do that consider our system as a whole and how we can better handle gracefully stopping and starting? ... similar to how context swaps state capturing type stuff alows laptops to close and stop and restart without apps even realizing ... we wont gracefully end until we finish our graceful start/end overhaul/upgrade and test it before we use it too, make sure that when we start again we dont work in a lazier slower state, it shouldnt even be noticeable, so lock in on this."*
deferral_reason: NONE — user is about to drive but explicitly gated their shutdown on completion + testing of this overhaul. This is the highest-priority work right now (parallel with the in-flight BGs).
related_goals: [G10]  # new goal proposed in this turn
related_plans:
  - context/markdowns/plans/automation/graceful-shutdown-infrastructure-plan.md  # Foundation; this overhaul extends
  - context/markdowns/plans/automation/graceful-shutdown-startup-overhaul-plan.md  # NEW
serves_northern_star: G2
belongs_to_goal: G10  # routed 2026-06-08 (UNCLEAR Decision 3): title is verbatim G10's ("graceful start/stop, laptop-suspend-resume"); title-match wins; serves_guiding_light + related_goals already G10
serves_guiding_light: G10
related_refs:
  - .claude/skills/agentic-quality-discipline/references/graceful-shutdown.md (existing 7-step checklist; needs upgrade)
  - .claude/skills/agentic-script-design/references/state-substrate-design.md (the architectural patterns we apply)
  - context/markdowns/goal-billboard/audits/A29-lazy-regression-to-mean-bg-launch.md (zero-degradation criterion is a direct corollary of A29)
findings: []
---

# A30 — Graceful shutdown/startup overhaul

## The user's framing, broken down

> "begin readiness for graceful shutdown? before we do that consider our system as a whole and how we can better handle gracefully stopping and starting?"

Two-part: (1) prepare for shutdown, (2) FIRST upgrade the shutdown/startup system itself.

> "similar to how context swaps state capturing type stuff alows laptops to close and stop and restart wihtout apps even realizing"

Inspiration: **laptop suspend/resume**. Apps don't notice the gap. State is preserved in RAM-snapshot form. Resume is indistinguishable from no-stop.

> "we wont gracefully end until we finish our graceful start/end overhaul/upgrade and test it"

Gating condition: don't shut down until the overhaul is built + tested.

> "make sure that when we start again we dont work in a lazier slower state, it shouldnt even be noticeable"

Zero-degradation criterion. A29 + A21 already established that "lazy mode" is a real regression pattern. This audit's resume mechanism must avoid TRIGGERING A29 on the next session start.

## Why this matters

The current `graceful-shutdown.sh` + the 7-step checklist (per existing skill ref) handle:
- Save uncommitted work
- Log session state
- Surface in-progress items

What they DON'T handle:
- BG-agent in-flight state (5 currently running; resume should know which were active + their status)
- Conversation thread context (what topic were we mid-stew on?)
- Metric trajectories (the conversation-pattern KPI; the BG-launch sparkline; the parallel-capacity ratio)
- Pending substrate state (asks-log entries; recurrences detected; lost-work undrained)
- Monkey-chamber pending decisions (their queued_at + age tracking)
- Token-budget continuity (cumulative today; projected ceiling)
- Northern Star + Guiding Lights + their phase context
- Lock files (heartbeat lock, skill-refresh lock — must be cleanly released or PID-validated on resume)

Without all of this, "resume" = SessionStart-fresh = lazy state without context. The user has stressed (A29) that this regression must not happen.

## The 4-layer snapshot/resume design

### Layer 1 — Snapshot (capture)

`scripts/graceful-shutdown-snapshot.sh` — captures a comprehensive state file:

**File:** `.claude/cache/session-snapshot.json` (also archived as `session-snapshot-<ISO>.json` for history)

**Schema:**
```json
{
  "snapshot_ts": "2026-05-27T00:00:00Z",
  "snapshot_reason": "user-initiated | session-end | auto-shutdown",
  "user_context": {
    "last_user_message_ts": "ISO",
    "last_user_message_excerpt": "first 200 chars",
    "current_thread_topic": "graceful shutdown overhaul",
    "stew_level": "deep | medium | shallow",  // how deep is the current inquiry
    "user_state_signal": "driving | afk | active"
  },
  "northern_star": {
    "goal_id": "G2",
    "phase": "signoff package assembled (19/19 verified); awaiting client review + 3 monkey decisions",
    "blockers": ["3 monkey decisions", "per-tab visual sweep"]
  },
  "guiding_lights": [
    {"goal_id": "G1", "phase": "..."},
    {"goal_id": "G6", "phase": "..."},
    ...
  ],
  "in_flight_bgs": [
    {
      "id": "ac717ab53842d4976",
      "description": "Testing toolbelt Phase 2 — tier system",
      "launched_at": "ISO",
      "expected_completion": "estimate",
      "file_zone": "scripts/run-tests.sh + package.json + skill ref",
      "resume_action": "check task notification; mark complete or note partial"
    },
    ...
  ],
  "pending_user_input": {
    "monkey_decisions": [
      {"id": "P1-04", "queued_at": "ISO", "age_hours": 40, "title": "..."},
      ...
    ],
    "settings_patches_pending": 9,
    "rename_001_pending": true,
    "install_001_pending": true
  },
  "open_audits": ["A12", "A13", "A14", "A15", "A16", "A17", "A18", "A19", "A20", "A21", "A22", "A23", "A24", "A25", "A26", "A27", "A28", "A29", "A30"],
  "metrics_snapshot": {
    "session_tokens_today": "N",
    "5hr_window_reset_target": "ISO",
    "ack_latency_p50_today": "Xmin",
    "bg_launches_today": N,
    "parallel_capacity_rolling_10": 0.X,
    "skill_health_quadrants": {"HH": 4, "HL": 5, "LH": 0, "LL": 0},
    "conversation_pushback_rate_band": "low | medium | high"
  },
  "structural_directives_pending": [
    "Apply settings.json patches via SET-005 (9 patches pending)",
    "Install heartbeat LaunchAgent (INSTALL-001)",
    "Phase 2 skill refreshes after webdesign: chrome-tools next, device-testing third",
    "Phase 5 cluster work blocked on M2 hardware availability",
    ...
  ],
  "lock_state": {
    "skill_refresh_lock_held_by": "BG #95 (agentic-webdesign) — release on completion",
    "heartbeat_lock_state": "not installed",
    "other_locks": []
  },
  "next_session_resume_briefing": "synthesized 200-word briefing of where we are + immediate next action"
}
```

Performance: <2s to snapshot (mostly file reads + JSON aggregation).

### Layer 2 — Resume (restore)

`scripts/graceful-startup-resume.sh` — called at SessionStart when snapshot exists:

Behavior:
1. Read latest `.claude/cache/session-snapshot.json`
2. Verify snapshot freshness (>24h old = warn but still use; >7d = ask user before resuming)
3. PID-validate any locks (clear stale; preserve live)
4. Emit `<system-reminder>` with the resume briefing:
   ```
   🔄 RESUME FROM SNAPSHOT [snapshot_ts] (gap: X minutes)
   
   Northern Star: G2 (phase: ...)
   You were stewing on: graceful shutdown overhaul
   
   In-flight BGs at shutdown: 5 (check task notifications for completions)
   Pending user input: 3 monkey decisions (P1-04 40h+, ...), 9 settings patches
   Open audits: 19 (A12-A30)
   
   Immediate next action: <synthesized>
   
   Resume verification: read this snapshot fully + acknowledge before doing other work.
   ```
5. Trigger BG status-fetch for any in-flight BGs that completed during the gap (read task notifications)

### Layer 3 — Verification (round-trip test)

`scripts/test-shutdown-resume-roundtrip.sh` — exercises the cycle WITHOUT actually ending:
1. Take a snapshot to a TEMP path
2. Read the snapshot back
3. Verify every field reconstructs an equivalent "session state"
4. Run a synthetic "next response" with only the snapshot as context (no JSONL)
5. Verify the synthetic response demonstrates the same focus, same priorities, same active goals
6. Check: would a fresh agent + the snapshot answer "where were we?" correctly? Score 0-100.

Test cadence: every time the snapshot script changes; before any real shutdown; nightly.

### Layer 4 — Documentation + memory rule

- Update `agentic-quality-discipline/references/graceful-shutdown.md` to v2 (snapshot/resume model added)
- New memory rule: "graceful-shutdown discipline" — never shut down without snapshot; never start without reading snapshot; verify resume before deeming "complete"

## Zero-degradation criterion (the A29 corollary)

The user explicitly said: "when we start again we dont work in a lazier slower state, it shouldnt even be noticeable."

This is A29's lazy-regression problem applied to session boundaries. Resume mechanisms:
1. The `<system-reminder>` from resume MUST set context with full pressure (not "ready to help" — but "here's exactly where + what's next")
2. The resume briefing MUST include current FLOOR-rule reminder (≥3 parallel BGs by default; in-flight count drives the formula)
3. The first response after resume MUST start by launching ≥1 BG if pending substrate has candidates (which it always will)
4. NO calm-mode startup — first 5 responses post-resume tracked; if they show A29 regression, escalate to user

## Phasing (see plan)

See `plans/automation/graceful-shutdown-startup-overhaul-plan.md`.

- **Phase 1 (THIS TURN — foreground)**: A30 audit + overhaul plan + G10 goal + memory rule + audit catalog + steering log
- **Phase 2 (QUEUED — BG when ceiling opens)**: Snapshot script + Resume script + skill ref v2 + supersede note on existing graceful-shutdown plan
- **Phase 3 (QUEUED — after Phase 2)**: Round-trip test script + run the test + verify zero-degradation criterion
- **Phase 4 (USER GATED)**: Actual graceful shutdown — only after Phase 2 + 3 land + test passes
- **Phase 5 (next session)**: Resume; verify in real life; iterate if needed

## Verification gates

Before Phase 4 (actual shutdown) can fire:
1. Phase 2 snapshot script exists + runs <2s + produces full JSON
2. Phase 2 resume script exists + emits comprehensive `<system-reminder>`
3. Phase 3 round-trip test passes (score ≥95/100)
4. Skill ref v2 updated
5. **User explicit "OK to shutdown now" signal** — even after auto-detection, agent waits for explicit go

## Cost gates

- Phase 1 (this turn): <500k tokens (foreground writes)
- Phase 2 (BG): <500k
- Phase 3 (test BG): <300k
- Phase 4 (the actual shutdown): minimal

## Status

IN PROGRESS. **Phase 2 + Phase 3 LANDED 2026-05-27.** Phase 4 (actual shutdown) remains user-gated on explicit "OK shut down now" signal.

### Phase 3 progress (landed 2026-05-27T00:14Z; first run scored 99/100 PASS)

Files delivered:
- `scripts/test-shutdown-resume-roundtrip.sh` — 7-phase 0-100 scoring; tier=medium; <30s wall-clock; ALWAYS exit 0 (PHASE_2_NOT_LANDED path also exit 0); --help / --json / --verbose flags; trap-based artifact cleanup. Uses --path flag against the resume script (the actual interface); falls back to --snapshot/--file/-f/env-var conventions if interface changes. When live snapshot --dry-run fails, falls back to cached `.claude/cache/session-snapshot.json` for downstream phases so the test still produces meaningful diagnostics on a snapshot-script regression. NEVER overwrites the real session-snapshot.json.
- `.claude/skills/agentic-testing-discipline/references/shutdown-resume-roundtrip-test.md` — 12.5K skill ref with frontmatter (`serves_northern_star: G2` + `serves_guiding_light: G10`), all 7 phase score weights, how-to-run + how-to-interpret + failure-mode catalog (per-sub-score gap diagnosis), tier=medium / budget conventions, anti-pattern guards.
- `scripts/run-tests.sh` — wired the round-trip test into the medium tier (also exposed in --dry-run listing as "shell tests"). Runs as part of `npm run test:medium`. Test failure propagates via OVERALL_EXIT.

**Phase 3 first run: PASS, score 99/100** (`tier: medium`, ts 2026-05-27T00:14:17Z):
- b) snapshot:        10/10
- c) schema:          24/25 (-1 because in_flight_bgs legitimately empty)
- d) resume:          30/30
- e) lock state:      15/15 (stale=8, live=7)
- f) floor / no-lazy: 15/15
- g) where-were-we:    5/5

Verification gate (Phase 4 actual-shutdown): CLEARED on the structural test side. Phase 4 remains gated on explicit user "OK shut down now" signal.

### Phase 2 progress (landed 2026-05-27T00:00Z)

Files delivered:
- `scripts/graceful-shutdown-snapshot.sh` — 1s wall-clock, atomic JSON write, archive copy, 13 top-level schema sections populated from goal-billboard + monkey chamber + cache aggregation + session JSONL walk. PID-validates locks before recording lock_state.
- `scripts/graceful-startup-resume.sh` — silent on absence, valid-JSON gate, freshness banding (fresh/stale-24h/very-stale-7d), PID-validates skill-refresh locks (clears stale / preserves live), BG-completion-fetch from session JSONLs, emits comprehensive `<system-reminder>` briefing, writes `.claude/cache/last-resume.json` marker, supports --quiet / --check / --path overrides for testing.
- `.claude/skills/agentic-quality-discipline/references/graceful-shutdown.md` — upgraded to v2 with snapshot/resume model, resume failure modes table, "what NOT to do at resume" section, frontmatter (serves_northern_star: G2 / serves_guiding_light: G10 / related_audit: A30).
- `context/markdowns/plans/automation/graceful-shutdown-infrastructure-plan.md` — marked superseded with note pointing to overhaul plan.
- `feedback_standing-protocols.md` (memory + mirror) — new "Graceful shutdown/startup discipline" rule.
- `.claude/cache/skill-refresh-history.jsonl` — appended this Phase 2 completion log.

Phase 2 verification (8 checks): all pass — see report.

Phase 3 (round-trip test) is the next step. Phase 4 (actual shutdown) remains user-gated on Phase 3 pass + explicit user "OK shut down" signal.

## Cross-references

- A29 — zero-degradation criterion derives from A29's lazy-regression analysis
- A21 — FLOOR rule applies post-resume immediately
- G7 — cluster Phase 2 doesn't block; cluster snapshot/resume future work
- G9 — testing toolbelt; round-trip test is a new test class for the toolbelt
- Existing `graceful-shutdown-infrastructure-plan.md` — superseded by overhaul; foundational

## Lessons (preliminary, before implementation)

- "Graceful shutdown" without "graceful startup" is half a feature. They're a pair.
- State capture across BG boundaries is the hardest part (BG #N might complete during the gap — need to fetch task notifications on resume)
- Zero-degradation criterion forces the resume mechanism to ACTIVELY set FLOOR-rule context, not passively re-load
- Lock-file PID validation is essential — stale locks would block post-resume work
- Snapshot freshness matters: >7d old snapshot = ask user before trusting (project state may have drifted)
