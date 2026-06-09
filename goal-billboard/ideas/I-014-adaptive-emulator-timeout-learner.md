---
idea_id: I-014
title: Dynamic self-readjusting emulator "sweet-spot" idle-timeout (the adaptive half of A42)
lifecycle: captured
serves_northern_star: G9   # testing toolbelt / anti-bottleneck; advisory
source: floated-ideas-audit-2026-06-07
source_ref: context/markdowns/goal-billboard/audits/findings/floated-ideas-audit-2026-06-07.md
leverage: med
related: A42 (emulator-killer) · scripts/hooks/sim-emu-session-teardown.sh (the static base, built)
created: 2026-06-07T00:00Z
updated: 2026-06-07T00:00Z
---

# I-014 — Adaptive emulator idle-timeout learner (the missing half of A42)

**What it is.** The emulator-killer should not use a fixed idle threshold — it should **learn and re-tune**
the idle-timeout sweet-spot from actual usage, folded into the stats system. A small learner records
emulator idle→reuse intervals and re-tunes the teardown threshold (a reactionary-style post-run recompute),
so we never kill an emulator we're about to reuse, nor leave a dead one burning.

**Why it matters.** The base killer shipped and is wired (`scripts/hooks/sim-emu-session-teardown.sh`, one
wire in settings.json) — but it's a **static** teardown. The user asked **twice** (0d739f65, 23f93bfd) for
*"some intense logic to not only back this up but also dynamically readjust the sweet spot time."* Grepping
the script for dynamic/adaptive/readjust/sweet-spot returns 0 hits — the parent built the teardown, the
self-readjusting brain the user wanted was never added.

**Concrete first step.** Add a learner that logs each emulator's idle-duration-before-reuse (and
killed-then-immediately-needed misfires), then recomputes the teardown threshold from that distribution on a
reactionary post-run pass — feeding the recomputed value back into the teardown hook.

**Leverage: MED.** Awaiting greenlight (surface-only by contract).
