---
id: A92
title: Git history forensic FACT-CHECK — real diffs vs commit messages vs git-page cards (the trust audit)
type: audit
status: findings-complete
created: 2026-05-31
scope: read-only
relates_to: [A87, A88, A90, A91]
extends: A91
files_audited:
  - "git show <sha> for all 28 git-page commits (the diffs = ground truth)"
  - scripts/dashboard/build_render_model.py          # READ-ONLY (_COMMIT_COPY + _A88_VISIBLE = the card claims)
  - scripts/dashboard/generate_dashboard.py          # READ-ONLY (beforeAfterViewerHTML + the gallery/banner renderers)
  - reports/dashboard-build/render-model.json        # the baked per-route pairs the page renders RIGHT NOW
  - reports/git-rerender/index.json
severity: HIGH (trust-gating — the user's commit waits on this)
---

# A92 — Git-history forensic FACT-CHECK (3-way: real diff vs message vs card)

> **Charter.** A12-style second pass. A91 lined up the git-page CARD copy vs the diff vs the
> shown picture for the 20 *visible* commits and found the `a553e1a` class of problem (a real
> visual side-effect — hero darkening — that no card mentioned). The user's exact fear:
> **"where ELSE did Claude darken an image / change something visual WITHOUT noting it."**
> This goes WIDER (every one of the 28 early commits, including the ones the page calls
> "mechanical" / "invisible" / "infra") and DEEPER (the full real diff of each, then a 3-way
> cross-check of REAL-DIFF vs COMMIT-MESSAGE vs GIT-PAGE-CARD).
>
> **Method.** `git show <sha>` for all 28 (the diff is the only ground truth). For the two mega
> commits (`02359cd` 1,887 files; `3d56df0` 640 files) the analysis focuses on the USER-VISIBLE
> src slice (`src/pages|components|styles|layouts|lib`, `.astro/.css/.ts`) and notes-but-doesn't-
> deep-dive the generated/tooling noise. Card claims read from `_COMMIT_COPY` + `_A88_VISIBLE`;
> the live shown pairs read from `render-model.json`. **Read-only throughout — no code/settings/src/git mutations.**

---

## 0. TL;DR — the verdict in three sentences

1. **A91's in-code fixes have LANDED and largely work.** The per-route gallery is real (`c.pairs`
   list → route-tab strip), `a553e1a` now renders **all three routes as EXACT re-renders** (its
   wrong-route `contact.jpg` artifact is gone — the single worst A91 card is FIXED), `3d56df0`
   shows all 4 routes exact, `382886e` shows both routes, the unreliable→code-only path defuses the
   no-delta/placeholder pairs, and a **strong stop-and-think banner** now fires for approximate
   pairs with other commits bracketed inside them. The five `_COMMIT_COPY` rewrites A91 recommended
   are all present (footer counters, services comet, uppercase eyebrow, tablet darkening, the
   4-page `3d56df0` re-author).

2. **The user's core fear has ONE big remaining hit, and it is NOT `a553e1a`.** It is **`02359cd`**
   — the "developing agentic multiplatform tool" commit. Its card says only *"the blog was rebuilt
   … an experimental page … image folders reorganised … minor touches site-wide."* The real diff
   ALSO contains, undocumented: a **global body-background color flip white → dark navy**, a
   **darkening of EVERY hero overlay on phones** (`.hero .bg-tint` ≤600px, 0.4/0.18/0.86 →
   0.6/0.5/0.92 — the exact `a553e1a`-class hidden darkening, but site-wide and earlier), a
   **fundamental scroll-behavior change** (fullPage snap-paging turned ON for mobile, was
   desktop-only), the **right-edge nav dots hidden below 991px**, **`text-wrap: balance/pretty`
   on every headline + body paragraph site-wide**, and **hero eyebrow labels deleted on 4 pages**
   + **install photos swapped + two testimonials rewritten**. "Minor touches site-wide" is hiding
   a major mobile-behavior + global-visual change. **This is the headline finding.**

3. **Everything else is accurate or only mildly under-described.** `3d56df0`'s card is now good
   but still omits **Services** (the §05 partner-logo strip was resized + re-gridded in that commit
   and `services` is not in its route list). The perf/image commits, the contrast bumps, the nav,
   the refactor, the three internal-dashboard commits, and the floor are all faithfully described.

**Bottom line for the commit gate:** the page is now broadly TRUSTWORTHY for 26 of 28 commits.
The two you must personally eyeball before trusting the page's summary are **`02359cd`** (heavily
under-described — a global mobile-behavior + dark-background + per-phone-hero-darkening change sold
as "minor touches") and, to a lesser degree, **`3d56df0`** (accurate on its 4 named pages but the
5th visibly-touched page, Services' logo strip, is unlisted).

---

## 1. THE HEADLINE FINDING — `02359cd` is the second `a553e1a`, only bigger

`02359cd` "developing agentic multiplatform parallelized development tool…" (2026-05-26).
**1,887 files** (mostly tooling), **31 src files**, **13 src-code files**. Category on the page:
**review** (correctly surfaced as a foundational decision). But its CARD undersells the *visual +
behavioral* reach enormously. Here is what the diff REALLY does on the rendered site, with the
exact changed lines, none of which the card mentions beyond "minor touches site-wide":

### 1.1 GLOBAL body background: white → dark navy  (`src/styles/app.css`)
```
-    background: var(--surface);            /* white */
+    background: var(--teal-deep);          /* #0c1c25 dark navy */
```
The body background is what shows in the iOS overscroll rubber-band and the safe-area-inset strip.
Flipping it white→dark is a real, global, visible change on every page (the commit's own comment
admits the prior white "bled badly… 'rail goes white at bottom-left' complaint"). **Undocumented.**

### 1.2 EVERY hero photo darkened on phones — the `a553e1a` twin  (`src/styles/app.css`)
```
+  @media (max-width: 600px) {
+    .hero .bg-tint {
+      background:
+        linear-gradient(180deg, rgba(12,28,37,0.6) 0%, rgba(12,28,37,0.5) 50%, rgba(12,28,37,0.92) 100%),
+        radial-gradient(120% 70% at 70% 40%, transparent 0%, rgba(12,28,37,0.5) 80%);
+    }
+  }
```
This is the **generic** `.hero .bg-tint` (NOT a per-page selector), so it darkens the hero overlay
on **every page with a photo hero, on every phone** (default was 0.4/0.18/0.86). This is precisely
the "Claude darkened an image without noting it" class the user is hunting — and it predates and
out-scopes `a553e1a` (which only touched products/highway/services on desktop). **Undocumented.**

### 1.3 fullPage snap-paging turned ON for mobile — a fundamental UX change  (`src/lib/fullpage.ts`)
```
-  // Desktop only. Below 992px fullPage disables itself … native scroll …
-  if (!window.matchMedia('(min-width: 992px)').matches) return null;
+  // 2026-05-24 — mobile fullPage enabled (was desktop-only until this turn).
…
-      responsiveWidth: 992,
-      responsiveHeight: 600,
+      responsiveWidth: 0,
+      responsiveHeight: 0,
+      touchSensitivity: 15,
```
Before this commit, phones/tablets scrolled the page **natively**. After it, every page snap-paginates
on swipe on **all** viewports. That is a large, felt behavioral change on mobile (one swipe = one full
section). **Undocumented** in both message and card.

### 1.4 Right-edge nav dots hidden on mobile/tablet  (`src/styles/app.css`)
```
+  @media (max-width: 991px) { #fp-nav { display: none; } }
```
The fullPage right-edge navigation dots disappear below 992px. Visible removal of a UI affordance on
mobile. **Undocumented.**

### 1.5 `text-wrap: balance` / `pretty` applied site-wide  (`src/styles/app.css`)
`text-wrap: balance` added to ALL SIX display type ramps (`.t-display-xl/-lg/-md/-sm`,
`.t-headline-lg`, `.t-quote`); `text-wrap: pretty` added to `.lede .lede-copy`, `.hero-headline-sub`,
`.stats-band .stat .desc`, `.text-on-dark-soft`. Every headline and many paragraphs **re-wrap** site-wide
(line-break positions change — a genuine, if subtle, visible typographic change everywhere). **Undocumented.**

### 1.6 Hero EYEBROW labels DELETED on four pages  (per-page `.astro`)
The mid-frame eyebrow labels were removed from the hero of **fleets** ("— For fleets that don't stop"),
**highway** ("— The new fuel stop"), **multifamily** ("— Multifamily Real Estate"), and **services**
("— Services Hub"). (Products' "— The lineup" is removed LATER, in `153b4fc`, which the page DOES
attribute.) The bulk eyebrow removal happened HERE and is **undocumented** — and note the page later
credits eyebrow work to `3d56df0`/`153b4fc`, so a reviewer would never trace it to `02359cd`.

### 1.7 Install PHOTOS swapped + testimonials rewritten  (per-page `.astro`)
Not pure re-foldering — the actual images changed: `install-caroline-01.jpg` → `seacoast-kittery/01.jpg`
(new photo + new alt + new caption), `install-atlantic-pointe-01.jpg` → `morrison-1240/01.jpg`,
`install-wyndham-01.jpg` → `wyndham-hampton/01.jpg`. Two testimonial paragraphs were **rewritten**
(multifamily: "Evium transformed our portfolio across Newark and the Bronx…" → "Their team handled
everything — from compliance and permits to installation and management…"). A headline gained a forced
`<br>` ("Electrifying the future. One property at a time." → `…future.<br><span>One property…`,
max-width 13ch→16ch). **Undocumented** (the card frames everything image-related as "folders reorganised").

### 1.8 What the page SHOWS for `02359cd`
`blog / chromium-desktop`, **method=existing-run, approximate=False, bracketed_others=0**, shown as-is
with the standard "⚠ approximate — nearest screenshots" chip (NO strong banner, since nothing brackets
it). So even the picture only shows the BLOG masthead — none of §1.1–1.7 (global bg, per-phone hero
darkening, fullPage, eyebrow removals, photo swaps) is visible OR described. A reviewer reading this
card learns about ~10% of what the commit did to the rendered site.

> **Recommendation (highest priority, copy-only — no code in this audit):** re-author
> `_COMMIT_COPY["02359cd"]` exactly as `3d56df0` was re-authored — enumerate the site-visible parts:
> (a) mobile now snap-paginates (fullPage), (b) hero overlays are darker on phones, (c) the page
> background is dark navy in the overscroll/safe-area bands, (d) every headline re-wraps
> (text-wrap), (e) hero eyebrow labels removed on fleets/highway/multifamily/services, (f) a few
> install photos + two testimonials changed, (g) the blog content-library rewrite + the new
> multifamilyv2 page. AND extend `_A88_VISIBLE["02359cd"].routes` from `["blog"]` to include
> `home, fleets, highway, multifamily, services, products, contact` (it touches every page via the
> global CSS + the per-page eyebrow/photo edits) so the gallery can show more than the blog frame.
> This is the same fix A91 applied to `3889ae6`/`3d56df0`; `02359cd` was missed because A91 only
> graded its blog-rewrite framing, not its global-CSS + per-page slice.

---

## 2. SECOND FINDING — `3d56df0` is now accurate on 4 pages but omits Services (logo strip)

`3d56df0` "snapshot 48h of sprint" (640 files / 7 src-code). The A91 re-author is GOOD — the card now
plainly lists Blog (equal-height cards), Highway (economics "Calculator" redesign), Contact (form
labels/placeholders), Products (hero crumb), and says "the earlier description ('just blog cards')
undersold it." The page shows **all 4 routes as EXACT re-renders**. Verified against the diff: all four
are real and correctly described.

**But the diff ALSO visibly changes Services** — the §05 partner-logo strip:
```
-    max-height: 38px; max-width: 116px; …
+    height: 30px; width: auto; max-width: 140px; …          (logos now render at a different uniform size)
…  + a full 3-breakpoint re-grid (7-up desktop / 4-up ≤1100px / 2-up ≤640px, last logo spans to kill the orphan)
```
`_A88_VISIBLE["3d56df0"].routes = ["blog","contact","highway","products"]` — **services is not in the
set**, so neither the card text nor the gallery surfaces the logo-strip change. Also undocumented (but
lower-stakes, invisible on engines already correct): `.hero { isolation: isolate }` (a WebKit-only
z-index bugfix so the crumb stops rendering behind the tint) and the unified `--eyebrow-padding-inline`
token (subtle floating-marker alignment shift on all heroes). **Recommendation:** add `services` to
`_A88_VISIBLE["3d56df0"].routes` and add one clause to the card: "Services — the partner-logo wall was
resized to a uniform height and re-laid-out so no logo is left orphaned."

---

## 3. PER-COMMIT 3-WAY FACT-CHECK TABLE (all 28)

Columns: **REAL DIFF** (ground truth, the visible/behavioral effects) · **COMMIT MSG** (the git
subject/body) · **GIT-PAGE CARD** (`_COMMIT_COPY` today) · **VERDICT**.
Verdict legend: `ACCURATE` · `UNDER-DESCRIBED` (card omits a real effect) · `MIS-CLASSIFIED-ROUTES`
(a visibly-touched page missing from the route set) · `MITIGATED` (shown via the honest code-only
card / strong banner — working) · `INVISIBLE-OK` (no rendered change, card says so).

| # | sha | REAL DIFF (visible/behavioral truth) | COMMIT MESSAGE | GIT-PAGE CARD (today) | VERDICT |
|---|-----|--------------------------------------|----------------|-----------------------|---------|
| 1 | `b77620d` | v1 of the whole site. | "first commit" | "Built the entire first version." Floor. | **ACCURATE** (floor) |
| 2 | `02359cd` | Blog→Content-Collection rewrite + new /multifamilyv2 + image re-folder **+ body bg white→navy + ALL phone heroes darkened (.hero .bg-tint ≤600px) + fullPage snap-paging ON for mobile + #fp-nav hidden ≤991px + text-wrap balance/pretty site-wide + 4 hero eyebrows deleted + install photos swapped + 2 testimonials rewritten + a forced <br>**. 31 src files. | "developing agentic multiplatform parallelized development tool…" (generic; says nothing about the site) | "Rebuilt the blog + added an experimental page… image folders reorganised… **minor touches site-wide.**" | **UNDER-DESCRIBED (severe) + MIS-CLASSIFIED-ROUTES** — see §1. The single biggest gap; contains the user's feared hidden hero-darkening, site-wide + earlier than a553e1a. |
| 3 | `3d56df0` | Blog equal-height cards; Highway economics "Calculator" redesign; Contact placeholders/labels; Products hero crumb; **Services §05 logo-strip resize + re-grid**; `.hero{isolation:isolate}` WebKit z-fix; eyebrow-inset token. 640 files. | "snapshot 48h of sprint work as ratchet point" (mislabels as a checkpoint) | Re-authored (A91): lists Blog/Highway/Contact/Products + "the earlier description ('just blog cards') undersold it." | **UNDER-DESCRIBED (minor) + MIS-CLASSIFIED-ROUTES** — accurate on its 4 named pages; omits **Services** logo strip (not in route set). See §2. |
| 4 | `3889ae6` | (1) Products `.prod-hero .bg-tint` **0.55→0.92 / 0.22→0.72 / 0.05→0.18** (heavy darken) + `.bg-image` object-position **60%→30%** (re-frame); (2) `.sec-foot` α 0.4→0.6 (global section counters, visible on products/highway/services); (3) Contact `.form-wrap align-items:start` + aside padding + input border 1.5px→2px `#b9c1cc`. | "phase a plow lands — 3 desktop bugs fixed in parallel" + detailed body | Re-authored (A91): "Darkened the Products hero, lifted faint section counters across Products/Highway/Services, straightened the Contact form." Carries the borderline tag + darkening caveat. | **ACCURATE** (post-A91; the darkening + counters + reframe are all now named) |
| 5 | `c9f52fe` | Nav: comment out "Multifamily v2"; "Highway / NEVI"→"Highway". `HeaderBatteryLefter.astro`. | "nav: drop multifamily v2, shorten highway label" | "Removed Multifamily v2 from menu; shortened Highway label." | **ACCURATE.** Page shows `home / existing-run / unreliable=no-delta → code-only card` (header crop not in masthead frame — MITIGATED, correct). |
| 6 | `1da5463` | `.econ-model span em` α 0.4→0.6 (highway fine print) + `.sec-foot.dark` α 0.5→0.75 (the LIGHT-bg footer-label variant, global). | "phase B: .sec-foot.dark alpha 0.5→0.75 + .econ-model span em alpha 0.4→0.6" | "Darkened two bits of faint grey text: footer labels + Highway econ fine print." | **ACCURATE.** (subtle/contrast → code-only card, MITIGATED) |
| 7 | `420b377` | Refines the nav comment pinning multifamilyv2 internal. Comment-only (+3/-1). | "nav: pin multifamilyv2 as internal-only (chamber decision)" | "Recorded the decision… only a note in code." | **INVISIBLE-OK** |
| 8 | `acb5648` | `.spec-card .stage`: green hairline `border-top`, green wash gradient (top 60%), `saturate(0.5)` on tellus-180/300 photos. Highway §04. | "P1-07a: /highway § 04 Tellus card light-panel tonal" | "Green-tinted, muted-photo treatment on Highway Tellus cards." | **ACCURATE** (exact re-render, right route) |
| 9 | `767e019` | Extract `Input.astro`; rewire contact form to it. Markup byte-identical (`id=f-{name}` preserved). **One latent detail:** message field `name` "message"→"msg" (form submits nowhere → zero effect). | "P2-12: extract reusable Input.astro + rewire /contact" | "Reorganised code… looks and behaves exactly the same." | **INVISIBLE-OK** (visually identical; the `name` rename is inert; not worth surfacing) |
| 10 | `7a96bee` | `.text-on-dark-mute` α 0.5→0.6 + `--on-surface-variant` **#5c6a75→#505c65** (a GLOBAL token used by `.text-ink-soft`, `.spec-card .meta`, info-card labels — darkens secondary text site-wide, not just on-dark). | "P2-09: bump .text-on-dark-mute α 0.5→0.6 + --on-surface-variant…" | "Muted text on dark a touch stronger, and darkened one grey UI colour." | **ACCURATE** (the "one grey UI colour" = the token; reach is broad but honestly flagged; MITIGATED via code-only card) |
| 11 | `2df2577` | `BaseLayout.astro` font CSS `media=print onload` + `<noscript>` mirror (FOUT on cold load only). | "perf: font CSS non-blocking via media=print onload" | "Fonts load faster; brief fallback on cold load." | **ACCURATE** (FOUT → code-only card, MITIGATED) |
| 12 | `08c4c5e` | New `src/pages/blog/[slug].astro` (+418). 17 posts stop 404ing. After-only. | "fix: add src/pages/blog/[slug].astro — 17 posts were silently 404ing" | "Built article-page template so all 17 posts open." | **ACCURATE.** Page: `blog / existing-run / placeholder-mismatch → code-only card` (after-only by nature, MITIGATED) |
| 13 | `c9a20d2` | 6 `href="#"` → `/blog/${id}` (blog index links). Behavioral. | "fix: rewire 6 dead /blog post-link placeholders" | "Pointed six dead links to real articles." | **ACCURATE** (behavioral → code-only card, MITIGATED) |
| 14 | `a553e1a` | Products scrim deepened (28%→34%, 0.18→0.32); **NEW** highway scrim; **NEW** services scrim; services scroll-comet brightened; `services.astro` extraClass. | "CB-1: hero scrim hardening (products + highway + services)" | Re-authored (A91): names all 3 routes + "brand-new dark overlay" on highway/services + the Services comet brightening + borderline tag. | **ACCURATE (now FIXED).** Page shows **all 3 routes as EXACT re-renders** — the A91 wrong-route `contact.jpg` artifact is gone. The worst A91 card is resolved. |
| 15 | `ef6faec` | Contact route cards: border 0.1→0.18 (`.route-row` + `.route-card`), white fill, hover translateY+shadow, `.go` underline-on-hover. §02. | "CB-2: contact § 02 card affordance" | "Made the Contact route cards clearly clickable." | **ACCURATE** (exact re-render) |
| 16 | `cda8348` | Contact install photo `<img>`→`<picture>` AVIF/WebP + `width/height` (CLS). Same photo. 131 files. | "perf: image pipeline foundation… /contact picture-element migration" | "Lighter formats + reserved space; same photo." | **ACCURATE** (CLS → code-only card, MITIGATED) |
| 17 | `cc5dc7c` | 17 Products card images → `<picture>`+srcset (CLS). `products.astro` +115. Same photos. 1 file. | "perf: /products image migration (17 images → picture+srcset)" | "Lighter formats + reserved space for 17 product images." | **ACCURATE + residual-mitigated.** Page: `products / existing-run / approx=Y / brk=9` → **the STRONG "other changes also happened between them" banner fires** (A91 residual symptom #3 now handled honestly; not demoted to code-only). |
| 18 | `dac177e` | 5 blog images → `<picture>`+srcset (CLS). 2 files. | "perf: /blog + /blog/[slug] image migration" | "Lighter formats for 5 blog images." | **ACCURATE** (CLS → code-only card, MITIGATED) |
| 19 | `6a36736` | Generated blog AVIF/WebP variants + 5 manifest entries. Zero src-code touch. | "perf: add 5 /blog source entries… generated AVIF/WebP variants" | "Generated lighter blog files… companion." | **INVISIBLE-OK** (infra card) |
| 20 | `382886e` | Shared `.cta-split .shot .caption` flex/z-index:2/gap + eyebrow `.k` **text-transform:uppercase** + line-heights. Fleets §04 + Products §07. | "fix(F-NEW-03+04): /fleets §04 + /products §07 caption-stack z-order" | Re-authored (A91): names both routes + "the eyebrow label now renders UPPERCASE… a small but real visible change on both." | **ACCURATE (now FIXED).** Page shows **both routes as EXACT re-renders** (products §07 no longer hidden). |
| 21 | `1389aad` | `.hero-top top:max(…, env(safe-area-inset-top)+12px)` notch-safe + `min-width:0` + new ≤600px `.highway-hero .bg-tint` narrow scrim (0.42→0.70 top). Highway §01 phone. | "fix(highway-01-mobile): notch-safe .hero-top + dense narrow scrim" | "Pushed Highway breadcrumb below notch + darkened hero overlay on narrow phones." | **ACCURATE** (and the card DOES say "darkened" here — good; exact re-render, right device) |
| 22 | `c19645e` | `.logo-cell img max-width: 140px → min(140px, 100%)` (no right-edge crop). Services §05 phone. | "fix(D-NEW-narrow-01): /services §05 conEdison logo right-edge crop" | "Stopped Con Edison logo getting cut off on phones." | **ACCURATE** (exact re-render, right device) |
| 23 | `153b4fc` | Remove `<span class="eyebrow no-rule">— The lineup</span>`; headline `margin-top:28px→30px` (2px, trivial). Products §01. | "fix(F-NEW-01+02): remove /products §01 mid-frame eyebrow dup" | "Removed duplicate '— The lineup' label." | **ACCURATE** (the 2px nudge is below notice; exact re-render) |
| 24 | `1158d35` | NEW `@media (641–1024px) .prod-hero .bg-tint` navy dark-tint overlay (tablet). Products §01 tablet. | "fix(M-NEW-01+D-NEW-tablet-01): /products §01 .bg-tint viewport-scope @media" | Re-authored (A91): "Darkened the Products hero photo on tablet screens… navy tint… heaviest at the bottom" + caveat. | **ACCURATE (now FIXED).** A91's UNDERSELL is resolved — the card now says "darker." Exact re-render. |
| 25 | `b8280e7` | Hero image pipeline (manifest-driven `<picture>` in Hero.astro) + 50 Products-hero variants + `width/height`. Same photo. 52 files. | "perf: Hero refactor — shared aura-dark-glow + secaucus-plaza picture+srcset" | "Reorganised hero image delivery; photo identical." | **INVISIBLE-OK** (format card; `<picture>` transparent to layout) |
| 26 | `57be1c8` | Internal dashboard + hub. Zero `src/pages|components|styles|layouts` touch. | "Dashboard findability: serve the new dash + style-picker, add a hub" | "Set up internal dashboard. Not part of the public website." | **INVISIBLE-OK** (verified: no public-site src) |
| 27 | `358b565` | Manifesto checklist. Zero public-site src touch. | "User owns 'done': manifesto checklist" | "Added the manifesto checklist. Not part of the public website." | **INVISIBLE-OK** (verified) |
| 28 | `0f76b39` | Re-skin internal dashboard. Zero public-site src touch. | "Dashboard re-skin: C/Blueprint canonical…" | "Re-skinned the dashboard. Not part of the public website." | **INVISIBLE-OK** (verified) |

### 3.1 Verdict rollup (28 commits, each gets exactly one primary verdict)
- **UNDER-DESCRIBED (card omits a real visible/behavioral effect): 2** — `02359cd` (severe — §1), `3d56df0` (minor — Services logo strip, §2).
- **ACCURATE — visible/subtle change fully matches the card: 13** — `b77620d, 3889ae6, c9f52fe, 1da5463, acb5648, 7a96bee, 2df2577, 08c4c5e, c9a20d2, a553e1a, ef6faec, cda8348, cc5dc7c, dac177e, 1389aad, c19645e, 153b4fc, 1158d35` → that is the visible+subtle remainder; the post-A91 rewrites (`3889ae6`, `a553e1a`, `382886e`, `1158d35`) are now all correct. (`382886e` also ACCURATE — listed under FIXED in §3.)
- **INVISIBLE-OK (no rendered change, card says so): 7** — `420b377, 767e019, 6a36736, b8280e7, 57be1c8, 358b565, 0f76b39`.
- **Cross-cut — MITIGATED (the slider would mislead, so the page shows the honest code-only card or the strong banner instead — working):** `c9f52fe, 1da5463, 7a96bee, 2df2577, 08c4c5e, c9a20d2` (code-only) + `cc5dc7c` (strong banner). These are a *display* property layered on top of the ACCURATE verdict, not a separate bucket.
- **No MIS-CLASSIFIED (wrong-route picture) remain** — A91's `a553e1a` wrong-route `contact.jpg` artifact is fixed; the gallery is per-route-exact.

*(Tally: 2 under-described + 19 accurate-visible/subtle + 7 invisible-OK = 28. `382886e` is counted in the accurate set.)*

---

## 4. THE PRIORITIZED "THINGS THE USER MUST KNOW" LIST

Ranked by how badly the page misleads about a real change. **All copy/data-only — recommend, don't implement.**

1. **`02359cd` — undocumented GLOBAL hero darkening on phones + dark body bg + mobile snap-paging.**
   *This is the answer to "where ELSE did Claude darken an image without noting it."* The `.hero .bg-tint`
   ≤600px block darkens every phone hero (0.4→0.6/0.18→0.5/0.86→0.92), the body bg flips white→navy, and
   fullPage snap-paging was turned on for mobile — none of it in the card ("minor touches site-wide").
   *Action:* re-author `_COMMIT_COPY["02359cd"]` (enumerate the 7 effects in §1) + expand
   `_A88_VISIBLE["02359cd"].routes` to all pages. **HIGHEST.**

2. **`02359cd` — undocumented hero EYEBROW removals (4 pages) + install-photo swaps + testimonial rewrites.**
   The bulk eyebrow deletion (fleets/highway/multifamily/services) and two changed install photos +
   two rewritten testimonials are folded into "image folders reorganised." A reviewer can't see them.
   *Action:* same re-author as #1 (these are the per-page slice). **HIGH.**

3. **`3d56df0` — Services partner-logo strip resized + re-gridded, but Services isn't a listed route.**
   The §05 logo wall changed (uniform 30px height, 7/4/2-up re-grid) and won't appear in the card text
   or the gallery. *Action:* add `services` to `_A88_VISIBLE["3d56df0"].routes` + one card clause. **MEDIUM.**

4. **`cc5dc7c` — approximate pair with 9 other commits bracketed inside it (informational, already mitigated).**
   The page now fires the strong "other changes also happened between these screenshots" banner, so this
   is *honestly* surfaced — but the slider on a busy Products route still shows a delta that is NOT this
   commit's CLS change. *Action:* none strictly required (the banner is honest); optionally demote to a
   pure code-only card like its siblings `cda8348`/`dac177e`. **LOW.**

5. **`02359cd` — its as-is blog pair shows only the masthead** (no strong banner because brk=0).
   Even after #1's copy fix, the single picture verifies ~10% of the commit. *Action:* once the routes
   are expanded (#1), the gallery picks up more frames; the blog masthead alone should carry the standard
   approximate chip (it does). **LOW.**

**NOT problems (explicitly cleared so they don't get re-litigated):**
- `a553e1a` — the A91 exhibit — is **fully fixed** (3 exact-route re-renders, comet documented). Do not re-flag.
- `3889ae6` — the products darkening + object-position reframe + counter lift are **now all in the card**.
- The contrast bumps (`1da5463`, `7a96bee`), the perf/image commits, the nav, the refactor, and the three
  internal-dashboard commits are **accurate**. `767e019`'s `name="message"→"msg"` is inert (no form handler).

---

## 5. WHAT THE PAGE GETS RIGHT NOW (do not regress — these are A91's wins, verified landed)
- **Per-route gallery is real.** `c.pairs` is a LIST; `beforeAfterViewerHTML` renders a route-tab strip
  with per-tab trust badges (exact / approx / code). `a553e1a`, `3d56df0`, `382886e`, `3889ae6` all show
  every affected route — A91's RC-1 ("one pair, no gallery") is closed.
- **The wrong-route artifact is gone.** `a553e1a` no longer falls back to `contact.jpg`; all three routes
  are exact parent→commit re-renders (A91's RC-3 worst case, fixed).
- **The five copy rewrites landed.** `_COMMIT_COPY` for `3889ae6` (counters), `a553e1a` (comet),
  `382886e` (uppercase), `1158d35` (tablet darken), `3d56df0` (4-page re-author) all present (A91 RC-2, fixed).
- **Unreliable→code-only works.** no-delta (`c9f52fe`, `1da5463`, `7a96bee`, `2df2577`) and
  placeholder-mismatch (`08c4c5e`, `c9a20d2`) are replaced by honest code-only cards.
- **The strong approximate banner exists and fires** (`cc5dc7c`) — A91's residual symptom #3 is handled
  honestly. PIL/missing-image paths still degrade to "trust the pair" (no false flags).
- **The borderline-review tag** is on `3889ae6` + `a553e1a` (the gray-zone scrim commits), surfaced for
  the user's eye with a one-tap "move to safe-to-keep."

---

*Audit method: `git show <sha>` for all 28 git-page commits (diff = ground truth); `_COMMIT_COPY` +
`_A88_VISIBLE` in build_render_model.py (the card claims today); `reports/dashboard-build/render-model.json`
(the per-route `pairs` the page renders RIGHT NOW — method/approximate/unreliable/bracketed_others);
`beforeAfterViewerHTML` + the gallery/banner renderers in generate_dashboard.py (how the pairs become the
UI). Read-only throughout — no edits to code, settings, src/, or git. Extends A91; supersedes none.*
