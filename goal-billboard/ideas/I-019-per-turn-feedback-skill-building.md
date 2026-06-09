---
idea_id: I-019
title: Per-turn "feedback-skill-building" end-of-turn automation — decide-or-decline (in limbo since May 25)
lifecycle: captured
serves_northern_star: G4   # skill-system maintenance; advisory
source: floated-ideas-audit-2026-06-07
source_ref: context/markdowns/goal-billboard/audits/findings/floated-ideas-audit-2026-06-07.md
leverage: low
related: feedback_skill-maintenance-cadence.md (the existing PERIODIC consolidation — the near-twin this must be disambiguated from)
created: 2026-06-07T00:00Z
updated: 2026-06-07T00:00Z
---

# I-019 — Per-turn feedback-skill-building automation (decide-or-decline)

**What it is.** The user floated (2842f478, 2026-05-25): *"maybe some feedback skill building stuff … that
run at the end of every turn."* The end-of-turn script framework and `record-matrix-timing.sh` shipped, but
the specific idea — **auto-build a feedback skill at the end of EVERY turn** — was never built as such. The
existing `skill-maintenance-cadence` is **periodic consolidation**, not per-turn, so this near-twin is
distinct and has sat in limbo since May 25.

**Why it matters.** It's been unresolved for the longest of any item here — neither built nor declined. The
right move is a **decision**, not necessarily a build: per-turn skill-building risks slop/noise and token
cost on every single turn, which may be strictly worse than the existing periodic cadence. This idea exists
to force that call so it stops floating.

**Concrete first step.** Decide whether per-turn feedback-skill-building is even desirable vs the existing
periodic `skill-maintenance-cadence`. If yes → scope it (cost-guarded, anti-slop). If no → explicitly
**decline** it with a one-line reason in `notes/steering-decisions-log.md` and set this idea's `lifecycle:`
to `dropped`. Either outcome closes the limbo.

**Leverage: LOW.** Awaiting greenlight / a decline decision (surface-only by contract; the user owns the
call).
