---
goal_id: G25
title: "Igor — the cluster's autonomous executor (frictionless 'post a task, come home to it done')"
priority: P1
status: active
northern_star: false
track: CLUSTER
created: 2026-07-13
gated_on: "none for Phase 1 build (Igor is owner-installed; Benny installs the systemd unit + sets the trust posture). Dispatch-by-data needs no per-launch gate."
project: AgenticOS
---

# G25 — Igor 🧟

The always-on worker on the Pi that takes tasks from the shared lane and runs them with a local
coding agent (Claude now, Codex later), reports back, and pings the phone. Kills the per-launch
"paste this to run a build" friction permanently by making dispatch *data* and authorization
*owner-granted-once*. The keystone connecting G23 (cluster alive), G24 (Codex compute), and the
cross-tool shared-brain.

**Trust model (Benny's call 2026-07-13):** full trust on the Pi (same as the Mac — the machine he
cares about more); safety via receipts (ledgered tasks + run logs + phone pings + nightly health),
not cages. Retained hygiene only: user owns pushes, per-run time/turn caps, kill-switch.

**Full roadmap + domino map + phases:**
`context/markdowns/plans/automation/igor-cluster-executor-roadmap-2026-07-13.md`

## Phase gate
1. **Igor v1** — watcher service + lane poller + local-Claude executor + ntfy ping; Mac dispatch.
2. Phone dispatch. 3. Codex lane. 4. Multi-node. 5. Self-directed idle-capacity work.

## Definition of alive (v1)
Post a task from the Mac → lid closed → Igor claims + runs the Pi's Claude → result rides the sync
home → phone buzzes. Zero per-launch friction, full audit trail.
