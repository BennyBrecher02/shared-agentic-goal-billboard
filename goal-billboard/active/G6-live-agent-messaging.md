---
goal_id: G6
title: Instant ack — drive chat-acknowledgement latency toward zero (live agent messaging)
status: active
track: tooling
phase: Phase 3 in flight — domino-trigger upgrade live (8-trigger fan-out per click). Server POST → `monkey-click-fan-out.sh` async-spawn; 5 fixed-baseline triggers (asks-log, steering-log, heartbeat, NS attribution, dashboard) + 3 conditional (scheduler-queue, bug-billboard, feedback-loop). data.json schema bumped to v3. Phase 2 (priority routing) complete; Phase 3+ (domino) per A31 Front 1. **A71 inbox-never-lost mutation deployed + VERIFY 5/5 PASS** (2026-05-28T10:18Z) — first 4/4 full-tether adaptive-immunity acid test; closes session-local-ack + cross-session-artifact bug class that was causing inbox messages to vanish across concurrent sessions.
priority: P1 (elevated from P2 — A20 trust-model)
created: 2026-05-26T14:54:26Z
updated: 2026-05-28T10:25:00Z
linked_plans:
  - context/markdowns/plans/automation/live-agent-message-queue-plan.md
  - context/markdowns/plans/automation/holdoff-routing-and-monkey-expansion-plan.md (A20)
  - context/markdowns/plans/automation/monkey-domino-trigger-upgrade-plan.md (A31 Front 1)
linked_audits:
  - A20 — Hedge holdoffs + unrouted blockers
  - A31 — Pre-restart prep master (Front 1 — domino-trigger upgrade)
linked_refs:
  - .claude/skills/agentic-quality-discipline/SKILL.md
  - .claude/skills/agentic-script-design/SKILL.md
linked_bugs: []
linked_changes: []
serves_northern_star: G2  # migrated 2026-05-26 - goal_id=G6 (not NS) → GL; NS defaults to G2
serves_guiding_light: G6
project: AgenticOS  # stamped by migrate-billboard-to-shared (shared store)
---
# G6 — Instant ack (drive chat-acknowledgement latency toward zero)

## Why it exists

**Verbatim user framing (2026-05-31):** *"g6 is us improving the ack response time in chat, i want a message sent to eventually be instantly ack'd by you but until thats possible we will try to push the ack speed."*

The GOAL is latency: when the user sends a message, the agent should acknowledge it as close to **instantly** as the harness allows. Today there is a lag between "user hits send" and "agent registers the message" — across the dashboard inbox channel, across hook drains, across turn boundaries. G6 exists to shrink that lag.

- **Aspiration (the target state):** a user message is *instantly* acknowledged by the agent — zero perceptible delay between send and ack.
- **Interim (what we do until instant is possible):** push the ack speed down, measurably, run over run. The `ack-trend` metric (currently ↓45%) is the yardstick: it tracks how far we've driven median ack latency down, and the goal is to keep driving it toward zero.

This reframes G6 from "build a messaging feature" to "make acknowledgement fast." The dashboard→agent-inbox messaging channel + the 8-trigger domino fan-out are the **delivery MECHANISM** that carries a message from the dashboard to the agent — they are how a message *arrives*, not the goal itself. The goal is how *fast* it gets acknowledged once sent. The mechanism is mature (Phases 1–3 shipped, inbox-never-lost hardened); the remaining work is squeezing latency out of that path until ack is effectively instant.

## Current state

**Delivery mechanism: SHIPPED. Latency reduction: IN FLIGHT — this is now the goal.**

The messaging-infra mechanism that *carries* a message from the dashboard to the agent is built and hardened (full history below):

- Phase 1–2 (file-based capture + dashboard wiring) landed: agent-inbox sidecar, dashboard form, drain hook, UserPromptSubmit re-surface, priority routing.
- Phase 3+ (8-trigger domino fan-out) landed per A31 Front 1 — a click fans out to 8 guarded triggers in <1.2s.
- A71 inbox-never-lost mutation deployed + VERIFY 5/5 PASS — closed the session-local-ack + cross-session-artifact bug class that was dropping messages.

With the pipe built, the live work is **latency**: the `ack-trend` metric tracks median time from "user sends" to "agent acknowledges." Currently ↓45% off the original baseline. The job is to keep driving it down toward instant.

## Blockers

- None hard. The mechanism is in place; remaining work is iterative latency reduction. Some of the gap is harness-bounded (the agent only checks the inbox at turn/tool boundaries) — "instant" is the aspiration, and how close we can get is partly bounded by what the runtime allows. Interim wins come from tightening the drain cadence and the surfacing path.

## Next action

Keep pushing `ack-trend` down:

1. Measure the current median ack latency (send → agent-acknowledges) so each change has a before/after number — never claim a speedup without the metric moving.
2. Identify the next-biggest contributor to the lag (drain cadence, hook ordering, turn-boundary wait, re-surface path) and shave it.
3. Re-measure; record the new `ack-trend` figure; log the delta in History.
4. Repeat. Each iteration ratchets the trend closer to zero; the aspiration is instant ack.

## Exit criteria

G6's terminal state is **ack latency driven as close to instant as the harness allows, and held there.** It is a latency goal, not a feature goal — the messaging mechanism is already shipped.

- [ ] **Baseline + metric live:** median ack latency (send → acknowledge) is measured and the `ack-trend` figure is tracked run-over-run on a real surface (not a one-off).
- [ ] **Sustained downward trend:** `ack-trend` keeps dropping across iterations (currently ↓45%) with each drop tied to a concrete latency-shaving change — no regressions.
- [ ] **Near-instant interim bar:** ack latency reaches the practical floor the harness allows (acknowledged within the same turn/tool-boundary the message arrives on, with no perceptible queue lag).
- [ ] **Aspiration (stretch / runtime-gated):** a user message is *instantly* acknowledged — zero perceptible send→ack delay. Reaching this may require runtime affordances beyond the current turn model; until then the interim bar above is "done enough" to consider G6 satisfied, with instant ack as the standing north star.

## History

- 2026-05-26 PM — Goal created. Plan-only. Awaiting GO signal for phase 1.
- 2026-05-26 PM — Phase 1 + Phase 2 inbox sidecar landed (`scripts/agent-inbox-server.py`, dashboard form, drain hook, UserPromptSubmit re-surface, heartbeat staleness alarm). A20 expanded monkey chamber to 4 categories.
- 2026-05-26 PM — **Phase 2 priority routing landed.** Server accepts `priority: urgent|normal|low`; new endpoints `/agent-inbox/recent?priority=urgent` (last-hour fast path) and `/agent-inbox/pending-urgent-count`. `inbox-on-prompt.sh` surfaces urgents FIRST in dedicated `<system-reminder>` blocks (🚨 prefix, cap=5, overflow falls to normal batch). `check-inbox.sh` tier-high queues `severity: urgent` notifications keyed `urgent-inbox-<filename>` (osascript pops when `EVIUM_HEARTBEAT_OSANOTIFY=1`). Dashboard inbox-console pins urgent rows + adds lefter `lefter-inbox` mini-block. Monkey chamber sorts decisions urgent → normal → low within category; long-pending (>24h queued) gets visual flag. `render-asks-data.py` emits `priority` field per `waiting_on_you` item.
- 2026-05-31 — **Goal REFRAMED around ack latency (user clarification).** User clarified G6's true purpose verbatim: *"g6 is us improving the ack response time in chat, i want a message sent to eventually be instantly ack'd by you but until thats possible we will try to push the ack speed."* G6 is a LATENCY goal: drive chat-acknowledgement latency toward instant (`ack-trend` ↓45% is the measure). The dashboard→agent-inbox messaging + 8-trigger domino fan-out are the delivery MECHANISM (already shipped/hardened), not the goal. Title, "Why it exists," "Current state," "Next action," and "Exit criteria" rewritten accordingly. Messaging-infra history below preserved intact.
- 2026-05-27 AM — **Phase 3+ domino-trigger upgrade landed (A31 Front 1).** A monkey-chamber click no longer just lands an inbox file; the POST handler async-spawns `scripts/monkey-click-fan-out.sh <decision_id> <option_id>` which runs 8 guarded triggers in <1.2s: (1) asks-log row, (2) steering-log row before closing `---`, (3) heartbeat-pending.jsonl `monkey-decision-applied` event, (4) ns-attribution.jsonl, (5) BG-dispatch billboard-data-refresh.sh, (6) scheduler-queue write under multi-change-queue/scheduled/ ONLY IF `option.proposed_change`, (7) bug-billboard inbox closure/creation ONLY IF `option.bug_resolution`, (8) feedback-loop-queue.jsonl ONLY IF `option.affects_shipped`. data.json bumped to schema_version=3 with 4 new optional per-option fields. 7 of 10 pending decisions enriched (P1-04 b/c, P1-07a a, P1-07b a/b/c, SET-005 yes-lift, RENAME-001 yes). Backward-compatible: missing fan-out script → WARN + normal flow; older clients ignore the new fields. Skill ref `monkey-domino-discipline.md` (~9K) documents the 8-trigger pipeline. 5 verification cycles pass: P1-04.a 5/8 baseline, SET-005.yes-lift baseline + trigger 7 (closes SET-001..004), INSTALL-001 5/8, RENAME-001.yes 5/8 + trigger 6, SYNTH-YESNO 5/8 graceful. LOG_ONLY=1 + DRY=1 env vars for cost-free verification. Server-side wiring; no settings.json hook entry needed (documented in `scripts/hooks/settings-patches/monkey-fan-out.json`).

## Notes

- **The goal is latency, not the feature.** The messaging channel (dashboard textbox → inbox → drain → surface) and the domino fan-out are the *mechanism* that delivers a message; G6 is about how *fast* that delivery is acknowledged. Don't let mechanism work (new triggers, new endpoints) be mistaken for goal progress — goal progress is the `ack-trend` number dropping.
- The design intentionally separates the delivery mechanism (no UI → UI wiring → domino) from the latency objective, so we can keep tightening ack speed without re-litigating the transport.
- Phase 3 features are explicitly **trigger-gated**: don't build unless user asks after using phase 2 for ≥1 week. Avoids over-investing in a channel that might not earn its keep.
- The `<system-reminder>` injection pattern matches existing project conventions (auto-memory, currentDate). Agent treats incoming messages as advisory nudges, not authoritative instructions.
- Security model is single-user, local-only, `127.0.0.1` bound. No auth in phase 1+2. Multi-developer concerns deferred to phase 3 (or never).
