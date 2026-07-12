---
id: G23
title: "Frankenstein cluster bring-up — the Pi/Mac distributed organism comes alive"
priority: P0
status: active
northern_star: true
track: CLUSTER
created: 2026-07-08
gated_on: user plugs in the (already-flashed) Pis + installs Tailscale
---

# G23 — THE NORTHERN STAR: bring the Pi/Mac cluster to life 🧟⚡

The user's flashed Pis are ready; the goal is to plug them in and animate the distributed organism
**Frankenstein-style** — laptop (M4) = cockpit, one Pi = always-on brainstem, the M2 = muscle. This is now the
guiding light (replaces the null Northern Star the overview flagged).

## The vision (plain)
- **M4 laptop = the cockpit** — where the user steers; allowed to sleep.
- **One Pi = the brainstem** — never sleeps; holds the shared memory + git remote + the task-watcher + phone
  alerts 24/7. When the laptop is closed, the Pi keeps the organism's heart beating.
- **M2 (the user's older Mac) = muscle** — SSH-driven device-test farm + burst compute. ($0 — it's owned hardware.)
- **The glue = a shared MIND, many BRAINS** (the user's "split-brain" = the GOAL): shared conventions (the kit) +
  shared state (the shared-billboard git repo) + published live chain-of-thought. NOT one merged brain; NOT
  siloed nodes. Each tool/node thinks independently but reads the same shared mind.

## Canon docs
- `context/markdowns/research/systems/cluster-state-of-union-2026-07-08.md` (the consolidated picture)
- `context/markdowns/plans/automation/tailscale-mac-setup-2026-07-08.md` (step 0: reachability)
- `context/markdowns/plans/automation/pi-brainstem-runbook.md` (the 9-section bring-up)

## Phase gate (the Frankenstein switch-throws, in order)
1. Tailscale on the M4 (reachability spine). 2. First-boot a Pi + ID the board. 3. Tailscale on the Pi.
4. Bare git remotes + first offsite push. 5. Shared-store sync timer. 6. Watcher systemd unit.
7. First sharded job: a task-lane round-trip (Mac posts → Pi claims+runs+replies). 8. ntfy phone alerts.

## Definition of alive
The M4 posts a task, closes its lid, and the Pi picks it up, runs it, and pushes the result back — proving the
organism outlives any single node being awake.
