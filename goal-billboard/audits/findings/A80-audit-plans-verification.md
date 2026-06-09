# A80 — G2 audit-plans adversarial verification

**Type:** READ-ONLY adversarial verification of committed G2 audit plans against real source. No plans edited, no src/ touched, no git ops.
**Date:** 2026-05-29
**Method:** source reads (`package.json`, `astro.config.mjs`, `app.css`, blog/header templates), `git ls-files` / `git check-ignore` for tracking status, `grep` for usage/orphan claims, and a from-scratch WCAG-2.x relative-luminance calculator (`/tmp/wcag.py`) for every contrast ratio.
**Plans verified:** `site-perf-optimization-plan.md`, `site-a11y-audit-plan.md`, `eyebrow-contrast-a11y-options.md`, `app-css-health-and-consolidation-plan.md`.

---

## 🔴 HEADLINE VERDICT (PRIORITY 1) — the build-wiring claim is WRONG

**`site-perf-optimization-plan.md` §0.1 / §1 / TL;DR claim: FALSE as stated. Do NOT act on it as written.**

The plan's single highest-leverage claim — *"The image pipeline is built but not wired into `build`… a clean checkout / CI build ships `<picture><source>` tags pointing at `/img-opt/...` files that don't exist → browsers fall through to the multi-MB raw `<img src>` fallback"* — **does not hold, because the variants are committed to the repo.**

Evidence:
- `git ls-files public/img-opt | wc -l` → **236 files tracked.** (Sample: `public/img-opt/blog/charger-plugs/ev-charging1-1200.avif`, …)
- `git check-ignore -v public/img-opt` → **not ignored.** `git check-ignore public/img-opt/chargers/siemens-l2-480.avif` → **not ignored.**
- `.gitignore` has **no rule** covering `public/img-opt/`. On the contrary, its header explicitly says *"Track: … public/img/, public/video/ …"* and the default-deny is scoped to `context/*`, not `public/`.
- `du -sh public/img-opt` = **8.9 MB / 236 files** — matches the plan's own "currently on disk" measurement.

**What this means:** a fresh `git clone` already contains all 236 AVIF/WebP variants. `astro build` copies `public/` → `dist/` 1:1, so the `<picture><source srcset="/img-opt/...">` tags resolve to **real committed files**. The "silent fallthrough to multi-MB raw on clean build / CI" scenario **cannot happen from a normal checkout.** The variants are a *committed artifact*, not a *manual-only* one.

**The plan's own §1 hedges toward this** ("variants currently exist on disk… 8.9 MB, 236 files") but then mis-frames them as "a manual artifact, not a build product" and concludes the clean build breaks. The missing fact is the **tracked-in-git** status — which flips the conclusion.

**The kernel that IS still true (and worth doing, at lower urgency):**
- `package.json` `build` = bare `astro build`; `images:build` = `node scripts/prebuild-image-pipeline.mjs`; **the two are not chained** (confirmed — lines 9 & 31-32). ✅ accurate.
- `astro.config.mjs` is minimal (sitemap + tailwind Vite plugin only); **no integration runs the pipeline** (confirmed — 13 lines, no image hook). ✅ accurate.
- So if someone **adds a new source image and forgets to run `images:build` + commit the variants**, the new `<picture>` falls through to its raw fallback. That is a real *maintenance* footgun → wiring the pipeline into `build` (Phase 1A) + a manifest-vs-disk build guard is still a **legitimate, good improvement**. But it is **prevention of a future drift**, not a fix for a *currently-shipping* "clean build serves raw multi-MB images" bug. **Reframe 1A from CRITICAL/"un-does the entire image effort"/"highest-leverage single fix" → MEDIUM hardening.** The catastrophe framing is unwarranted.

> One nuance: `npm run build` (not `astro build` directly) *would* auto-run a `prebuild` script if one existed — but **none exists** in package.json, so today nothing regenerates variants on build regardless of entry point. The plan's Phase-1A note about `prebuild` only-auto-firing-on-`npm run build` is itself correct; it just doesn't change the verdict because the variants are committed anyway.

### Perf plan — the rest

| Claim | Verdict | Notes |
|---|---|---|
| `hero-electricity.mp4` is **orphaned** (zero `src/` refs) | ✅ **CONFIRMED** | `grep -rn hero-electricity src/` → zero. Broadened grep across all `.astro/.ts/.js/.mjs/.json/.html` (excl. node_modules/.git/markdowns) → zero. File exists on disk at **13.9 MB** (`public/video/hero-electricity.mp4`). Safe-to-archive claim sound. (Plan said "13 MB"; real = 13.9 MB — rounded-down but right order.) |
| `hero-abstract.mp4` used on `/`, mp4-only, `preload="auto"`, no poster | ✅ sound (not re-opened in browser) | On disk **7.69 MB** (plan said 7.3 MB — slightly under-stated, immaterial). |
| `images:build` and `build` not chained | ✅ CONFIRMED | package.json lines 9, 31-32. |
| `astro:assets`/`<Image>` not used; rolls own `<picture>` | ✅ plausible/consistent | Not exhaustively grepped, but consistent with config + the manifest-driven design described. |
| Raw originals (siemens-l2 6.6 MB etc.) also ship in `dist/` as fallback `src` | ✅ sound | They're in tracked `public/img/`; `public/`→`dist/` copy is 1:1. The Phase-3 "trim deployed originals" rationale stands independent of the §0.1 error. |

**Net on perf plan:** ⚠ **the headline framing is materially wrong** (committed variants ≠ "clean build ships raw"). Sub-claims about orphan video, missing chaining, and originals-in-dist are accurate. The remediation is still worth doing as *drift-prevention hardening*, not *active-bug rescue* — re-rank 1A accordingly before the user acts on the "highest-leverage / silently un-does everything" urgency.

---

## ✅ PRIORITY 2 — contrast math is EXACT

Re-computed every ratio from the **real tokens** (`app.css`: `--primary:#2dc856`, `--teal:#1a2f3a`, `--teal-deep:#0c1c25`, `--surface:#ffffff`, `--plaza:#eef1f5` — all confirmed by direct grep) using the WCAG-2.x sRGB relative-luminance formula, with proper alpha compositing of `rgba(255,255,255,α)` over the opaque section backgrounds.

**Every headline value the task flagged is correct to 0.01:1.**

| Claim (plan) | Plan value | Recomputed | Verdict |
|---|---|---|---|
| white@**0.40** on `--teal` | 3.52:1 | **3.52:1** | ✅ EXACT |
| white@**0.45** on `--teal` | 4.08:1 | **4.08:1** | ✅ EXACT |
| white@0.40 on `--teal-deep` | 3.79:1 | **3.79:1** | ✅ EXACT |
| white@0.45 on `--teal-deep` | 4.43:1 | **4.43:1** | ✅ EXACT |
| `#1c7d36` on white | 5.20:1 | **5.20:1** | ✅ EXACT |
| `#1c7d36` on plaza `#eef1f5` | 4.59:1 | **4.59:1** | ✅ EXACT |
| `#2dc856` brand-green on white | 2.21:1 | **2.21:1** | ✅ EXACT |
| `#2dc856` on plaza | 1.95:1 | **1.95:1** | ✅ EXACT |

**Plus every supporting figure in both a11y plans also checks out exactly:**
- Eyebrow alt-candidates: `#1d8137` → 4.94 / 4.36 ✅; `#1a7532` → 5.78 / 5.10 ✅; `#29b54e` (`--primary-deep`) → 2.69 / 2.37 ✅; `#00b831` full-sat → 2.66 ✅ (confirms Option 2's "even max-saturation fails 3:1" claim).
- Dark-bg eyebrows (must-stay-passing): `#2dc856` on teal **6.29:1** ✅, on teal-deep **7.87:1** ✅.
- Fix target white@**0.6**: teal **6.01:1** ✅, teal-deep **6.89:1** ✅ (matches the `.sec-foot` precedent). white@0.5 on teal **4.65:1** ✅.
- A2 plus-glyph `rgba(45,200,86,0.6)` on teal-deep → **3.63:1** ✅.
- A3 input border `#b9c1cc` on white → **1.82:1** ✅; fp-nav dot `rgba(140,150,150,0.9)` on teal-deep → **4.89:1** ✅.

**One sub-0.05 nit (immaterial):** the *proposed fix* shade `#8a94a0` on white — plan says **3.05:1**, I compute **3.08:1**. Both clear the 3.0 non-text bar; either way the recommendation (darken `#b9c1cc` to ≥`#8a94a0`) is valid. Rounding/formula-precision, not an error.

**Verdict: both `site-a11y-audit-plan.md` and `eyebrow-contrast-a11y-options.md` contrast math = ✅ SOUND.** The "FAIL AA-normal" classifications (0.40/0.45 white labels at 10-11px → 4.5:1 applies, not the large-text 3:1 relaxation) are correctly reasoned. The `#1c7d36` recommendation genuinely clears AA on both light backgrounds. (Note: these are static-formula confirmations on flat backgrounds — the plans themselves flag `[verify-live]` for text-over-photo and focus/keyboard items; that honesty caveat is appropriate and not re-litigated here.)

---

## ✅ PRIORITY 3 — CSS-health spot-checks all CONFIRMED

`app-css-health-and-consolidation-plan.md` — spot-checked 6 of its grep-backed claims:

| Claim | Verdict | Evidence |
|---|---|---|
| `.rule-soft` (line 1072) is **truly unused** | ✅ CONFIRMED dead | Defined once at `app.css:1072`; `grep -rn rule-soft src/` returns **only the definition**, zero markup hits. Safe to remove. |
| `.fp-auto-height` is **truly live** | ✅ CONFIRMED live | Used at `FooterSection.astro:10` (`class="section fp-auto-height"`) + referenced in SiteFooter/FooterSection doc-comments. Correctly flagged KEEP. |
| `.section.fit` half (line 206) is dead | ✅ CONFIRMED | Rule structure at `app.css:206-207` matches plan exactly (`.section.fit, .section.fp-auto-height { min-height: 0; }`); zero `class="…fit…"` word-boundary matches in any `.astro`. Drop the `.fit,` half, keep the `.fp-auto-height` half. |
| `@layer components` boundary ~1073 / unlayered after | ✅ CONFIRMED | `@layer components {` opens at **130**, closes `}` at **1073**; first unlayered rule (`#fp-nav { z-index: 30 !important; }`) at **1082**. Plan's "~1073/1075" is essentially exact (closer 1073, content resumes 1082). The cascade-origin risk argument (unlayered beats layered) is sound. |
| `.protocol-row .p para` phantom (line 1514) | ✅ CONFIRMED dead half | Line 1514 = `.protocol-row .p para, .protocol-row .p p {…}`; **zero `<para>` elements** anywhere in `src/`. The `para` half can never match — safe to drop, keep the `.p p` half. |
| `#2DC856` literal count = 33 | ✅ CONFIRMED exact | `grep -rino 2dc856 src` = **33**. Per-file: HeaderBatteryLefter **13** ✅, ScrollHint **2** ✅, app.css **6** ✅, blog/[slug] **12**. |

**Minor framing note (not an error):** the plan tallies blog/[slug] as "11" in one place (§3.1) by splitting it as 9 CSS lines + 2 JS literals; the raw grep is **12 occurrences** because line 475 contains `#2DC856` twice (so 10 CSS occurrences across 9 lines + 2 JS = 12). The plan's own enumeration (`…455, 475(×2) + JS 52, 190`) is internally consistent and correctly identifies the 2 JS literals (lines 52, 190) as non-CSS "leave-as-is." No actionable discrepancy — the Tier-A "25 literal→var()" replacement count (header 13 + blog-CSS 9-lines + fp-nav 3) is unaffected.

**Verdict: `app-css-health-and-consolidation-plan.md` spot-checks = ✅ SOUND.** Its dead/live census is grep-accurate, the Tier-A (G2-safe, value-identical) vs Tier-B (cascade-moving, post-G2) split is well-reasoned, and the `@layer` preservation list rests on a correctly-read layer geography.

---

## Summary table

| Plan | Verdict | Headline |
|---|---|---|
| **`site-perf-optimization-plan.md`** | ⚠ **ERROR in the headline claim** | "Clean build ships raw multi-MB images / pipeline un-wired un-does the image effort" is **FALSE** — `public/img-opt/` (236 variants) is **committed & not gitignored**, so a fresh clone/CI ships the variants. Re-rank Phase-1A from CRITICAL → MEDIUM drift-hardening. Orphan-video, no-chaining, originals-in-dist sub-claims are accurate. |
| **`site-a11y-audit-plan.md`** | ✅ SOUND | All §A contrast math exact (0.40/0.45 white-alpha fails, white@0.6 fix target, A2/A3 figures). [verify-live] caveats appropriate. |
| **`eyebrow-contrast-a11y-options.md`** | ✅ SOUND | `#2dc856` 2.21/1.95 FAIL and `#1c7d36` 5.20/4.59 PASS both verified exact. Option-2 "max-sat still <3:1" math confirmed. |
| **`app-css-health-and-consolidation-plan.md`** | ✅ SOUND | `.rule-soft` dead, `.fp-auto-height` live, `.section.fit`/`.p para` dead halves, layer boundary 1073/1082, `#2DC856`×33 — all confirmed. |

**One actionable correction for the user before acting:** the perf plan's flagship "highest-leverage single fix" is built on a false premise. The work it proposes (chain pipeline into build + manifest-vs-disk guard) is still worth doing, but as **future-proofing against an add-image-forget-to-regenerate footgun**, not as a rescue of a currently-broken production build. Nothing on the live site is shipping raw multi-MB images via the `<picture>` path due to missing variants — the variants are in the repo. The contrast and CSS-health plans are safe to act on as written (their math and grep census check out).

---

---

## Round 2 — content/forms/products/nav

**Type:** READ-ONLY adversarial verification of the second wave of G2 audit plans against real source.
**Date:** 2026-05-29
**Method:** direct source reads of `src/pages/contact.astro`, `src/pages/blog.astro`, `src/pages/blog/[slug].astro`, `src/pages/products.astro`, `src/layouts/BaseLayout.astro` + `ContentLayout.astro`, `src/components/forms/Input.astro`, `src/components/chrome/HeaderBatteryLefter.astro` + `SiteFooter.astro`, `src/consts.ts`, `astro.config.mjs`; `ls`/`grep` for API/action/backend existence; per-file frontmatter-stripped body grep for in-body links; awk-computed real `<title>` string lengths.
**Plans verified:** `site-forms-ux-conversion-audit.md`, `site-products-ux-audit.md`, `site-content-seo-strategy-audit.md`, `site-nav-ia-audit.md`.

---

### 🔴 PRIORITY — the CRITICAL "forms submit to nothing" bug — ✅ CONFIRMED (definitively)

**`site-forms-ux-conversion-audit.md`'s claim that the forms submit to nothing is TRUE, fully verified at the source.** The bug-billboard critical entry stands.

Both forms `preventDefault()` then **only manipulate the DOM** — there is no `fetch`, no `FormData`, no network send of any kind, and there is no backend for one to talk to even if it existed.

**Contact form** (`src/pages/contact.astro`, handler lines 200-212):
- `form.addEventListener('submit', (e) => { e.preventDefault(); … })` — line 200-201.
- Body does exactly three things: (1) checks the three required fields `['f-first','f-last','f-email']` are non-empty (toggling an `.invalid` class), (2) `modal.classList.add('is-open')`, (3) `modal.setAttribute('aria-hidden','false')`. Lines 202-212.
- **No `fetch`, no `FormData`, no `XMLHttpRequest`, no `action`/`method` on the `<form>`** (`<form class="form-card reveal" id="contact-form" novalidate>` — line 37, bare). The user's data never leaves the browser. The "Message received" success modal (lines 172-179) is shown unconditionally on valid input. `close` handler just `form.reset()`s.
- (Cross-check on the validation IDs: `Input.astro` line 42 sets `id = \`f-${name}\``, so `name="first"|"last"|"email"` → `f-first|f-last|f-email`. The handler's `getElementById` lookups DO resolve — i.e. client validation works — but that only gates the no-op, it doesn't add a send.)

**Blog newsletter form** (`src/pages/blog.astro`, handler lines 1501-1517, source `<form id="sub-form" novalidate>` line 1338):
- The code's own comment is `// ── Subscribe form (no-op acknowledgement) ────`.
- `e.preventDefault()` (1505); validates the email via `input.checkValidity()`; on success it merely mutates the button text to `'Subscribed '`, disables pointer-events, and dims the input (`input.disabled = true; input.style.opacity = '0.5'`). **No `fetch`/`FormData`.** The address is discarded.

**No backend exists anywhere:**
- `ls src/pages/api/` → **No such file or directory.** `ls src/actions/` → **No such file or directory.**
- `grep -rni "formspree|netlify|data-netlify|action="` across `src/` → **zero matches** (no third-party form endpoint, no `action=` attribute on any form site-wide).
- `astro.config.mjs` (13 lines, read in full) has **no `output`** key and **no `adapter`** — it's the Astro static default (`integrations: [sitemap()]` + a tailwind Vite plugin, nothing else). A static build has no server to receive a POST even if a form had an `action`.

**Verdict: ✅ SOUND — CONFIRMED.** Every leg of the claim holds: preventDefault-then-DOM-only on both forms, no fetch/FormData, no `/api`, no `/actions`, no `output`/`adapter`, no Formspree/Netlify/`action=`. **A real lead-loss bug: 100% of contact + newsletter submissions are silently dropped while showing a success state.** The plan's framing is accurate and the critical severity is warranted. *(Scope note: this verifies the two named forms. The site has no other `<form>` elements that send data — products uses a modal, not a form.)*

---

### `site-products-ux-audit.md` — the context-blind quote-CTA P0 — ✅ CONFIRMED

The claim that the product modal's quote CTA is a **static `<a href="/contact">` the open-handler never rewrites** is TRUE.

- Source markup: `<a id="prod-modal-cta" href="/contact" class="prod-modal-cta">Request a quote …</a>` — `src/pages/products.astro` line 517. Hardcoded `/contact`, no query string, no `data-*`.
- `openModal(card)` (lines 530-563) writes **eight** things into the modal: `prod-modal-img` (src+alt), `prod-modal-name`, `prod-modal-brand`, `prod-modal-tier` (text+class), `prod-modal-power`, `prod-modal-desc`, `prod-modal-specs` (innerHTML), `prod-modal-builtfor` (innerHTML). **It never references `prod-modal-cta`.** There is no `.href =` assignment anywhere in the `<script>` (lines 524-584).
- Consequence: open the modal for any of the **16** chargers (9 L2 + 7 L3), click "Request a quote," and you land on bare `/contact` with **zero** signal about which unit you were viewing. The product identity the user just expressed interest in is dropped at the handoff — and `/contact` itself can't capture it (per the forms finding above, contact submits to nothing anyway, compounding the loss).

**Verdict: ✅ SOUND.** The P0 is real and correctly described as context-blind. Fix is trivial in shape (set `cta.href = '/contact?product=' + card.dataset.product` or similar inside `openModal`), though it only becomes *useful* once the contact form actually transmits.

---

### `site-content-seo-strategy-audit.md`

**(a) Blog title double-suffix — ✅ CONFIRMED (real lengths computed).**
- `src/pages/blog/[slug].astro` line 95 passes `title={\`${post.data.title} — The Current\`}` into `ContentLayout`, which forwards `title` verbatim to `BaseLayout`.
- `BaseLayout.astro` line 19: `pageTitle = title ? \`${title} — ${SITE_TITLE}\` : …`, and `SITE_TITLE = 'Evium Charging'` (`consts.ts` line 5).
- So the rendered blog `<title>` = **`{post title} — The Current — Evium Charging`** — the suffix is applied **twice** (`— The Current` from the page, `— Evium Charging` from the layout; +29 chars of boilerplate tail).
- Real computed lengths (awk over the 17 `.md` titles): worst case **104 chars** ("EV Adoption Is Accelerating — What Property Owners Need to Know Now — The Current — Evium Charging"); five posts ≥100; shortest still **97**. All far exceed the ~60-char Google SERP display budget, so the distinctive head of every blog title is truncated in results while the redundant double brand tail is what gets cut/cluttered. *(Confirmed this is blog-specific: non-blog pages like `/contact` get a single clean suffix — `Contact — Evium Charging`, 26 chars — because they pass a bare `title` and only BaseLayout's one suffix applies. The double-suffix is unique to the `[slug]` route hard-coding its own `— The Current`.)*
- **Verdict: ✅ SOUND.** Double-suffix real; over-length real.

**(b) Zero in-body internal links across 17 posts — ✅ CONFIRMED (exact).**
- Post count: `ls src/content/blog/*.md | wc -l` = **17**. ✅
- Stripped frontmatter from each file (everything after the 2nd `---`) and grepped the body for markdown inline links `](…)`: **TOTAL = 0** across all 17. A second scan for any internal-path token (`](/`, `(/blog`, `(/services`, `(/products`, `(/multifamily`, `(/highway`, `(/fleets`, `(/contact`, `href=`) in the bodies also returned **zero**. The bodies render through `<Content />` (`[slug].astro` line 43), so what's not in the markdown isn't on the page.
- **Verdict: ✅ SOUND.** Literally zero in-body internal links across the whole canon — a real internal-linking / crawl-equity gap, exactly as claimed.

**(c) OCPP rendered 3 ways — ✅ CONFIRMED.**
- Normalizing every OCPP version string in `src/`, there are exactly **3** distinct forms: **`OCPP 1.6+`** (14 occurrences), **`OCPP 1.6`** (6), **`OCPP 1.6J`** (1). Same protocol/version, three inconsistent surface renderings (e.g. `products.astro:42` "OCPP 1.6" vs `:50` "OCPP 1.6J" vs the pervasive "OCPP 1.6+"; the blog body `ocpp-…md` uses bare "OCPP 1.6" while also mentioning "2.0.1").
- **Verdict: ✅ SOUND.** The "3 ways" count is accurate as a count of distinct version-string renderings. *(Minor framing note, not an error: the bulk of the site standardizes on `OCPP 1.6+`; the inconsistency is concentrated in 2 product spec strings + the explainer post. So it's a real but localized consistency nit, not site-wide chaos — the audit's count is right; calibrate the effort to "fix 7 stray strings," not "rewrite everything.")*

---

### `site-nav-ia-audit.md`

**(a) `NAV_LINKS` is dead; `MENU_LINKS` is the live one; footer is a 3rd source — ✅ CONFIRMED (all three).**
- `NAV_LINKS` (`consts.ts:18`, a fully-typed `NavLink[]` with `sub` descriptors) is referenced **nowhere** — repo-wide grep across `*.astro/*.ts/*.tsx/*.js/*.mjs` (excl. node_modules) returns **only its own definition line.** **Dead.** ✅
- The live primary nav is **`MENU_LINKS`**, hard-coded *inline* inside `HeaderBatteryLefter.astro` (defined line 38, consumed line 170). It is a **separate, divergent** list — e.g. it labels the blog **"Journal"** and carries a commented-out `Multifamily v2` entry; `NAV_LINKS` labels things differently (`Highway · NEVI`, `Hardware` with sub-copy). Two hand-maintained sources already out of sync. ✅
- The footer is indeed a **3rd** independent hardcoded list: `SiteFooter.astro` lines 23-28 spell out `<li><a href="/services">Services</a></li> … /products`Hardware</li>` by hand, sharing **no** data source with either of the other two (it pulls only `CONTACT`/`APPS` from consts, not the link set). ✅
- **Verdict: ✅ SOUND.** Three separate nav definitions (one dead in consts, one live inline in the header, one inline in the footer) with no single source of truth — exactly as claimed. The dead `NAV_LINKS` is the trap: it *looks* like the canonical nav (typed, documented, in consts) but nothing consumes it, so editing it changes nothing on the site.

**(b) Market pages link only to `/contact` (siloing) — ✅ CONFIRMED.**
- Enumerated every `href` on the four market/service pages. Their only **cross-page** destination is `/contact` (services ×3, multifamily ×4, highway ×3, fleets ×2); everything else is an on-page anchor (`#scope`, `#models`, `#economics`) or `/#faq` (home anchor). **None of the four link laterally to each other or to `/products`/`/blog`.**
- **Verdict: ✅ SOUND.** The market pages are dead-end silos whose only forward path is the (currently non-transmitting) contact form. Real IA gap. *(This compounds (b)-content above: no in-body blog links + no lateral market links = almost no internal cross-linking anywhere outside the global chrome.)*

---

### Round 2 summary table

| Plan | Claim | Verdict |
|---|---|---|
| **`site-forms-ux-conversion-audit.md`** | Forms submit to nothing (preventDefault + DOM-only, no backend) | ✅ **CONFIRMED** — critical bug real. No fetch/FormData on either form; no `/api`, no `/actions`, no `output`/`adapter`, no Formspree/Netlify/`action=`. 100% of submissions silently dropped. |
| **`site-products-ux-audit.md`** | Quote-CTA is static `<a href="/contact">` the modal handler never rewrites | ✅ **CONFIRMED** — `prod-modal-cta` set once in markup (line 517), never touched in `openModal`; all 16 products → context-blind `/contact`. |
| **`site-content-seo-strategy-audit.md`** | Blog title double-suffix | ✅ **CONFIRMED** — `{title} — The Current — Evium Charging`; real worst case 104 chars, 5 posts ≥100, all ≥97. Blog-specific. |
| | Zero in-body internal links across 17 posts | ✅ **CONFIRMED** — 0 markdown body links across all 17 (frontmatter-stripped grep). |
| | OCPP rendered 3 ways | ✅ **CONFIRMED** — `1.6+`(14) / `1.6`(6) / `1.6J`(1). Localized to ~7 strings; count accurate. |
| **`site-nav-ia-audit.md`** | `NAV_LINKS` dead; `MENU_LINKS` live; footer 3rd source | ✅ **CONFIRMED** — `NAV_LINKS` zero imports repo-wide; live nav is inline `MENU_LINKS` in HeaderBatteryLefter; footer is a separate inline `<li>` list. 3 unsynced sources, the consts one a dead decoy. |
| | Market pages link only to `/contact` | ✅ **CONFIRMED** — services/multifamily/highway/fleets cross-page links are `/contact`-only + self-anchors; no lateral or `/products` links. |

**Net on Round 2:** unlike Round 1 (which caught 2 over-statements), **all six Round-2 findings verify as sound against real source** — no errors, no over-statements detected. The forms-submit-to-nothing CRITICAL is definitively confirmed (and is compounded by the products quote-CTA dropping context into that same dead form). Two immaterial calibration notes (OCPP inconsistency is ~7 localized strings not site-wide; the products CTA fix is only *useful* once the form actually transmits) — neither changes a verdict.

BG-COMPLETE-SENTINEL
