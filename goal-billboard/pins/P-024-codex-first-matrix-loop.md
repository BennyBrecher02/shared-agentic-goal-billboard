---
id: P-024
kind: paused-work
title: "Codex first-use: wire one Codex worker to run a matrix-run task end-to-end"
created: 2026-07-08
state: pinned
surface_when: user says "start codex" / "wire codex" / next Codex session
settle_how: one matrix-run task claimed from the inbox by a Codex worker + run + result posted back; cost observed
---

# P-024 — The concrete FIRST Codex move (pinned, remind me)

Wire ONE Codex worker to claim a `matrix-run` task from its task-inbox and run it — a single end-to-end loop to
LEARN Codex's real reliability + cost before trusting it with more. LOW effort: the executor already exists
(`scripts/task-run-matrix.sh`), the hand-off rail exists (`task-inbox.sh --to codex`), the gate exists
(`codex-block-dangerous.sh`). First-use = teach a Codex worker to poll its inbox + invoke the executor.
Full research: context/markdowns/research/codex-orchestration-slamdunks-2026-07-08.md. Sibling pin: [[P-023]]
(the idle-fill executor idea, AFTER this proves out).
