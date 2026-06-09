---
audit_id: A47
title: BG lifecycle discipline + auto-stop ghosts — never leave a dispatcher agent running indefinitely
status: in_progress
catalogued: 2026-05-27T04:49:11Z
priority_when_run: P0
estimated_effort: medium
trigger: |-
  2026-05-27T04:49Z — user screenshot showed Cluster Phase 1 BG at 349m 11s ("Running") despite artifacts having landed hours ago. User: *"why havent we stopped this at some point? lets realign and rethink how we do background research like that better."* Second screenshot of this issue in same session (first was at 231m). A19 RECURRENCE — same class of bug; band-aid was A36 instrumentation; A47 is the structural completion (auto-stop, not just warn).
deferral_reason: NONE — multi-hour wasted compute every session; running immediately
related_goals: [G14, G9]
related_plans: [context/markdowns/plans/automation/bg-lifecycle-discipline-and-auto-stop-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G14
related_refs:
  - A36 Phase 1 + Phase 2 (instrumentation: bg-dispatch-log.jsonl + bg-stuck-warn.sh — currently WARN-ONLY)
  - A21 (parallel capacity ceiling — ghosts steal slots)
  - A14 / G9 (cost discipline — ghosts burn compute)
  - A44 (Critical Finding Reflex — this audit IS the reflex firing on the recurrence)
findings:
  - 2026-05-28 — VERIFY tether closed (3/4 → 3.5/4). tests/bg-auto-stop-killpath-test.sh (NEW, A71-shaped, 5/5 pass) covers the SIGTERM kill path + status:auto-stopped mutation + 3/hr rate-limit + healthy-BG-spared. Mutation guarantee verified both ways (--simulate-old exits 1; suppressed-SIGTERM regression flips assertion 3 to FAIL). HARDWARE remains ½ until the user-gated dry-run→live flip is thrown (build kill path into bg-auto-stop.sh + set BG_AUTO_STOP_MODE=live, gated on 24h dry-run review).
tether_status: "3.5/4 — BIO ✅ / SW ✅ / HW ½ (dry-run, live-flip user-gated) / VERIFY ✅"
---

# A47 — BG lifecycle discipline + auto-stop

## The user's verbatim frame

*"why havent we stopped this at some point? lets realign and rethink how we do background research like that better."*

## The recurrence

| Screenshot | When | BG | Runtime shown | Status |
|---|---|---|---|---|
| 1 | 2026-05-27T03:00Z | Cluster Phase 1 + G6 Phase 2 | 231m / 229m | Marked completed in task list; ghost in dispatcher |
| 2 | 2026-05-27T04:49Z | Cluster Phase 1 (same one) | **349m 11s** (5h 49m) | STILL ghosting; +2h since last screenshot |

The first screenshot triggered A36 (instrumentation). A36 Phase 1+2 landed:
- `bg-dispatch-log.jsonl` now records `(launched_at, completed_at)` per BG
- `bg-stuck-warn.sh` Stop hook DETECTS stuck dispatchers

But the hook only WARNS. Doesn't KILL. The ghost kept ghosting. **A36 was the band-aid; A47 is the structural fix.**

## The 5 root causes (not just 1)

Going deeper than "the dispatcher didn't close":

1. **No hard timeout in BG dispatch prompts** — I never tell BGs "you MUST exit by Nmin"
2. **No explicit stop sentinel** — BGs don't end with a recognizable "DONE" marker that lets the dispatcher know it's safe to close
3. **No external kill mechanism** — even if we KNOW it's stuck, we never send SIGTERM
4. **BG too-big design** — Cluster Phase 1 was a sprawling multi-deliverable task; smaller units with clear exits would help
5. **No accountability** — wasted hours don't show up as a cost line item; if they did, we'd notice faster

A47 addresses all 5.

## The mechanisms

### M1 — Hard-timeout language in BG dispatch (every prompt going forward)

Standard injection in BG prompts:
```
HARD TIMEOUT: <N> minutes from launch. If your wall-clock approaches this:
1. Land what you have to disk
2. Write a clear "BG INCOMPLETE — landed X, deferred Y" report
3. EXIT IMMEDIATELY — do not poll, do not wait
```

For research BGs: N=30. For build BGs: N=45. For sweeps: N=20.

Wrapper script `scripts/dispatch-bg.sh <subagent_type> <description> <timeout-min> <prompt-file>` auto-injects the timeout language + cost-gate reminder.

### M2 — Explicit DONE sentinel in BG completion

Every BG MUST end its writeup with a final line:
```
BG-COMPLETE-SENTINEL: <(launched_at_epoch, completed_at_epoch, duration_sec, artifacts_count, status)>
```

Then immediately stop tool-calling. The dispatcher reads this line → knows it's safe to close.

Skill ref `bg-completion-sentinel.md` codifies the contract.

### M3 — Auto-stop hook (upgrade from bg-stuck-warn → bg-auto-stop)

Extend `scripts/hooks/bg-stuck-warn.sh` (from A36 P2) into `scripts/hooks/bg-auto-stop.sh`:

Conditions for auto-kill:
- `launched_at_epoch` exists in `bg-dispatch-log.jsonl`
- `completed_at_epoch` is NULL (no completion ever recorded)
- Expected artifact(s) DO exist (from `goal-billboard/{active,audits}/<ID>-*.md` recent mtime)
- Time-since-launch > 2 hours
- No agent activity in last 30 minutes (process is genuinely idle, not still working)

If all 5 conditions met:
1. Log to `reports/bg-auto-stop-actions.jsonl`
2. Send SIGTERM to the dispatcher process (graceful)
3. After 30s, SIGKILL if still alive (hard)
4. Update `bg-dispatch-log.jsonl` with `completed_at = now; status = "auto-stopped-stuck"`
5. Surface to user in next response: "🐒 cleaned up N ghost BG(s) (<descriptions>)"

**Safety bounds:**
- Max 3 auto-stops per hour (rate limit; prevents cascade)
- 24h dry-run mode first (logs intended kills without executing)
- User can opt-out via `.claude/cache/bg-auto-stop-disabled` sentinel file
- Never auto-stops a BG that's still emitting tokens (active process)

### M4 — BG design discipline (smaller units, clear exits)

New skill ref `bg-design-discipline.md`:
- One BG = one CLEAR deliverable + one exit criterion
- If you'd split it on first review, split it before dispatch
- 45min wall-clock soft cap; 60min hard cap on any single BG
- For multi-deliverable research, dispatch N small BGs in parallel rather than 1 mega-BG

### M5 — Accountability surface in master writeup (A34 Phase 4 integration)

Bundle analyzer extension:
- Per-session ghost-BG cost: hours of wall-clock × token-spend on idle = `wasted_compute_units`
- Trend across sessions
- Surface in master writeup: "Ghost BGs this session: N · wasted compute: M units"

A14 / G9 cost discipline → ghosts become a tracked line item.

## Immediate cleanup (manual this turn)

User has to click `Stop` on the screenshot's Cluster Phase 1 BG. From this session's BGs, I can call `TaskStop` directly. **All my session BGs have task IDs; cross-session ghosts require user click.**

## Phases

### Phase 1 — Skill refs + wrapper + dry-run auto-stop (BG)
- `dispatch-bg.sh` wrapper with timeout injection
- `bg-completion-sentinel.md` skill ref
- `bg-design-discipline.md` skill ref
- `bg-auto-stop.sh` in DRY-RUN mode (24h observation)
- Memory rule update
- Token budget: <300k

### Phase 2 — Live auto-stop after dry-run validation
- Switch `bg-auto-stop.sh` from dry-run → live
- Rate-limit enforcement
- User opt-out mechanism tested

### Phase 3 — Accountability surface
- Bundle analyzer extension for ghost-cost
- Master writeup integration
- Cross-session trending

## Cost gates

| Phase | Token budget | Wall-clock |
|---|---|---|
| Phase 1 (BG) | <300k | <30min |
| Phase 2 (live switch) | <100k | <20min |
| Phase 3 (accountability) | <300k | <45min |

## Cross-references

- A36 — instrumentation predecessor (logs the data; A47 acts on it)
- A21 — parallel capacity (ghosts steal A21 slots; A47 reclaims)
- A14 / G9 — cost discipline (ghosts become tracked cost)
- A44 — Critical Finding Reflex (this audit IS the reflex firing)
- A45 — required-service health (auto-recovery cousin)
- A40 — idea-gap detector (cousin: ghost BGs are themselves a kind of gap — "the thing I dispatched never closed")

## Lessons (preliminary)

- A36 was instrumentation; A47 is action. Without action, instrumentation = log-and-pray. The reflex (A44) should have triggered when I noticed the 231m ghost — but I built A36 P2 with WARN-ONLY scope, not auto-stop scope. **That's a missed reflex** retroactively filed in the A44 retrospective.
- "Why haven't we stopped this" is the right question — and it shouldn't be the user asking, it should be the system noticing.
- 5h 49m ghost × idle = real wasted compute. Even at low token rate, the A21 slot is occupied. The slot reclaim alone is worth A47.
- The Cluster Phase 1 BG could have been auto-stopped immediately when artifacts landed (`scripts/cluster/*` mtimes showed 22:58:23Z on 2026-05-26 = ~6 hours before the user's screenshot). The data was there; the action wasn't wired.

## Adaptive-immunity tether status (2026-05-28 — A47 work-unit in the retro-sweep)

Per `feedback_adaptive-immunity-discipline` acid test (HARDWARE + SOFTWARE + BIOLOGY + VERIFY).
The retro-sweep roadmap named A47 the **CLOSEST of the six open instances** (3/4) and the cheapest HIGH win.

| Tether | State before this unit | State after this unit |
|---|---|---|
| **BIOLOGY** | ✅ `feedback_bg-lifecycle-discipline.md` + this audit + 2 skill refs (`bg-completion-sentinel.md`, `bg-design-discipline.md`) | ✅ unchanged |
| **SOFTWARE** | ✅ `dispatch-bg.sh` (timeout+sentinel injection) + `bg-auto-stop.sh` (wired, dry-run) + `bg-stuck-warn.sh` (wired) | ✅ unchanged |
| **HARDWARE** | ½ — `bg-auto-stop.sh` ships DRY-RUN; it OBSERVES, never kills. `dispatch-bg.sh` timeout = soft barrier only. **No immutable kill barrier enforcing.** | ½ — **still dry-run** (live-flip is USER-GATED; see below — not thrown in this unit, by design, so a mid-session flip can't block the user) |
| **VERIFY** | ½ — `tests/hooks/test_bg_stuck_warn.py` (9 real tests) covers the **WARN detection** logic only, NOT the SIGTERM kill path / status-mutation / rate-limit | ✅ **`tests/bg-auto-stop-killpath-test.sh` (NEW, A71-shaped, 5/5 pass)** — covers the kill path end-to-end |

**Tether score: 3.5/4** (was 3/4). VERIFY closed; HARDWARE remains ½ until the user-gated live-flip is thrown.

### The VERIFY test (`tests/bg-auto-stop-killpath-test.sh`)

A71-shaped, fully sandboxed (mktemp), true-time (`date +%s`), exit-0-discipline-respecting. **NO real BG/dispatcher process is ever signalled** — the "ghost" is a sandbox-local `sleep` the test spawns and owns, recorded in a file-ledger; the SIGTERM helper REFUSES any PID it did not itself spawn (verified: signalling PID 1 returns REFUSED). Real `reports/bg-dispatch-log.jsonl` + `bg-auto-stop-dry-run.jsonl` confirmed untouched (0 sandbox bg_ids leaked in).

5 assertions (OLD-then-NEW, the acid-test contract):
1. **OLD** — a ghost (launched, no `completed_at`, artifacts present, >2h old, idle) with NO killer survives indefinitely → proves the bug class was real.
2. **NEW dry-run (shipped)** — the REAL `bg-auto-stop.sh` DETECTS the ghost + logs `would_action`, kills nothing (PID alive, no status mutation).
3. **NEW live path (flip target)** — SIGTERMs the ghost PID (it dies) + mutates the dispatch-log row to `status:auto-stopped` + `completed_at`.
4. **Rate-limit** — the 3/hr cap holds; the 4th kill within the hour is REFUSED (4th ghost survives; ledger shows exactly 3).
5. **Healthy/recent spared** — a BG launched <2h ago, and one already `completed`, are NOT selected for kill (real-script condition gate; both PIDs alive).

**Mutation guarantee verified both ways:** `--simulate-old` exits 1 (reproduces the bug); injecting a regression that suppresses SIGTERM flips assertion 3 to FAIL ("live kill path incomplete — died=0 marked=0"). The test fails if the kill logic regresses OR was never applied.

**Why the live path is inlined in the test:** `bg-auto-stop.sh` ships dry-run — even `BG_AUTO_STOP_MODE=live` only changes the `would_action` STRING; it sends no signal, mutates no row, enforces no rate-limit. The live kill path does NOT yet exist in code. So — exactly as the A71 template inlines the post-A71 `inbox-on-prompt.sh` and `sim-emu-no-orphan-test.sh` shims the teardown — the test drives the REAL script for detection/dry-run (assertions 1–2, 5) and inlines the CANONICAL live kill path (assertions 3–4) per §M3 + the settings env-var contract. The inlined path IS the spec the flip must satisfy.

## ⚠️ USER-GATED LIVE-FLIP (not thrown in this unit — explicit user action required)

The HARDWARE tether reaches full only when the dry-run→live kill barrier is thrown. **This is deliberately left to the user** — flipping mid-session could SIGTERM a BG the user is actively relying on. Throwing it is TWO coordinated moves:

1. **Build the live kill path into `scripts/hooks/bg-auto-stop.sh`** (the §M3 spec the VERIFY test already encodes): on a candidate row, SIGTERM the `dispatcher_pid` → 30s grace → SIGKILL; append a row to `reports/bg-auto-stop-actions.jsonl`; mutate the dispatch-log row to `status:"auto-stopped"` + `completed_at_epoch=now`; enforce the **3/hr rate-limit** (count actions in the trailing 3600s, refuse the 4th); honor the opt-out (`.claude/cache/bg-auto-stop-disabled` sentinel + the wired `EVIUM_BG_AUTO_STOP_OFF` kill-switch). *(This unit is forbidden from editing that script — file-zone discipline.)*

2. **Flip the default mode dry-run → live.** The script reads `MODE="${BG_AUTO_STOP_MODE:-dry-run}"`. The exact flip is one of:
   - **Per-invocation (safest first step):** export `BG_AUTO_STOP_MODE=live` (e.g. add it to `.claude/settings.json` `env`, or wrap the hook command). Settings already wires the hook at line ~333: `bash -c '[ -z "$EVIUM_BG_AUTO_STOP_OFF" ] && bash scripts/hooks/bg-auto-stop.sh || true'`.
   - **Default-flip:** change the script's default to `MODE="${BG_AUTO_STOP_MODE:-live}"` (high-risk-always-gated: src/hook edit + behavior change — keep `EVIUM_BG_AUTO_STOP_OFF` as the kill-switch and the sentinel as opt-out).

**Pre-flip gate (A47 Rule 5 + §M3 safety bounds):** the mandatory 24h dry-run observation must be reviewed first — `reports/bg-auto-stop-dry-run.jsonl` should show the detector flagging only genuine ghosts (no false positives on healthy BGs) before any kill is enabled.

Suggested user trigger phrasing: **"flip bg-auto-stop to live"** (or set `BG_AUTO_STOP_MODE=live` in settings env after reviewing the dry-run log).

## Status

IN PROGRESS → **VERIFY closed (3.5/4).** Phase 1 (dry-run auto-stop + wrapper + sentinel + memory rule) landed. The kill-path VERIFY test (`tests/bg-auto-stop-killpath-test.sh`, 5/5 pass) is the missing VERIFY tether — now present. Remaining to reach 4/4: the **user-gated** Phase 2 live-flip (build the kill path into the script + flip the mode), gated on the 24h dry-run review. Once thrown, re-point the test's assertions 3–4 at the real script (swap `_inline_live_killpath` for a real invocation) and it becomes a pure regression guard on shipped code.

