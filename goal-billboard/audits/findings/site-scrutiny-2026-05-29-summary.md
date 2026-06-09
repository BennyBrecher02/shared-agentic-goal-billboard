# Site-scrutiny 2026-05-29 — consolidated durable findings (tracked copy)

The full per-area outputs live in gitignored `reports/audit-findings/site-scrutiny-2026-05-29/`. This is the TRACKED consolidation of the findings that are **durable** (survive the impending redesign) — so they're not lost. The visual-scrutiny half (4 BGs) was killed as redundant (pages about to be redesigned) + was colliding on a shared browser; the CODE-audit half (a11y / CSS / content-SEO; perf killed mid-flight) surfaced the real value below.

## Honest context

This came from an over-reached "burn the budget" plow (user called it out — re-auditing a soon-to-be-redesigned site is largely make-work). The visual half was waste. But the code half found genuine net-new defects that hold regardless of the redesign. Kept only the durable findings.

## Real defects (independent of the redesign — worth fixing regardless)

| # | Defect | Detail | Severity | Survives redesign? |
|---|---|---|---|---|
| 1 | **Eyebrow contrast WCAG AA fail** | `.eyebrow` = `#2dc856` (brand green) at 12px on white/`bg-plaza` = **2.21:1 / 1.95:1** (needs 4.5:1). On services/products/multifamily/highway/fleets/blog. The a11y test has color-contrast DISABLED, so it was never caught. | High (a11y) | YES — it's the brand accent; the eyebrow pattern likely carries forward |
| 2 | **Zero structured data** | No JSON-LD anywhere across 16 charger models + 17 blog posts. Data is fully modeled in code (productsL2/L3 arrays, typed blog frontmatter) — just never emitted as schema. | High (SEO) | YES — content/SEO survive any visual redesign |
| 3 | **`/multifamilyv2` indexable clone** | Live, sitemapped, no `noindex` — a near-byte-identical clone of `/multifamily` (only hero differs). Duplicate-content SEO risk. | Medium | YES |
| 4 | **Head-tag gaps** | 0 canonical tags, 0 Twitter-card tags, `og:type` always "website", OG image hardcoded site-wide. RSS + sitemap integrations installed but RSS never wired. | Medium (SEO) | YES |
| 5 | **"OCPP" written 3 ways** | 1.6 / 1.6+ / 1.6J — terminology inconsistency. | Low (content) | YES |

**Top SEO wins** (data already structured, just unemitted): Product ItemList JSON-LD (16 models) · BlogPosting + BreadcrumbList (17 posts, add `author` to the Zod schema) · Organization JSON-LD from `consts.ts` · complete BaseLayout head (canonical + per-page OG + Twitter).

## Redesign-import PRESERVATION list (the import MUST respect these)

The CSS-consistency audit's highest-value output — **19 distinct device/engine fix-clusters** (~80 provenance-tagged lines across 60 `@media` blocks) that look redundant out of context but **break silently if "cleaned up."** The design-import DIFF + any "modernize the CSS" instinct must preserve:

- Anything tagged `P0/P1/P2/B-K/CB/g2/audit#/-NEW-` in app.css comments
- Every `env(safe-area-inset-*)`, `100dvh`, `isolation:isolate`
- Per-page `*-hero .bg-tint` / scrim overrides (multi-tier hero scrims)
- WebKit-stacking fixes, iOS safe-area fixes, narrow-viewport clipping fixes

**Cascade-layer risk (critical for the import):** app.css deliberately mixes `@layer components` with a large UNLAYERED block (fp-nav, all `.prod-*`, modals, per-page hero scrims). Several device fixes rely on the unlayered rules winning the cascade. **If the import reorganizes `@layer` structure, those fixes break silently.** Flag any layer reorg for explicit re-verification.

## Refactor opportunity (for the redesign, not urgent now)

- **Card-grid 12-way duplication**: `card-row / surface-row / chip-row / info-row / compliance-row / models / spec-row / connectors / protocol-row / brands / route-row / portal-cards` are all the same grid+border skeleton. `.card-row` proves a parameterized version works. The redesign is the natural time to consolidate to one component.
- Token drift: HeaderBatteryLefter + blog/[slug].astro hardcode brand teal/green ~30× as literal hex vs `var()`. Dead rules: `.text-sage`, `.section.fit`, `.rule-soft`, `.protocol-row .p para` (phantom). The "brighter green" hover-glow (`#3AE86E`/`#7DFA9F`) has no token + 3 spellings.

## Disposition

- Defects 1-5 → become bug-billboard items + fix-candidates when the user triages (defects 1+2 are the high-value, redesign-independent ones).
- Preservation list + cascade risk → feed the design-import (the "do not discard" reference).
- This doc is the tracked record; the gitignored `reports/` originals have the full per-area detail.
