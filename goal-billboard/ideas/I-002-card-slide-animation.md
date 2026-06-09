---
idea_id: I-002
title: Card-slide animation when work flows between lifecycle lanes
lifecycle: captured
serves_northern_star: G2
impact_score: 4.5
source: research
created: 2026-05-31T00:00Z
updated: 2026-05-31T00:00Z
---

# I-002 — Card-slide animation between lifecycle lanes

When an item changes lifecycle state (an audit completes → moves IN-FLIGHT to
LANDED), have its card visibly *slide* to the new column on next render. Held out
of the core pass deliberately: it is only meaningful if the user is watching a
live re-render, otherwise it risks the autoplay anti-pattern (the
dashboard-centerpiece rule: never autoplay; motion must be meaningful, not
ambient). Needs a careful "diff the previous baked model and only animate genuine
transitions" design. Captured for review — not greenlit.
