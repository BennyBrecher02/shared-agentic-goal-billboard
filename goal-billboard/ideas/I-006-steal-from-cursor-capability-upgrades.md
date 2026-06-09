---
idea_id: I-006
title: Steal-from-Cursor capability upgrades (semantic-index MCP · glob-scoped rule auto-attach · post-edit ripple check)
lifecycle: captured
serves_northern_star: G2   # capability wins (per RH-020 part-3); advisory
source: rabbit-hole        # RH-020 part-3
created: 2026-06-03T05:00Z
updated: 2026-06-03T05:00Z
---

# I-006 — Steal-from-Cursor capability upgrades (backlog / icebox)

Three Cursor-derived upgrades to fold into OUR Claude-Code substrate (no Cursor dependency) — captured as backlog so they're not forgotten. Benny 2026-06-03: *"too big a leap for where we are, keep this in our backlog so we dont forget it"* → **NOT greenlit, NOT scheduled.** Source: `research/rabbit-holes/RH-020-cursor-integration/part-3-cursor-does-better-slam-dunks.md`.

1. **Local semantic code-index MCP** (the slam-dunk, ~1hr spike) — adopt an existing local-only MCP server (e.g. `claude-context-local`) giving agents `@codebase` semantic retrieval + an auto-fresh index. Highest leverage of the three; it's ADOPT-an-existing-tool, not build-from-scratch.
2. **Glob-scoped rule auto-attach** (BUILD, medium effort) — a PreToolUse hook + rule frontmatter so the right convention fires automatically when editing matching files (e.g. CSS conventions when touching `src/styles/*.css`). Mirrors Cursor's `.mdc` glob-scoped rules.
3. **Post-edit ripple-reference checklist** (CONCEPT-steal, lower priority) — a post-edit hook that surfaces sibling references to something just changed (Cursor's "next-edit" ripple), so cross-file follow-ups aren't missed.

**Why backlog, not now:** system-capability improvements, separate from the agent-benchmark handoff. Greenlight only when capacity frees + after the more pressing OS/site work. Do NOT auto-start (the ideas lane is surface-only by contract).
