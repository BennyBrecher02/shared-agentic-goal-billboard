---
id: 20260609T074517Z-dashboard-wip-claude-main-bfe2aedf
discovered: 2026-06-09T07:45:17Z
agent: claude-main
worktree: dashboard-wip
route: /services, /multifamily, /multifamilyv2, /highway, /fleets, /products, /contact, /blog
section: global
viewport-bucket: all
severity: high
status: open
defect-type: infrastructure
dedupe-key: AgenticOS|stale-evium-routes|global|all|infrastructure
project: AgenticOS
---
# Playwright specs still target 8 dead Evium routes — the full matrix fails on stale test-data, not on real defects

## Description
The Playwright specs (`tests/visual.spec.ts`, `tests/responsive.spec.ts`, `tests/a11y.spec.ts`, and
`tests/rail.spec.ts`) iterate a 9-route list left over from the Evium site
(`/`, `/services`, `/multifamily`, `/multifamilyv2`, `/highway`, `/fleets`, `/products`, `/contact`, `/blog`).
But `src/` is now the **BLS test-data site** with only 3 pages: `/`, `/zmanim`, `/color-comparison`. Verified
live against the dev server on 43700: **8 of the 9 spec routes return 404** (`/services /multifamily
/multifamilyv2 /highway /fleets /products /contact /blog`), only `/` resolves. Every spec that does
`page.goto('<dead route>')` then asserts on the page therefore fails. This is the dominant failed-count in the
full matrix run and it is **stale-test-data drift from the Evium→BLS pivot — NOT a site regression.** This is
benchmark output: the specs need re-pointing at the 3 real BLS routes; the BLS `src/` is NEVER fixed here
(`feedback_bls-is-test-data`).

## Repro
- capture: full-matrix run 2026-06-09 (headless, 6.6min, 434 passed / 168 failed / 108 skipped); summary in context/markdowns/agent-recommendations/timing/matrix-cost.md
- the 168 failures break down EXACTLY as the stale-test-data thesis predicts: 108 visual-snapshot (9 routes × 12 projects — 8 routes 404, `/` diffs vs the old Evium baseline), 48 rail (4 rail tests × 12 projects — the HeaderBatteryLefter rail is an Evium component absent from BLS, tested on the 404 /contact), 12 responsive (the `h1 present+visible` check on `/` × 12 — a GENUINE BLS defect, filed separately to dev-handoff). 0 a11y failures.
- steps: 1) `curl -s -o /dev/null -w "%{http_code}" http://localhost:43700/services` → 404 (repeat for the 8 routes) 2) `bash scripts/matrix-cost-capture.sh --run` 3) observe the failed count concentrated on the 8 dead routes × {visual,responsive,a11y} × 12 projects + rail on /contact.

## Files
- tests/visual.spec.ts:14-24 (the 9-route ROUTES array — 8 are dead)
- tests/responsive.spec.ts:14-24 (same stale ROUTES array)
- tests/a11y.spec.ts:20-30 (same stale ROUTES array)
- tests/rail.spec.ts (RAIL_TEST_ROUTE = '/contact' — 404 on BLS)

## Fix attempted
(none — routing to handoff; src/ + the BLS test-data specs are NOT fixed in this repo)

## Verification
(empty — the fix is a Cursor-side spec re-point against the 3 real BLS routes; re-run the matrix there)
