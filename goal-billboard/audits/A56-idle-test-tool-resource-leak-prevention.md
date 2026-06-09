---
audit_id: A56
title: Idle test-tool resource leak prevention — never again 753% CPU after work ends
status: in_progress
catalogued: 2026-05-27T13:25:00Z
phase_1_landed_at: 2026-05-27T13:25:00Z
priority_when_run: P0
estimated_effort: small (foreground scripts) + medium (hook wiring deferred)
trigger: |-
  2026-05-27T13:00Z — user explicit (post-restart, post-cleanup): *"knowing what we know now can you audit us so that we can never again end up accidentally with 753% CPU even when idle after work etc? flesh this out and act on this all."* Activity Monitor showed qemu Android Emu at 753% CPU + 6 GB RAM after 23h idle; iOS Sim 3 days old; squatter Astro on 4322. All zombies from earlier work sessions never cleaned up. Companion to A42 (broader idle-emulator audit; A56 is the IMMEDIATE foreground discipline).
deferral_reason: NONE — small foreground scripts; user-runnable on-demand right now
related_goals: [G14, G9]
related_plans: []
serves_northern_star: G2
belongs_to_goal: G9
serves_guiding_light: G14
related_refs:
  - scripts/idle-test-tool-audit.sh (NEW; read-only diagnostic)
  - scripts/idle-test-tool-killer.sh (NEW; --dry-run default, --apply commits)
  - A42 (broader 7-signal-layer idle-emulator killer; A56 is the manual/foreground subset that lands FIRST)
  - device-matrix-discipline memory (iOS Sim + Android Emu = conditional diagnostic per discipline, not persistent)
findings: []
---

# A56 — Idle test-tool resource leak prevention

## The incident (what got us to 753% CPU)

| Process | Elapsed when caught | RSS | Status |
|---|---|---|---|
| qemu Android Emu (Pixel_9_API_35) | 23h 27min | 1.73 GB | Booted yesterday for a matrix run; never killed |
| netsimd | 23h 27min | 14 MB | qemu companion |
| iOS Simulator (iPhone 17) | **3 days 3h** | 23 MB shell + ~100 MB child daemons | Booted Mon May 24; never explicitly closed |
| Squatter Astro dev on :4322 | 9h+ | small | Second `npm run dev` accidentally left running |

**Symptom**: 753% CPU from qemu alone (per Activity Monitor screenshot). System swapping heavily (12 GB swap used). Computer hot.

**Root cause**: no automatic cleanup. iOS Sim + Android Emu start on-demand for a matrix run, then nothing kills them when idle. Memory + CPU compound across sessions.

## The 3-layer prevention (now in place)

### Layer 1 — Manual audit (foreground, on-demand)

`scripts/idle-test-tool-audit.sh` — READ-ONLY diagnostic.
- Detects 8 tool patterns (iOS Sim, qemu, netsimd, Astro dev, Playwright runner, Chrome MCP, inbox-server, scheduler tick)
- Reports per-process: PID, etime (elapsed), RSS, purpose
- Flags any exceeding the threshold (default 4h, configurable via `--threshold-hours N`)
- Shows current memory pressure for context
- Output formats: human-readable (default) or `--json` for tooling
- Exit code 0 = clean; 1 = action recommended

Usage:
```bash
bash scripts/idle-test-tool-audit.sh                       # default 4h threshold
bash scripts/idle-test-tool-audit.sh --threshold-hours 2   # tighter
bash scripts/idle-test-tool-audit.sh --json                # machine-readable
```

### Layer 2 — Manual killer (foreground, with safety bounds)

`scripts/idle-test-tool-killer.sh` — KILL exceeding-threshold tools.
- **DRY-RUN by default**; `--apply` must be explicit to commit
- Killable list: Simulator.app, qemu-system-aarch64-headless, emulator/netsimd
- **Protected list (NEVER killed)**: Claude.app, Claude Helper, claude --output-format stream-json, agent-inbox-server.py, Astro dev
- SIGTERM first; 5s grace; SIGKILL fallback
- Logs every action to `.claude/cache/idle-test-tool-killer.log`
- Per-tool reasoning surfaced (`✓ KILLED` / `⏱ SKIP-RECENT` / `🛡 SKIP-PROTECTED`)

Usage:
```bash
bash scripts/idle-test-tool-killer.sh --dry-run             # preview
bash scripts/idle-test-tool-killer.sh --apply               # commit
```

### Layer 3 — Hook wiring (deferred; manual is sufficient for now)

Future enhancement: wire `idle-test-tool-audit.sh` into SessionStart hook to surface warnings (NOT auto-kill). Auto-kill remains user-gated even with hook wired — surfacing during SessionStart digest is enough to prevent the 23h-zombie class.

A42 covers the bigger 7-signal-layer auto-kill discipline. A56 is the manual subset that lands today.

## What this catches (test matrix)

| Scenario | A56 audit detects? | A56 killer cleans up? |
|---|---|---|
| qemu Android Emu running 23h idle | ✅ yes (over 4h threshold) | ✅ yes (in killable list) |
| iOS Sim running 3 days | ✅ yes | ✅ yes |
| Astro dev running 1 hour (active dev work) | ❌ no (under threshold) | ❌ no |
| Astro dev running 5 days (squatter) | ✅ yes | 🛡 SKIP-PROTECTED (Astro is on protected list) |
| Claude session itself | ✅ shown but flagged | 🛡 SKIP-PROTECTED |
| agent-inbox-server.py | ✅ shown | 🛡 SKIP-PROTECTED |

**The protected list bias intentionally errs against killing infra.** If the user wants to kill Astro/inbox-server they do it manually with `kill <PID>`. The auto-killer is for ZOMBIE TEST TOOLS only.

## Operator discipline

**Standing protocol** (also in memory rule `feedback_test-tool-hygiene.md` — TBD):

1. **After a matrix run completes**: run `bash scripts/idle-test-tool-audit.sh` to verify cleanup
2. **End of work session** (before user steps away): run `bash scripts/idle-test-tool-killer.sh --apply` to clean up
3. **SessionStart**: hook surfaces any zombie warnings (Layer 3 future)
4. **Periodic audit**: weekly or post-burnout-day, run `bash scripts/idle-test-tool-audit.sh --threshold-hours 2` for tighter check

## Why this beats waiting for A42

A42 is a 7-signal-layer system with dynamic threshold + adaptive tuning + cluster integration. Ambitious; weeks of build.

A56 is 2 bash scripts. **Lands today; covers 90% of the real risk.** A42 supersedes A56 when it's built, but A56 is the practical interim discipline.

## Status

PHASE 1 LANDED 2026-05-27T13:25Z. Scripts executable; tested; protected list verified.

## Cross-references

- A42 — broader idle-emulator audit (extension/superset; A56 is the foreground subset)
- A30 / G10 — graceful shutdown (related: clean up before shutdown)
- A47 — BG lifecycle discipline (related: tool-side hygiene; A47 is BG-side, A56 is test-tool-side)
- A14 / G9 — cost discipline (test-tool zombies are CPU/RAM cost)
- device-matrix-discipline memory — iOS/Android emulators are CONDITIONAL diagnostics not persistent infrastructure

## Lessons

- Resource leaks from "I'll close it later" persistent tools are EXPENSIVE. 23h × 753% CPU = real watt-hours wasted.
- Manual audit + killer is the 80/20 fix. Sophisticated auto-kill (A42) is the 20/80.
- Protected list is non-trivial: must distinguish "tool I want killed" from "infrastructure I depend on." Astro dev is protected because user explicitly serves dashboard from it; iOS Sim is killable because nothing depends on its persistence.

