---
idea_id: I-004
title: Audit status-drift sweep (the 54-in-progress WIP signal)
lifecycle: captured
serves_northern_star: G3
impact_score: 6.5
source: research
created: 2026-05-31T00:00Z
updated: 2026-05-31T00:00Z
---

# I-004 — Audit status-drift sweep

The Board's lifecycle-lane WIP counts make a latent problem undeniable: ~54 of
~85 audits sit at `status: in_progress` with inconsistent vocabulary (`complete`
vs `verified_complete` vs `phase_1_landed` vs `phase2_patches_applied`). Half are
likely done-but-never-moved or abandoned-but-never-closed. This idea: normalize
the audit `status:` field to the canonical vocabulary
(`available | in-progress | completed | cancelled`) and move the stale ones, so
the WIP count reads calmly-accurate. A catalog-hygiene pass. Captured for review
— not greenlit.
