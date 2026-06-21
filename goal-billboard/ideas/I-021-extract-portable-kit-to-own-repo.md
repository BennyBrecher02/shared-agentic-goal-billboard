---
idea_id: I-021
title: Extract the portable-kit to its own repo — timing at the first real external deploy
lifecycle: captured        # captured = awaiting your review (the user-gate per the ideas-lane safety contract)
serves_northern_star: G21  # the portable kit IS the deliverable core of the portable agentic-OS
belongs_to_goal: G21       # repo-creation todo #2 of 3, tracked under the Northern Star
user_gated: true           # USER-ONLY: repo creation + first push are gated; the user decides WHEN to extract
timing: at-first-real-external-deploy
source: user-prompt        # the user-approved goal-grooming triage (2026-06-21) appended these 3 repo-creation todos
source_ref: context/markdowns/audits/goal-grooming-proposal-2026-06-21.md
created: 2026-06-21T21:01Z
updated: 2026-06-21T21:01Z
---

# I-021 — Extract the portable-kit to its own repo (timing: at the first real external deploy)

**What it is.** The `portable-kit/` (deploy-kit.sh, generate-agent-configs.sh, templates, adapter contract —
the tool-/repo-/machine-agnostic core of G21) currently lives **inside this repo**. This todo: when the kit
first needs to be deployed somewhere **external** (a fresh repo / a different machine / another person), give
it **its own repo** so it can be cloned + deployed standalone, versioned independently of this OS-host repo.

**Why "at the first real external deploy," not now.** While the only consumer is this repo, an in-repo kit is
simpler — one history, one place to iterate. The cost of a premature split (two repos to keep in lockstep, a
sync seam) only pays off once the kit is actually consumed from outside. The trigger is the **first real
external deploy**; extracting before that is speculative structure.

**Why USER-GATED.** New repo creation + the first push are user-owned (gated). The user also owns the *timing*
call — "is this the real external deploy that justifies the split?" is a judgment, not an automation.

**Concrete first step (when the trigger fires).** Carve `portable-kit/` (+ the templates/refs it needs) into a
new repo with its own README + deploy entrypoint; decide the sync story back to this host (submodule vs
periodic copy vs the kit becomes canonical). Until then: keep iterating in-repo.

**Timing: at the first real external deploy.** Premature extraction adds a sync seam for no consumer.
