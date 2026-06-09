---
idea_id: I-016
title: "All agents TIME their testing" — the universal matrix-timing mandate
lifecycle: captured
serves_northern_star: G9   # testing toolbelt / anti-bottleneck; advisory
source: floated-ideas-audit-2026-06-07
source_ref: context/markdowns/goal-billboard/audits/findings/floated-ideas-audit-2026-06-07.md
leverage: med
related: scripts/record-matrix-timing.sh (mechanism built; invocation opt-in)
created: 2026-06-07T00:00Z
updated: 2026-06-07T00:00Z
---

# I-016 — Universal test-timing mandate (turn an opt-in mechanism into an enforced discipline)

**What it is.** Make per-test timing a **universal, enforced discipline** across every testing agent, so the
system builds a hard-data view of the matrix's time-complexity — the raw material for proving (and improving)
parallelization with numbers instead of intuition.

**Why it matters.** The *mechanism* exists (`scripts/record-matrix-timing.sh`) but invoking it is **opt-in** —
no rule mandates that every testing agent times its run, so the "clearer wider view" the user wanted is never
systematically collected. The user floated it (2842f478, 2026-05-25): *"maybe all agents should now time
their testing so we can get a clearer wider view of our matrixs time complexity and further improve our
parallelization backed by logic and agents proof."* The mechanism is built; the universal rule was never
written into memory or a skill as enforced. (Stays inside the anti-bottleneck contract — timing is cheap and
must never itself become the bottleneck.)

**Concrete first step.** Add a one-line discipline to the `agentic-device-testing` SKILL ("every matrix run
pipes through `record-matrix-timing.sh`") and note the same rule in the device-matrix memory feedback file,
so the timing collection becomes default behavior rather than something an agent has to remember.

**Leverage: MED.** Awaiting greenlight (surface-only by contract).
