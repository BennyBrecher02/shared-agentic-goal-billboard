---
idea_id: I-022
title: os-tools MCP repo — timing LATER; FLAG it may NOT need its own repo (can ride inside portable-kit)
lifecycle: captured        # captured = awaiting your review (the user-gate per the ideas-lane safety contract)
serves_northern_star: G21  # the os-tools MCP is the STATE-carrying surface of the portable agentic-OS
belongs_to_goal: G21       # repo-creation todo #3 of 3, tracked under the Northern Star
user_gated: true           # USER-ONLY: repo creation + first push are gated; AND the user decides IF a separate repo is even wanted
timing: LATER
needs_decision: true       # OPEN QUESTION — may NOT need its own repo (see flag below)
source: user-prompt        # the user-approved goal-grooming triage (2026-06-21) appended these 3 repo-creation todos
source_ref: context/markdowns/audits/goal-grooming-proposal-2026-06-21.md
created: 2026-06-21T21:01Z
updated: 2026-06-21T21:01Z
---

# I-022 — os-tools MCP repo (timing: LATER — and maybe never its own repo)

**What it is.** The os-tools MCP (thin-shell, 9 tools, stdio-local; designed in
`research/systems/os-tools-mcp-design-2026-06-21.md`, working server at `scripts/mcp/os_tools_server.py`) is
the STATE-carrying surface of the portable OS. This todo asks whether it should eventually get **its own repo**.

**⚠ FLAG — it may NOT need its own repo.** The MCP can comfortably **ride inside the portable-kit** (I-021):
it deploys with the kit, versions with the kit, one less repo to maintain. A **separate** repo only earns its
keep if the user wants the MCP **pip-installable standalone** (`pip install os-tools-mcp`) — i.e. distributed
and consumed independently of the kit, on its own release cadence, by people who don't take the whole OS. That
is the deciding question, and it is the **user's** call. Default assumption: it rides inside portable-kit
unless the user explicitly wants the standalone-installable package.

**Why LATER.** Lowest urgency of the three — the MCP is designed + has a working server, but nothing external
consumes it yet, and the "own repo?" question is downstream of the portable-kit extraction (I-021) even
landing. No action until the kit is real externally AND the pip-installable need is confirmed.

**Concrete first step (when revisited).** Decide the one question: **pip-installable standalone?** If YES →
its own repo with packaging (pyproject.toml, entrypoint, release flow). If NO → fold it into portable-kit and
close this todo. Do not create the repo speculatively.

**Timing: LATER.** And gate it on the standalone-installable decision before creating any repo at all.
