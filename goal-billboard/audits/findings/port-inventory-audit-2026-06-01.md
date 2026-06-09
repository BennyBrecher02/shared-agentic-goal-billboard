---
title: Port-number inventory audit — every port the agentic-OS uses, where it's set, collision risk
audit_id: port-inventory
date: 2026-06-01T16:23Z
method: read-only (grep/read; nothing changed)
serves_northern_star: G2
belongs_to_goal: G4
purpose: >
  Inventory feeding a port-reassignment synthesis. User wants to move off common
  AI-default ports (4321/3000/8080/8000/5173) onto unique, collision-resistant ports.
sources:
  - scripts/services-registry.json (declared single-source-of-truth for service ports)
  - scripts/project-config.json / project-config.sh / project_config.py (DEV_SERVER_PORT)
  - playwright.config.ts, astro.config.mjs, package.json
  - scripts/serve-dashboard.sh, scripts/agent-inbox-server.py, scripts/wake-restart.sh
  - scripts/hooks/inbox-server-guard.sh, scripts/hooks/test-e2e-dev-server-check.sh
  - scripts/scheduler/{__main__.py,scheduler.py,shards/*.py}
  - scripts/dashboard/{generate_dashboard.py,rerender_git_pairs.py,rerender_listA_extend.py}
  - scripts/{run-deep-audit.sh,mobile-sim-sweep-orchestrator.sh,ios-simulator-capture.sh,android-emulator-capture.sh}
  - scripts/cluster/{readiness-check.sh,inventory.sh}
  - .claude/launch.json, .claude/workflows/bracketed-research-audit.js
  - context/markdowns/required-services.json (legacy fallback registry)
  - tests/inbox-server-resilience-test.sh, tests/inbox-server-wrong-port-test.sh
---

# Port-number inventory audit — 2026-06-01

Read-only inventory of every port the system uses, where each is defined, whether it is
config-derived or hardcoded, and its collision risk. This is the substrate a clean
port-reassignment will work from.

## 0. Executive summary

- **9 distinct fixed ports** are referenced in code (one of which — **4322** — is a
  DEAD legacy port that lives only in guard/scan/comment code, never bound on purpose).
  **8 are live/intended.**
- **2 are HIGH collision-risk** common AI-defaults: **4321** (Astro dev — Astro's
  framework default, the exact port the user flagged) and **8000** is NOT used, but
  **8787 / 8799 / 8801** sit in the crowded 8xxx band (Wrangler/Cloudflare default is
  8787 — MED-HIGH). No 3000 / 5173 / 8080 / 9229 in code (good — only `9000` appears
  once as a usage-example in a comment, not bound).
- **Centralization is PARTIAL and split-brain.** There are **two** "single source of
  truth" claims: `services-registry.json` (4321 / 4323 / 4399 / 8787) and
  `project-config.json` `DEV_SERVER_PORT` (4321). They **agree today** but **duplicate
  4321**. Beyond those, **4399 is hardcoded as a string literal in ~6+ places**, the
  scheduler pool **4400–4409** is hardcoded in `__main__.py`, and the dashboard
  launcher ports (**8787 / 8799 / 8801 / 4477**) are scattered hardcoded literals with
  NO central definition. A clean reassignment must collapse these.

## 1. Port inventory table

| Port | Service / purpose | Defined at (file:line) | Hardcoded vs config-derived | Collision risk |
|---|---|---|---|---|
| **4321** | Astro dev server (interactive site + dashboard preview); also the CORS `ORIGIN` for the inbox-server and the dashboard's `INBOX_API` host | **CONFIG anchor:** `scripts/project-config.json:19` (`DEV_SERVER_PORT`), mirrored `scripts/project-config.sh:84`, `scripts/project_config.py:46`; **ALSO** `scripts/services-registry.json:25` (`astro-dev.port`); **HARDCODED literals** at `scripts/agent-inbox-server.py:76` (`ORIGIN="http://localhost:4321"`), `scripts/dashboard/generate_dashboard.py` (`INBOX_API` host, CORS expectation), `scripts/hooks/test-e2e-dev-server-check.sh:37`, `scripts/cluster/readiness-check.sh:86`, `scripts/mobile-sim-sweep-orchestrator.sh:129`, `scripts/capture-baseline.mjs:38`, `context/markdowns/required-services.json:17` | **MIXED** — config-derived for the dev server itself, but **duplicated as a hardcoded literal in ≥7 other files** (CORS origin, e2e check, cluster probe, capture defaults). `DEV_SERVER_PORT` is NOT actually threaded into most consumers. | **HIGH** — Astro's framework default port; the canonical "AI-default" the user wants to move off. Anything else starting Astro on this machine collides. |
| **4322** | **DEAD legacy** inbox-server port (pre-A45). Never bound on purpose; survives only as the "wrong port" the guard/scan/tests must DETECT and the "never use this" comment. | `scripts/wake-restart.sh:27` (zombie-kill scan), `scripts/hooks/inbox-server-guard.sh:212` (`WRONGPORT_SCAN`), `scripts/agent-inbox-server.py:54` (comment), `scripts/dashboard/generate_dashboard.py:3383` (comment "never 4322") | Hardcoded literal (intentionally, as a negative sentinel) | **LOW** (not bound). NOTE for reassignment: if 4323 moves, the dead-4322 scan/comments should move too, or they become stale noise. |
| **4323** | **agent-inbox-server** (the P0 agent channel — user→agent click delivery). HTTP server, `serve_forever`. | **CONFIG anchor:** `scripts/services-registry.json:13` (`agent-inbox-server.port`, declared canonical). Read via env→registry→literal in `scripts/agent-inbox-server.py:57` (`CANONICAL_INBOX_PORT=4323` floor). **HARDCODED literals** at `scripts/wake-restart.sh:38`, `scripts/hooks/inbox-server-guard.sh:161`, `scripts/dashboard/generate_dashboard.py:3383` (`INBOX_API`), `scripts/hooks/inbox-on-prompt.sh:325`, `.claude/workflows/bracketed-research-audit.js`, `context/markdowns/required-services.json:7` | **MIXED (best-centralized of the set):** `agent-inbox-server.py` genuinely reads the registry (env → registry → 4323 floor) so the *server* can't drift. BUT the dashboard `INBOX_API`, the guard default, wake-restart, and the JS workflow each **hardcode 4323** rather than reading the registry — the "port drift impossible" claim only holds for the server binary, not its clients. | **MED** — 4323 is not a famous default, but it's adjacent to 4321/4322 and lives in the crowded 43xx band. |
| **4399** | **e2e-playwright** ephemeral dev server (Playwright `webServer`, runs only during a test run, deliberately separate from 4321). | `playwright.config.ts:35` (`webServer.command: 'npm run dev -- --port 4399'` + `baseURL`/`url` `http://localhost:4399`); `scripts/services-registry.json:43`; **HARDCODED string** `"4399"` in `scripts/scheduler/scheduler.py:748`, `scripts/scheduler/shards/playwright_project.py:37`, `shards/ios_simulator.py:60`, `shards/android_emulator.py:75`; plus `scripts/run-deep-audit.sh:114-115`, `scripts/ios-simulator-capture.sh:31`, `scripts/android-emulator-capture.sh:43`, `scripts/mobile-sim-sweep-orchestrator.sh` | **HARDCODED everywhere.** The scheduler comment is explicit: "Hardcode port 4399 — playwright.config.ts webServer.url is hardcoded to 4399." It's NOT in `project-config.json`. The scheduler `DevServerPorts` pool exists but is bypassed — code always USES 4399 and only acquires the pool to track ownership. | **MED** — uncommon literal, but the most-duplicated hardcoded port in the repo (≥8 files), so the highest *centralization debt*. |
| **4400–4409** | Scheduler `DevServerPorts` resource Pool (reserved for future multi-tick parallelism; currently acquired-but-unused — code always binds 4399). Memory/docs describe the intended pool as **4400–4499**. | `scripts/scheduler/__main__.py:118` (`Pool(name="DevServerPorts", items=[str(p) for p in range(4400, 4410)])`); tests use `"4400"`/`"4401"`/`"4402"` literally; docs say 4400–4499 (`.claude/memory-mirror/feedback_multi-change-scheduler.md:29`, `feedback_script-design-fundamentals.md:20`) | **HARDCODED** — the `range(4400, 4410)` literal is the only definition; not config-derived. **Doc/code mismatch:** docs say 4400–4499, code allocates 4400–4409. | **MED** — uncommon band, but the doc-vs-code range mismatch is a latent footgun for reassignment. |
| **4477** | Private preview port for historical-build re-renders (git before/after pair generator). | `scripts/dashboard/rerender_git_pairs.py:49` (`PORT = 4477`); `scripts/dashboard/rerender_listA_extend.py:58,182` (`lsof -ti:4477` kill + `H.PORT`) | **HARDCODED literal**, single-purpose. Not in any registry. | **LOW** — obscure dedicated port; only used during ad-hoc re-render runs. |
| **8787** | **dashboard-launcher** — static `python3 -m http.server` serving `public/` (operational dashboards). | `scripts/serve-dashboard.sh:25` (`PORT=8787`, `--port` overridable, default 8787); `scripts/services-registry.json:34` (`dashboard-launcher.port`) | **MIXED** — registry declares it AND the launcher hardcodes the same default independently (the launcher does NOT read the registry; it just happens to match). Overridable via `--port`. | **MED-HIGH** — **8787 is the Cloudflare Wrangler default**; a common dev tool. Worth moving off. |
| **8799** | `dashboard-static` IDE/launch profile — `python3 -m http.server 8799 --directory public`. | `.claude/launch.json:13-14` | **HARDCODED** in the IDE launch config; no registry entry; diverges from 8787 (the actual launcher default) — a 3rd dashboard-serving port. | **MED** — crowded 8xxx band; and it's a *third* way to serve the dashboard (8787 vs 8799 vs 8801) → confusion risk. |
| **8801** | `dashboard-root` IDE/launch profile — `python3 -m http.server 8801` (serves repo root). | `.claude/launch.json:19-20` | **HARDCODED** in the IDE launch config; no registry entry. | **MED** — crowded 8xxx band; redundant 4th dashboard-server variant. |
| _(9000)_ | _Not bound._ Appears once as a `--port 9000` **usage-example in a comment** in `scripts/serve-dashboard.sh:16`. Listed only so the reassignment doesn't think it's free-and-clear / doesn't trip over the example. | `scripts/serve-dashboard.sh:16` (comment) | n/a (documentation) | **n/a** |
| _(dynamic)_ | Test sandboxes (`inbox-server-resilience-test.sh`, `inbox-server-wrong-port-test.sh`) allocate ports at runtime via a `free_port()` helper (`SANDBOX_PORT`, `SQUAT_PORT`, `CANON_PORT`, `WRONG_PORT`, `FOUROHFOUR_PORT`). | `tests/inbox-server-resilience-test.sh:95-98`, `tests/inbox-server-wrong-port-test.sh:54-58` | **Dynamic / ephemeral** — no fixed port; OS-assigned. | **LOW** — no fixed binding; correctly avoids collisions by design. Good pattern to emulate. |

**Distinct FIXED ports = 9** (4321, 4322-dead, 4323, 4399, the 4400–4409 pool, 4477, 8787, 8799, 8801). **Live/intended = 8** (excluding dead-4322). Counting the pool as one logical allocation. (9000 is example-only; test ports are dynamic.)

## 2. Central-config-vs-scattered assessment

**Verdict: PARTIALLY centralized, with a split-brain and a long hardcoded tail.**

There are **two competing "single source of truth" registries**, plus a legacy third:

1. **`scripts/services-registry.json`** — schema v2, self-described as the SINGLE SOURCE
   OF TRUTH for *service* ports. Declares **4321 / 4323 / 4399 / 8787** (+ a `scheduler:
   null` placeholder). `scripts/services-check.sh` reads it AND has real **config-level
   port-collision detection** (`jq` group_by(port) → exit 4 if two services share a
   port). This is the strongest piece of the system.
2. **`scripts/project-config.json` `DEV_SERVER_PORT: 4321`** — the non-derivable
   identity, read by `project-config.sh` + `project_config.py`. **Duplicates 4321**
   (the registry's `astro-dev.port`). Its own doc even says the inbox port is
   "intentionally not duplicated here" and lives in the registry — so the split is known
   but 4321 itself is still in both.
3. **`context/markdowns/required-services.json`** — A45-era legacy registry (4321 /
   4323), explicitly "superseded" by services-registry.json but "kept as a fallback."
   A third place 4321/4323 appear.

**Scattered hardcoded offenders (what a clean reassignment MUST centralize):**

- **4399 (e2e) — the worst offender.** Hardcoded as a literal in ≥8 files and NOT in any
  registry/config: `playwright.config.ts` (the true anchor), `scheduler.py:748`,
  `shards/playwright_project.py:37`, `shards/ios_simulator.py:60`,
  `shards/android_emulator.py:75`, `run-deep-audit.sh`, `ios-simulator-capture.sh`,
  `android-emulator-capture.sh`, `mobile-sim-sweep-orchestrator.sh`. Moving 4399 = touch
  all of these by hand.
- **4321 CORS/client coupling.** Even though 4321 is config-anchored, its *consumers*
  hardcode it: `agent-inbox-server.py:76` (`ORIGIN`), `generate_dashboard.py`
  (`INBOX_API` host + CORS), `test-e2e-dev-server-check.sh:37`,
  `cluster/readiness-check.sh:86`, `capture-baseline.mjs:38`. `DEV_SERVER_PORT` is the
  named anchor but is **not actually threaded** into these.
- **4323 clients.** Server reads the registry; clients (`generate_dashboard.py`
  `INBOX_API`, `inbox-server-guard.sh:161`, `wake-restart.sh:38`,
  `inbox-on-prompt.sh:325`, `bracketed-research-audit.js`) hardcode 4323.
- **8787 / 8799 / 8801 (dashboard-serving) — no central home at all.** 8787 in the
  launcher default (matches registry by coincidence, not by reading it); 8799 + 8801 in
  `.claude/launch.json` with zero registry linkage. Three+ ways to serve the same
  dashboard, on three different ports.
- **4400–4409 scheduler pool** — hardcoded `range(4400, 4410)` in `__main__.py`; docs
  say 4400–4499 (mismatch).
- **4477** — hardcoded single-purpose literal in two dashboard re-render scripts.

**Bottom line:** the *service* layer (4321/4323/4399/8787) has a real registry +
collision-check, but (a) it's duplicated against project-config (4321) and a legacy file,
and (b) the registry is read by very few consumers — most code hardcodes the same numbers
inline. The dashboard-serving ports and the scheduler pool have no central definition.

## 3. Reassignment-readiness note (centralization debt = # files to touch per port)

| Port | # files to change to move it | Centralization debt |
|---|---|---|
| **4399 (e2e)** | **~9** (playwright.config.ts + 4 scheduler/shard files + 4 capture/audit scripts) | **HIGHEST.** Not in any config. Pure hardcoded sprawl. Pre-req: introduce an `E2E_PORT` config key + thread it everywhere (or have shards read playwright.config). |
| **4321 (astro-dev)** | **~8** (project-config trio acts as 1 anchor + services-registry + required-services + ≥5 hardcoded consumers: CORS origin, dashboard INBOX_API/CORS, e2e-check hook, cluster probe, capture-baseline) | **HIGH.** Anchor exists (`DEV_SERVER_PORT`) but isn't threaded; consumers hardcode. Also must update Astro's own default expectation. |
| **4323 (inbox)** | **~6** (registry = anchor + 5 hardcoded clients: dashboard, guard, wake-restart, inbox-on-prompt, JS workflow) + move the dead-4322 scan/comments | **MED-HIGH.** Server is clean (reads registry); clients are not. |
| **8787 (dashboard-launcher)** | **~2** (serve-dashboard.sh default + services-registry) — but reconcile with 8799/8801 | **MED.** Small file count, but tangled with the 8799/8801 duplicates. |
| **8799 / 8801 (launch.json)** | **1 each** (`.claude/launch.json`) — NOTE: editing `.claude/` may trip the settings-edit-guard class; launch.json is separate from settings.json but confirm | **LOW count, but** these are orphan duplicates; the real fix is *consolidating* the 3 dashboard ports into 1, not just renumbering. |
| **4400–4409 (scheduler pool)** | **~2** (`__main__.py` range literal + the tests' "4400" literals) + fix the 4400–4499-vs-4409 doc mismatch | **MED.** Single code definition; reconcile docs. |
| **4477 (git-rerender preview)** | **~2** (`rerender_git_pairs.py` + `rerender_listA_extend.py`) | **LOW.** |
| **4322 (dead)** | **~4** scan/comment sites — only matters as cleanup when 4323 moves | **LOW.** |

**Recommended reassignment shape (for the synthesis to build on, NOT applied here):**
collapse to **ONE registry** (`services-registry.json`) holding *every* fixed port
(add e2e-port 4399, the scheduler pool base, the dashboard ports, the rerender port);
make `project-config.json` `DEV_SERVER_PORT` *read from* the registry (or delete the dup);
thread the registry value into the hardcoded consumers (CORS origin, INBOX_API,
playwright.config, shards, capture scripts) instead of inline literals; consolidate the
3 dashboard-serving ports (8787/8799/8801) into one; and pick **uncommon, clustered,
non-AI-default** numbers (e.g. a private 49xxx or a contiguous unusual block) so the
existing `services-check.sh` collision-detector covers the whole set. The dynamic
`free_port()` test pattern is the model for anything that doesn't need a fixed port.

---

_Read-only audit. No ports, config, or files were changed (this findings doc only)._
