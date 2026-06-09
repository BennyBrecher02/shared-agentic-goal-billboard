# A80 — SEO plans verification (adversarial, read-only)

**When:** 2026-05-29 · **Scope:** QA the two G2 SEO plans against real source BEFORE implementation · **Mode:** read-only, no git, no plan edits
**Targets:** `context/markdowns/plans/structured-data-jsonld-plan.md` (JSON-LD) · `context/markdowns/plans/baselayout-head-completeness-plan.md` (head completeness)
**Why this matters:** the design-ref verification caught a material WCAG error; same adversarial pass applied here. Every cited file/line/data-shape was re-derived from source, every schema/tag mapping checked for validity, every "resolved gap" re-tested for fabrication.

**Headline verdict:** Both plans are **GO to implement** — they are unusually accurate and honest (no fabricated data, gaps correctly identified, deferrals sound). I found **zero blocking errors** and a short list of **minor inaccuracies + implementer trip-hazards** to fix in the morning. Details below.

---

## Verification method

Read both plans in full, then re-derived every factual claim from source:
- `src/pages/products.astro` (product arrays, type, line ranges) · `src/consts.ts` (org data) · `src/content.config.ts` (Zod) · `src/layouts/{BaseLayout,ContentLayout,FullPageLayout}.astro` · `astro.config.mjs` · all 17 `src/content/blog/*.md` frontmatter · `src/pages/blog/[slug].astro` · `src/pages/blog.astro` · `src/pages/multifamily{,v2}.astro` (diffed) · `node_modules/@astrojs/rss` (v4 API) · asset existence on disk · grep sweeps for existing JSON-LD / author / canonical / twitter.

---

## PLAN 1 — `structured-data-jsonld-plan.md` (JSON-LD)

### ✅ Facts verified ACCURATE

| Claim | Verified |
|---|---|
| Canonical origin `https://eviumcharging.com` in `astro.config.mjs` `site:` | ✅ line 8 |
| Product counts: `productsL2` (9) + `productsL3` (7) = **16** | ✅ counted 9 + 7 = 16 |
| `Product` type `{ id, tier:'l2'\|'l3', brand, tag, img, name, power, desc, builtFor:string[], specs:{l,v}[] }` at **lines 6–10** | ✅ exact (Spec line 6, Product 7–10) |
| Product data lines 12–96 | ✅ L2 = 12–58, L3 = 60–96 |
| **No per-product routes** — `src/pages/products/` does not exist | ✅ confirmed (single `/products` + JS modal) |
| 17 blog `.md` files, typed `blog` collection | ✅ counted 17 |
| Blog frontmatter `{ title, category(enum), categoryLabel, date, excerpt, readTime, image? }` | ✅ exact match to Zod |
| **All 17** have `image` + valid ISO `date` + `categoryLabel` | ✅ dumped all 17 — every one populated |
| Blog date range **2024-08-12 → 2026-05-02** | ✅ exact (oldest `ev-adoption-accelerating`, newest `ev-tires-vs-regular-tires`) |
| Longest title ≈70 (≤110 headline cap holds) | ✅ longest = **69** chars |
| `Hero` `crumb` is editorial decoration, NOT nav breadcrumb | ✅ confirmed (`{ now:'Hardware', ref:'The lineup' }`) — warning is sound |
| **ZERO** existing JSON-LD / schema.org / `author` anywhere in `src/` | ✅ grep clean |
| Logo `evium-logo.svg` + `evium-mark.svg` exist in `public/img/logos/` | ✅ both present |
| `SITE_TITLE` / `SITE_DESCRIPTION` / `CONTACT` (phone `+19499452000`) / `SOCIAL` (LinkedIn+FB, instagram empty) / `APPS` | ✅ all exact in `consts.ts` |
| `src/lib/jsonld.ts` is **NEW** (doesn't exist) | ✅ only `fullpage.ts` + `reveal.ts` present |
| `@astrojs/sitemap` installed + active | ✅ `astro.config.mjs` line 9 |
| `@astrojs/rss` dep present but never wired | ✅ `package.json` line 35, no route |

### ✅ Schema mappings — VALID

- **Organization** (A): all props legit schema.org Organization. `@id` fragment pattern (`.../#organization`) correct. `sameAs` from `Object.values(SOCIAL).filter(Boolean)` correctly drops empty instagram. SVG `logo` accepted by validators (warning-only). ✅
- **WebSite** (B): valid; correctly declines to fabricate a `SearchAction` (no real search endpoint). ✅
- **ItemList → ListItem → Product** (C): structurally valid. `position` 1..16, `@id` fragments unique per product id, `additionalProperty[]` from `specs[]` mapped `{l→name, v→value}` is the correct schema.org pattern for arbitrary spec key/values. `category` derived from `tier`. ✅
- **BlogPosting** (D): all props valid. `datePublished` as `YYYY-MM-DD` is a valid schema.org Date. `author`/`publisher` as Organization references valid. ✅
- **BreadcrumbList** (E): 3-level Home › The Current › {title} all map to real URLs (`/`, `/blog`, `/blog/<slug>`). Valid. ✅
- **Data gaps (§5):** GAP-1..6 all correctly identified. The **"do not fabricate offers/price/rating"** stance (GAP-3) is exactly right — fabricating those risks a Google manual action. The org-default `author` fallback (GAP-1a) is valid and ships without touching `.md`. **No fabricated resolutions found.** ✅

### ⚠ Issues (none blocking)

| # | Severity | Real value vs plan's claim | Fix for the morning |
|---|---|---|---|
| **1.1** | ⚠ minor — trip hazard | Plan §4 says the products ItemList can be passed "**via a layout `head` slot** OR emit in page body." **No head-slot mechanism exists** in `ContentLayout`/`BaseLayout` today (grep: zero `<slot name="head">`). | Harmless because the plan's *recommended* path is "emit in page body" (which works — Astro serves body-level `<script type=ld+json>` and crawlers read it). Just **delete the "head slot" option** so an implementer doesn't go build a slot that isn't there. |
| **1.2** | ⚠ cosmetic | §1 table example cites the `crumb` ref as `"SITE-REF-01"` / `"FLEET-OP-A9"`. Products' actual crumb is `{ now:'Hardware', ref:'The lineup' }`. | Plan flags these as illustrative ("e.g."), so not wrong — but the literal current values differ. No action needed; the load-bearing point (don't derive breadcrumbs from crumb) is correct. |
| **1.3** | ⚠ note | `abs(path)` is specified two ways: §4 helper = `new URL(path, SITE_ORIGIN).href`; product-image row (line 154) = `new URL('/img/' + p.img, Astro.site).href`. | Equivalent in output. **Recommend the helper standardize on the `SITE_ORIGIN`-string form** — it works in BOTH `.astro` and `.ts` contexts, which the companion plan's `rss.xml.ts` endpoint needs (see composition note below). Caller still prepends `/img/` for product `img` values (they're stored as `chargers/x.png`, no leading slash) — plan says this correctly. |
| **1.4** | ⚠ verify-at-build | `headline` truncation: plan adds defensive `.slice(0,110)`. Longest real title is 69, so it never fires — fine, harmless. | No action. |

### Plan 1 verdict: **✅ GO to implement.** Sound. Fix 1.1 (drop the phantom head-slot option) and adopt 1.3 (SITE_ORIGIN-based `abs`) for clean composition. Everything else is accurate and the honesty discipline (no fake offers/ratings/search) is exactly correct.

---

## PLAN 2 — `baselayout-head-completeness-plan.md` (head completeness)

### ✅ Facts verified ACCURATE

| Claim | Verified |
|---|---|
| `<head>` owner = `BaseLayout.astro` **lines 25–57** | ✅ exact (`<head>`=25, `</head>`=57) |
| Existing OG tags at **lines 33–36**: `og:title`, `og:description`, `og:type="website"` (hardcoded), `og:image="/img/heroes/multifamily.jpg"` (hardcoded, root-relative) | ✅ exact, all four lines confirmed |
| Title composition (line 19), description default (line 20) | ✅ exact |
| **ZERO** canonical, **ZERO** twitter, **no** `og:url`/`og:site_name`/`og:locale` today | ✅ grep clean |
| `og:image` asset `/img/heroes/multifamily.jpg` exists (2.3 MB) | ✅ exists, 2,299,893 bytes ≈ 2.3 MB |
| All 3 layouts accept only `{ title?, description? }` | ✅ exact in all three `.astro` |
| Page→layout map (FullPage: index/services/fleets/highway/multifamily/multifamilyv2; Content: products/contact/blog/[slug]) | ✅ confirmed by imports |
| `[slug].astro` passes `title="{title} — The Current"`, `description=excerpt` (lines ~94–97) | ✅ exact (94–97) |
| `blog.astro` → `ContentLayout title="The Current"` at **lines 1026–1029** | ✅ exact |
| `robots.txt` does **not** exist in `public/` | ✅ confirmed absent |
| `@astrojs/rss@4.0.18` present, never imported, no `/rss.xml` | ✅ exact version, no route |
| RSS v4 API: default `rss(options)`, `rssSchema`, item fields `title/description/pubDate/link/categories[]/author/content/enclosure{url,length,type}`, `site` accepts string\|URL | ✅ verified in `dist/index.d.ts` |
| RSS `author` expects an email per spec | ✅ `.d.ts`: "item author's **email address**" — omit-not-fabricate is correct |
| Per-page hero images (§3 OG table): index `wyndham-hampton/01.jpg` · services `secaucus-plaza/05.jpg` · multifamily `secaucus-plaza/01.jpg` · highway `heroes/highway.png` · fleets `heroes/fleet.png` · products `decorative/aura-dark-glow.png` · contact `seacoast-kittery/01.jpg` | ✅ **all 7 match source exactly + all assets exist on disk** |
| products hero is an abstract glow (weak preview → pick a charger image) | ✅ confirmed `decorative/aura-dark-glow.png`; charger assets exist for the swap |
| `trailingSlash`/`build.format` not set → Astro default `format:'directory'` → trailing slash | ✅ confirmed neither set; HG-5 concern is real and correctly handled |

### ✅ Tag mappings + decisions — VALID

- **W1 canonical** `new URL(Astro.url.pathname, Astro.site).href` — correct syntax; trailing-slash-match-to-sitemap concern (HG-5) correctly raised. ✅
- **W2 OG** `og:url`/`og:site_name`/`og:locale="en_US"` (site is `lang="en"` ✅) + per-page absolute `og:image` — all valid OG syntax. ✅
- **W3 `og:type`** `"article"` only on blog posts via prop; optional `article:published_time`/`article:section` conditional — valid OG-article namespace. ✅
- **W4 Twitter** `summary_large_image` + title/description/image mapped from OG; **correctly OMITS `twitter:site`** (no real handle in `SOCIAL` — confirmed, only LinkedIn+FB). Not fabricating a handle is right. ✅
- **W5 RSS** field map valid against v4 API; `context.site` is the correct way to get site in an endpoint; newest-first sort `b.data.date.localeCompare(a.data.date)` correct for ISO dates; excerpt-only Phase 2 + deferred full-content via `compiledContent()` is sound. ✅
- **W5 robots.txt** `Sitemap: …/sitemap-index.xml` — correct: `@astrojs/sitemap` emits `sitemap-index.xml`. The **"don't `Disallow` the page you're canonicalizing"** rule (re `/multifamilyv2`) is textbook-correct. ✅
- **D3 `/multifamilyv2`** canonical→`/multifamily/` over noindex — correct SEO reasoning (consolidate vs suppress); `sitemap({ filter })` exclusion is a real `@astrojs/sitemap` option; gating the `src/` edit on user sign-off respects decision-handling discipline. ✅

### ⚠ Issues (none blocking)

| # | Severity | Real value vs plan's claim | Fix for the morning |
|---|---|---|---|
| **2.1** | ⚠ minor — inaccurate claim | §1 + §6 say `/multifamily` vs `/multifamilyv2` differ by hero `src` **AND "a slightly different hero sub-headline."** The actual `diff` shows only **TWO** deltas: (a) hero `src` (`installations/secaucus-plaza/01.jpg` vs `heroes/multifamily-old-site.jpg`) and (b) **a code-comment block** — **NOT** a sub-headline. The sub-headline is identical. | Correct the prose: the only *rendered* difference is the hero image. (Strengthens the dedup case — they're even closer clones than stated. No impact on the canonical recommendation.) |
| **2.2** | ⚠ cosmetic | §3 W5 feed-description claims it "matches `/blog` description — single truth." Actual `/blog` description = `"The Current — Evium's journal on charging infrastructure… Filed monthly."` The plan's RSS feed text drops the `"The Current — "` prefix. | Truly cosmetic (RSS feed title already carries "The Current"). If "single truth" is the goal, source the RSS description from the same string or accept the deliberate trim. |
| **2.3** | ⚠ verify-at-build | §3 W1/HG-5: with `trailingSlash` unset, Astro's default is `'ignore'` (routes resolve both ways) while `build.format:'directory'` emits trailing-slash URLs. Canonical, `og:url`, sitemap, and RSS `link` must ALL use the trailing-slash form to avoid self-conflict. | Plan already calls this out (HG-5) and says "verify at build." Just **confirm at build** that `dist/sitemap-0.xml` entries and the emitted canonical both carry the trailing slash, and that `rss.xml.ts` `link` uses `/blog/${post.id}/` (with slash — plan's W5 table already specifies the slash ✅). No code change anticipated; it's a build-time assertion. |
| **2.4** | ⚠ note | §4 `articleMeta` is threaded page→ContentLayout→BaseLayout. The two wrappers (`ContentLayout`, `FullPageLayout`) currently destructure only `{ title, description }` — the new optional props (`ogType`, `ogImage`, `ogImageAlt`, `canonical`, `articleMeta`) must be added to **both** wrapper `Props` AND forwarded. | Plan §4 + §10 explicitly say to widen + forward in both wrappers. Accurate. Just the one easy-to-miss step an implementer trips on — flagged so it's not forgotten. The `…Astro.props` spread approach works. |

### Plan 2 verdict: **✅ GO to implement.** Sound. Fix 2.1 (sub-headline claim is wrong — it's a comment, not rendered copy). Treat 2.3 as a build-time assertion (already anticipated). 2.2/2.4 are cosmetic/reminder. No fabricated data, decisions are SEO-correct.

---

## COMPOSITION — do the two plans compose cleanly?

**✅ YES, with one recommendation already covered by 1.3.**

- **Shared `SITE_ORIGIN`:** both plans add `export const SITE_ORIGIN = 'https://eviumcharging.com'` to `consts.ts` and reference it from `astro.config.mjs` `site:`. One constant, two readers. No conflict. ✅
- **Shared `src/lib/jsonld.ts` `abs()`:** the head plan §8 explicitly *references* the companion's `abs()` and says "do NOT write a second absolute-URL helper." Both "companion-first" and "head-first" sequencing orders are spelled out and both work (head-first creates `jsonld.ts` with just `abs()` as a seed; companion adds `orgNode()`/`jsonLd()` later). No duplication, no collision. ✅
- **Same `BaseLayout.astro` head, distinct commits:** companion §7 anticipates "same head-pass, distinct commits"; head plan §8 echoes it (JSON-LD `<script>` blocks vs `<meta>`/`<link>` tags as separate commits). Clean. ✅
- **Shared single-truth data points:** blog `date` feeds both `article:published_time` (head) and `BlogPosting.datePublished` (companion); `categoryLabel` → both `article:section` and `articleSection`; `post.data.image` → both `og:image`/`twitter:image` and `BlogPosting.image`. All read from `post.data` — no copy. ✅

**One composition subtlety to lock in (ties to issue 1.3):** the companion's `abs()` MUST source `SITE_ORIGIN` (a plain string constant), NOT `Astro.site`. Reason: the head plan's **`src/pages/rss.xml.ts` is a static endpoint, not a `.astro` component — `Astro.site` is unavailable there** (endpoints get `site` via the `context` arg only). An `abs()` that closes over the `SITE_ORIGIN` string works in `.astro` frontmatter, in `.ts` endpoints, and in the RSS item `link` builder alike. The companion plan already specifies `new URL(path, SITE_ORIGIN).href` in its §4 helper definition — so **as written it's correct**; just make sure the implementer doesn't "simplify" it to `Astro.site` inside the helper (that would break RSS). Flagging because it's the single most likely place the two plans could silently diverge at implementation time.

---

## GO / NO-GO TO IMPLEMENT

| Plan | Verdict | Conditions |
|---|---|---|
| **`structured-data-jsonld-plan.md`** | **✅ GO** | Drop the phantom "head slot" option (1.1); standardize `abs()` on the `SITE_ORIGIN` string form (1.3). Both are doc tweaks, not blockers. |
| **`baselayout-head-completeness-plan.md`** | **✅ GO** | Correct the "different sub-headline" claim — it's a code comment, not rendered copy (2.1); confirm trailing-slash agreement at build (2.3, already anticipated). |
| **Composition** | **✅ CLEAN** | One rule: keep `abs()` sourced from `SITE_ORIGIN` (string), never `Astro.site`, so the RSS `.ts` endpoint can reuse it. |

**Bottom line:** Unlike the design-ref verification that surfaced a material WCAG error, these two plans contain **no material errors** — the cited line numbers, data counts, asset paths, Zod shape, RSS API, and trailing-slash behavior are all accurate; the schema/tag mappings are valid; and the data-gap handling is honest (no fabricated offers, ratings, search endpoint, Twitter handle, or author email). The findings above are 6 minor doc-accuracy nits + 2 build-time assertions. Safe to implement both, in either order, with the §8 composition rules.

### Findings summary (for the board)
- **0 blocking errors** across both plans.
- **2 minor inaccuracies:** plan-1 illustrative crumb values (cosmetic); plan-2 "different sub-headline" (it's a comment — fix the prose).
- **2 trip-hazards:** plan-1 phantom head-slot option; the `abs()`-must-use-`SITE_ORIGIN` composition rule (so RSS endpoint can share it).
- **2 build-time assertions:** trailing-slash agreement (sitemap/canonical/og:url/RSS link); wrapper-prop forwarding in both layouts.
- All asset paths exist on disk; all 17 posts carry image+date+categoryLabel; product count is exactly 16; zero pre-existing JSON-LD/canonical/twitter/author.

BG-COMPLETE-SENTINEL
