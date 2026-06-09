---
idea_id: I-008
title: REST adapter for the work-substrate (capture from any device — phone/remote)
lifecycle: greenlit
serves_northern_star: G2
source: user-prompt
created: 2026-06-03
updated: 2026-06-03
priority: HIGH — today, after the core foundational work
---

# I-008 — REST adapter for the work-substrate (HIGH urgency, today)

**User-greenlit 2026-06-03:** build the REST interface — the 3rd of the CLI/MCP/REST "big 3" — as a **higher-urgency
todo to tackle today**, but **after the important foundational work first** (1b memory-split → 2a core ops lib →
2b CLI → 2c MCP). REST is the adapter that makes "from anywhere" literal: capture an idea / research / bug from a
**phone or another machine** over HTTP, not just this laptop.

- Build as a **thin adapter over the same core ops library** (don't duplicate logic).
- **Gated:** bind to localhost/tailnet + require a token — never expose unauthenticated (per the research).
- Tracked in the milestone plan as **Phase 2d, un-deferred → today-priority.**
