---
title: Git rollback-page before/after capture integrity audit
date: 2026-05-31T17:24Z
trigger: user inspecting commit 382886e — "products slider seems no diff which is worrying"
severity: HIGH (data integrity / user trust on the keep-undo rollback page)
scope: every git item with a re-rendered before/after pair (reports/git-rerender/index.json)
method: READ-ONLY. md5 byte-equality + full-resolution pixel diff (fraction of changed pixels + max channel delta) + in-tool visual confirmation of every flagged pair. No code edits, no commits.
verdict: products case REAL & confirmed (byte-identical pair). 3 of 29 route-pairs definitively broken (0 pixel diff). 1 more is real-but-not-representative. Root cause = capture frame does not contain the pixels the commit changed (two distinct mechanisms).
---

# Git rollback-page before/after capture integrity audit

## Headline verdict

- **The products case is REAL.** For commit `382886e`, the `/products` before and after
  images are **byte-for-byte identical** (same MD5, same inode-distinct files, 0 pixels
  changed). The fleets pair, by contrast, is a genuine diff. The user is exactly right:
  sliding the products image shows nothing because the two images are literally the same.
- **Spread: 3 of 29 re-rendered route-pairs (≈10%) are definitively broken** — zero visible
  diff (MD5-identical, 0.0000% pixels changed). These span **3 commits** (`382886e`,
  `3889ae6`, `3d56df0`) on **2 routes** (`/products`, `/highway`). One **additional** pair
  (`3d56df0 /blog §2`) is "real-but-not-representative" — the only delta is a section-counter
  digit (`02/06` → `02/07`), not the change the card advertises.
- **Root cause is NOT "the change is too subtle."** In every broken case the change is real
  and clearly visible *on the page* — it is simply **outside the captured frame**. Two
  distinct frame-selection bugs in the capture harness produce this (see §3).
- **Coverage: 29/29 route-pairs checked** (all have both images on disk; 0 missing). Every
  one of the 10 pairs flagged by the cheap screen was then visually confirmed in-tool.

---

## 1. The reported case — `382886e` confirmation (with evidence)

Commit `382886e` = `fix(F-NEW-03+04): /fleets §04 + /products §07 caption-stack z-order`.
It edits the **shared** `.cta-split .shot .caption` rule in `src/styles/app.css` (adds
`display:flex; flex-direction:column; gap:10px`, `z-index:2`, `text-transform:uppercase`
on `.k`, and resets `.t`/`.k` margins). That component is used by **both** `/fleets` and
`/products`, so a correct capture must show a real diff on **both**.

The index (`reports/git-rerender/index.json`, commit `382886e`) defines two route-pairs,
both bracketing parent-state `d0a2ea6` (before) → `382886e` (after):

| Route | before image | after image |
|---|---|---|
| fleets §4 | `git-rerender/d0a2ea6…/chromium-desktop/fleets.jpg` | `git-rerender/382886e…/chromium-desktop/fleets.jpg` |
| products §6 | `git-rerender/d0a2ea6…/chromium-desktop/products.jpg` | `git-rerender/382886e…/chromium-desktop/products.jpg` |

(full dir SHAs: before = `d0a2ea64593902883392ea1a1f331522b2a7838b`, after =
`382886ed541abb8d1c07703fbbc6e1cb2af5f8f0`.)

### Evidence

| pair | dims | SHA-256 (before / after) | MD5 equal? | pixels >8/255 changed | verdict |
|---|---|---|---|---|---|
| **fleets** | 1344×900 / 1344×900 | `d0f1187e…` / `be4529d7…` | **no** | (real diff) | ✅ genuine before/after |
| **products** | 1344×637 / 1344×637 | `f1b7f45e…` / `f1b7f45e…` | **YES — identical** | **0.0000%** | ❌ BROKEN (no diff) |

- Both files exist, were written at **distinct mtimes** (06:21:54 vs 06:22:42) with
  **distinct inodes** (22985564 vs 22988363) — so this is **not** a symlink/hardlink and
  not one file reused; they are two independent captures that happened to produce the
  identical JPEG.
- Both states built successfully (`index.json.states`: `d0a2ea6` = "built", `382886e` =
  "built") — so this is **not** a failed-build fallback.

### Why (visual confirmation)

Loading the four images:

- **fleets after** correctly shows the `.cta-split` — the bus-fleet photo with the caption
  stack "DEPOT TRANSITION / Bus fleet · Mixed AC + DC banks" bottom-right. This is the
  component the commit fixed, and the before/after differ. ✅
- **products after** shows the **"Four brands. One backend." manufacturers grid** with
  footer "— MANUFACTURERS · 04 … 06 / 06". This is **the wrong section** — it does not
  contain `.cta-split` at all. The products `.cta-split` ("Build a quote — Pick the units.
  We'll price the deploy.") is a *different, later* section that was never captured. Because
  the captured section is untouched by the CSS change, before == after, byte for byte. ❌

**Conclusion:** confirmed. The products before/after is identical because the harness
captured the wrong section of the products page.

---

## 2. Full sweep — every re-rendered before/after pair

29 route-pairs across 12 commits in `index.json`. Method: (a) cheap screen = MD5 equality +
16×16 average-hash Hamming distance; (b) every pair with Hamming ≤ 1 then re-checked with a
**full-resolution** pixel diff (fraction of pixels with any channel delta > 8, plus max
channel delta and the bounding box of the diff region); (c) every flagged pair then **opened
and eyeballed**. The fine diff matters: a 16×16 average-hash cannot see a thin underline or a
removed one-line eyebrow, so it over-flags. The fine diff + eyeball separates "broken" from
"real but small."

### Broken / suspicious pairs (the 10 the cheap screen flagged), after fine-diff + eyeball

| Commit | Route | Device | § | MD5= | px>8 changed | diff bbox | **classification** |
|---|---|---|---|---|---|---|---|
| `382886e` | products | chromium-desktop | 6 | **YES** | 0.0000% | — | ❌ **BROKEN** (wrong section; `.cta-split` never in frame) |
| `3889ae6` | highway | chromium-desktop | 4 | **YES** | 0.0000% | — | ❌ **BROKEN** (right section, changed `.sec-foot` clipped out of frame) |
| `3d56df0` | highway | chromium-desktop | 4 | **YES** | 0.0000% | — | ❌ **BROKEN** (same `.sec-foot`-out-of-frame as above; shares the after-state file) |
| `3d56df0` | blog | chromium-desktop | 2 | no | 0.0226% | (1232,832)-(1288,856) | ⚠️ **real-but-not-representative** — only the section counter `02/06`→`02/07` changes; the card advertises "equal-height cards," not visible here |
| `153b4fc` | products | chromium-desktop | 1 | no | 0.0574% | (47,319)-(177,337) | ✅ REAL — the "— THE LINEUP" eyebrow line is removed (small but correct; matches the commit) |
| `02359cd` | products | webkit-iphone | 1 | no | 2.9456% | (72,1200)-(976,1984) | ✅ REAL (coarse-hash false alarm) |
| `3889ae6` | contact | chromium-desktop | 1 | no | 4.5159% | (159,352)-(1264,897) | ✅ REAL — contact §01 form-card affordance (B-K7-b) |
| `ef6faec` | contact | chromium-desktop | 2 | no | 4.9226% | (56,46)-(1296,507) | ✅ REAL — "GO →" route-card links gain green underlines |
| `3d56df0` | products | chromium-desktop | 1 | no | 0.5625% | (0,0)-(561,900) | ✅ REAL (low-contrast hero tweak) |
| `7a96bee` | products | chromium-desktop | 3 | no | 0.3514% | (104,112)-(1216,689) | ✅ REAL (subtle contrast change) |

The other 19 route-pairs passed the cheap screen with Hamming > 1 (real diffs) and were not
individually opened — they are not at risk of the identical-pair failure (a non-zero Hamming
on a 16×16 hash already proves the pixels differ).

### Tally

| outcome | count | of 29 |
|---|---|---|
| ✅ genuine before/after diff | **25** | 86% |
| ❌ **broken** — zero pixel diff (MD5-identical) | **3** | 10% |
| ⚠️ real-but-not-representative (counter-only delta) | **1** | 3% |
| missing image(s) | 0 | 0% |

**The 3 broken pairs by commit/route:**

1. `382886e` `/products §6` (chromium-desktop) — the user's reported case.
2. `3889ae6` `/highway §4` (chromium-desktop).
3. `3d56df0` `/highway §4` (chromium-desktop) — note: its after-state file is the same
   `3d56df0/highway.jpg` that serves as `3889ae6`'s **before**, so the identical highway §4
   frame propagates across **three consecutive states** (`02359cd` → `3d56df0` → `3889ae6`).

### Honesty / coverage note

- All 29 pairs had both images on disk and were screened. The 10 screen-flagged pairs were
  each opened and visually verified — no blanket "the rest are clean" beyond the Hamming>1
  pairs, which are clean *by construction* (a positive Hamming distance is proof of pixel
  difference; it cannot be a false "broken").
- The 19 unopened pairs could still have the *milder* defect of being a real diff that
  doesn't represent the card's described change (the `3d56df0 blog §2` failure mode). I did
  not exhaustively eyeball all 19 against their card copy; that mild class is plausibly
  present elsewhere but is a labeling weakness, not the identical-pair trust-breaker the
  user flagged.
- Multi-page / multi-part commits were prioritized as requested: `02359cd` (7 routes),
  `3d56df0` (5), `3889ae6` (4), `a553e1a` (3), `382886e` (2) were all examined; the broken
  pairs concentrate in the multi-route commits, exactly where mis-targeting is most likely.

---

## 3. Root cause + spread

The capture pipeline:
`scripts/dashboard/rerender_git_pairs.py` (the `PLAN` list + the per-state build/serve) →
`tests/git-rerender-capture.spec.ts` (the Playwright capture, driven by `GR_ROUTE` +
`GR_SECTION`). For each commit it builds the parent-state and the commit-state in a sandbox
worktree, serves each, and captures **one section** of one route per project. A section is
selected by a **1-based `GR_SECTION` index** that the spec maps to a DOM section and clips.

**All three broken pairs share one umbrella cause: the captured frame does not contain the
pixels the commit changed**, so the unchanged region renders identically in both builds and
the JPEGs come out equal. There are two distinct underlying bugs:

### Bug A — off-by-one section index (Hero is an uncounted `<section>`) → `382886e /products`

The spec counts sections with `main.content-main > section` (plus `> .blog-hero`). On
`/products` the section list is:

```
index 0  Hero  (the <Hero> component ALSO renders as <section> — products.astro:147)
index 1  Level 2 vs Level 3
index 2  bg-surface #level-2
index 3  bg-teal-deep #level-3
index 4  bg-teal
index 5  bg-teal-deep "Four brands. One backend." (footer counter "06 / 06")  ← captured
index 6  bg-teal     ".cta-split — Build a quote"  (NO counter; the 7th <section>)  ← wanted
```

The `PLAN` sets `{"sha":"382886e","route":"/products","section":6}`. The spec computes
`i = min(6, sectionCount) - 1 = 5` → it captures **index 5** (the brands grid), not the
`.cta-split` at **index 6**. The `PLAN` comment literally reasons "6 top-level `<section>`s
→ §6," but it (a) counted the page's own `sec-counter` labels, which stop at "06 / 06," and
(b) **did not count the `<Hero>`** (which is a real `<section>` DOM node) and **did not count
the unnumbered `.cta-split` CTA** that sits *after* the 06/06 section. The `.cta-split` is the
7th DOM `<section>`, so the correct value is `section: 7`, not `6`. Off-by-one, caused by
mixing the page's display-numbering with the DOM `<section>` count.

(Contrast: `382886e /fleets §4` works because fleets is a `FullPageLayout` page where the
`.cta-split` happens to sit inside the section-4 panel that the capture lands on.)

### Bug B — the changed element is at the section's bottom edge, clipped out of the panel → `3889ae6 /highway §4` and `3d56df0 /highway §4`

`3889ae6` (and the `3d56df0` checkpoint that precedes it) change `.sec-foot` — the
section-**footer** counter row's text color (`rgba(255,255,255,0.4)` → `0.6`, an AA contrast
lift). The `PLAN` targets `/highway §4` (the "hardware" panel). The capture lands on the
**top** of the hardware section (the "Infrastructure-grade hardware" spec-card view); the
`.sec-foot` row ("— Hardware · NEVI-spec · GSA-approved   04 / 07") lives at the **bottom**
of that section and is **not in the captured frame** (highway is a `FullPageLayout` page;
the panel paints the upper region). The only thing the commit changed is therefore off-screen
in both builds → byte-identical capture.

This is a *frame-extent / element-position* mismatch, distinct from Bug A's *wrong-index*
mismatch, but the user-visible symptom is the same: an identical before/after slider.

### Why `3d56df0 /blog §2` is the milder "real-but-not-representative" class

The blog page gained a 7th section in the 640-file checkpoint, so the section counter at the
captured frame's corner changed `02/06` → `02/07`. That *is* a real pixel diff (so it is not
"broken"), but it does **not** show "blog equal-height cards" (the change the card
advertises) — those live in a different section, or the §2 "Featured" frame is a single hero
rather than the 3-card grid. Net: the slider "works" (something moves) but is misleading
about what changed. This is the labeling-weakness tail noted under coverage.

### Spread summary

- **Identical-pair (trust-breaking) defect: 3 / 29 route-pairs (10%)**, across 3 commits
  (`382886e`, `3889ae6`, `3d56df0`) and 2 routes (`/products`, `/highway`). Because
  `3d56df0/highway.jpg` is reused as `3889ae6`'s before-frame, the same dead highway §4 frame
  poisons two adjacent commit cards.
- **Bug A (off-by-one on `ContentLayout` + `<Hero>` pages)** is the more dangerous class going
  forward: any `ContentLayout` page (`/products`, `/contact`, `/blog`) whose target section is
  computed from the page's *display* counter rather than the *DOM `<section>`* count is at
  risk. `382886e /products` is the one instance that actually fired, but the same `PLAN`-
  authoring method was used for every entry.
- This is **not** a "checked vs unchecked" issue — the broken sliders are baked from the same
  index regardless of the user's keep/undo state, so an unchecked item is just as exposed as
  a checked one. The risk is uniform across the rollback page.

---

## 4. Fix recommendation

Ordered by leverage. (All in `scripts/dashboard/rerender_git_pairs.py` + the capture spec /
its consumers — note another process is currently editing
`scripts/dashboard/generate_dashboard.py`; none of the fixes below need to touch that file.)

1. **Immediate (unblocks the user's exact complaint): correct the `382886e` products section
   index `6 → 7`** in the `rerender_git_pairs.py` `PLAN`, and re-run the harness for that
   commit (or just that state-pair). This makes the products slider show the real
   `.cta-split` z-order fix. While there, re-target the two highway `.sec-foot` entries
   (`3889ae6`, `3d56df0` `/highway §4`) so the changed footer row is in frame — either point
   them at a whole-page / full-section frame (`section: 0`) or add a capture mode that scrolls
   the section's **bottom** into view before clipping.

2. **Structural — kill Bug A: stop hand-authoring section indices from display numbering.**
   The `PLAN` should target sections by a **stable selector or `data-anchor`**, not a fragile
   integer that silently disagrees with the DOM `<section>` count once `<Hero>` and unnumbered
   CTA sections are present. Resolve the element at capture time (`page.locator('.cta-split')`
   / `[data-anchor="…"]`) and clip to *its* bounding box. This removes the entire class of
   "captured the neighbor section" bugs.

3. **Structural — kill Bug B: clip to the changed element, not the whole panel.** For
   bottom-of-section changes like `.sec-foot`, a panel-top clip will always miss them. Either
   capture the full section height (already partly supported via the `scrollHeight` clip, but
   `FullPageLayout` panels defeat it) or, better, drive the clip from the *changed selector*
   per item (same mechanism as fix 2).

4. **Add a build-time integrity gate (prevents silent regressions).** After the harness
   writes `index.json`, assert for every route-pair that
   `md5(before_img) != md5(after_img)` (and ideally that a full-res pixel diff exceeds a small
   floor, e.g. > 0.05% changed pixels). On failure, mark that route's pair with a `note`
   ("identical before/after — capture did not surface a diff; falling back to code-only card")
   and have the dashboard **suppress the slider** for that pair rather than show a dead one. A
   dead slider is worse than an honest "no visual diff captured" note. This is the durable fix
   — it converts the discipline-fragile "author the right section index" into a
   structurally-enforced invariant.

5. **Audit the remaining 19 unopened pairs for the mild class** (real diff that doesn't match
   the card copy, à la `3d56df0 blog §2`) once the trust-breakers above are fixed. Lower
   priority — a labeling weakness, not a trust breaker.

---

## Appendix — exact file paths used as evidence

- Index / model: `reports/git-rerender/index.json`,
  `reports/dashboard-build/render-model.json`.
- Capture harness: `scripts/dashboard/rerender_git_pairs.py` (the `PLAN`, the off-by-one
  comment at the `382886e` entry), `tests/git-rerender-capture.spec.ts` (the `GR_SECTION`
  index → `min(SECTION,count)-1` clip logic), `scripts/dashboard/build_render_model.py` (the
  consumer route-map).
- Reported case images:
  - products before: `reports/git-rerender/d0a2ea64593902883392ea1a1f331522b2a7838b/chromium-desktop/products.jpg`
  - products after:  `reports/git-rerender/382886ed541abb8d1c07703fbbc6e1cb2af5f8f0/chromium-desktop/products.jpg`  (MD5-identical to the before)
  - fleets before:   `reports/git-rerender/d0a2ea64593902883392ea1a1f331522b2a7838b/chromium-desktop/fleets.jpg`
  - fleets after:    `reports/git-rerender/382886ed541abb8d1c07703fbbc6e1cb2af5f8f0/chromium-desktop/fleets.jpg`  (genuine diff)
- Other broken-pair images:
  - `reports/git-rerender/3d56df0b9f19e3f0da72f002970f91bdc6943f39/chromium-desktop/highway.jpg`
    (== `reports/git-rerender/3889ae6e8e0f7656b77610a83deab0f6e149443e/chromium-desktop/highway.jpg`,
    and == `reports/git-rerender/02359cd384ee7cc89c449940c74d4641c028abf3/chromium-desktop/highway.jpg`).
- Page sources at `382886e`: `src/pages/products.astro` (Hero at :147, `.cta-split` at :444),
  `src/components/Hero.astro` (root `<section>` at :40), `src/layouts/ContentLayout.astro`
  (`main.content-main` at :19), `src/layouts/FullPageLayout.astro` (`#fullpage` at :20).
