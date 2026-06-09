---
id: A88
title: Git rollback — VISIBLE-change vs INVISIBLE-change reclassification (re-render scope)
type: audit
status: completed
mode: read-only
created: 2026-05-31
audited_by: subagent (BG dispatch, read-only — diffs read, nothing applied)
scope: re-audit the 24 src/-touching commits on the axis that matters for visual review —
  does the commit VISIBLY change a rendered page, or is it visually invisible? Corrects the
  design-vs-mechanical bucketing that conflated "mechanical" with "no before/after needed."
cross_refs:
  - context/markdowns/goal-billboard/audits/A85-git-rollback-safety-audit.md
  - context/markdowns/goal-billboard/audits/A87-manifesto-realignment.md
  - context/markdowns/plans/git-rollback-page-ux-upgrade.md
  - scripts/dashboard/build_render_model.py  # _classify_one / _MECH_SUBJECT_SIGNALS (the current split)
  - scripts/dashboard/parse_git_timeline.py  # CURATED source_class map
supersedes_axis: the "design-choice (review) vs mechanical (keep)" split for re-render purposes
---

# A88 — Git rollback: VISIBLE vs INVISIBLE reclassification

**Premise (user, 2026-05-31):** the rollback page sorts the 24 src/-touching commits into
**6 "design choices" (review)** vs **17 "mechanical" (keep)** + 1 floor. The user's correction:
**"mechanical" ≠ "invisible."** A device-responsive fix or a crop/overlap fix is *mechanical*
(it wasn't a subjective design call) but it is **VISIBLE** — a person sees the page differently.
The design-vs-mechanical axis answers *"is this a judgment call?"*; it does **not** answer
*"is this worth a before/after look?"* This audit re-runs the judgment on the axis that actually
sets re-render scope: **does the commit change what a person SEES on the live site?**

**Where the current split comes from (verified):** `build_render_model.py::_classify_one`
(lines 837–907) + `_MECH_SUBJECT_SIGNALS` (783–801). Read the design intent verbatim in the
module comment (751–764): *"most of Claude's changes were MECHANICAL and good — device-responsive
fixes, perf, image optimization — which the user would never revert and **never needs to
review**."* That parenthetical is exactly the conflation the user is flagging — it equates
"mechanical" with "no review / no before-after," which is wrong for a *visual* review pass. The
classifier fires `mechanical` on verbs like `@media`, `crop`, `z-order`, `notch-safe`, `clamp`,
`scrim` — every one of which is a **visible rendering change** (just not a subjective one).

**Bottom line:**
- Of the **17 "mechanical"** commits, **15 are VISIBLE** (yes or subtle), **2 are INVISIBLE.**
  The "mechanical" bucket is overwhelmingly mis-labelled for re-render purposes — it is mostly
  *visible-but-not-subjective* work (responsive fixes, scrim/contrast, glitch fixes), which is
  precisely the class that benefits MOST from a before/after pair (a person can't eyeball a
  responsive fix from a commit subject).
- Of the **6 "review" (design)** commits, **5 are VISIBLE**, **1 is INVISIBLE** (a comment-only
  nav commit — `420b377` — whose code change had already landed in the prior commit).
- The **1 floor** commit (`b77620d`) is VISIBLE by definition (it is the entire first render) but
  has **no before** — it is the baseline, not a before/after candidate.
- **Net corrected re-render scope: 20 of 24 commits warrant a before/after pair** (or after-only
  where no before exists). 3 are INVISIBLE (no pair). 1 is the floor (after-only / baseline label).

The two axes are **orthogonal and BOTH worth keeping** — design-vs-mechanical answers "whose call
was this / should the user reconsider the decision," visible-vs-invisible answers "should we
re-render a before/after." The git-page should carry **both** flags, and the before/after
re-render scope should key off **VISIBLE**, not off `category==='review'`.

---

## (1) Per-commit table — the VISIBLE-change axis

Columns: **sha** · **current bucket** (review / mech / floor, as `_classify_one` bakes it today)
· **VISIBLE?** (yes / subtle / no) · **what's visibly different** · **route × device the diff
shows on** · **before/after pair?** (Y / Y-after-only / N).

"subtle" = the steady-state painted page is identical but something visibly changes during load
(FOUT, CLS/layout-settle) or is a behavior/navigation change that looks identical at rest — worth
a *labelled* look but a slider may show no delta at rest.

| sha | cur. bucket | VISIBLE? | What's visibly different | Route × device | B/A pair? |
|---|---|---|---|---|---|
| `b77620d` | floor | **yes** (no before) | The entire first render of the site — every page. | all routes × all devices | **Y — after-only** (it IS the floor; "reverting undoes the whole site") |
| `02359cd` | **review** | **YES** | NOT a pure rename. `blog.astro` rewritten hardcoded→Content-Collection (547 lines), **17 new blog post bodies**, a **new `multifamilyv2.astro` page** (225 lines), nav edits, app.css +155, edits to every page. The blog index + all 17 article pages render from collections; a whole new page exists. | `/blog` + `/blog/*` (×all); new `/multifamilyv2`; header (×all) | **Y** (the user's #1 redo-by-hand candidate; correctly in review) |
| `3d56df0` | mech | **YES** ⚠️**mis-bucketed** | Subject "snapshot…ratchet" tripped the *checkpoint→mechanical* signal, but the diff is **390 insertions of real visual work**: eyebrow inset alignment (`left: var(--eyebrow-padding-inline)`), **equal-height blog cards** (`height:100%`+`flex:1`), `.mast-foot` responsive `@media` for iPad/phone (fixes the "17 / ARTICLES · / 4 / CATEGORIES" 4-line wrap), `white-space:nowrap` on the live counts. Touches Hero, blog, contact, highway, products. | `/blog`,`/contact`,`/highway`,`/products` × **iPad-portrait + phone** (and desktop card heights) | **Y** |
| `3889ae6` | mech | **YES** ⚠️**mis-bucketed** | Three real desktop fixes: `/products` hero `object-position 60%→30%` (pillar readout moves ~64px to clear eyebrow overlap) + `.bg-tint` gradient stops `0.55/0.22/0.05 → 0.92/0.72/0.18` (much darker scrim); `/contact` input border `1.5px→2px` + grid `align-items:stretch→start` (column realign); `.sec-foot` alpha `0.4→0.6`. | `/products` §01, `/contact` §01 × desktop (chromium/webkit/firefox) | **Y** |
| `c9f52fe` | **review** | **YES** | Nav item **removed** ("Multifamily v2") + label **"Highway / NEVI" → "Highway"**. The site header changes on every page. | header × all routes × all devices | **Y** |
| `1da5463` | mech | **subtle** | Two alpha lifts: `.sec-foot.dark 0.5→0.75` and `.econ-model span em 0.4→0.6` (text gets visibly darker/more opaque). Small but a person can see the contrast change on the affected counters/labels. | `/highway` (econ model) + sec-foot wherever `.dark` × desktop | **Y** (subtle — slider shows a faint darkening) |
| `420b377` | **review** | **NO** ⚠️**mis-bucketed (other direction)** | **Comment-only diff.** The actual nav code (`// { label: 'Multifamily v2' … }`) was already commented out in `c9f52fe`; this commit only rewrites the *explanatory comment* above it. Zero rendered change. (It IS a real *decision* — hence "review" — but it is **invisible**, so no re-render.) | none | **N** |
| `acb5648` | **review** | **YES** | `/highway` §04 Tellus cards: **green hairline `border-top`** added, a green wash gradient on the stage top 60%, and `filter: saturate(0.5)` on two product photos (visibly de-saturates vendor magenta/cyan). | `/highway` §04 × desktop (+ tablet/phone — same selector) | **Y** |
| `767e019` | mech | **NO** | `Input.astro` extracted + `/contact` rewired to use it. The component docstring is explicit: emits "the exact `.field`/`label`/`input` DOM the stylesheet already targets … lights up automatically **without any CSS change**." Pure refactor — **identical rendered output**. | none (renders identically) | **N** |
| `7a96bee` | mech | **subtle** | `.text-on-dark-mute α 0.5→0.6` + `--on-surface-variant #5c6a75→#505c65` (AA contrast lift, measured). Colors change a perceptible step; muted-on-dark text reads slightly stronger. | wherever those tokens apply (multiple routes) × desktop/all | **Y** (subtle) |
| `2df2577` | mech | **subtle** | Font CSS made non-blocking (`media="print" onload`). **Steady-state is identical** (Outfit still applies). But during load the user now sees a **FOUT** — fallback font first, then swaps to Outfit — where before it was render-blocking (no flash, slower paint). A transient visible change on first/cold load. | all routes × all devices (slow/cold connection) | **Y — load-sequence** (a filmstrip, not a single before/after, shows it; at rest no delta) |
| `08c4c5e` | mech | **YES** | Adds `src/pages/blog/[slug].astro` — **17 article pages that were silently 404ing now render** as full magazine-layout articles. Going from a 404 error page to a designed article is a maximal visible change. | `/blog/<slug>` × all 17 posts × all devices | **Y** (after-only per page — before = the 404 page) |
| `c9a20d2` | mech | **subtle** (behavioral) | Rewires 6 dead `href="#"` placeholders → real `/blog/${id}` URLs. The page **looks identical at rest** (same link text/styling), but **clicking now navigates** instead of doing nothing. Visible *as a behavior* (and it's what makes `08c4c5e`'s 17 pages reachable). | `/blog` link targets × all devices | **Y — behavioral** (slider shows no rest-state delta; note it as a nav fix) |
| `a553e1a` | mech | **YES** | Hero **scrim hardening** on 3 routes: `/products` stops `0.18→0.32` + `28%→34%`; new `.highway-hero .bg-tint` override (deep 0.70 band at 65%); new `.services-hero .bg-tint` override + brightened scroll-hint comet. The dark overlay over hero photos visibly deepens. | `/products`,`/highway`,`/services` §01 × desktop (1440 canvas) | **Y** |
| `ef6faec` | **review** | **YES** | `/contact` §02 route cards: border opacity `0.1→0.18` (hairline becomes visible), **white card fill**, **hover lift + soft shadow**, and the `GO` link gains an **underline** that extends on hover. The cards now read as interactive. | `/contact` §02 × desktop (+ hover state) | **Y** |
| `cda8348` | mech | **subtle** | `/contact` install photo → `<picture>` AVIF/WebP/JPG (same photo) **+ adds `width="1200" height="1600"`** to an `<img>` that had none → reserves aspect-ratio space, changes CLS/layout-settle on load. Steady-state image identical. | `/contact` install photo × all devices (CLS on load) | **Y — CLS/load** (slider shows no rest delta; note layout-stability) |
| `cc5dc7c` | mech | **subtle** | `/products` 17 card images → srcset **+ adds `width="250" height="350"`** (were unsized) → layout reservation/CLS change; the **reflection** image now points to an explicit `-320.webp` variant (potentially different sharpness than the prior CSS-scaled source). Same product photos. | `/products` card grid × all devices (CLS + reflection sharpness) | **Y — CLS/load** (note possible reflection-sharpness delta — worth one real look) |
| `dac177e` | mech | **subtle** | `/blog` + `/blog/[slug]` 5 PNGs → srcset **+ adds `width="1537" height="1023"`** (were unsized) → CLS/layout reservation on the blog thumbnails/covers. Featured img kept `eager`. Same photos. | `/blog`, `/blog/*` thumbnails/cover × all devices (CLS) | **Y — CLS/load** (slider shows no rest delta) |
| `382886e` | mech | **YES** | `/fleets` §04 + `/products` §07 caption stack: `z-index:2`, switch to `display:flex;flex-direction:column;gap:10px`, `text-transform:uppercase` on the eyebrow, `line-height`+`margin` resets. Fixes eyebrow/caption text **collision** — the caption block visibly re-lays-out. | `/fleets` §04, `/products` §07 × all engines (esp. when text wraps) | **Y** |
| `1389aad` | mech | **YES** | `/highway` §01 mobile: `top: max(clamp(...), safe-area-inset-top+12px)` (crumb row drops below the **iPhone notch** — fixes clipped "THE NEW FUEL STOP" top-bar) + a much darker narrow `@media (max-width:600px)` scrim (top stop `0.42→0.70`). Phone-only, very visible. | `/highway` §01 × **iPhone (notch) + narrow Android** | **Y** (phone) |
| `c19645e` | mech | **YES** | `/services` §05 conEdison logo: `max-width: 140px → min(140px,100%)` — fixes the **right-edge crop** of the wide logo at the 2-up phone breakpoint. The logo goes from clipped to fully shown. | `/services` §05 × **phone / narrow** (≈411px Android, iPad-mini) | **Y** (phone) |
| `153b4fc` | mech | **YES** ⚠️**mis-bucketed** | Subject "remove … dup" → was read as mechanical, but it **removes a visible text line** — the mid-frame `— The lineup` eyebrow on the `/products` hero (and bumps `margin-top 28→30px`). A line of copy disappears from the rendered hero. (This is also arguably a content/IA call.) | `/products` §01 hero × all devices | **Y** |
| `1158d35` | mech | **YES** | New `@media (min-width:641px) and (max-width:1024px)` band for `.prod-hero .bg-tint` — gives **tablets** an intermediate scrim (fixes the "H" of HARDWARE crumb occluded on iPad-mini/iPad portrait). A viewport range that previously fell through the cascade now renders a different scrim. | `/products` §01 × **tablet (744–1024px portrait)** | **Y** (tablet) |
| `b8280e7` | mech | **NO** (srcset caveat) | Hero refactor centralizes `<picture>` in `Hero.astro`. The `<img class="bg-image" src={src}>` value is **unchanged**; docstring: "`<picture>` is transparent to layout … zero regression." Displayed hero photo is the **same image** at steady state. The "50 new variants / secaucus-plaza" are generated `-opt` assets, not a swap; "aura-dark-glow" is in a comment. Pure byte/format optimization. | none at rest (only smaller bytes; srcset CLS caveat shared by all migrations) | **N** (steady-state identical) |

**Tally (24):** VISIBLE **yes = 13** · **subtle = 7** · **no = 3** · floor (yes/after-only) = 1.
→ **20 warrant a before/after** (or after-only) · **3 are INVISIBLE** (no pair) · 1 is the
after-only floor.

---

## (2) Summary — mis-bucketed commits + the corrected re-render scope

### 2a. Currently "mechanical" but actually VISIBLE (mis-bucketed *for review*)

**15 of the 17 "mechanical" commits are visible.** They split into:

- **Clearly visible (yes) — 9:** `3d56df0`, `3889ae6`, `08c4c5e`, `a553e1a`, `382886e`,
  `1389aad`, `c19645e`, `153b4fc`, `1158d35`.
  These are responsive fixes, scrim/contrast hardening, glitch fixes (crop/z-order/notch),
  a new-route render, and a removed copy line. **This is exactly the class the user named** —
  "mechanical" by *decision-type* but plainly visible on screen. A commit subject like
  *"conEdison logo right-edge crop (max-width clamp)"* cannot be eyeballed without a phone
  before/after; burying it as "safe to keep, no review" hides the most before/after-worthy work.

- **Subtle / load-time / behavioral (subtle) — 6:** `1da5463` (alpha), `7a96bee` (alpha+hex),
  `2df2577` (FOUT on load), `c9a20d2` (link-target behavior), `cda8348`/`cc5dc7c`/`dac177e`
  (srcset + **added `width`/`height` → CLS/layout-settle**). Steady-state may look identical, but
  each has a visible facet (a faint contrast step, a font-swap flash, a now-working click, a
  layout-stability change) worth a *labelled* look — though a naive rest-state slider may show no
  delta, so the page should annotate the *kind* of change (see §2d).

- **Two flagged as ⚠️ subject-misleading** (`3d56df0`, `153b4fc`): the *subject* tripped a
  mechanical signal (`snapshot`, `remove…dup`) but the *diff* is visible design/layout work.
  These are the clearest "the heuristic read the commit message, not the change" cases.

**Only 2 "mechanical" commits are genuinely INVISIBLE:** `767e019` (Input.astro extraction —
identical DOM by design) and `b8280e7` (Hero `<picture>` wrap — identical rendered image).
These are the *textbook* invisible class (pure refactor; pure format/byte optimization with
unchanged output).

### 2b. Currently "review" but actually INVISIBLE (mis-bucketed *the other way*)

- **`420b377`** — comment-only diff (the nav code already changed in `c9f52fe`). It is a real
  *decision* (rightly "review" on the design axis) but renders **nothing new**. It needs **no
  before/after** — there's nothing to render. Keep it in the review/decision list; exclude it
  from the re-render scope.

### 2c. The two gray zones the user called out, resolved

- **Image migrations** (`cda8348`, `cc5dc7c`, `dac177e`, `b8280e7`): the *image bytes/format*
  change but the *photo* does not — so on the "does the picture look different" question they are
  **invisible**. BUT three of them (`cda8348`/`cc5dc7c`/`dac177e`) **add `width`/`height` to
  previously-unsized `<img>`s**, which changes **layout reservation / CLS** (and `cc5dc7c`'s
  reflection now uses a fixed `-320.webp` that may differ in sharpness). That tips them to
  **subtle** (a layout/load-stability change, not a content change). **`b8280e7` adds no
  dimensions and keeps the same `<img>`** → the one genuinely **invisible** migration. Verdict:
  migrations are **subtle (CLS/load)** when they add dimensions, **invisible** when they don't.
- **The v1 restructure** (`02359cd`): **not** "mechanical renames." It rewrites the blog to
  Content-Collections, ships **17 new posts + a new page**, and edits every route — unambiguously
  **VISIBLE**, and the user's stated redo-by-hand candidate. Correctly in review; correctly a
  before/after candidate (where a before exists — the blog before/after is the richest).

### 2d. CORRECTED list of commits that warrant a before/after pair (the re-render scope)

This is the scope the git-page **second pass** should re-render against — keyed off **VISIBLE**,
not `category==='review'`:

**Tier 1 — full before/after slider (steady-state visibly differs) — 13:**
`02359cd`, `3d56df0`, `3889ae6`, `c9f52fe`, `acb5648`, `08c4c5e` (after-only per post),
`a553e1a`, `ef6faec`, `382886e`, `1389aad`, `c19645e`, `153b4fc`, `1158d35`.

**Tier 2 — pair WITH an explicit "kind" annotation (rest-state may show little/no delta) — 7:**
- contrast/alpha (slider shows faint darkening): `1da5463`, `7a96bee`
- load-sequence (needs a filmstrip, not a single pair): `2df2577`
- behavioral / link-target (annotate "navigation fix"): `c9a20d2`
- CLS / layout-stability (annotate "layout-shift / load"): `cda8348`, `cc5dc7c`, `dac177e`

**Floor — after-only / baseline label — 1:** `b77620d` (no before exists).

**NO pair (INVISIBLE — exclude from re-render) — 3:** `420b377` (comment-only),
`767e019` (identical-DOM refactor), `b8280e7` (identical-image format swap).

→ **Re-render scope = 20 commits** (13 Tier-1 + 7 Tier-2) **+ 1 after-only floor**; **3 excluded.**

### 2e. Device coverage the pairs must hit (so the matrix re-render isn't desktop-only)

Several visible diffs are **viewport-specific** — a desktop-only re-render would miss them entirely:
- **Phone / notch:** `1389aad` (iPhone notch), `c19645e` (phone crop), `3d56df0` (phone mast-foot).
- **Tablet (641–1024 portrait):** `1158d35`, `3d56df0` (iPad mast-foot), `1389aad` (narrow).
- **Desktop (1440 canvas):** `3889ae6`, `a553e1a`, `acb5648`, `ef6faec`, `382886e`.
- **All viewports:** the route/content changes (`02359cd`, `08c4c5e`, `c9f52fe`, `153b4fc`).

The before/after capture for each commit should default to **a route × device on which its diff
is actually visible** (the §2.3 `touched_routes` inference in the UX plan already supports this) —
not a fixed `chromium-desktop`, or the responsive fixes will render identical pairs.

---

## (3) Recommendation for the git-page (carries forward to the UX plan)

1. **Keep BOTH axes; they answer different questions.** Add a `visible` field
   (`yes`/`subtle`/`no`) alongside the existing `category` (`review`/`mechanical`/`floor`) in
   `_classify_one`. Design-vs-mechanical = "should the user reconsider the decision";
   visible-vs-invisible = "should we re-render a before/after." Do not collapse them.
2. **Key the before/after re-render off `visible != 'no'`, NOT off `category`.** The current
   page leads with the 6 "review" cards and de-emphasizes the 17 "mechanical" — but 15 of those
   17 are visible and are the *hardest* to judge from text. The visible-but-mechanical commits
   are where a before/after pays off most.
3. **Stop reading the commit *subject* as the visibility oracle.** `3d56df0` ("snapshot") and
   `153b4fc` ("remove…dup") prove the subject lies about visibility. Visibility must come from the
   **diff shape** (does it touch rendered CSS/markup/content/media) — the model already has
   `file_list` + `insertions/deletions`; a `visible` heuristic can read those (e.g. any change to
   `.astro` markup, `src/styles/*.css` non-comment lines, `src/content/**`, or an `<img>`
   `src`/`width`/`height` → visible; comment-only or pure `.ts`/component-extraction-with-identical-DOM
   → invisible). When uncertain → **VISIBLE** (the user's stated safe default).
4. **Annotate the "subtle" kind on the card** (contrast / FOUT / CLS / behavioral) so a rest-state
   slider that shows no delta isn't misread as "nothing changed" — point the reviewer at the load
   filmstrip (Mode-3) or the behavior, per the kind.
5. **Three commits need NO re-render** (`420b377`, `767e019`, `b8280e7`) — render them as
   "no visual change" cards (keep them in the timeline for completeness; don't spend a capture).

---

## Findings ledger

| ID | Finding | Impact on re-render scope |
|---|---|---|
| V1 | 15 of 17 "mechanical" commits are VISIBLE (9 yes + 6 subtle) | promote to before/after candidates |
| V2 | `3d56df0` + `153b4fc` mis-read by the SUBJECT heuristic (snapshot / remove-dup) — diffs are visible design work | switch visibility signal to diff-shape, not subject |
| V3 | `420b377` is "review" but INVISIBLE (comment-only; code already changed in `c9f52fe`) | exclude from re-render; keep as decision card |
| V4 | Image migrations are subtle (CLS via added `width`/`height`), except `b8280e7` which is invisible (no dims, same `<img>`) | 3 migrations → subtle pair; `b8280e7` → no pair |
| V5 | `767e019` + `b8280e7` are the only genuinely invisible "mechanical" commits (identical-DOM refactor / identical-image format) | exclude from re-render |
| V6 | Several visible diffs are phone/tablet-only — a desktop-only re-render misses them | pairs must capture the route×device where the diff shows |
| V7 | The two axes are orthogonal; the page currently has only one and keys review off it | add a `visible` field; key re-render off VISIBLE not category |

## What was NOT done (read-only audit)
- No code edits, no model regeneration, no captures, no classifier change. All recommendations in
  §3 are for the git-page second pass / the UX-upgrade plan (`git-rollback-page-ux-upgrade.md`),
  to be built when the user greenlights. Nothing applied.
