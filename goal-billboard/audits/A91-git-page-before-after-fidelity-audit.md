---
id: A91
title: Git/rollback page — before/after picture-vs-description FIDELITY audit
type: audit
status: findings-complete
created: 2026-05-31
scope: read-only
relates_to: [A87, A88, A90]
files_audited:
  - scripts/dashboard/build_render_model.py   # READ-ONLY (another BG editing concurrently)
  - scripts/dashboard/generate_dashboard.py   # READ-ONLY (another BG editing concurrently)
  - reports/git-rerender/index.json
  - reports/dashboard-build/render-model.json
severity: HIGH (trust-breaking — the page's own evidence contradicts its descriptions)
---

# A91 — Git/rollback page before/after FIDELITY audit

> **Charter.** The git/rollback page leads each commit card with a plain-language
> description ("what changed") + a before/after picture. The user reports the pictures
> and descriptions are **misleading**: multi-route commits show only ONE route; the
> visible delta doesn't match the description; approximate pairs may show capture-noise.
> This audit lines up, for every visible commit, three things — **(a)** the authored card
> copy (`_COMMIT_COPY`), **(b)** the ACTUAL diff (`git show`), **(c)** the before/after the
> page currently shows (from the live `render-model.json`) — and flags every mismatch.
>
> **Read-only.** No edits to `build_render_model.py` / `generate_dashboard.py` (a sibling
> BG is editing them live), no src/settings/git mutations. Recommendations only.

---

## 0. TL;DR — the three confirmed defects are REAL and STRUCTURAL

| # | Defect | Verdict | Root cause |
|---|--------|---------|------------|
| **1** | Multi-route commits show only ONE route | **CONFIRMED** | The model bakes a **single** `c["pair"]` (one route×device×section); the renderer (`beforeAfterViewerHTML`) reads `c.pair` as one object — there is **no gallery / route-tab loop over `c.visible_routes`**. 4 commits hide 1-3 routes each. |
| **2** | Visible delta ≠ description | **CONFIRMED (3 commits)** | `_COMMIT_COPY` was authored from *intent*, not from the *diff*. Brightness/overlay/scrim/uppercase effects present in the diff are omitted or mis-framed. `3d56df0`'s copy is for a different change than the bulk of its diff. |
| **3** | Approximate pairs show capture-noise | **PARTLY MITIGATED, gap remains** | A91-in-code already replaces the WORST approximate pairs (`unreliable` flag → code-only card; 9 commits). But **2 truly-approximate pairs still render as-is, flagged "reliable"** (`a553e1a`, `cc5dc7c` — both `approximate=True`, other src-commits sit between their brackets) plus **1 existing-run-but-not-flagged-approximate pair** (`02359cd`); they pass the >3 perceptual-diff gate so the shown delta is real but **not provably this commit's change**. |

**The page surfaces 21 visible commits** (yes/subtle/floor) + 7 invisible (no picture, correctly). Of the 21 visible: **8 show an EXACT re-rendered pair** (reliable), **3 show an existing-run pair as-is** (2 of them flagged `approximate=True`: `a553e1a`, `cc5dc7c`; 1 not: `02359cd`), **9 have their pair REPLACED by an honest code-only card** (the A91-in-code mitigation, working), **1 is the floor** (after-only baseline).

The system is **already half-fixed**: the `_detect_unreliable_pair` + code-only-card machinery (tagged "A91" in the source) catches the no-delta and placeholder-mismatch failure modes. The **remaining** trust gap is (1) the missing multi-route gallery, (2) ~5 descriptions that undersell the diff, and (3) the 3 approximate pairs shown without an "approximate" caveat strong enough to stop a wrong read.

---

## 1. The three flagged commits (the user's exhibits) — dissected

### 1.1 `1158d35` — "Added a tablet-size version of the Products hero overlay"
- **Diff (truth):** adds `@media (min-width:641px) and (max-width:1024px) { .prod-hero .bg-tint { background: linear-gradient(180deg, rgba(12,28,37,0.50)…0.88), radial-gradient(…0.34) } }`. `.bg-tint` is a near-black navy (`rgba(12,28,37,…)`) **darkening overlay**. One file, `src/styles/app.css`, +19 lines. Scope: `/products` §01 hero, tablet portrait only.
- **Picture shown:** `products / webkit-ipad`, **method=rerender, reliable** (an EXACT isolated parent→commit re-render).
- **Is the brightness delta the real @media overlay or noise?** → **The REAL overlay.** It's an exact re-render of the parent vs this commit on `webkit-ipad` (an in-tablet-range viewport), so the photo-darkening the slider reveals IS exactly what this @media block does. **Not noise.**
- **Is "overlay" honest for a darkening?** → **Technically yes, but the copy under-discloses.** The change literally adds a dark-tint overlay, so "overlay" is not wrong. But the *visible effect* is "the tablet hero photo gets noticeably darker," and the card never says "darker." Unlike its siblings `3889ae6` and `a553e1a` (which carry a `choosing:` note "it does make the photo darker — worth a glance"), `1158d35` has **no such note**. A reviewer sees a darkening they weren't told to expect → exactly the "did it do MORE than the card says?" fear.
- **Verdict:** picture-fidelity **GOOD** (exact, right route). Description-fidelity **WEAK** — omits the darkening side-effect that the (correct) picture plainly shows.

### 1.2 `382886e` — the caption-collision commit ("Un-stacked overlapping caption text on Fleets + Products")
- **Diff (truth):** edits the **shared** `.cta-split .shot .caption` rule (used by BOTH `/fleets §04` and `/products §07`): adds `z-index:2`, `display:flex; flex-direction:column; gap:10px`; on the eyebrow `.k` adds `text-transform:uppercase` + `line-height:1.2` + `margin:0`; on `.t` sets `line-height:1.3; margin:0`. `src/styles/app.css`, +13/-3.
- **`_A88_VISIBLE` routes:** `["fleets","products"]` — correctly multi-route.
- **Picture shown:** `fleets / chromium-desktop` §4, **method=rerender, reliable.** Exact and correct — for **fleets only**.
- **Coverage gap:** **`/products §07` is HIDDEN.** The identical shared-component fix on the Products lineup section is never shown — the user cannot review the change on the second of the two routes the card itself names. **This is symptom #1 in its purest form.**
- **Description side-effect omission:** the card says "they stack cleanly with spacing" but does not mention the eyebrow now renders **UPPERCASE** (`text-transform:uppercase` is a genuine new visible change). Minor, but it's an undocumented visual effect.
- **Verdict:** picture-fidelity **PARTIAL** (correct route shown, second route hidden). Description-fidelity **MOSTLY GOOD** (routes named) but omits the uppercase change.

### 1.3 `a553e1a` — "Deepened the dark overlay behind three hero headlines" (products + highway + services)
- **Diff (truth):** three distinct effects, `src/styles/app.css` +55 and `src/pages/services.astro` +1:
  1. **Products** `.prod-hero .bg-tint`: 96deg gradient stops moved/deepened — `0.72@28% → 0.74@34%` and notably **`0.18@55% → 0.32@60%`** (a real darkening reaching further into the headline column).
  2. **Highway** — a **brand-new** `.highway-hero .bg-tint` dark gradient (`0.42→0.22→0.70@65%→0.92@100%` + radial). Real darkening.
  3. **Services** — a **brand-new** `.services-hero .bg-tint` (`0.40→0.20→0.65@60%→0.93@100%`) **AND** a brightened scroll-hint comet (`.services-hero .hero-scroll .scroll-hint-comet` gradient lifted). `services.astro` gains `extraClass="services-hero"` so the selector resolves.
- **`_A88_VISIBLE` routes:** `["products","highway","services"]` — correctly multi-route.
- **Picture shown:** `products / chromium-desktop`, **method=existing-run, approximate=Y, flagged "reliable" (shown as-is).**
- **Does the shown pair actually show the scrim change?** → **Cannot be trusted to.** It is NOT one of the 8 exact re-renders. Its re-render directory on disk holds only `chromium-desktop/contact.jpg` (a **wrong-route** capture — contact, not products), so the build fell back to an **existing-run bracket**. The "reliable" flag means it merely cleared the `>3` mean-per-channel perceptual-diff gate — but `attach_visible_and_pairs` confirms **other src-touching commits sit between the bracketing runs**, so the products delta on screen may include OTHER changes, or may not isolate the scrim deepening at all.
- **Coverage gap:** **highway + services are HIDDEN** — the two routes where this commit added *entirely new* scrim rules (the most consequential part of the change) are invisible. Plus the **services scroll-hint comet brightening** is undocumented in the card.
- **Verdict:** the single worst card on the page. Picture-fidelity **POOR** (products-only, approximate, possibly contaminated, two new-rule routes hidden). Description-fidelity **GOOD on routes/intent** but **omits the comet-brightening secondary effect.** This is the exact "did the change do MORE, and WHERE ELSE?" alarm — and here the answer is yes (comet) and two hidden routes.

---

## 2. PER-COMMIT FIDELITY TABLE (all 28 commits)

Legend — **Flag**: `OK` clean · `UNDERSELL` copy omits a real visual effect · `HIDDEN-ROUTES` multi-route, only one shown · `APPROX-ASIS` approximate pair shown without a strong caveat · `MISLABEL` copy describes a different change than the diff · `MITIGATED` an unreliable pair correctly replaced by the code-only card.

| sha | What the diff REALLY did (all routes + all visual effects) | What the card CLAIMS | What the PICTURE shows now | Flag |
|-----|-----------------------------------------------------------|----------------------|----------------------------|------|
| `b77620d` | v1 of the whole site. | "Built the entire first version." | Floor: after-only baseline. | **OK** |
| `02359cd` | Blog → Content-Collection rewrite (547-line blog.astro), +12 article .md, +multifamilyv2 page, HeaderBatteryLefter/BaseLayout/fullpage.ts touches, image re-folder. | "Rebuilt the blog + added an experimental page… minor touches site-wide." | `blog / chromium-desktop`, **existing-run, `approximate=False`, shown as-is** (before_lag ~20h / after_lag ~12h, no intervening src commit). | **APPROX-ASIS (mild)** (existing-run not exact, and a 547-line blog rewrite is too large to verify from one masthead frame — but no intervening commit, so lower-risk than `a553e1a`/`cc5dc7c`) |
| `3d56df0` | **640-file** "snapshot 48h sprint." src slice touches blog (equal-height cards — matches copy), **highway.astro +129**, contact.astro, Hero.astro, products.astro, fleets.astro, app.css +183. Bulk is non-site infra (memory/hooks/audits). | "Made the blog cards line up to equal height + aligned the scroll hint." | `blog / webkit-ipad`, existing-run, **flagged no-delta → REPLACED by code-only card.** | **MISLABEL + HIDDEN-ROUTES** (copy covers a sliver; contact/highway/products visual changes unmentioned & unshown) |
| `3889ae6` | (1) `.prod-hero .bg-tint` darken + object-position shift; (2) `.form-wrap` align + aside padding (contact); **(3) `.sec-foot` α 0.4→0.6 — comment says it anchors counters across /products /highway /services.** | "Products hero and the Contact form, on desktop." | `products / chromium-desktop`, **rerender, reliable** (exact). | **UNDERSELL + HIDDEN-ROUTES** (copy omits the section-footer contrast lift on highway+services; pair is products-only) |
| `c9f52fe` | Nav: comment out Multifamily-v2 item; "Highway / NEVI" → "Highway". `HeaderBatteryLefter.astro`. | "Removed Multifamily v2 from menu; shortened Highway label." | `home / chromium-desktop`, existing-run, **no-delta → code-only card.** | **OK** (copy accurate; no-delta correctly mitigated — header crop not in the masthead frame) |
| `1da5463` | `.econ-model span em` α 0.4→0.6 (highway) + `.sec-foot.dark` α 0.5→0.75 (global footer labels). | "Darkened two bits of faint grey text: footer labels + Highway econ fine print." | `highway / chromium-desktop`, existing-run, **no-delta → code-only card.** | **OK** (copy exact; subtle contrast correctly mitigated) |
| `420b377` | Code comment only (pin multifamily internal). | "Recorded the decision… only a note in code." | None (invisible) — comment card. | **OK** |
| `acb5648` | `.tellus` cards: green hairline `border-top`, green wash gradient (top 60%), `saturate` filter muting 2 product photos. Highway §04. | "Green-tinted, muted-photo treatment on Highway Tellus cards." | `highway / chromium-desktop` §4, **rerender, reliable** (exact). | **OK** (accurate + exact + right route) |
| `767e019` | Refactor: extract reusable form field. No visual change. | "Reorganised code… looks the same." | None (invisible) — refactor card. | **OK** |
| `7a96bee` | `.text-on-dark-mute` α 0.5→0.6 + `--on-surface-variant` darkened (AA). | "Muted text on dark a touch stronger… measured a11y bump." | `home / chromium-desktop`, existing-run, **no-delta → code-only card.** | **OK** (exact; correctly mitigated) |
| `2df2577` | `BaseLayout.astro` font-load change (FOUT). | "Fonts load faster; brief fallback on cold load." | `home / chromium-desktop`, existing-run, **no-delta → code-only card.** | **OK** (FOUT correctly mitigated — rest-state identical) |
| `08c4c5e` | New `blog/[slug].astro` dynamic route (+418) — 17 posts stop 404ing. | "Built article-page template so all 17 posts open." | `blog / chromium-desktop`, existing-run, **placeholder-mismatch → code-only card.** | **MITIGATED** (after-only by nature; placeholder-mismatch correctly replaced) |
| `c9a20d2` | Rewire 6 dead blog links → `/blog/${id}`. | "Pointed six dead links to real articles." | `blog / chromium-desktop`, **placeholder-mismatch → code-only card.** | **MITIGATED** (behavioral; correctly replaced) |
| `a553e1a` | **Products scrim deepened + NEW highway scrim + NEW services scrim + services comet brightened.** (see §1.3) | "Deepened the dark overlay behind hero text on Products, Highway, Services." | `products / chromium-desktop`, **existing-run, approx, shown as-is.** | **HIDDEN-ROUTES + APPROX-ASIS + UNDERSELL** (highway/services hidden; products pair approximate & possibly contaminated; comet omitted) |
| `ef6faec` | Contact route cards: border `0.1→0.18`, white fill, hover lift+shadow, `.go` underline-on-hover. §02. | "Made Contact route cards clearly clickable." | `contact / chromium-desktop` §2, **rerender, reliable** (exact). | **OK** (accurate + exact + right route) |
| `cda8348` | Contact install photo → picture+srcset + reserved space (CLS). 131-file image commit. | "Lighter formats + reserved space; same photo." | `contact / chromium-desktop`, existing-run, **no-delta → code-only card.** | **OK** (CLS; correctly mitigated) |
| `cc5dc7c` | 17 Products card images → picture+srcset (~85% byte cut), CLS. `products.astro` +115. | "Lighter formats + reserved space for 17 product images." | `products / chromium-desktop`, **existing-run, approx, shown as-is.** | **APPROX-ASIS** (perf/CLS; copy is honest but the as-is approximate pair on a busy route risks showing unrelated delta — should be a CLS/no-delta-style note) |
| `dac177e` | 5 blog images → picture+srcset, CLS. | "Lighter formats for 5 blog images." | `blog / chromium-desktop`, **placeholder-mismatch → code-only card.** | **MITIGATED** |
| `6a36736` | Generated blog image variants + build-list entries. No page change. | "Generated lighter blog files… companion." | None (invisible) — infra card. | **OK** |
| `382886e` | Shared `.cta-split .shot .caption` flex/z-index/uppercase fix — **fleets §04 + products §07.** (see §1.2) | "Un-stacked overlapping captions on Fleets §4 + Products §7." | `fleets / chromium-desktop` §4, **rerender, reliable** (exact). | **HIDDEN-ROUTES + UNDERSELL** (products §07 hidden; uppercase eyebrow omitted) |
| `1389aad` | `.hero-top top: max(…, env(safe-area-inset-top)+12px)` notch-safe + narrow scrim 0.42→0.70 (phone only). Highway §01. | "Pushed Highway breadcrumb below notch + darkened hero overlay on narrow phones." | `highway / webkit-iphone` §1, **rerender, reliable** (exact). | **OK** (accurate + exact + right device — note: copy DOES mention the darken here, good) |
| `c19645e` | conEdison logo `max-width:100%` clamp (no right-edge crop). Services §05 partners. | "Stopped Con Edison logo getting cut off on phones." | `services / webkit-iphone` §5, **rerender, reliable** (exact). | **OK** (accurate + exact + right device) |
| `153b4fc` | Remove duplicate `— The lineup` eyebrow above Products headline. §01. | "Removed duplicate '— The lineup' label." | `products / chromium-desktop` §1, **rerender, reliable** (exact). | **OK** (accurate + exact) |
| `1158d35` | NEW tablet `@media` dark-tint overlay on `.prod-hero .bg-tint`. (see §1.1) | "Added a tablet-size version of the Products hero overlay." | `products / webkit-ipad` §1, **rerender, reliable** (exact). | **UNDERSELL** (real darkening; copy omits "photo gets darker", has no `choosing:` caveat like its siblings) |
| `b8280e7` | Hero image pipeline + lighter Products hero variants. `Hero.astro` +88. No visible change. | "Reorganised hero image delivery; photo identical." | None (invisible) — format card. | **OK** |
| `57be1c8` | Internal dashboard + hub. Not public site. | "Set up internal dashboard." | None (invisible) — infra card. | **OK** |
| `358b565` | Manifesto checklist. Not public site. | "Added the manifesto checklist." | None (invisible) — infra card. | **OK** |
| `0f76b39` | Re-skin internal dashboard. Not public site. | "Re-skinned the dashboard." | None (invisible) — infra card. | **OK** |

### 2.1 Flag rollup
- **HIDDEN-ROUTES (multi-route, only one shown): 4 commits** — `3d56df0`, `3889ae6`, `a553e1a`, `382886e`. Hidden routes: `3d56df0`→{contact, highway, products}; `3889ae6`→{contact + the highway/services footer-counter effect}; `a553e1a`→{highway, services}; `382886e`→{products}.
- **APPROX-ASIS (existing-run pair rendered without a strong "this may not be the change" caveat): 3 commits** — `a553e1a` + `cc5dc7c` (both `approximate=True`, intervening src-commits), and `02359cd` (existing-run, not flagged approximate, but a 547-line rewrite unverifiable from one frame). (`a553e1a` is double-flagged.)
- **UNDERSELL / MISLABEL (copy ≠ diff): 5 commits** — `3d56df0` (mislabel: copy is for a sliver of a 640-file commit), `3889ae6` (omits footer-contrast on highway/services), `a553e1a` (omits services comet), `382886e` (omits uppercase eyebrow), `1158d35` (omits tablet photo darkening).
- **MITIGATED (working correctly): 9 commits** — the code-only-card replacement is doing its job for the no-delta + placeholder-mismatch failure modes.
- **Clean OK: the remaining ~13.**

---

## 3. ROOT CAUSES (why the page misleads)

### RC-1 — The model bakes ONE pair per commit; the renderer has no gallery
`attach_visible_and_pairs` writes a single `c["pair"]` (picked from the first rerender entry, else the first bracketing existing-run found by iterating `want_routes` and returning on the **first** hit). `c["visible_routes"]` correctly lists ALL affected routes, but it is used only for the *text* reach note — never to build multiple pairs. `beforeAfterViewerHTML(c, idx)` then reads `const pair = c.pair` (one object) and renders one `<div class="ba-viewer">` with `ba-pairhead-loc = ${rt}` (one route). **There is no loop, no tab strip, no "also affected" list in the viewer.** → symptom #1.

### RC-2 — `_COMMIT_COPY` was authored from intent, not diffed against the change
The copy reads like a designer's changelog ("added a hero overlay") rather than a description of the *rendered delta* ("the photo gets darker on tablets"). Where a diff has a secondary effect (footer-contrast in `3889ae6`, comet-brightening in `a553e1a`, uppercase eyebrow in `382886e`, the whole non-blog half of `3d56df0`), the copy silently drops it. There is **no check that the copy mentions every route in `_A88_VISIBLE[sha].routes` nor every visual-effect class in the diff.** → symptom #2.

### RC-3 — The re-render harness produced only 8 usable pairs for the multi-route commits, and one is wrong-route
`reports/git-rerender/index.json` has 13 built states but only **8** entries in `commits{}` (the usable pairs). The multi-route commits (`3d56df0`, `a553e1a`, `382886e`) each have only ONE route re-rendered, and **`a553e1a`'s on-disk re-render is `contact.jpg` — a route the commit doesn't even visibly touch** — so it silently falls back to an approximate existing-run pair. The harness never re-rendered highway/services for `a553e1a` or products §07 for `382886e`. → feeds both #1 and #3.

### RC-4 — The `unreliable` gate is necessary but not sufficient for approximate pairs
`_detect_unreliable_pair` only demotes a pair when before≈after (`<3` mean-channel diff) or a green-placeholder mismatch. An existing-run pair that has a *large* delta passes as "reliable" **even when other src-commits sit between its brackets** — i.e. the delta is real but **not attributable to this commit**. `a553e1a` and `cc5dc7c` are exactly this (`approximate=True`, intervening commits); `02359cd` is the milder case (existing-run, no intervening commit, but a 547-line rewrite no single masthead frame can verify). All three get only the small "⚠ approximate — nearest screenshots around this change" provenance chip (the renderer shows it for ANY non-rerender method), not a stop-and-think caveat that the delta may be a *different* change. → residual symptom #3.

---

## 4. RECOMMENDED FIX (spec — recommend only; do NOT implement here)

### Fix-1 — Show ALL affected routes' before/afters (per-commit gallery, not one pair)  ⟶ closes symptom #1
**Model (`build_render_model.py`):** replace the single `c["pair"]` with `c["pairs"]` — a **list**, one entry per route in `c["visible_routes"]` (keep `c["pair"]` as `pairs[0]` for back-compat). For each route, pick its own pair with the existing priority (exact re-render → existing-run bracket), each independently `unreliable`-tested. A route with no available pair still gets a stub `{route, device:null, no_capture:true}` so the gallery can say "no saved frame for /services yet — here's the code."
**Renderer (`generate_dashboard.py`):** in `beforeAfterViewerHTML`, when `c.pairs.length > 1`, render a **route tab strip** (or stacked sections) above the viewer — one tab per affected route, each carrying its own before/after (or its own code-only card / no-capture stub). Default-select the route with an EXACT re-render; badge tabs that are approximate / code-only. The pair head's `ba-pairhead-loc` becomes the active tab's route.
**Harness (`rerender_git_pairs.py`):** extend the re-render set so EVERY route in `_A88_VISIBLE[sha].routes` gets an exact parent→commit frame — specifically add highway+services for `a553e1a`, products §07 for `382886e`, and the contact/highway/products frames for `3d56df0`. Fix the `a553e1a` wrong-route artifact (it has `contact.jpg`, should have products/highway/services). This is the single highest-leverage capture-coverage gap.

### Fix-2 — Regenerate descriptions to MATCH the diff  ⟶ closes symptom #2
Rewrite the 5 flagged `_COMMIT_COPY` entries so the copy names **every route** and **every visual effect** the diff produces:
- `1158d35`: add to `what`/`choosing` — "this makes the Products hero photo **visibly darker on tablets**" (mirror the `3889ae6`/`a553e1a` darkening caveat).
- `3889ae6`: add the **section-footer contrast lift** and that it touches **highway + services** counters, not just products+contact. Update `_A88_VISIBLE["3889ae6"].routes` to include highway, services.
- `a553e1a`: add the **services scroll-hint comet brightening**.
- `382886e`: add that the **eyebrow now renders uppercase**.
- `3d56df0`: re-author entirely — it is a 640-file snapshot whose *site* effect spans blog (equal-height cards) **+ highway + contact hero + products**, not just "blog cards." Either split the description per route or state plainly "a large checkpoint commit; the site-visible parts are: …" and list them.
**Guardrail (recommend, not in this BG):** a build-time lint that asserts each `_COMMIT_COPY[sha]` text mentions every route in `_A88_VISIBLE[sha].routes`, and (best-effort) flags a copy that omits a brightness/overlay/scrim keyword when the diff for that sha adds an `rgba(12,28,37` gradient or a `filter:`/`text-transform:` change. Warning-only; never blocks.

### Fix-3 — Honestly mark approximate-but-large-delta pairs  ⟶ closes residual symptom #3
For the 2 truly-approximate pairs (`a553e1a`, `cc5dc7c`): when `pair.method=="existing-run"` AND `pair.approximate` AND other src-commits bracket it, **strengthen the caveat** beyond the current small "⚠ approximate" chip — render a banner inside the viewer: "⚠ This is the nearest pair of saved screenshots around this change. **Other changes also happened between them**, so what you see may not be only this change. Go by the code below." Optionally extend `_detect_unreliable_pair` to add a third reason `bracketed-by-others` (demote to the code-only card) when the between-commit count exceeds a threshold (e.g. ≥2) — but a strong caveat is the minimum.

### Fix-4 (supporting) — keep the gallery honest when a route has no frame
Where Fix-1's per-route pick finds nothing for a route (e.g. services on `a553e1a` until the harness re-renders it), show that route's tab with the **honest code-only card** scoped to that route's diff hunks, not a blank tile.

### Priority order
1. **Fix-1 harness coverage** (`a553e1a` wrong-route artifact + missing highway/services/products-§07 re-renders) — highest leverage, fixes the worst card and the coverage gap at the source.
2. **Fix-2 descriptions** (5 rewrites + `_A88_VISIBLE` route corrections) — cheap, directly answers "did it do MORE / WHERE ELSE."
3. **Fix-1 model+renderer gallery** — the structural fix for all future multi-route commits.
4. **Fix-3 approximate caveat** + **Fix-4 per-route honest fallback**.

---

## 5. WHAT'S ALREADY RIGHT (do not regress)
- The **`unreliable` → code-only-card** path (the in-source "A91" work) correctly defuses 9 commits' misleading no-delta / placeholder pairs. The placeholder-green and perceptual-diff detectors are sound and degrade safely (missing PIL → trust the pair, no false flags).
- **Invisible commits** (7) correctly show a labelled "why there's no picture" card, never a broken tile.
- **Exact re-renders** (8) are right-route, right-device, and reliable — for the single route they cover.
- `visible_routes` already carries the full route set in the model — Fix-1 is wiring already-present data into the viewer, not re-deriving it.

---

*Audit method: `git show <sha>` for every commit (the diff = ground truth), `_COMMIT_COPY` + `_A88_VISIBLE` in `build_render_model.py` (the authored copy + curated routes), `reports/git-rerender/index.json` + `reports/dashboard-build/render-model.json` (the baked pair the page renders), and `beforeAfterViewerHTML` in `generate_dashboard.py` (how one pair becomes one viewer with no gallery). Read-only throughout.*
