---
idea_id: I-013
title: Cross-substrate idea-gap sweep — apply the A40 lens to event-bus / dispatch-log / bug-billboard inbox
lifecycle: captured
serves_northern_star: G14   # closure discipline; advisory
source: floated-ideas-audit-2026-06-07
source_ref: context/markdowns/goal-billboard/audits/findings/floated-ideas-audit-2026-06-07.md
leverage: med
related: A40 (idea-gap detector) · scripts/detect-idea-gaps.py
created: 2026-06-07T00:00Z
updated: 2026-06-07T00:00Z
---

# I-013 — Cross-substrate idea-gap sweep (the named A40 follow-up)

**What it is.** A40 built `detect-idea-gaps.py` to sweep **session JSONLs** for dropped/unserviced requests.
The explicitly-named-but-undone extension: run that same dropped-request lens over the OTHER coordination
substrates — `event-bus.jsonl`, `dispatch-log.jsonl`, and `bug-billboard/inbox/` — so an idea dropped in ANY
substrate (not just chat) gets caught.

**Why it matters.** The system's own lesson from the A40 build was *"the substrate was sitting there FOR
WEEKS — built-but-not-queried."* The fix was scoped to ONE substrate; the other three coordination
substrates still have no gap-detection. This very floated-ideas audit is itself downstream of the original
gap-mining ask — proof the class is still live and leaking. Floated as the explicit deferred follow-up to
the A40 build (0d739f65, 2026-05-27).

**Concrete first step.** Extend `scripts/detect-idea-gaps.py` (or add a sibling) to also scan
`event-bus.jsonl` + `dispatch-log.jsonl` + `bug-billboard/inbox/` for unserviced items, emitting into the
same gap-surface the session-JSONL sweep already feeds.

**Leverage: MED.** Awaiting greenlight (surface-only by contract).
