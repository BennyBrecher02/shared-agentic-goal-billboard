---
idea_id: I-009
title: Exclude dev-only dashboard bridges from production dist/
lifecycle: captured
source: chamber
created: 2026-06-03
updated: 2026-06-07
---

# I-009 — Exclude dev-only dashboard bridges from production dist/

**Source:** taken over from a spawned chip's "Suggested task" (2026-06-03). Benny: *"id rather u handle it
yourself"* — OWNED here as a tracked idea, deliberately NOT re-spawned as a chip.
**State:** PRE-DEPLOY · LATENT — no deploy pipeline is committed yet, so nothing ships today.

**Problem:** `npm run build` copies `public/`'s gitignored dev-bridge symlinks wholesale into `dist/` (Astro
copies `publicDir` with no per-entry ignore) → `dist/cache` (~98 files from `.claude/cache`),
`dist/audit-timelapse` (~2,950 files), `dist/git-rerender`, `dist/audit-comparisons`, etc. Internal dev-only
data would ship in a public deploy.

**Fix (do this BEFORE the first Vercel deploy):** a `postbuild` npm step that removes those internal bridge
dirs from `dist/` (Astro can't ignore them at copy-time), or a deploy-time ignore. The build-robustness fix
(07af4d5) does not worsen this — it was already the case whenever the build succeeded.

**Trigger:** the moment we wire the Vercel deploy (tied to `cursor-composer-evaluation` / the vercel-serverless
research). Not urgent for tonight's push (dist/ is gitignored — never pushed).
