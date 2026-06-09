---
id: A90
title: Git rollback page — review-worthiness classification + per-commit copy + image-route correctness
type: audit
status: completed
mode: read-only
created: 2026-05-31
audited_by: subagent (BG dispatch, read-only — real git diffs read, render-model inspected, nothing applied)
hard_timeout: 45min
scope: >
  The git/rollback page (#page-git) is the user's MAIN BOTTLENECK. This audit re-derives its three
  broken layers FROM THE REAL DIFFS: (1) the review-worthy-vs-mechanical split, (2) the per-commit
  copy, (3) the per-commit image/route. Produces a per-commit implementation spec for the next pass.
cross_refs:
  - context/markdowns/goal-billboard/archived/audits/A88-git-visible-change-reclassification.md  # the VISIBLE/INVISIBLE axis (orthogonal — see §0)
  - context/markdowns/goal-billboard/archived/audits/A85-git-rollback-safety-audit.md
  - context/markdowns/goal-billboard/archived/audits/A87-manifesto-realignment.md
  - scripts/dashboard/build_render_model.py   # _classify_one + _MECH_SUBJECT_SIGNALS + _A88_VISIBLE + _pick_best + attach_captures + _decision_intent (READ-ONLY this pass)
  - scripts/dashboard/parse_git_timeline.py   # CURATED source_class map + infer_routes
  - reports/dashboard-build/render-model.json # look_back.git_timeline.commits (current per-commit category/route/thumb/pair)
  - reports/git-rerender/index.json           # the 13 re-rendered isolated before/after pairs
axes_note: >
  THREE distinct axes now exist for these commits. Do not conflate:
   • REVIEW-WORTHINESS (this audit, §1): "is this a judgment call the user should reconsider?"  → category review|mechanical|floor|infra
   • VISIBLE-vs-INVISIBLE (A88, §0): "does a person SEE a difference on the live page?"           → visible yes|subtle|no|floor
   • These are ORTHOGONAL: a commit can be VISIBLE but MECHANICAL (a responsive crop fix — you
     see it, but it wasn't a subjective call) or INVISIBLE but REVIEW (a nav-comment decision).
---

# A90 — Git rollback page: logic + content + image spec (the implementation brief for the next pass)

**Premise (user, 2026-05-31).** The git/rollback page lets the user review Claude's ~24 early
pre-Cursor commits and decide keep-vs-flag-to-undo per commit. Three problems remain:
1. The **review-vs-mechanical division is probably wrong** — several of the 17 "mechanical" feel
   judgment-worthy like the 6 "review."
2. The **per-item text is too generic/templated** ("something was clipped, mis-stacked, or out of
   place") and confusing.
3. The **images are mismatched** — the collapsed thumbnail shows the BLOG page on commits that
   have nothing to do with the blog.

This audit grounds every verdict in the ACTUAL `git show <sha>` diff (not the commit subject, which
the current classifier over-trusts) and in the live `render-model.json`. **Recommend only — nothing
applied.** Another BG is editing the dashboard files; this audit stayed strictly read-only on them.

---

## §0 — What the system ALREADY has (so this spec is a delta, not a rebuild)

Reading the live code + model, the git-page is **further along than the user's framing assumes**.
Confirmed present in `build_render_model.py` + `render-model.json` today:

- **The A88 VISIBLE axis is already wired.** `_A88_VISIBLE` (lines 736–762) carries a curated
  `visible ∈ {yes,subtle,no,floor}` + `kind` + `routes` per commit; `attach_visible_and_pairs`
  (923) writes `c["visible"]`, `c["visible_kind"]`, `c["visible_note"]`, `c["visible_routes"]`.
  `visible_counts = {yes:13, subtle:7, no:7, floor:1}`. **So A88's recommendation is built.** A90
  does NOT touch the visible axis — it is the review-worthiness axis + copy + images.
- **Real before/after PAIRS exist.** `reports/git-rerender/index.json` holds **13 re-rendered
  isolated pairs** (exact parent→commit, the right route×device): `3889ae6, acb5648, a553e1a,
  ef6faec, 382886e, 1389aad, c19645e, 153b4fc, 1158d35` + 4 non-site. `attach_visible_and_pairs`
  prefers these (`method:"rerender"`), else falls back to an existing-run time-bracket
  (`method:"existing-run"`). **The PAIR resolves to the CORRECT route on every commit** (verified
  §3). The bug is NOT the pair — it is the collapsed THUMBNAIL (`best_capture`).
- **A category split exists.** `classify_commits` → `_classify_one` produces
  `category_counts = {review:6, mechanical:17, floor:1, infra:4}` — i.e. **28 site commits**, not
  24 (the page grew by the 4 dashboard-build "infra" commits + extra perf since A88's 24).
- **Per-commit plain-language copy is GENERATED, not authored.** `_decision_intent` (1252) +
  the decision-brief builder template the copy from `(category, source_class, subject, visible_kind)`.
  This is the source of CONCERN 2's genericness — it is literally a switch over coarse buckets.

The corrected commit universe (chronological, site-touching, from `git log --reverse -- src/ public/`):

| # | sha | date | subject (truncated) |
|---|---|---|---|
| 1 | `b77620d` | 05-21 | first commit (the v1 floor) |
| 2 | `02359cd` | 05-26 | "developing agentic multiplatform…" — the 102-file restructure |
| 3 | `3d56df0` | 05-27 | snapshot 48h of sprint work as ratchet point |
| 4 | `3889ae6` | 05-27 | phase a plow — 3 desktop bugs fixed in parallel |
| 5 | `c9f52fe` | 05-27 | nav: drop multifamily v2, shorten highway label |
| 6 | `1da5463` | 05-27 | phase B: .sec-foot.dark + .econ-model span em alpha |
| 7 | `420b377` | 05-27 | nav: pin multifamilyv2 internal-only (chamber decision) |
| 8 | `acb5648` | 05-27 | P1-07a: /highway §04 Tellus card light-panel tonal |
| 9 | `767e019` | 05-27 | P2-12: extract reusable Input.astro + rewire /contact |
| 10 | `7a96bee` | 05-27 | P2-09: bump text-on-dark-mute α + on-surface-variant |
| 11 | `2df2577` | 05-27 | perf: font CSS non-blocking via media=print onload |
| 12 | `08c4c5e` | 05-27 | fix: add src/pages/blog/[slug].astro (17 posts 404ing) |
| 13 | `c9a20d2` | 05-27 | fix: rewire 6 dead /blog post-link placeholders |
| 14 | `a553e1a` | 05-27 | CB-1: hero scrim hardening (products/highway/services §01) |
| 15 | `ef6faec` | 05-27 | CB-2: contact §02 card affordance |
| 16 | `cda8348` | 05-27 | perf: image pipeline foundation + /contact migration |
| 17 | `cc5dc7c` | 05-27 | perf: /products image migration (17 images) |
| 18 | `dac177e` | 05-27 | perf: /blog + /blog/[slug] image migration (5 PNGs) |
| 19 | `6a36736` | 05-27 | perf: add 5 /blog source entries to image manifest |
| 20 | `382886e` | 05-28 | fix: /fleets §04 + /products §07 caption-stack z-order |
| 21 | `1389aad` | 05-28 | fix: /highway §01 mobile notch-safe + narrow scrim |
| 22 | `c19645e` | 05-28 | fix: /services §05 conEdison logo right-edge crop |
| 23 | `153b4fc` | 05-28 | fix: remove /products §01 mid-frame eyebrow dup |
| 24 | `1158d35` | 05-28 | fix: /products §01 bg-tint tablet @media (641–1024) |
| 25 | `b8280e7` | 05-28 | perf: Hero refactor + secaucus-plaza picture+srcset |
| 26 | `57be1c8` | 05-29 | Dashboard findability: serve the new dash + hub (infra) |
| 27 | `358b565` | 05-29 | User owns 'done': manifesto checklist (infra) |
| 28 | `0f76b39` | 05-29 | Dashboard re-skin: C/Blueprint canonical (infra) |

`6a36736` touches only `public/img-opt/` (generated AVIF/WebP) + `scripts/` → INFRA (no src/). The
parser's CURATED map in `parse_git_timeline.py` still lists stale SHAs (`b3178f4`, `5fd1114`,
`1158d35`-as-`1158d35`, `b8280e7`) and is missing `57be1c8/358b565/0f76b39` — a housekeeping note,
not load-bearing (the model already classifies all 28 correctly via the fallbacks).

---

## §1 — CONCERN 1: corrected per-commit review-worthiness classification

**The axis (define it precisely).** A commit is **REVIEW** (surface it; the user should eyeball the
decision) iff a human exercised **subjective judgment that a different person could reasonably make
differently** — a look-and-feel call, a content/microcopy/IA call, an affordance call, or a
foundational architectural call. A commit is **MECHANICAL** (safe to keep, no judgment) iff it is a
**corrective or routine** change with no subjective latitude — a pure rename/format/comment, a
dependency/build tweak, a measured a11y-contrast bump to hit a ratio, an image-format/srcset
swap, a dead-route/404 wiring fix, or a viewport-responsiveness fix that makes an EXISTING design
render correctly on a screen where it was broken. **FLOOR** = the v1 baseline. **INFRA** = no
`src/` touch (dashboard/build/manifesto plumbing).

**The classification LOGIC that should drive it (real diff content, not keyword heuristics).**
The current `_classify_one` reads the **subject string** first (`_MECH_SUBJECT_SIGNALS` /
`_REVIEW_SUBJECT_SIGNALS` regexes). That is the root defect — the subject lies (A88 proved
`snapshot` and `remove…dup` mislead). The corrected logic keys off the **diff shape**:

1. `source_class==V1` → **floor**.
2. No `src/` file in `file_list` → **infra**.
3. Foundational structural mega-commit (≥40 files OR adds a whole new `src/pages/*.astro` page OR
   a Content-Collection architecture rewrite) → **review** (judgment, surfaced — the A87 §3 C2 fix).
4. Diff touches ONLY: comment lines / a pure component-extraction with byte-identical emitted DOM /
   image-format(`<picture>`/srcset/avif/webp) with unchanged `src` / generated `-opt` assets /
   font-loading mechanics / a dead-`href`→real-URL rewire / a single measured contrast α/hex value
   to hit a stated ratio → **mechanical**.
5. Diff adds/edits **rendered copy or markup text content** (a headline, eyebrow, label, nav item),
   or adds **aesthetic CSS chosen for look (not to hit a measured threshold)** — new gradient
   *moods*, brand-color washes, decorative borders, hover/affordance treatments, saturate filters
   on photos → **review**.
6. A viewport `@media`/clamp/safe-area/z-order/crop fix that makes an **already-decided** design
   render correctly on a broken screen (no new aesthetic choice) → **mechanical**.
7. Uncertain → **review** (the user's safe default).

The discriminator between rules 5 and 6 is the question **"did Claude invent a look, or restore an
intended one?"** — read the diff, not the verb.

### §1a — Corrected classification table (all 28)

Columns: **current** (model today) · **A90 verdict** · **Δ** (change?) · **why (review-worthiness)**
· **quoted diff signal** · **A88 visible** (cross-ref, NOT the basis).

| sha | current | **A90** | Δ | Why (judgment vs mechanical) | Diff signal (quoted) | A88 vis |
|---|---|---|---|---|---|---|
| `b77620d` | floor | **floor** | — | The entire first build; reverting undoes the site. Not a candidate. | 78 adds, 5769 ins, 0 del | floor |
| `02359cd` | review | **review** | — | Foundational: blog→Content-Collection rewrite, **+17 post bodies**, **a NEW `multifamilyv2.astro` (225 lines)**, whole-tree image re-folder, app.css +155. A pile of architecture+content calls. | `A  src/pages/multifamilyv2.astro`; `public/img/{ => _archive}/…` (R100 ×dozens) | yes |
| `3d56df0` | mechanical | **REVIEW** | **→review** | Subject "snapshot/ratchet" is misleading; the SITE diff is real layout judgment: **equal-height blog cards** (`height:100%`+`flex:1`+`min-height:3.75em`) + a unified eyebrow-inset token. Card-height balancing is a design call. | `flex: 1;` "the three cards now align bottom-to-bottom"; `left: var(--eyebrow-padding-inline)` | yes |
| `3889ae6` | mechanical | **mechanical (low-conf — borderline)** | — | Corrective: a measured contrast lift (`0.4→0.6`, "3.52:1 → ~6.9:1") + a grid `align-items:start` realign + input border `1.5px→2px`. Intent is fix-the-defect, values are measured. **BUT** the hero `object-position 60%→30%` + scrim darkening (next row's sibling) shade into aesthetics — flag low-confidence. | `measured 3.52:1 … Lifted to 0.6 → ~6.9:1`; `align-items: start; /* prevents the right…` | yes |
| `c9f52fe` | review | **review** | — | Removes a nav item ("Multifamily v2") + relabels "Highway / NEVI"→"Highway". A content/IA call on every page's header. | nav array edit + label string change | yes |
| `1da5463` | mechanical | **mechanical** | — | Two alpha bumps (`.sec-foot.dark 0.5→0.75`, `.econ-model em 0.4→0.6`) for legibility. Routine contrast tune; no new look. | `-…0.4…  +…0.6…` two-line diff, 2 files | subtle |
| `420b377` | review | **review (decision) — but flag as INVISIBLE-decision** | keep | Comment-only diff (the nav code already changed in `c9f52fe`). It IS a real *decision* (chamber: pin internal-only) → keep in review, but it renders nothing — pair NONE (A88 agrees). | comment-block rewrite only; 0 code lines | no |
| `acb5648` | review | **review** | — | Tellus cards get a **green hairline border, a green atmospheric wash gradient, AND `saturate(0.5)` on vendor photos**. A deliberate brand-aesthetic treatment. | `border-top: 1px solid var(--primary)`; `filter: saturate(0.5)`; "gradients model atmosphere" | yes |
| `767e019` | mechanical | **mechanical** | — | Extracts `Input.astro`; emits "the exact `.field`/`label`/`input` DOM … without any CSS change." Byte-identical render. Textbook refactor. | `A src/components/forms/Input.astro` + contact.astro -28 | no |
| `7a96bee` | mechanical | **mechanical** | — | Token contrast bump (`α 0.5→0.6`, `--on-surface-variant #5c6a75→#505c65`) "per Mode-9 measurement." Measured a11y; no aesthetic latitude. | `#5c6a75→#505c65 (AA+margin)` | subtle |
| `2df2577` | mechanical | **mechanical** | — | Makes font CSS non-blocking (`media="print" onload`). Pure load-perf mechanic; steady page identical. | `media="print" onload` one-file +12 | subtle |
| `08c4c5e` | mechanical | **mechanical** | — | Adds `blog/[slug].astro` so 17 posts stop 404ing. A correctness/wiring fix (route that should exist now exists) — not a look choice. | `A  src/pages/blog/[slug].astro` | yes |
| `c9a20d2` | mechanical | **mechanical** | — | Rewires 6 dead `href="#"`→`/blog/${id}`. Pure wiring; identical at rest. | `-href="#"  +href={…/blog/${id}}` ×6 in blog.astro | subtle |
| `a553e1a` | mechanical | **mechanical (low-conf — borderline)** | flag | Corrective scrim hardening so hero text reads (cross-engine defect, "eyebrow + body contrast borderline"). **BUT** the chosen gradient stops change the photo's mood — defensibly a look call. Keep mechanical, mark low-confidence + offer "move to review." | `0.18 → 0.32 … 28% → 34%`; new `.highway-hero .bg-tint{…0.70 65%…}` | yes |
| `ef6faec` | review | **review** | — | Contact route-cards gain **white fill, visible border, hover-lift + soft shadow, GO underline** — "the entire card reads as an interactive target." An affordance/visual decision. | `border:1px solid rgba(26,47,58,0.18)`; "hover state adds a transform-lift + soft shadow" | yes |
| `cda8348` | mechanical | **mechanical** | — | `/contact` photo → `<picture>` + `width/height`. Format/CLS mechanic; same photo. | `<picture>` + `width="1200" height="1600"` | subtle |
| `cc5dc7c` | mechanical | **mechanical** | — | `/products` 17 imgs → srcset + dims. Format/CLS mechanic. (Reflection now `-320.webp` — sharpness caveat, still mechanical.) | `width="250" height="350"` + srcset ×17 | subtle |
| `dac177e` | mechanical | **mechanical** | — | `/blog` 5 PNGs → srcset + dims. Format/CLS mechanic. | `width="1537" height="1023"` + srcset | subtle |
| `6a36736` | infra | **infra** | — | Only `public/img-opt/` generated variants + `scripts/`. No src/. | 0 src files; `public scripts` only | no |
| `382886e` | mechanical | **mechanical** | — | `/fleets §04 + /products §07` caption-stack z-order/flex fix — un-collides text. Corrective layout, no new look. | `z-index:2; display:flex;flex-direction:column;gap:10px` | yes |
| `1389aad` | mechanical | **mechanical** | — | `/highway §01` mobile: notch-safe `top: max(clamp(…), safe-area-inset-top+12px)` + darker narrow scrim. Responsiveness fix on a broken screen. | `safe-area-inset-top` + `@media(max-width:600px)` scrim | yes |
| `c19645e` | mechanical | **mechanical** | — | `/services §05` logo `max-width:140px → min(140px,100%)` — un-crops the wide logo on phone. Pure responsive fix. | `max-width: min(140px, 100%)` | yes |
| `153b4fc` | review | **review** | — | **Removes a line of rendered copy** — the `— The lineup` eyebrow on `/products` hero (cites "redundant hierarchy/microcopy, axis 9+12", a "systemic decision"). A content/IA judgment call. | `-<span class="eyebrow no-rule">— The lineup</span>` | yes |
| `1158d35` | mechanical | **mechanical** | — | New `@media (641–1024px)` band giving tablets an intermediate scrim. A viewport range that fell through the cascade now renders — responsive fix. | `@media (min-width:641px) and (max-width:1024px){ .prod-hero .bg-tint…}` | yes |
| `b8280e7` | mechanical | **mechanical** | — | Hero `<picture>` centralization; `<img src>` unchanged ("transparent to layout … zero regression"); 50 generated `-opt` variants. Format optimization. | unchanged `src`; +50 `-opt` assets | no |
| `57be1c8` | infra | **infra** | — | Dashboard/hub serving. No src/. | `context public` only | no |
| `358b565` | infra | **infra** | — | Manifesto checklist + standing rule. No src/. | `.claude context public` only | no |
| `0f76b39` | infra | **infra** | — | Dashboard re-skin. No src/. | `.claude context public scripts` only | no |

### §1b — The delta from the current split (the answer to "is it really 6, or 9/12?")

- **Net REVIEW change: 6 → 7.** The single firm reclassification is **`3d56df0` mechanical→review.**
  Its subject ("snapshot 48h … ratchet point") tripped `_MECH_SUBJECT_SIGNALS`'
  `\bsnapshot\b|ratchet point` rule, but its *site* diff is **equal-height blog-card balancing +
  unified eyebrow inset** — a layout-aesthetic decision. This is the clearest "the classifier read
  the message, not the change" case on the review axis (A88 caught the same commit on the visible
  axis). **Recommended firm review set (7):** `02359cd, 3d56df0, c9f52fe, acb5648, ef6faec,
  153b4fc, 420b377`.

- **Two BORDERLINE (low-confidence mechanical, offer "move to review"): `3889ae6` and `a553e1a`.**
  Both are *corrective* scrim/contrast work driven by measured cross-engine defects (legitimately
  mechanical by intent), but both **chose specific gradient/opacity/object-position values that
  visibly change a hero photo's mood** — which is the exact gray zone the user is sensing. The
  honest call is: **keep them mechanical, but render them with `confidence:"low"` and a one-tap
  "move to review" affordance**, with copy that says *why* it's borderline (see §2). If the user
  wants the aggressive read, the review set is **9** (`+3889ae6 +a553e1a`). It is NOT 12 — the
  remaining mechanical commits (`1da5463, 7a96bee, 2df2577, 08c4c5e, c9a20d2, cda8348, cc5dc7c,
  dac177e, 382886e, 1389aad, c19645e, 1158d35, 767e019, b8280e7`) are genuinely mechanical: measured
  contrast bumps, format/srcset swaps, dead-route wiring, notch/crop/z-order responsive fixes, and
  two true invisible refactors. None involve inventing a look or editing copy.

- **Why the user *feels* like more than 6 are judgment-worthy (and the real fix).** The confusion is
  mostly the **VISIBLE axis leaking into the review framing**, not a misclassification.
  **15 of the 17 "mechanical" are VISIBLE** (A88). Burying 15 visible commits under "Mechanical —
  safe to keep" makes the page feel like it's hiding substantial work. **The fix is presentational,
  not a re-bucketing:** the page should LEAD with `visible != no` (the 20 you can see), and use
  review-vs-mechanical as a *secondary* "needs-your-judgment" badge — not as the primary
  keep/hide gate. Concretely: 7 review (+2 borderline) on the judgment axis; 20 visible on the
  see-it axis; the page should make both legible without implying "mechanical = nothing happened."

- **Classifier change to implement it (next pass).** Replace `_classify_one`'s subject-first order
  with the §1 diff-shape logic. Minimum viable fix that captures the one firm delta: **add a
  `_FOUNDATIONAL_REVIEW`-style curated override for `3d56df0` → review** ("equal-height blog cards +
  eyebrow-inset — a layout call, mis-read as a checkpoint by its subject"), exactly as `02359cd`
  is already overridden. Add `confidence:"low"` + a `borderline_review:true` flag to `3889ae6` and
  `a553e1a`. Do NOT move the measured-contrast / format / wiring / responsive commits.

---

## §2 — CONCERN 2: per-commit copy spec (specific, not templated)

**The defect.** `_decision_intent` (build_render_model.py:1252) generates copy from coarse buckets.
The literal offending string is line 1283: *"A fix for something that was displaying wrong —
clipped, mis-stacked, or broken on some screens."* Every SCRUTINY/`fix`-prefixed mechanical commit
gets that identical sentence — which is why the page reads as templated and confusing.

**The fix.** Author **specific copy per commit**, keyed by SHORT sha (the same curated-map pattern
the code already uses for `_A88_VISIBLE` and `_FOUNDATIONAL_REVIEW`). Recommended new structure:
`_COMMIT_COPY: dict[str, dict]` with fields `what` / `why` / `choosing` / `reach` / `if_undo`. The
generic `_decision_intent` stays ONLY as the fallback for future (post-handoff) commits not in the
map. Below is the authored copy for all 28 (plain language, no git terms, no "clipped/mis-stacked").

> Field meanings: **what** = what THIS commit actually changed · **why** = why it's here ·
> **choosing** = what the user is deciding · **reach** = which page(s) it shows on (honest about
> shared code) · **if_undo** = what reverting does.

**1. `b77620d` — floor.**
- what: "The very first build of the entire site — every page, image, and style in one go."
- why: "This is the starting point. Everything else is a change layered on top of it."
- choosing: "Nothing to decide here — this is the floor, not a tweak."
- reach: "The whole site."
- if_undo: "Reverting this would erase the entire website. Don't."

**2. `02359cd` — review (foundational).**
- what: "A big early restructure: the blog was rebuilt to pull its 17 articles from a content
  library instead of hand-coded HTML, a second experimental Multifamily page was added, and the
  image folders were reorganised."
- why: "Early scaffolding to make the blog manageable and to try a nav variant — done before the
  careful review process was in place."
- choosing: "Whether you'd rather rebuild the blog's structure (and the experimental page) your own
  way by hand, or keep this foundation."
- reach: "The blog and every article page; a new /multifamilyv2 page; minor touches site-wide."
- if_undo: "Reverting drops the content-library blog and the experimental page — a large undo.
  Best treated as one all-or-nothing unit."

**3. `3d56df0` — review (was mechanical).**
- what: "Made the three blog cards line up to the same height so their bottom edges match, and
  aligned the scroll hint to the page's left edge."
- why: "The cards were ending at different heights, which looked ragged on tablet and desktop."
- choosing: "Whether you like the cards forced to equal height, or prefer them to size to their own
  text."
- reach: "The blog index (card layout); a small alignment tweak on hero scroll hints site-wide."
- if_undo: "The blog cards go back to varying heights."

**4. `3889ae6` — mechanical (borderline).**
- what: "Darkened the photo overlay behind the Products hero text and nudged the background image
  so the charger readout stops colliding with the headline; also straightened the Contact form's
  right column and thickened its input borders slightly."
- why: "On desktop the headline was sitting on a bright part of the photo and was hard to read, and
  the form column was misaligned."
- choosing: "This was a fix for a readability problem, but it does make the Products photo darker —
  worth a glance if you care about that photo's mood. (Tap 'needs my eye' to move it up.)"
- reach: "Products hero and the Contact form, on desktop."
- if_undo: "The Products photo gets lighter behind the headline and the form column shifts back."

**5. `c9f52fe` — review.**
- what: "Removed the 'Multifamily v2' item from the site menu and shortened the 'Highway / NEVI'
  menu label to just 'Highway'."
- why: "To tidy the navigation and hide an experimental page from visitors."
- choosing: "Whether those are the menu names and items you want."
- reach: "The header menu, on every page."
- if_undo: "The extra menu item and the longer label come back."

**6. `1da5463` — mechanical.**
- what: "Made two bits of faint grey text a little darker: the small section-footer labels and the
  fine print under the Highway economics figures."
- why: "The text was too light to read comfortably."
- choosing: "Nothing subjective — a routine legibility bump."
- reach: "The Highway economics block, plus footer labels wherever they appear, on desktop."
- if_undo: "That text goes back to lighter grey."

**7. `420b377` — review (decision; invisible).**
- what: "Recorded the decision to keep the experimental Multifamily page available internally but
  out of the public menu. (This commit only updates a note in the code — the page itself didn't
  change here.)"
- why: "To document the chamber decision so it isn't undone by accident later."
- choosing: "Whether you agree with keeping that page internal-only."
- reach: "Nothing visible — it's a note in the code."
- if_undo: "Removes the note. Nothing on the site changes."
- present-as: NO before/after (see §3 — invisible-decision card).

**8. `acb5648` — review.**
- what: "Gave the Highway 'Tellus' product cards a green hairline along the top, a faint green
  glow over the top of each card, and toned down the bright magenta/cyan in the product photos."
- why: "To make those cards feel on-brand and calmer."
- choosing: "Whether you like this green-tinted, muted-photo treatment on those cards."
- reach: "The Highway page, section 4 (Tellus cards), on desktop."
- if_undo: "The green accent and glow go away and the product photos return to full saturation."

**9. `767e019` — mechanical (invisible).**
- what: "Reorganised the code behind the Contact form's input fields into one reusable building
  block. The form looks and behaves exactly the same."
- why: "So future forms can reuse the same field design without copy-pasting."
- choosing: "Nothing — the page is pixel-identical."
- reach: "None — the Contact form renders the same."
- if_undo: "The code goes back to the older copy-pasted layout. No visible change."
- present-as: NO before/after (code-only card).

**10. `7a96bee` — mechanical.**
- what: "Made muted text on dark backgrounds a touch stronger, and darkened one grey UI colour, to
  meet readability targets."
- why: "Measured contrast was just below the accessibility threshold."
- choosing: "Nothing subjective — a measured accessibility bump."
- reach: "Muted text across several pages, on desktop."
- if_undo: "That text goes back to slightly fainter."

**11. `2df2577` — mechanical (subtle/load).**
- what: "Changed how the site's fonts load so pages paint faster. The finished page looks the same;
  on a slow first visit you may briefly see a fallback font before the real one swaps in."
- why: "Font files were blocking the page from showing — this speeds up first paint."
- choosing: "Nothing — a routine speed tune-up."
- reach: "Every page, only during the first (cold) load."
- if_undo: "Pages paint a little slower but with no font flash."
- present-as: subtle/load — show pair with the 'font-swap on load' note (§3).

**12. `08c4c5e` — mechanical.**
- what: "Built the article-page template so all 17 blog posts actually open. Before this, clicking a
  post led to a 'page not found' error."
- why: "The article pages had never been created — the links went nowhere."
- choosing: "Nothing subjective — these articles should open, and now they do."
- reach: "All 17 blog article pages."
- if_undo: "Every blog article goes back to a 'not found' error."
- present-as: after-only per post (before = the 404 page).

**13. `c9a20d2` — mechanical (behavioral).**
- what: "Pointed six blog links that did nothing to their real article pages. The links look the
  same; now they actually go somewhere when clicked."
- why: "They were placeholder links left pointing at nothing."
- choosing: "Nothing — they should work, and now they do."
- reach: "The blog index links."
- if_undo: "Those six links stop working again."
- present-as: behavioral — pair looks identical at rest; show the 'links now navigate' note (§3).

**14. `a553e1a` — mechanical (borderline).**
- what: "Deepened the dark overlay behind the hero text on Products, Highway, and Services so the
  headlines read clearly over the photos."
- why: "On some browsers the headline and body text were hard to read against bright parts of the
  hero photos."
- choosing: "This was a readability fix, but it does make those three hero photos darker — worth a
  look if the photo mood matters to you. (Tap 'needs my eye' to move it up.)"
- reach: "The hero sections of Products, Highway, and Services, on desktop."
- if_undo: "Those three hero photos get lighter behind the text."

**15. `ef6faec` — review.**
- what: "Turned the three Contact route cards into clearly clickable cards — a white fill, a visible
  border, a lift-and-shadow on hover, and an underline on the 'GO' link."
- why: "The cards didn't look clickable — they read as plain text."
- choosing: "Whether you like this card-and-hover treatment for those three cards."
- reach: "The Contact page, the three-card row, on desktop (hover effect on desktop)."
- if_undo: "The cards go back to looking like plain panels with no hover."

**16. `cda8348` — mechanical (subtle/load).**
- what: "Switched the Contact install photo to lighter, modern image formats and reserved its space
  so the page doesn't jump while it loads. Same photo."
- why: "To shrink the image and stop the layout shifting during load."
- choosing: "Nothing — a routine image/loading tune-up."
- reach: "The Contact install photo; you'd only notice it as a steadier load."
- if_undo: "The photo loads as a heavier file and the page may jump slightly while loading."
- present-as: subtle/load — pair looks identical at rest; show the 'layout-shift' note (§3).

**17. `cc5dc7c` — mechanical (subtle/load).**
- what: "Did the same lighter-format + reserved-space treatment for the 17 Products card images.
  Same photos."
- why: "To shrink the images and steady the layout while they load."
- choosing: "Nothing — a routine image tune-up."
- reach: "The Products card grid; noticeable only as a steadier, faster load."
- if_undo: "Those images load heavier and the grid may jump while loading."
- present-as: subtle/load (note possible reflection-sharpness change).

**18. `dac177e` — mechanical (subtle/load).**
- what: "Same lighter-format + reserved-space treatment for the 5 blog images. Same photos."
- why: "To shrink the blog images and steady the layout while they load."
- choosing: "Nothing — a routine image tune-up."
- reach: "The blog thumbnails and article covers; noticeable only as a steadier load."
- if_undo: "Those images load heavier and may jump while loading."
- present-as: subtle/load.

**19. `6a36736` — infra.**
- what: "Generated the lighter image files for the blog and added their entries to the build list.
  Behind-the-scenes companion to the image work above."
- why: "To finish wiring up the blog image optimisation."
- choosing: "Nothing — build plumbing."
- reach: "None on the pages directly."
- if_undo: "Removes the generated files; pair them with the related image commit if undoing."
- present-as: code/infra — no before/after.

**20. `382886e` — mechanical.**
- what: "Fixed overlapping caption text on the Fleets section 4 and Products section 7 — the lines
  were stacking on top of each other; now they stack cleanly with spacing."
- why: "The caption block was colliding when the text wrapped."
- choosing: "Nothing — a fix for text that was overlapping."
- reach: "Fleets section 4 and Products section 7."
- if_undo: "Those captions can overlap again when the text wraps."

**21. `1389aad` — mechanical.**
- what: "Pushed the Highway breadcrumb row below the iPhone notch so it stops getting clipped, and
  darkened the hero overlay on narrow phones."
- why: "On notched phones the top row was hidden under the notch and the headline was hard to read."
- choosing: "Nothing — a phone-display fix."
- reach: "The Highway hero, on phones (especially notched iPhones)."
- if_undo: "The breadcrumb can clip under the notch again on phones."

**22. `c19645e` — mechanical.**
- what: "Stopped the Con Edison logo from getting cut off at its right edge on narrow phone
  screens by letting it shrink to fit."
- why: "The wide logo was being cropped in the two-up layout on small screens."
- choosing: "Nothing — a phone-display fix."
- reach: "The Services partners section, on phones / narrow screens."
- if_undo: "The logo can get clipped again on small screens."

**23. `153b4fc` — review.**
- what: "Removed the small '— The lineup' label that sat above the Products headline, because the
  breadcrumb already says the same thing."
- why: "The same phrase was showing twice — once in the breadcrumb, once as a label."
- choosing: "Whether you want that label gone, or prefer to keep it (or replace it with different
  words)."
- reach: "The Products hero, on all screens."
- if_undo: "The '— The lineup' label comes back above the headline."

**24. `1158d35` — mechanical.**
- what: "Added a mid-size-screen version of the Products hero overlay so the breadcrumb stays
  readable on tablets (a range that previously fell through the cracks)."
- why: "On iPad-sized screens the overlay wasn't kicking in, so the breadcrumb was hard to read."
- choosing: "Nothing — a tablet-display fix."
- reach: "The Products hero, on tablets (portrait)."
- if_undo: "The breadcrumb can be hard to read on tablets again."

**25. `b8280e7` — mechanical (invisible).**
- what: "Reorganised how hero images are delivered and generated lighter versions of the Products
  hero photo. The photo on screen is the same."
- why: "To speed up the Products page without changing how it looks."
- choosing: "Nothing — the hero looks identical."
- reach: "None visible; the Products hero just loads lighter."
- if_undo: "The hero loads as a heavier image. No visible change."
- present-as: NO before/after (code/format-only card).

**26. `57be1c8` — infra.**
- what: "Set up the project's internal dashboard and a hub page. Not part of the public website."
- why: "Tooling for you to review the work."
- choosing: "Nothing — internal tooling."
- reach: "None on the public site."
- if_undo: "Removes the dashboard wiring. The public site is unaffected."
- present-as: code/infra — no before/after.

**27. `358b565` — infra.**
- what: "Added the manifesto checklist (the one where only you tick items off). Not part of the
  public website."
- why: "To track which goals you've personally signed off."
- choosing: "Nothing — internal tracking."
- reach: "None on the public site."
- if_undo: "Removes the checklist. The public site is unaffected."
- present-as: code/infra — no before/after.

**28. `0f76b39` — infra.**
- what: "Re-skinned the internal dashboard. Not part of the public website."
- why: "A visual refresh of your review tooling."
- choosing: "Nothing — internal tooling."
- reach: "None on the public site."
- if_undo: "Reverts the dashboard's look. The public site is unaffected."
- present-as: code/infra — no before/after.

### §2a — Titles flagged as confusing vs their actual content

The page should show a **plain re-title** (the `what` headline) in place of the raw git subject for:
- `02359cd` — subject "developing agentic multiplatform parallelized development tool…" describes
  the SESSION, not the site change. **Re-title: "Rebuilt the blog + added an experimental page."**
- `3d56df0` — subject "snapshot 48h of sprint work as ratchet point" is a checkpoint label; the
  site change is blog-card heights. **Re-title: "Made the blog cards line up to equal height."**
- `3889ae6` — subject "phase a plow lands — 3 desktop bugs fixed in parallel (B-K7-…)" is internal
  shorthand. **Re-title: "Darkened the Products hero overlay + straightened the Contact form."**
- `a553e1a` / `ef6faec` — "CB-1 / CB-2" prefixes are internal codes. **Re-title to the `what`.**
- `1da5463`, `7a96bee` — the raw α/hex/selector subjects are unreadable to a non-coder. Re-title to
  the plain `what`.
- `382886e`/`1389aad`/`c19645e`/`1158d35`/`153b4fc` — `fix(CODE):` prefixes + `§`/selector jargon.
  Show the `what` headline; keep the raw subject available under a "details" toggle.

Keep the raw subject accessible (collapsed), but never lead a card with it.

---

## §3 — CONCERN 3: per-commit image/route spec + the mismatch root cause

### §3a — ROOT CAUSE of the blog-thumbnail bug (verified, with the exact mechanism)

The collapsed THUMBNAIL is `c["best_capture"]`, chosen by `_pick_best(captures, touched_routes)`
in `attach_captures` (build_render_model.py:512, 693). The before/after PAIR is chosen separately by
`attach_visible_and_pairs` using `c["visible_routes"]` (the A88 curated narrow route). **These two
use DIFFERENT route inputs, and that is the entire bug:**

1. For any commit that edits **shared CSS** (`src/styles/app.css`) or a shared component,
   `parse_git_timeline.infer_routes` returns `["*"]` (or `*` plus a page) — see `touched_routes` in
   the model.
2. In `_pick_best`, `shared = "*" in touched` makes `on_touched(c)` return **True for every route**,
   so the "prefer a touched route" score term (`0 if on_touched else 1`) is **0 for all captures**.
3. The remaining tiebreakers (`match==exact`, `has after`, `source==vq`, `device==chromium-desktop`)
   don't distinguish routes, so ordering falls to the **alphabetical dedup sort** (lines 686–690:
   `x.get("route") or ""`). **`blog` sorts first** → the blog capture becomes the thumbnail.
4. `attach_captures` (line 1525) runs BEFORE `attach_visible_and_pairs` (line 1545), so
   `visible_routes` doesn't exist yet when `_pick_best` runs — it CAN'T use the right route.

**Verified blast radius: 14 of 28 commits have a thumbnail on the wrong route** (all showing
`blog`), while their before/after PAIR already resolves to the CORRECT route:

`b77620d, c9f52fe, 1da5463, acb5648, 767e019, 7a96bee, 2df2577, a553e1a, ef6faec, cda8348,
382886e, 1389aad, c19645e, 1158d35`.

**Crucial confirmation: the correct-route capture EXISTS in the matched run for every one of these**
(each commit's `combos.by_route` contains the right route — e.g. `acb5648` has a `highway` capture
in run `20260527-1728`, but the thumb points at that run's `blog/section-01.jpg`). So this is a
**selection bug, not a missing-image problem** — no re-capture needed, just pick the right one.

### §3b — THE FIX (for the next pass)

Make the thumbnail key off the **visible/curated route**, not `touched_routes`:

- **Option A (preferred, minimal): reorder + re-key.** Move the `best_capture` selection so it runs
  AFTER `attach_visible_and_pairs`, and call `_pick_best(deduped, c["visible_routes"] or
  [r for r in touched_routes if r != "*"] or touched_routes)`. With a concrete narrow route,
  `on_touched` discriminates and the right route wins. (If the commit already has a `pair`, simplest
  of all: **set `best_capture` to the pair's `before` image** so the thumbnail and the opened pair
  always agree.)
- **Option B (in-place, no reorder): pass a route hint into `attach_captures`.** Compute the curated
  route from `_A88_VISIBLE[sha]["routes"]` inside `attach_captures` (it already imports the map's
  data indirectly) and feed it to `_pick_best` instead of `touched_routes`.
- **Either way:** when `touched_routes == ["*"]` and there's no curated route, fall back to a
  representative page (`home`/`index`) rather than alphabetical `blog`. **`blog` must never be the
  default just because it sorts first.**
- **Guard test:** assert that for every commit with a non-empty `visible_routes`, `best_capture.route
  ∈ visible_routes` (this fails today for the 14 above; it should pass after the fix).

### §3c — Per-commit image spec (correct route/device, or NONE)

**route** = the route the thumbnail + (if visible) the pair should show — keyed to the A88 curated
visible route (already in the model as `visible_routes`), NOT `touched_routes`. **pair** = the
existing pair method the model already computed (verified correct). **Δthumb** = is the thumbnail
wrong today?

| sha | A90 image verdict | Correct route × device | Current thumb route | Δthumb | Pair today (already correct) |
|---|---|---|---|---|---|
| `b77620d` | after-only (floor) | **home** (index) × desktop | blog ❌ | **FIX** | none (floor) |
| `02359cd` | pair | **blog** × all | blog ✅ | ok | existing-run (blog) ✅ |
| `3d56df0` | pair | **blog** × iPad+phone | products | **FIX → blog** | existing-run (blog) ✅ |
| `3889ae6` | pair | **products** × desktop | products ✅ | ok | **rerender (products)** ✅ |
| `c9f52fe` | pair | **home/header** × all | blog ❌ | **FIX** | existing-run (home) ✅ |
| `1da5463` | pair (subtle/contrast) | **highway** × desktop | blog ❌ | **FIX** | existing-run (highway) ✅ |
| `420b377` | **NONE** (invisible-decision) | — | blog ❌ | **drop thumb** | none ✅ |
| `acb5648` | pair | **highway** × desktop | blog ❌ | **FIX** | **rerender (highway)** ✅ |
| `767e019` | **NONE** (code-only refactor) | — | blog ❌ | **drop thumb** | none ✅ |
| `7a96bee` | pair (subtle/contrast) | **home** × desktop | blog ❌ | **FIX** | existing-run (home) ✅ |
| `2df2577` | pair (subtle/FOUT) | **home** × all (cold load) | blog ❌ | **FIX** | existing-run (home) ✅ |
| `08c4c5e` | after-only per post | **blog** (/blog/*) × all | blog ✅ | ok | existing-run (blog) ✅ |
| `c9a20d2` | pair (behavioral) | **blog** × all | blog ✅ | ok | existing-run (blog) ✅ |
| `a553e1a` | pair | **products** (+highway/services) × desktop | blog ❌ | **FIX** | existing-run (products) ✅ |
| `ef6faec` | pair | **contact** × desktop | blog ❌ | **FIX** | **rerender (contact)** ✅ |
| `cda8348` | pair (subtle/CLS) | **contact** × all | blog ❌ | **FIX** | existing-run (contact) ✅ |
| `cc5dc7c` | pair (subtle/CLS) | **products** × all | products ✅ | ok | existing-run (products) ✅ |
| `dac177e` | pair (subtle/CLS) | **blog** × all | blog ✅ | ok | existing-run (blog) ✅ |
| `6a36736` | **NONE** (infra) | — | (none) | ok | none ✅ |
| `382886e` | pair | **fleets** (+products) × all | blog ❌ | **FIX** | **rerender (fleets)** ✅ |
| `1389aad` | pair | **highway** × phone (notch) | blog ❌ | **FIX** | **rerender (highway, webkit-iphone)** ✅ |
| `c19645e` | pair | **services** × phone | blog ❌ | **FIX** | **rerender (services, webkit-iphone)** ✅ |
| `153b4fc` | pair | **products** × all | products ✅ | ok | **rerender (products)** ✅ |
| `1158d35` | pair | **products** × tablet | blog ❌ | **FIX** | **rerender (products)** ✅ |
| `b8280e7` | **NONE** (format-only) | — | blog ❌ | **drop thumb** | none ✅ |
| `57be1c8` | **NONE** (infra) | — | (none) | ok | none ✅ |
| `358b565` | **NONE** (infra) | — | (none) | ok | none ✅ |
| `0f76b39` | **NONE** (infra) | — | (none) | ok | none ✅ |

**Summary of image actions:**
- **11 thumbnails to FIX** (wrong route → correct route): `b77620d, c9f52fe, 1da5463, acb5648,
  7a96bee, 2df2577, a553e1a, ef6faec, cda8348, 382886e, 1389aad, c19645e, 1158d35` — and `3d56df0`
  (products→blog). All correct-route captures already exist in the matched run; pure re-selection.
- **3 thumbnails to DROP** (commit is invisible — must NOT show any screenshot): `420b377`
  (comment), `767e019` (refactor), `b8280e7` (format). Today they each wrongly show a blog
  thumbnail, which reads as "broken." See §3d for how to present them.
- **4 infra** already show no thumbnail (correct): `6a36736, 57be1c8, 358b565, 0f76b39`.
- **The 13 re-render pairs are all correct** (right route×device) — do not touch them.

### §3d — How to present the INVISIBLE commits CLEARLY (not a confusing "No visual change")

7 commits have `visible == "no"` (`420b377, 767e019, b8280e7, 6a36736, 57be1c8, 358b565, 0f76b39`).
A blank tile or a bare "No visual change" reads as broken. Specify a distinct, self-explaining card
per kind (the model already carries `visible_kind` — use it to pick the card copy):

- **Comment/decision (`420b377`):** card titled **"A recorded decision — nothing on the page
  changed."** Show the §2 copy + the actual comment diff (it's tiny). No image frame at all.
- **Code refactor (`767e019`):** card titled **"Same page, tidier code."** Show "the page is
  pixel-identical" + the code diff (Input.astro). A small "show code" toggle, no screenshot tile.
- **Image-format optimization (`b8280e7`, `6a36736`):** card titled **"Lighter files, same
  picture."** Optionally show ONE still of the (unchanged) image with a "same image, smaller file"
  caption — explicitly NOT a before/after slider (there's nothing to slide).
- **Dashboard/manifesto infra (`57be1c8, 358b565, 0f76b39`):** card titled **"Behind-the-scenes —
  not part of the public site."** Show the code-diff only, no image frame.

The unifying rule: **an invisible commit gets a labelled explanation of WHY there's no picture, never
an empty/placeholder image tile.** This is the difference between "looks broken" and "reads as
intentional."

---

## §4 — Implementation checklist for the next git-page pass (what to build)

1. **§1 classifier:** add a curated review override for `3d56df0` (→review, "equal-height blog
   cards"); add `confidence:"low"` + `borderline_review:true` to `3889ae6` and `a553e1a`; replace
   `_classify_one`'s subject-first ordering with the §1 diff-shape logic for future commits.
2. **§1 presentation:** lead the page with the VISIBLE axis (the 20 you-can-see commits), use
   review-vs-mechanical as a secondary "needs-your-judgment" badge — stop letting "mechanical" read
   as "nothing happened / hidden." Offer a one-tap "move to review" on the 2 borderline cards.
3. **§2 copy:** add `_COMMIT_COPY` curated per-sha map (what/why/choosing/reach/if_undo) for all 28;
   keep `_decision_intent` only as the fallback for post-handoff commits. Delete the templated
   "clipped, mis-stacked, or out of place" sentence from the lead position.
4. **§2a titles:** show the plain `what` headline instead of the raw git subject for the 9 flagged
   commits; keep the raw subject under a details toggle.
5. **§3 thumbnail fix:** re-key `best_capture` off `visible_routes` (or set it to the pair's
   `before`); fall back to `home`, never alphabetical `blog`; add the guard test
   `best_capture.route ∈ visible_routes`.
6. **§3d invisible cards:** render the 7 `visible=="no"` commits as labelled "why there's no picture"
   cards by `visible_kind`, never an empty image tile.

**Order of impact (the user's bottleneck):** #5 (thumbnail) + #6 (invisible cards) kill the
"images look wrong/broken" confusion with the least code — and require NO re-capture (every correct
image already exists). #3 (copy) + #2 (presentation) kill the "generic/templated/confusing" feel.
#1 (classifier) is the smallest factual change (one firm reclassification + two borderline flags).

---

## Findings ledger

| ID | Finding | Action |
|---|---|---|
| C1-1 | Review split is mostly right; ONE firm miss: `3d56df0` (equal-height blog cards) is review, mis-read as a checkpoint by its subject. | Curated override → review. Net 6→7. |
| C1-2 | `3889ae6` + `a553e1a` are genuine gray-zone (corrective scrim work with aesthetic side-effects). | Keep mechanical, `confidence:low` + "move to review" affordance. Aggressive read = 9, not 12. |
| C1-3 | The "feels like more than 6" intuition is the VISIBLE axis (15/17 mechanical are visible) leaking into the review framing. | Fix is presentational: lead with visible, demote review-vs-mechanical to a badge. |
| C1-4 | Classifier reads the subject first; the subject lies (snapshot / ratchet / fix-codes). | Re-key `_classify_one` to diff-shape. |
| C2-1 | Per-commit copy is templated from coarse buckets; literal "clipped, mis-stacked, or out of place" repeats. | Authored `_COMMIT_COPY` per-sha (§2); fallback only for future commits. |
| C2-2 | 9 commit TITLES are internal jargon / session labels, not what changed. | Show the plain `what` headline; raw subject under a toggle. |
| C3-1 | 14/28 thumbnails show the wrong route (all `blog`) because `_pick_best` uses `touched_routes` (`*` → all match → alphabetical blog wins) and runs before `visible_routes` exists. | Re-key thumbnail off `visible_routes`/pair-before; fall back to home, never blog. |
| C3-2 | The correct-route capture EXISTS in every mismatched commit's matched run — selection bug, not missing image. | No re-capture; pure re-selection + a guard test. |
| C3-3 | 3 invisible commits (`420b377`, `767e019`, `b8280e7`) currently show a blog thumbnail → reads as broken. | Drop the tile; render labelled "why there's no picture" cards by `visible_kind`. |
| C3-4 | The 13 re-rendered before/after pairs are all on the correct route×device. | Leave untouched. |

## What was NOT done (read-only audit)
No edits to `generate_dashboard.py` / `build_render_model.py` / `parse_git_timeline.py` /
`settings.json` / `src/`. No git mutations. No model regeneration, no captures. Dashboard files were
read only (another BG is editing them). Every recommendation here is for the next git-page pass.
