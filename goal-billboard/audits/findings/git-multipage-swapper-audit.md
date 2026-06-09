---
title: "Git-page multi-page before/after SWAPPER — completeness + noticeability + history audit"
audit-id: git-multipage-swapper-audit
date: 2026-05-31
status: read-only-findings
scope: read-only (audit only — the single write is this file). DID NOT edit generate_dashboard.py (another BG owns it). No commits.
boundary: ONLY the multi-page before/after SWAPPER (the expanded `.ba-gallery` route-picker). Collapsed-card visual hierarchy is a SEPARATE BG's scope — not scrutinized here.
trigger: user inspecting 382886e — "i do see an option to swap above the image … thats good but it needs to be a bit more noticeable" + "how many other items are victim of this previously warned error."
verified-against:
  - reports/dashboard-build/render-model.json            # the LIVE baked per-route `pairs` the page renders now (28 commits)
  - scripts/dashboard/generate_dashboard.py              # READ-ONLY — beforeAfterViewerHTML / .ba-gallery / .ba-gtabs CSS + baTabBadge
  - http://localhost:4321/dashboard-build/dashboard.html#git   # in-browser getComputedStyle (Chrome plugin, own tab, Mode 4)
  - 28 session JSONLs (~/.claude/projects/<PROJECT_SLUG>/*.jsonl)   # repeated-request history
relates_to: [A88, A90, A91, A92]
extends: git-page-beforeafter-overhaul-targets.md (A92 worklist) — this layers the SWAPPER noticeability dimension on top.
---

# Git multi-page SWAPPER audit — completeness · noticeability · history

## 0. HEADLINE (inline)

- **5 multi-page items** in the timeline (commits affecting >1 page): `02359cd` (8 pages), `3d56df0` (5), `3889ae6` (4), `a553e1a` (3), `382886e` (2).
- **Completeness: 0 victims.** Every one of the 5 HAS the per-page swapper, and every swapper lists **ALL** affected pages — verified live (8/8, 5/5, 4/4, 3/3, 2/2 tabs). No page is silently omitted from the picker, at the model layer or the rendered layer.
- **Noticeability: 5 of 5 are victims.** The swapper is present and complete but visually *quiet* — the route-picker tabs render at **10px** dim pills, the "This change touched N pages" header at **12px**, and the picker has **no visual priority over the less-important `side by side / slider / flip` view-mode toggle** (also 10px, sitting louder right under the image). A user scanning the card reads the pills as passive status tags, not "switch to the other affected page." That is exactly the "nearly missed it" the user reports.
- **One-line fix:** Promote the picker to a labelled, segmented control — replace the quiet `.ba-ghead` + dim `.ba-gtab` pills with a single bordered banner reading **"▸ This change affects N pages — view each:"** followed by **larger (≥13px), segmented, equal-weight tabs** that read as a real switcher (active tab filled, not just tinted), placed immediately above the image with a clear visual "these control the picture below" relationship — and rank it visually ABOVE the view-mode toggle.

> **Important nuance for the fix author:** this is NOT a contrast/legibility bug. All swapper text passes WCAG AAA (measured below). It is a **visual-salience / hierarchy** bug — the right element is legible but under-ranked. The fix is size + weight + framing + a directive label, not color-darkening.

---

## PART 1 — COMPLETENESS SWEEP

Source: `reports/dashboard-build/render-model.json` → `look_back.git_timeline.commits[]` (28 commits). The multi-page signal is `visible_routes` length > 1; the swapper data is the `pairs[]` array (one entry per affected route — this is what builds the route-tab strip). Live render verified in-browser by expanding each card and counting `.ba-gtab` elements.

### 1a. The multi-page items (the only ones the swapper applies to)

| item (sha · title) | #pages affected | swapper present? | all pages listed? | victim? |
|---|---|---|---|---|
| **02359cd** · "developing agentic multiplatform … (first big checkpoint)" | **8** (home, highway, fleets, services, products, multifamily, blog, contact) | YES — 8 tabs live | YES — 8/8 (set-equal to visible_routes); badges: 6 EXACT + 2 approx | **noticeability only** |
| **3d56df0** · "snapshot 48h of sprint work as ratchet point…" | **5** (blog, contact, highway, products, services) | YES — 5 tabs live | YES — 5/5 | **noticeability only** |
| **3889ae6** · "phase a plow lands — 3 desktop bugs fixed in parallel" | **4** (products, contact, highway, services) | YES — 4 tabs live | YES — 4/4 | **noticeability only** |
| **a553e1a** · "CB-1: hero scrim hardening (products §01 + highway §0…)" | **3** (products, highway, services) | YES — 3 tabs live | YES — 3/3 | **noticeability only** |
| **382886e** · "fix(F-NEW-03+04): /fleets §04 + /products §07 caption-stack z-order" | **2** (fleets, products) | YES — 2 tabs live | YES — 2/2 | **noticeability only** (the user's exhibit) |

**Data-layer integrity check (all 5):** `pairs[].route` is set-equal to `visible_routes` for every multi-page item (verified via `sort/unique` comparison). **Multi-page items with FEWER pairs than affected routes (an incomplete picker): NONE.** So the A91 gallery builder is feeding the picker a complete per-route list — no page is dropped before render.

**Render-layer integrity check (all 5):** expanded each card live; `beforeAfterViewerHTML` takes the `pairs.length>1` branch → builds `.ba-gallery` with header "This change touched N pages" + one `.ba-gtab` per pair. Live tab counts and routes matched the model exactly (8/8, 5/5, 4/4, 3/3, 2/2; one tab `.active` each). **No silent omission at render.**

**Verdict, Part 1:** the *completeness* failure the user feared (a page mentioned but with "no clear way to see" its change) is **NOT present** in any current multi-page item. This is the thing A91+A92 fixed and it held. The remaining problem is purely *noticeability* (Part 2).

### 1b. A SEPARATE latent class worth flagging (not the swapper bug, but the same perception risk)

The same scan surfaced commits whose **raw commit subject NAMES routes that are not in their `pairs`/`visible_routes`** (i.e. the card's verbatim "ORIGINAL NOTE" line names more pages than the picker shows):

| sha | visible/paired routes | extra routes named in the commit subject |
|---|---|---|
| c9f52fe | home | highway, multifamily *(named as nav labels being changed, not visible diffs)* |
| 153b4fc | products | highway, multifamily, services *(named in "M-NEW / D-NEW" tag context)* |
| 767e019 | contact | contact *(extract-Input refactor — correctly single)* |
| 6a36736 / b8280e7 / 420b377 | (none — code/infra) | blog / products / multifamily(v2) |

These are **mostly correct scoping** — A92 properly narrowed `visible_routes` so the *card copy* no longer over-claims, and a single (or zero) pair is the honest answer. They are **not** swapper victims. **BUT**: the card still renders the verbatim git subject in its "ORIGINAL NOTE" footer, which names the extra routes. That is the *exact perception trigger* the user hit on 2026-05-31T09:19 ("it notes a change in fleets … but then it also notes something about products and we dont see anything for products"). Recommend (out of this BG's scope, flag only): when the ORIGINAL NOTE names a route that has no pair, add a one-line reconciler ("note names other pages as context; only <route> changed visibly"). Logged as a watch-item, not a fix here.

---

## PART 2 — NOTICEABILITY (in-browser, computed-style)

Method: Chrome plugin, **own new tab** (not reused), `dashboard.html#git`, expanded `382886e` (the user's exhibit), `getComputedStyle` on the rendered swapper. Viewport 1440×757, devicePixelRatio 2, Blueprint skin (the live default).

### 2a. Measured rendered prominence (382886e swapper)

Selector reference (NOT line numbers — the monolith is being edited):

| element | selector | font-size | weight | color | box | role |
|---|---|---|---|---|---|---|
| page-count header | `.ba-gallery .ba-ghead-t` | **12px** | 700 | `#ffffff` | h≈18px | "This change touched 2 pages" |
| header note | `.ba-gallery .ba-ghead-note` | **10px** | 400 | `rgb(154,166,194)` (dim) | — | "pick a page to see its before/after — every affected page is here…" |
| **route picker tab** | `.ba-gtab` (active) | **10px** | 600 | `rgb(122,212,251)` accent | **h≈20px**, pad 3×10px, pill r999 | THE control — switch affected page |
| route picker tab (inactive) | `.ba-gtab` | **10px** | 400 | `rgb(154,166,194)` (dim) | h≈20px | the *other* affected page(s) |
| tab badge | `.bgt-badge` | **9px** | 700 | green/amber/dim | tiny | exact / approx / code |
| view-MODE toggle | `.ba-modes` buttons | **10px** (bar 13px) | 400 | accent | h≈18px | "side by side / slider / flip" — only changes HOW you view ONE page |

### 2b. Spatial relationship (the under-appreciated finding)

Measured top offsets after pinning the gallery to the top of the viewport:

- `.ba-ghead` (page-count header): y≈121
- `.ba-gtabs` (the picker strip): y≈156→192 (a 36px-tall strip)
- `.ba-stage` (the before/after IMAGE): y≈228
- `.ba-modes` (the "side by side / slider / flip" toggle): y≈488 — **sits directly under the image, where the eye lands after looking at the picture**

**The inverted hierarchy:** the picker (high-stakes: "did I see every page this touched?") is a 10px dim pill strip ABOVE the image, where the eye arrives first and skims past it as metadata. The view-mode toggle (low-stakes: cosmetic view style) is the same 10px but lands in the eye's natural resting spot *under* the picture and reads as the primary interactive control. So the user's attention is drawn to the toggle that does the *less* important thing, and glides over the picker that does the *more* important thing. The picker has **no font-size, weight, or framing advantage** over the toggle to correct this.

### 2c. Contrast math (proves it's NOT a legibility problem)

WCAG ratios (computed): header title **18.93:1**, header note **7.76:1**, inactive picker tab **7.09:1**, active tab **11.04:1**, badge **9.69:1**. All clear AAA (≥7.0) for normal text. The text is perfectly *readable*. The defect is salience, not contrast — so the fix must NOT be "make it brighter," it must be "make it bigger, framed, and ranked as a control."

### 2d. VERDICT

**The swapper is present, complete, and legible — but NOT prominent enough. The user is right.** A reader scanning a multi-page card will register the image and the loud "side by side / slider / flip" toggle, and will skim past the 10px dim pill row that is actually the page-switcher. The header "This change touched N pages" (12px) states the fact but does not *direct* the eye to the control that acts on it, and the note that says "pick a page…" is the smallest, dimmest text in the block (10px/400/dim).

### 2e. CONCRETE, BUILDABLE FIX (selectors, not line numbers)

Target the `.ba-gallery` header + tab strip rendered by `beforeAfterViewerHTML` (multi-route branch) and styled by the `.ba-ghead` / `.ba-ghead-t` / `.ba-gtabs` / `.ba-gtab` rules. Four changes, all CSS + one copy tweak (no logic change):

1. **Directive label, not a passive statement.** Change the `.ba-ghead-t` copy from "This change touched N pages" to an action header with the caret the user already recognises: **"▸ Affects N pages — pick one to see its before/after:"** and bump it to **≥14px / 700**. Fold the dim `.ba-ghead-note` INTO this line (kill the separate 10px dim sentence) so there is one loud instruction, not a loud-ish title + a whisper.

2. **Make the tabs read as a SWITCHER, not status chips.** Bump `.ba-gtab` to **≥13px**, increase padding (e.g. `6px 14px`), and render them as a **segmented control** (shared container with a hairline, tabs butted together) rather than free-floating pills — segmented = "these are mutually-exclusive views of the same thing," which is exactly the semantics. Give the **active** tab a *filled* background (not the current subtle 12% accent tint) and the inactive tabs a clearly clickable affordance (visible border + hover lift, which already exists but is too quiet at 10px).

3. **Rank it above the view-mode toggle.** The picker is currently visually co-equal with (or quieter than) `.ba-modes`. The picker should be the louder of the two — give the `.ba-gtabs` strip a subtle accent left-border or a tinted background band so it reads as the primary control of the card, and keep `.ba-modes` as the secondary, smaller control it should be.

4. **Tie the picker to the image.** Add a visual connector so it's unmistakable the tabs drive the picture below — e.g. the active tab's fill color flows into the top edge of `.ba-stage` (shared accent border), or a small "▾ showing: <route>" caption on the stage that echoes the active tab. This closes the "I didn't realise those pills changed the picture" gap.

Single-page commits (the `pairs.length===1` branch) are untouched by all four — they never render `.ba-gallery`, so this is purely the multi-page path.

---

## PART 3 — HISTORY (the genuine repeated-request dimension)

Searched all 28 session JSONLs for prior user warnings about multi-page-change before/after visibility. The thread is **documented, recent, and precise** — and the current complaint is the user catching a *known, previously-warned* issue that was fixed at the data layer but shipped under-prominent.

### 3a. Timeline of the ask (all in session `666b8e4c`, 2026-05-31)

| when (UTC) | line | what the user said | what was promised / built |
|---|---|---|---|
| **09:19** | 8723 | **THE ORIGINAL WARNING.** Point 3: *"it notes a change in fleets and the pic is fleets but then it also notes something about products and **we dont see anything for products** so…????"* Point 4: *"this is the products page in the pic but description mentions **Products, Highway, and Services** … it's not just these 3 it's an **overarching flaw**"* | An audit + fix of the git page's multi-page handling. This is the seed of A91 (multi-route gallery) + A92 (forensic fact-check). |
| **10:03** | 8981 | *"second pass full force actual code change history deepdive and then fact check our git page and commit history"* — where else did a change touch pages without showing them. | A92 forensic fact-check (per-commit copy-vs-diff); spawned `git-page-beforeafter-overhaul-targets.md`. |
| **10:07** | 9000 | **THE "previously-warned" line the current prompt references:** *"rethink our items in the what we see list … **im just worried from last time when one item had 3 pages and idk how well we've accounted for that since then.**"* | Keep coherent multi-page commits as **gallery items** (the route-tab swapper), split genuinely-bundled ones. → A91 `c.pairs` per-route gallery built. |
| **17:03** | 10780 | *"another scrutinization pass at the git page's unopened changed items as the last change to add the **noticeable badges on the multi change ones wasn't handled as neat and professionally as i wanted**"* | (Collapsed-card scope — separate BG. But note: the user is already saying the multi-change treatment isn't prominent/clean enough.) |
| **17:12** | 10811 | **THE CURRENT PROMPT.** Pastes the full 382886e card, then: *"im still seeing us blatantly mention a change effects many pages with **no clear way to see every effected pages changes, we see just the chosen one** … (i do see an option to swap above the image … that's good but it **needs to be a bit more noticeable**) … how many other items are victim of this **previously warned error we couldve avoided had you been listening**."* | (this audit.) |

### 3b. Asked-vs-built gap — why it regressed to "not noticeable"

- **Asked (09:19 + 10:07):** when a change touches many pages, the user must be able to *see every affected page's* before/after — not just one — and was "worried from last time" about the 3-page case.
- **Built (A91):** the per-route `pairs` gallery + `.ba-gallery` route-tab strip. This **fully satisfies the literal ask** — every affected page IS in the picker (Part 1 confirms 0 omissions). The data/completeness problem was genuinely solved.
- **The gap:** A91 solved *completeness* but shipped the picker with the system's **smallest type tokens** (10px tabs, 9px badges) and pill styling that reads as metadata, placed above the image where the eye skims, and **not ranked above the cosmetic view-mode toggle.** So a user re-encountering a multi-page card 7 hours later (the user, at 17:12) *nearly missed the very control built to answer his 10:07 worry.* The fix was correct in **substance** and under-built in **prominence** — which is the user's exact framing: "i DO see an option to swap … that's good but it needs to be a bit more noticeable."
- **Why "previously warned" stings:** the 09:19 warning (points 3 & 4) is almost verbatim the 17:12 complaint, ~8 hours earlier. The completeness fix landed; the *noticeability* of that fix was never separately verified before declaring the git page done — so the regression is "built it, didn't make sure the user would FIND it." The structural lesson: a "show me X" request isn't satisfied until X is *prominent enough to be found*, not merely *present in the DOM*.

---

## Appendix — files & selectors referenced (repo-relative)

- Model (source of the completeness table): `reports/dashboard-build/render-model.json` → `look_back.git_timeline.commits[].{visible_routes,pairs}`
- Renderer (READ-ONLY — do not edit; another BG owns it): `scripts/dashboard/generate_dashboard.py` — function `beforeAfterViewerHTML` (multi-route `pairs.length>1` branch), `baTabBadge`; CSS rules `.ba-gallery`, `.ba-ghead`, `.ba-ghead-t`, `.ba-ghead-note`, `.ba-gtabs`, `.ba-gtab`, `.ba-gtab.active`, `.bgt-route`, `.bgt-badge`, `.ba-modes`.
- Live page: `http://localhost:4321/dashboard-build/dashboard.html#git` (expand a card via `.commit-card .cc-expand`; multi-page cards: 02359cd, 3d56df0, 3889ae6, a553e1a, 382886e).
- Sibling audits (cross-link, do not duplicate): `git-page-beforeafter-overhaul-targets.md` (A92 per-commit recreate/split worklist), `git-button-persistence-ack-audit.md` (action-control persistence).
- History source: session JSONL `666b8e4c-2ea7-4e92-a51e-7d04196328de.jsonl` lines 8723 / 8981 / 9000 / 10780 / 10811.
