---
title: "Session-request audit — 2026-06-07 (127 requests, thorough)"
kind: audit-finding
created: 2026-06-07
source: workflow wddppcvf8 (session-request-audit, 10 agents, full transcript)
status: active — punch-list being worked
---

# Session-request audit — 127 requests

**Histogram:** 108 DONE · 14 PARTIAL · 3 IN_FLIGHT · 1 USER_PENDING · **1 DROPPED**. Full result: workflow `wddppcvf8` output.

## The 1 genuine DROP
- **[39]** OpenClaw/Hermes app↔harness↔OS-layer spectrum placement — never captured. → needs the original context (grep transcript) before placing; do NOT fabricate.

## Claude-actionable (✅ worked / ⏳ remaining)
- ✅ **[124]** settings-layer de-shim — `settings.proposed.json` had 52 `EVIUM_` kill-switch names; now 0 (AOS_-only).
- ✅ **[126]** `revert`-ungate — was claimed-folded but absent; now in `permissions.allow`, removed from deny.
- ✅ **[92]** "no core-ops library" — AUDIT WAS WRONG; `work-tools/{core,cli,mcp-tools,rest}.mjs` + ops fns exist.
- ✅ **[127]** this audit doc (capture).
- ⏳ **[3]/[123]** fast-tier agnosticism FAIL ("line 1846") = the **stale worktree `agent-a265`** the path-test scans (live tree clean); fix = remove that worktree / exclude `.claude/worktrees/` from the scan.
- ⏳ **[116]/[118]** RH→goal-board pipeline: only the 2 point-fixes built; full data-model + AIS watchdog are design-only (build-gated on user sign-off, canonical surfaces).
- ⏳ **[17]** stale `reference_repo-geography.md` note (says captures NOT restored; they ARE) — memory reconcile (backup-first).
- ⏳ owner=claude cleanups (low-pri): [8] pins index drift · [48] BLS mobile plan stale · [56] research docs untracked · [24] spawn-tracking F4 · [16] sync-audit findings open · [38] handoff-hardening cadence · [112] 5-naming-schemes artifact unverifiable.

## Gated-on-USER (PARTIAL only because the gated step is yours)
- **THE ONE PASTE:** `settings.proposed.json` → `settings.json` — now fully assembled: AOS_ guards + result-loops + never-idle wire + revert-ungate. [11][121][126]
- **Daemon relabel:** `reports/plist-staging/APPLY-runbook.sh` (com.evium → com.agentic-os on the 6 live daemons) + `scripts/daemons/reload-all.sh`. [20][100][123]
- **AIS manifesto pillar** + the 30-row change-list across 9 canonical surfaces. [117][118]
- **research/external/ `domain:` retrofit** decision [63].

## The standing fix
The Part-5 alignment-immune-system (designed, not yet wired) AUTOMATES exactly this audit so it never needs asking again. The non-idle keystone (built) + this audit are the manual stopgap until that pillar is greenlit + wired.
