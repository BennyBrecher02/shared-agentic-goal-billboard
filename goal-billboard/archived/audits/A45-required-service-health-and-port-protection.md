---
audit_id: A45
title: Required-service health-check + port-conflict detection + auto-protection — sidecars never silently down again
status: completed
catalogued: 2026-05-27T04:35:19Z
closed: 2026-05-28T21:17Z
closed_by: A74-D1+D5 → 4/4-tether closure (the A71 pattern applied to the server-death class)
priority_when_run: P0
estimated_effort: medium
trigger: 2026-05-27T04:35Z — companion of A44 (Critical Finding Reflex). The SPECIFIC instance that triggered A44's general pattern. User explicit: *"flesh that fully also"* (referring to server-protection specifically, alongside the general thing).
deferral_reason: NONE — concrete protection; structurally tied to A44 first firing
related_goals: [G14, G9]
related_plans: [context/markdowns/plans/automation/required-service-health-protection-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - A44 — meta-pattern parent
  - A71 — sibling 4/4-tether closure (inbox-loss class); supplied the VERIFY template
  - A62 — adaptive-immunity discipline (the 4/4-tether acid test)
  - scripts/agent-inbox-server.py (the canonical example; canonical port 4323 post-migration, default env 4322)
  - scripts/hooks/inbox-server-guard.sh (the HARDWARE+SOFTWARE tether — NEW 2026-05-28)
  - tests/inbox-server-resilience-test.sh (the VERIFY tether — NEW 2026-05-28)
  - .claude/settings.json (where Stop/SessionStart hooks live)
findings:
  - "2026-05-28: pre-existing A45 work (services-check.sh + wake-restart.sh + sessionstart-services-digest.sh) was READ-ONLY (digest) or USER-RUN (wake-restart). No agent-invokable auto-recover guard existed → that missing HARDWARE barrier is exactly why A74-D1/D5 scored the class 0/4 tethers despite the partial build."
  - "2026-05-28: confirmed the squatter is STILL present on the real system — port 4322 held by node/Astro (PID 63627). Post-A45 migration moved the canonical inbox-server to 4323, so the guard checks 4323 (registry) + the 'count' marker and correctly ignores the 4322 squatter. The marker-not-port lesson holds in production."
---

# A45 — Required-service health-check + port-conflict protection

## The triggering incident

User mid-monkey-chamber-run pressed YES on SET-003 → got `POST failed: http 404. Is agent-inbox-server.py running on port 4322?`. Investigation revealed:
1. `agent-inbox-server.py` was NOT running
2. Port 4322 was squatted by a SECOND Astro dev server (PID 63944)
3. Dashboard POSTs received 404 (wrong server responding) → 404 ≠ network error → offline-queue fallback never triggered for the BUTTON path
4. User's single YES click was LOST (text "Other" entries were safe in localStorage)

## The 4 things that should have prevented this

1. **Pre-startup health-check** — before any user action that depends on a sidecar, verify it's running
2. **Port-conflict detector** — verify "the thing on port X is the SERVICE we expect" not "any random process"
3. **Auto-recovery** — if sidecar missing, try to start it (with safety bounds)
4. **Visible status** — dashboard surface shows green/yellow/red per service so user knows BEFORE clicking

## The required-services registry

`config/required-services.json` — single source of truth:

```json
{
  "agent-inbox-server": {
    "default_port": 4322,
    "fallback_ports": [4323, 4324, 4325],
    "script": "scripts/agent-inbox-server.py",
    "env": {"AGENT_INBOX_PORT": "<port>"},
    "health_endpoint": "/agent-inbox/recent",
    "health_method": "GET",
    "expected_status": 200,
    "identification_marker": "agent-inbox-server",
    "auto_restart": true,
    "restart_max_attempts": 3,
    "criticality": "P0",
    "consumed_by": ["monkey-chamber clicks", "monkey-chamber Other text", "billboard inbox bar"]
  },
  "astro-dev-1": {
    "default_port": 4321,
    "fallback_ports": [],
    "script": "npm run dev",
    "health_endpoint": "/",
    "expected_status": 200,
    "identification_marker": "astro",
    "auto_restart": false,
    "criticality": "P1",
    "consumed_by": ["dashboard at localhost:4321"]
  },
  "heartbeat": {
    "default_port": null,
    "process_marker": "heartbeat-tick",
    "auto_restart": false,
    "criticality": "P2",
    "consumed_by": ["session-bundle ticks", "scheduler awareness"],
    "install_instructions": "INSTALL-001 (LaunchAgent opt-in)"
  }
}
```

## The 5 mechanisms

### M1 — `scripts/required-service-check.sh` (CLI + library)

Reads registry. For each required service:
1. Check if expected port has a listener
2. If yes: verify it's OUR service via identification_marker (e.g., curl health_endpoint + check response)
3. If wrong service on port: report SQUATTER with PID + recommend migration to fallback_ports
4. If port empty AND auto_restart: try to start the script (logged); retry up to restart_max_attempts
5. Output structured status: `{service, port, state: 'healthy'|'squatted'|'down'|'starting'|'failed', recovery_action}`

Used by:
- SessionStart hook
- Stop hook (lighter-weight check)
- Manual CLI: `bash scripts/required-service-check.sh [--json] [--service=X]`
- Dashboard service-status panel

### M2 — SessionStart hook `scripts/hooks/required-service-startup-check.sh`

Wired into SessionStart. Runs the check; surfaces in digest:
```
─── Required services ───
✓ agent-inbox-server (port 4323)
⚠ agent-inbox-server expected 4322; running on 4323 (squatter on 4322: PID 63944 = astro dev)
✗ heartbeat: not installed (INSTALL-001 pending your approval)
```

If any P0 service is down without auto-recovery success: surface in URGENT INBOX as a `service-down-N` entry.

### M3 — Pre-action guard (PreToolUse on monkey-chamber writes)

A new hook `scripts/hooks/sidecar-precheck-monkey.sh` fires before any tool that COULD trigger a monkey-chamber write. Checks `agent-inbox-server` is healthy. If not, blocks the write with explanation rather than letting it fail silently downstream.

(Conservative: only on PreToolUse `.*` matching specific tool patterns; ~5ms overhead.)

### M4 — Dashboard service-status panel

New lefter mini-block + Stats-tab panel showing live service status. Updates via SessionStart's `data.json` + 60s polling.

```
─── Sidecars ───
🟢 inbox-server :4323
🟢 astro :4321
⚪ heartbeat (install opt-in)
```

Click → drill-down into recent health-check history + manual restart button.

### M5 — Auto-recovery with safety bounds

`scripts/sidecar-restart.sh <service-name>`:
1. Check current state (skip if already healthy)
2. Find a free port from `fallback_ports`
3. Start the script with that port
4. Wait up to 5s for it to become healthy
5. If success: log + update runtime-config so dashboard knows the new port
6. If fail after 3 attempts in 1 hour: disable auto-restart for that service + alert user

**Bounds:**
- Max 3 restart attempts per service per 1-hour window
- Cooldown: 30s between attempts
- After 3 fails → user must manually intervene + reset counter
- Dry-run mode: first 24h logs intended actions without executing

## Recovery path for runtime-config

When sidecar moves to fallback port, write `.claude/cache/sidecar-runtime-ports.json`:
```json
{"agent-inbox-server": 4323, "updated_at": "2026-05-27T04:30:35Z"}
```

Dashboard reads THIS at boot to construct INBOX_API URL — eliminates the need for hardcoded constants. Falls back to default_port if file missing.

(Future-proofing: today I hardcoded `:4323` in dashboard JS as the immediate fix. M5 makes that dynamic.)

## Phases

### Phase 1 — Foundational (BG)
- `config/required-services.json` registry
- `scripts/required-service-check.sh` CLI + library
- SessionStart hook + status surface
- Settings delta queued
- Token budget: <300k

### Phase 2 — Auto-recovery + dynamic port
- `scripts/sidecar-restart.sh` with safety bounds
- `.claude/cache/sidecar-runtime-ports.json` runtime config
- Dashboard reads runtime ports instead of hardcoded
- Token budget: <300k

### Phase 3 — Dashboard panel
- Lefter mini-block + Stats-tab panel
- Click-through to history + manual restart UI
- Token budget: <300k

### Phase 4 — Pre-action guards
- `sidecar-precheck-monkey.sh` (and similar for other write-paths)
- Token budget: <200k

## Cost gates per phase as above. Total <1.1M across 2-3 sessions.

## Cross-references

- A44 — meta-pattern parent (this audit is A44's first proper auto-spawn)
- INSTALL-001 (heartbeat LaunchAgent) — adjacent pattern; A45 absorbs the heartbeat install case
- SET-005 (settings batch) — port-config fixes
- A36 — BG-dispatcher instrumentation (similar substrate-needs-instrumentation pattern)
- A37 — state-batch-digest broken (similar untested-code pattern)
- bug-billboard `20260527T0430-main-claude-main-monkey-yes-click-lost-on-port-conflict.md` (the precipitating incident)

## Lessons (preliminary)

- The bug today was a 2-part stack: (1) wrong service on the port; (2) asymmetric offline-queue coverage. A45 closes part 1 structurally. Part 2 was fixed in `onMonkeyOptionClick` per the same turn.
- "Identification marker" is critical — checking "is anything on port X" isn't enough; must verify "is OUR service on port X." Otherwise an Astro squatter responds with 404 and looks like a server error.
- Auto-recovery bounded by 3-attempts-per-hour is the right balance: covers transient crashes; doesn't infinite-loop on broken installs.
- Runtime port config eliminates the brittle "hardcoded constants" pattern — pairs naturally with A36 dispatcher instrumentation philosophy.

## Status

CLOSED 2026-05-28T21:17Z — 4/4 tethers achieved + VERIFY 5/5. See closure below.

---

# A45 CLOSURE — 4/4-tether mutation (the A71 pattern applied)

**Closed 2026-05-28T21:17Z** via A74-D1+D5 mandate: A45 was named the BIGGEST adaptive-immunity hole (0/4 tethers, class still live-hurting — inbox-server-DEAD recurred 2026-05-27). This closure makes silent monkey-chamber click-loss STRUCTURALLY impossible the same way A71 closed the inbox-loss class.

## Why "0/4 tethers" despite the partial A45 build

Pre-existing A45 work (`services-check.sh`, `wake-restart.sh`, `sessionstart-services-digest.sh`, `required-services.json`) covered SOFTWARE-detection + SessionStart-surface — but:
- `services-check.sh` is **read-only** (its digest hook explicitly never restarts).
- `wake-restart.sh` is **user-run** (needs a human in a terminal).
- There was **no agent-invokable, automatic detect-AND-recover guard** — no HARDWARE barrier that runs without the user remembering anything, and **no VERIFY test** proving the failure can't recur.

Per `feedback_adaptive-immunity-discipline`: "Hardware tier is systematically under-used … VERIFY step is systematically skipped." Both gaps were exactly the missing tethers here.

## The 4 tethers (now complete)

| Tether | Artifact | What it does |
|---|---|---|
| **HARDWARE** | `scripts/hooks/inbox-server-guard.sh` | Agent-invokable structural barrier. DETECTS down/squatted server (marker-based, not port-based). `--apply` auto-restarts a DOWN server with bounded attempts (3/hr, cooldown). Dry-run default; kill-switch `EVIUM_INBOX_SERVER_GUARD_OFF=1`; exit 0 in hook contexts. NEVER kills a squatter (destructive/out-of-scope) — reports PID+cmd for human decision. |
| **SOFTWARE** | same guard + `--sessionstart` mode | Executable guard logic + a SessionStart surface that emits a `<system-reminder>` when the server is down/squatted, so the user KNOWS their chamber clicks aren't landing in purgatory BEFORE they click. Reads the `required-services.json` registry (falls back to baked-in canonical defaults if jq/registry absent). |
| **BIOLOGY** | this audit closure + cross-refs | Persistent memory: A45 closed, cross-referenced to A71 (sibling closure) + A62 (discipline) + A44 (parent). The class is encoded as a positive 4/4 example. |
| **VERIFY** | `tests/inbox-server-resilience-test.sh` | Sandboxed 5-assertion acid test (mirrors A71). Spins up REAL inbox-server copies + a fake 404 squatter on ephemeral ports. FAILS if server-death can silently lose clicks again. |

## VERIFY result — 2026-05-28T21:17Z

Test ran clean in an isolated sandbox (ephemeral high ports; never touches real 4322/4323 or real cache). **5/5 assertions PASS, exit 0:**

```
Assertion 1: OLD behavior — squatter on inbox port silently loses a chamber click
  [PASS] click LOST against squatter (proves the bug class existed) — 404 = silent loss, as in 2026-05-27
Assertion 2: NEW — guard detects a DOWN server (no listener on port)
  [PASS] guard reports state=down, exit 1
Assertion 3: NEW — guard --apply restarts a DOWN server to healthy
  [PASS] guard --apply recovered server (click LOST→LANDED) — exit 0; chamber click now lands
Assertion 4: NEW — port-conflict detected; --apply refuses to kill squatter
  [PASS] port-conflict reported (exit 2); squatter untouched; no blind restart
Assertion 5: IDEMPOTENT — guard on a healthy server is a no-op
  [PASS] repeated guard runs on healthy server = no-op (0 restarts)

Total: 5 pass / 0 fail
VERIFY: PROTECTION HOLDS — A45 server-death mutation is durable
```

Assertion 1 is the structural proof the failure existed (no guard ⇒ a 404 from a squatter = silently-lost click). Assertions 2–5 prove the guard closes it (detect → recover → port-conflict-safe → idempotent). The test fails BOTH ways (regression OR un-applied mutation), by design — same contract as A71.

## Real-system status at closure

`bash scripts/hooks/inbox-server-guard.sh` →
```
  ✓ agent-inbox-server [P0] healthy on port 4323
```
The real Python inbox-server (PID 9835) is UP on the registry's canonical port 4323. A node/Astro squatter still holds 4322 — the guard correctly checks 4323 + the `count` marker and ignores 4322. (The post-A45 port migration to 4323 is what neutralizes the original squatter; the guard validates the migration held.)

## Relationship to the earlier A45 design (M1–M5)

This closure does NOT re-litigate the M1–M5 design above. It lands the **two missing tethers** (agent-side auto-recover HARDWARE barrier + VERIFY test) on top of the already-built SOFTWARE/registry/SessionStart pieces. M3 (PreToolUse pre-action guard) and M4 (dashboard panel) remain deferred — the guard + its SessionStart surface + the bounded `--apply` recovery are the highest-leverage moves that make the silent-loss failure structurally impossible. Per `feedback_decision-finality`, those deferred sub-mechanisms are not re-raised here.

## A62 4/4-tether achievement

This is the second audit (after A71) in the retroactive adaptive-immunity map (A40/A44/A47/A48/A56/A57/A61/A71/A45) to achieve all 4 tethers verified end-to-end. The two together close BOTH halves of the chamber feedback loop: **A71** = the message a session might never SEE; **A45** = the click that might never LAND.
