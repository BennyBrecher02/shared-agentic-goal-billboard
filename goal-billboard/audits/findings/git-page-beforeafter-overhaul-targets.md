---
title: Git-page before/after OVERHAUL TARGETS — per-commit recreate-vs-split map
type: audit-findings
status: read-only-findings
created: 2026-05-31
scope: read-only (audit only — single write is this file)
relates_to: [A88, A90, A91, A92]
extends: A92
sources_read:
  - scripts/dashboard/build_render_model.py        # READ-ONLY (_COMMIT_COPY, _A88_VISIBLE, A91 unreliable-pair detection ~1353-1588, pairs builder)
  - scripts/dashboard/generate_dashboard.py         # READ-ONLY (baPairBodyHTML / baCodeOnlyHTML / beforeAfterViewerHTML — the code-only fallback trigger)
  - reports/dashboard-build/render-model.json       # the LIVE baked per-route pairs (method/unreliable/approximate) the page renders right now
  - reports/git-rerender/index.json                 # which SHAs have REAL exact rebuilt frames
  - context/markdowns/goal-billboard/audits/A92-git-history-forensic-factcheck.md   # the per-commit content map = source of truth
  - git show --stat per commit (all 28)             # file counts = ground truth
intent: drive (a) recreating missing before/afters + (b) splitting bundled commits in the upcoming git-page overhaul.
---

# Git-page before/after OVERHAUL TARGETS (all 28 tracked commits)

> **What this is.** A92 fact-checked the CARD COPY vs the diff. This file is the
> *production worklist* layered on top: for every one of the 28 commits, what before/after
> treatment it gets TODAY (read live from `render-model.json`, not inferred), whether it is
> **code-only only because the screenshots missed it** (→ recreate, with the exact selector +
> page-region to target), and whether it **bundles separable sub-changes** (→ split into
> sub-cards, with file/hunk groupings). Read-only audit; the single write is this file.

## How to read the "current treatment" column (the live render decision tree)

The renderer (`baPairBodyHTML` in generate_dashboard.py) decides per-route, in this order:
1. `pair.no_capture` → **CODE-ONLY** card ("no reliable before/after for this one").
2. `pair.unreliable` (build flagged it) → **CODE-ONLY** card. Two reasons:
   - `no-delta` — before≈after on the captured section (perceptual diff < 3.0/channel). The
     change is real but **off-frame / below the fold** in the only screenshots that exist.
   - `placeholder-mismatch` — one frame is the blog green/black placeholder cover-tile, the
     other real art (a content-state diff unrelated to the commit).
3. `method === 'rerender'` → **REAL exact pair** (harness rebuilt parent→commit frames). Trusted.
4. else (approximate existing-run bracket) → **picture shown**; if `approximate`/`bracketed_others>0`
   it ALSO fires the **STRONG "other changes also happened between them" banner**.

`visible=='no'/'floor'` commits get **NO pair at all** (code-diff only + "not part of the public site"/floor label).

The **critical insight for the overhaul**: every `no-delta` code-only card is a SUBTLE-BUT-RENDERABLE
candidate — the change *would* show if the harness re-rendered the right region at the right viewport.
That is the whole "CODE-ONLY-BUT-SHOULD-RECREATE" list below.

---

## PER-COMMIT TABLE (all 28)

Legend — **Treatment today**: `REAL n×` = n exact rerendered route pairs · `CODE-ONLY(reason)` = explain-away card ·
`APPROX+BANNER` = approximate pair with strong caveat banner · `NONE` = no pair (invisible/floor/infra).
Mixed rows list per-route. **Recreate?** = is a code-only/missing treatment actually renderable if re-targeted.
**Split?** = does it bundle separable sub-changes per A92.

| # | sha | current card title (short) | files (src/total) | nature | treatment TODAY (live) | SUBTLE-BUT-RENDERABLE? | BUNDLED-SHOULD-SPLIT? |
|---|-----|----------------------------|-------------------|--------|------------------------|------------------------|------------------------|
| 1 | `b77620d` | Built the entire first version | 21 / 87 | floor | NONE (floor label) | No — it's the baseline inventory | No (the floor; it IS everything, splitting is meaningless) |
| 2 | `02359cd` | Flipped to dark bg, darkened phone heroes, mobile snap-scroll | 31 / 1887 | bundled-checkpoint | **REAL 4×** (home/highway/fleets/services exact) + **CODE-ONLY(no-delta)** products & multifamily + **APPROX** blog & contact | **YES — products & multifamily** (see recreate list #1). Their share of the global change (dark bg band, ≤600px hero darken) didn't register on the captured frame. | **YES — heavily** (see split list #1). Bundles ≥7 separable sub-changes. |
| 3 | `3d56df0` | Large checkpoint — several pages at once | 7 / 640 | bundled-checkpoint | **REAL 5×** (blog/contact/highway/products/services exact) | No — all 5 routes captured exactly | **YES** (see split list #2). 5 unrelated page changes + a WebKit z-fix + token. |
| 4 | `3889ae6` | Darkened Products hero, lifted counters, straightened Contact form | 1 / 6 | multi-file (1 src) | **REAL 3×** (products/highway/services exact) + **CODE-ONLY(no-delta)** contact | **YES — contact** (see recreate list #2). The form-alignment + border-thickening delta doesn't show at rest in the bracketing run. | **YES (mild)** (see split list #3). 3 logically-distinct hunks in one CSS file. |
| 5 | `c9f52fe` | Removed Multifamily v2 from menu, shortened Highway label | 1 / 1 | CSS/markup (header) | **CODE-ONLY(no-delta)** home | **Marginal** — the change IS the header nav, croppable. But A92 calls it correctly code-only (header above masthead frame). Renderable only with a header-strip capture (low value). | No |
| 6 | `1da5463` | Darkened two bits of faint grey text | 2 / 2 | CSS-only (contrast) | **CODE-ONLY(no-delta)** highway | **YES but low-yield** — `.econ-model span em` α 0.4→0.6 + `.sec-foot.dark` α 0.5→0.75. A faint-text darkening; renderable as a tight crop of the Highway econ fine-print + a light-bg footer label, but the delta is ~0.1 alpha. | No |
| 7 | `420b377` | Recorded the internal-only decision | 1 / 1 | comment-only | NONE (invisible) | No — comment-only (+3/-1) | No |
| 8 | `acb5648` | Green-tinted muted-photo on Highway Tellus cards | 1 / 1 | CSS-only (visible) | **REAL 1×** (highway exact) | No — captured exactly | No |
| 9 | `767e019` | Reorganised Contact form code (identical) | 2 / 2 | refactor | NONE (invisible) | No — byte-identical markup; the `name="message"→"msg"` is inert | No |
| 10 | `7a96bee` | Muted text on dark stronger + 1 grey colour darker | 1 / 1 | CSS-only (contrast) | **CODE-ONLY(no-delta)** home | **YES — the canonical case** (see recreate list #3). `.text-on-dark-mute` α 0.5→0.6 + global token `--on-surface-variant #5c6a75→#505c65`. Renderable at the exact elements below. | No |
| 11 | `2df2577` | Fonts load faster (cold-load flash) | 1 / 1 | perf (FOUT) | **CODE-ONLY(no-delta)** home | No — FOUT is cold-load-only; steady page identical. A rest screenshot genuinely shows nothing; only a cold-load filmstrip would. Keep code-only. | No |
| 12 | `08c4c5e` | Built article template so 17 posts open | 1 / 1 | new template (after-only) | **CODE-ONLY(placeholder-mismatch)** blog | No (true after-only) — before = 404, no "before" page exists. Code-only is correct. | No |
| 13 | `c9a20d2` | Pointed six dead blog links to articles | 1 / 1 | behavioral (href) | **CODE-ONLY(placeholder-mismatch)** blog | No — link-target change; identical at rest. Behavioral, not visual. Keep code-only. | No |
| 14 | `a553e1a` | Deepened 3 hero overlays + brightened Services comet | 2 / 2 | CSS-only (visible) | **REAL 3×** (products/highway/services exact) | No — captured exactly (the A91 wrong-route artifact is fixed) | **Optional (mild)** (see split list #4). Products scrim deepen vs 2 brand-NEW scrims vs comet brighten. |
| 15 | `ef6faec` | Made Contact route cards clickable | 1 / 1 | CSS-only (visible) | **REAL 1×** (contact exact) | No — captured exactly | No |
| 16 | `cda8348` | Lighter formats + reserved space (Contact photo) | 1 / 131 | perf (CLS) | **CODE-ONLY(no-delta)** contact | No — same photo, CLS-only. Identical at rest; only a load filmstrip differs. Keep code-only. | No (131 files = generated AVIF/WebP variants of ONE photo) |
| 17 | `cc5dc7c` | Lighter formats + space for 17 Products images | 1 / 1 | perf (CLS) | **APPROX + STRONG BANNER** products | No — CLS-only (same photos). The banner is honest; A92 suggests optionally demoting to pure code-only like its siblings. | No |
| 18 | `dac177e` | Lighter formats for 5 blog images | 2 / 2 | perf (CLS) | **CODE-ONLY(placeholder-mismatch)** blog | No — CLS-only, same photos. Keep code-only. | No |
| 19 | `6a36736` | Generated lighter blog files (companion) | 0 / 61 | infra (no src) | NONE (infra) | No — generated variants + manifest only | No |
| 20 | `382886e` | Un-stacked caption text (eyebrow now uppercase) | 1 / 1 | CSS-only (visible) | **REAL 2×** (fleets/products exact) | No — both routes captured exactly | **Optional (mild)** (see split list #5). The z-order fix vs the uppercase eyebrow are separable user-facing facts. |
| 21 | `1389aad` | Pushed Highway breadcrumb below the notch | 1 / 1 | CSS-only (visible, phone) | **REAL 1×** (highway phone exact) | No — captured exactly | **Optional (mild)** (see split list #6). Notch-safe `.hero-top` vs the new ≤600px narrow scrim darkening. |
| 22 | `c19645e` | Stopped Con Edison logo getting cut off | 1 / 1 | CSS-only (visible, phone) | **REAL 1×** (services phone exact) | No — captured exactly | No |
| 23 | `153b4fc` | Removed duplicate '— The lineup' label | 1 / 1 | markup (visible) | **REAL 1×** (products exact) | No — captured exactly (the 2px margin nudge is below notice) | No |
| 24 | `1158d35` | Darkened Products hero on tablet | 1 / 1 | CSS-only (visible, tablet) | **REAL 1×** (products tablet exact) | No — captured exactly | No |
| 25 | `b8280e7` | Reorganised hero image delivery (identical) | 1 / 52 | perf (format) | NONE (invisible) | No — `<picture>` transparent to layout; same photo | No (52 files = 50 generated hero variants) |
| 26 | `57be1c8` | Set up internal dashboard (not public) | 0 / 4 | infra (no public src) | NONE (infra) | No | No |
| 27 | `358b565` | Added the manifesto checklist (not public) | 0 / 7 | infra (no public src) | NONE (infra) | No | No |
| 28 | `0f76b39` | Re-skinned internal dashboard (not public) | 0 / 9 | infra (no public src) | NONE (infra) | No | No |

### Treatment rollup
- **REAL exact pairs (fully captured, no action):** `acb5648, a553e1a, ef6faec, 382886e, 1389aad, c19645e, 153b4fc, 1158d35, 3d56df0` (all routes) + the captured-route slices of `02359cd` (4/8), `3889ae6` (3/4).
- **CODE-ONLY that SHOULD be recreated (re-renderable, just mis-captured):** `02359cd` (products, multifamily), `3889ae6` (contact), `7a96bee` (home). → recreate list.
- **CODE-ONLY that is CORRECTLY code-only (genuinely un-renderable at rest — keep):** `2df2577` (FOUT), `08c4c5e` (after-only), `c9a20d2` (behavioral), `cda8348`/`cc5dc7c`/`dac177e` (CLS), `c9f52fe`/`1da5463` (marginal/low-yield).
- **NONE (floor/infra/invisible — correct):** `b77620d, 420b377, 767e019, 6a36736, b8280e7, 57be1c8, 358b565, 0f76b39`.

---

## LIST A — CODE-ONLY-BUT-SHOULD-RECREATE (commit → exact selector + page-region to re-render)

These are flagged code-only ONLY because the existing screenshots didn't capture the changed
region/viewport. Each names the EXACT selector/element + the page-area + viewport the harness
must target so the recreated before/after actually SHOWS the change.

1. **`7a96bee`** — *the canonical "subtle-but-renderable" case (like the `.text-on-dark-mute` example given).*
   - **Selectors:** (a) `.text-on-dark-mute` opacity **0.5 → 0.6**; (b) global token `--on-surface-variant` **#5c6a75 → #505c65** (consumed by `.text-ink-soft`, `.spec-card .meta`, info-card labels — darkens secondary text **site-wide**, not just on-dark).
   - **Render target #1 (on-dark):** any hero/dark-section caption or sub-label using `.text-on-dark-mute` — e.g. the **home** hero sub-line / on-dark muted captions. Tight crop on that muted line, `chromium-desktop`.
   - **Render target #2 (the broader token):** a **Products `.spec-card .meta`** label (the small grey spec text on a product card) — this is the clearest place the `--on-surface-variant` darkening reads, on a light card where the hex shift is visible. `chromium-desktop`, crop to one spec card's meta row.
   - Pair the two renders so the reviewer sees BOTH the on-dark lift and the global-token darkening (A92 notes the reach is broad-but-honest).

2. **`3889ae6` → `contact` route only** (its products/highway/services hunks are already REAL exact).
   - **Selector:** `.form-wrap { align-items: start }` (was stretched) + Contact aside padding + input border **1.5px → 2px** colour `#b9c1cc`.
   - **Render target:** the **Contact §02 form block** — the right-column form + its inputs. Re-render parent→commit cropped to the form-wrap so the column-top re-alignment and the thicker/recoloured input borders are visible. `chromium-desktop`. (Today this route shows no-delta because the bracketing run framed a different section.)

3. **`02359cd` → `products` and `multifamily` routes** (4 of its 8 routes already REAL exact: home/highway/fleets/services).
   - **What didn't register:** these two routes' share of the GLOBAL change — the body-bg white→navy band and (on phone) the `.hero .bg-tint` ≤600px darkening — produced ~0 delta on whatever section the bracketing run captured.
   - **Selector(s):** `.hero .bg-tint` (≤600px block: gradient stops 0.4/0.18/0.86 → 0.6/0.5/0.92) + body `background: var(--surface)→var(--teal-deep)`.
   - **Render target — products:** the **Products hero** on a **phone** viewport (`webkit-iphone`) — shows the per-phone hero darkening directly; plus a desktop overscroll/safe-area band frame for the dark-bg flip if the harness can capture overscroll.
   - **Render target — multifamily:** the **Multifamily hero** on **phone** (`webkit-iphone`) for the same `.hero .bg-tint` darkening, AND the multifamily section that carries the **rewritten testimonial** + the **swapped install photo** (`install-*` → `morrison-1240/…`) — that content swap is renderable on desktop as a real before/after of the testimonial/photo block.
   - Note: `02359cd`'s `_A88_VISIBLE.routes` already lists all 8; the gap is purely that the *existing-run bracket* for these 2 framed a no-delta section. A targeted re-render at the hero/testimonial region fixes it.

**Explicitly NOT on this list (verified genuinely un-renderable at rest — leave as code-only):**
`2df2577` (FOUT = cold-load only), `08c4c5e` (after-only — no "before" page existed), `c9a20d2`
(behavioral href change), `cda8348`/`cc5dc7c`/`dac177e` (CLS — identical at rest, only a load
filmstrip would differ). `c9f52fe` (header nav) + `1da5463` (≤0.1 alpha fine-print) are
*technically* renderable but low-yield — recreate only if a header-strip / fine-print-crop capture
is cheap.

---

## LIST B — BUNDLED-SHOULD-SPLIT (commit → ordered sub-change breakdown + file/hunk groupings)

Each sub-change is a future sub-card. Ordered most-visible-first. File/hunk groupings per A92's map.

1. **`02359cd`** — *the big one; A92's headline finding. Split into 7 sub-cards.* (31 src files; group by effect, not by file, since most live in the shared `src/styles/app.css`.)
   1. **Dark page background site-wide** — `src/styles/app.css`: body `background: var(--surface) → var(--teal-deep)` (#0c1c25). [global; visible in overscroll/safe-area bands]
   2. **Every phone hero darkened** — `src/styles/app.css`: new `@media (max-width:600px) .hero .bg-tint` block (0.4/0.18/0.86 → 0.6/0.5/0.92 + radial). [the `a553e1a`-class hidden darkening, but site-wide]
   3. **Mobile snap-paging turned ON** — `src/lib/fullpage.ts`: removed the `min-width:992px` desktop-only guard; `responsiveWidth/Height 992/600 → 0/0`; `touchSensitivity:15`. [behavioral — one swipe = one section on all viewports]
   4. **Right-edge nav dots hidden on mobile/tablet** — `src/styles/app.css`: `@media (max-width:991px){ #fp-nav{display:none} }`.
   5. **`text-wrap: balance/pretty` site-wide** — `src/styles/app.css`: added to `.t-display-xl/-lg/-md/-sm`, `.t-headline-lg`, `.t-quote` (balance) + `.lede .lede-copy`, `.hero-headline-sub`, `.stats-band .stat .desc`, `.text-on-dark-soft` (pretty). [every headline re-wraps]
   6. **Hero eyebrow labels DELETED on 4 pages** — per-page `.astro`: `fleets.astro`, `highway.astro`, `multifamily.astro`, `services.astro` (removed the `— …` mid-frame eyebrow). [NOTE: products' eyebrow removal is a SEPARATE later commit `153b4fc` — do not fold]
   7. **Install photos swapped + 2 testimonials rewritten + a forced `<br>`** — per-page `.astro`: `contact.astro` + `multifamily.astro` (install-caroline→seacoast-kittery, install-atlantic-pointe→morrison-1240, install-wyndham→wyndham-hampton; 2 multifamily testimonial paragraphs; headline `<br>` + max-width 13ch→16ch).
   - *(Plus the originally-labelled work, which can stay as one "blog + scaffolding" sub-card: blog → Content-Collection rewrite + new `/multifamilyv2` page + image re-foldering.)*

2. **`3d56df0`** — *checkpoint; split into 5 page sub-cards + 2 invisible notes.* (7 src files)
   1. **Blog** — equal-height feature cards (matching bottom edges).
   2. **Highway** — economics section reworked into the "estimated host revenue / Calculator" outcome layout.
   3. **Contact** — form field labels + placeholders rewritten ("Jane"/"Doe" vs "Type here…").
   4. **Products** — hero crumb tweak.
   5. **Services** — §05 partner-logo wall resized to uniform height (`.logo-cell img` 38px/116px → 30px/140px) + 3-breakpoint re-grid (7/4/2-up). [A92's "omitted Services" finding — now in `_A88_VISIBLE` but still bundled in copy]
   - *(Invisible companions, fold into one note:* `.hero{isolation:isolate}` WebKit z-fix; unified `--eyebrow-padding-inline` token.)*

3. **`3889ae6`** — *3 separable hunks in one CSS file; split into 3 sub-cards.* (1 src file `src/styles/app.css`)
   1. **Products hero darkened + re-framed** — `.prod-hero .bg-tint` 0.55→0.92 / 0.22→0.72 / 0.05→0.18 + `.bg-image` object-position 60%→30%.
   2. **Section-footer counters lifted** — `.sec-foot` α 0.4→0.6 (the "01/06" numbers; visible on products/highway/services). [global selector — affects 3 routes]
   3. **Contact form straightened** — `.form-wrap align-items:start` + aside padding + input border 1.5px→2px #b9c1cc. [the route currently shown as code-only — see recreate list #2]

4. **`a553e1a`** — *optional split (3 sub-cards) — the 3 effects are distinct user-facing facts.* (2 src files)
   1. **Products scrim deepened** — existing overlay 28%→34%, 0.18→0.32 (a darken of an EXISTING scrim).
   2. **Highway + Services got brand-NEW dark scrims** — overlays that did not exist before (a bigger change than #1; A92 flags these as the heaviest). Could be 1 card or 2.
   3. **Services scroll-comet brightened** — the scroll-cue brightened so it survives the now-darker hero + `services.astro` extraClass.
   - Low priority — the current single card already names all three, but a reviewer may want to keep the Products darken and undo the brand-new Highway/Services scrims independently.

5. **`382886e`** — *optional split (2 sub-cards).* (1 src file)
   1. **Caption z-order fix** — `.cta-split .shot .caption` flex/z-index:2/gap (un-stacks overlapping caption text on Fleets §04 + Products §07).
   2. **Eyebrow now UPPERCASE** — eyebrow `.k { text-transform: uppercase }` (a real visible casing change a user might not want). [the two are separable: one is a bug-fix, one is a style decision]

6. **`1389aad`** — *optional split (2 sub-cards).* (1 src file)
   1. **Notch-safe breadcrumb** — `.hero-top top:max(…, env(safe-area-inset-top)+12px)` + `min-width:0` (a pure phone-display fix).
   2. **Narrow-phone hero darkened** — new ≤600px `.highway-hero .bg-tint` scrim 0.42→0.70 top (a darkening — separable from the notch fix). [the card DOES already mention both; split only if you want darkening decisions isolated]

**Not split (single coherent change):** `acb5648, c9f52fe, 1da5463, ef6faec, c19645e, 153b4fc, 1158d35`,
and the perf/infra commits. The floor (`b77620d`) is intentionally one unit.

---

## Notes for the overhaul build (carried from the live model + render logic)

- The machinery to do per-route splitting **already exists**: `c["pairs"]` is a LIST, the gallery
  renders one tab per route, and `_A88_VISIBLE[sha].routes` controls which routes appear. Splitting a
  bundled commit into sub-cards is primarily a **`_COMMIT_COPY` + a sub-card data structure** job, not
  a re-architecture — the route-tab gallery is the natural home for the per-effect breakdown.
- Recreating a mis-captured before/after means producing an **exact rerender frame** for that
  (route, section, device) so the build picks branch (3) `method==='rerender'` instead of the
  no-delta existing-run bracket. The rerender index (`reports/git-rerender/index.json`) currently
  has frames for 11 commits; `7a96bee`, and the `contact`/`products`/`multifamily` route-slices of
  `3889ae6`/`02359cd`, are the missing targets in List A.
- `cc5dc7c` is the ONLY commit currently on the APPROX+BANNER path (every other approximate pair was
  either re-rendered or demoted to code-only). A92 suggests optionally demoting it to pure code-only
  like its CLS siblings — low priority, the banner is already honest.
