# A78 — Dashboard Rendering-Pipeline Code Audit

**Scope:** READ-ONLY code audit of how the operational dashboard is generated. Where are the UX problems rooted in the pipeline? How hard is a re-skin/redesign (separation of concerns)? Which panels are real vs stubs? What couples it to annoying reload/jump behavior?

**Date:** 2026-05-28
**Author:** A78 sub-audit (pipeline-code zone)
**Method:** read of `render-billboard-page.py`, `scripts/timelapse/generate.py`, `scripts/timelapse/template.html`, the 11 `render-*-data.py` mergers, both data-refresh hooks, and the live artifact `reports/timelapse/2026-05-25-overnight/index.html`.

---

## TL;DR — root-cause summary

The dashboard's UX problems are rooted in **one structural fact**: the dashboard the user actually looks at is a **9,017-line, 769 KB hand-authored monolithic HTML file** (`reports/timelapse/2026-05-25-overnight/index.html`) that **no generator script produces**. The `generate.py` + `template.html` pipeline only builds the *audit-heatmap* portion (1 of 7 tabs). The other 6 tabs (Billboard, Scheduler, Subagents, Monkey, Improvement, Stats) were hand-written directly into the artifact and accreted audit-by-audit (A14 → A48), each tab bolting on more inline `<style>` (3,159 lines of CSS) and more inline panel-render JS (~5,000 lines).

So there are **two parallel, partially-overlapping pipelines** with a hard seam between them:

| Layer | What builds it | What it renders |
|---|---|---|
| **Data** | 11 `render-*-data.py` mergers + `render-billboard-page.py` | JSON blocks → `reports/billboard/data.json` + `reports/timelapse/<run>/data.json` |
| **Heatmap view** | `generate.py` ← `template.html` (placeholder substitution) | The audit time-lapse tab ONLY (heatmap / reels / findings / rail) |
| **Operational view (6 tabs)** | **NOTHING — hand-edited artifact** | Billboard, Scheduler, Subagents, Monkey, Improvement, Stats |

**This is the redesign blocker.** The data layer is clean and re-runnable; the *view* layer for the part that matters most (the operational tabs) is an un-regenerable hand-built blob. You can't "re-skin without rewriting the data logic" easily — not because the data logic is entangled, but because the **view has no source template at all**. A redesign means either reverse-engineering the 769 KB artifact into a real template, or rebuilding the operational view from scratch against the (already-clean) data.json contract.

---

## Architecture map (file + line citations)

### 1. Data layer — clean, real, re-runnable ✅

**`scripts/render-billboard-page.py`** (607 lines):
- `main()` (line 388) collects goals/audits/plans/verified-changes from `context/markdowns/goal-billboard/` + `plans/` + `multi-change-queue/verified/`.
- Computes Northern Star (line 442), Guiding Lights track-grouping (line 464–553), verified-changes rail (line 558).
- Writes `reports/billboard/data.json` (line 579) — **this is the live source the dashboard fetches.**
- **Dead code:** the entire `render()` HTML function (lines 257–385) and the `CSS` string (lines 179–254) are **no longer called**. `main()` writes a redirect stub instead (lines 416–432) and the full-HTML render is commented out (lines 436–437). So this file *named* `render-billboard-page.py` no longer renders a page — it's a **data extractor** misnamed as a page renderer. ~200 lines of dead HTML/CSS sit in the file.

**`scripts/timelapse/generate.py`** (629 lines): builds the audit-heatmap data + view.
- `build_data()` (line 221) → the routes/sections/devices matrix + severity tally.
- `render_html()` (line 332) does **5 string `.replace()` calls** against `template.html`: `__DATA_JSON__`, `__FINDINGS_HTML__`, `__RUN_ID__`, `__RUN_LABEL__`, `__RUNS_JSON__` (lines 351–356). That's the *entire* templating mechanism — no engine, just `str.replace`.

**11 `render-*-data.py` mergers** — each reads its own sources and **merges one block into the run's `data.json`**:
`render-stats-data.py` (`stats`), `render-asks-data.py` (`asks` + `waiting_on_you`), `render-cluster-data.py` (`cluster`), `render-skill-metrics-data.py` (`skill_metrics`), `render-meta-monitoring-data.py` (`meta_monitoring`), `render-meta-loop-eval-data.py` (`meta_loop_eval`), `render-bg-pattern-data.py` (`bg_launch_pattern`), `render-subagent-data.py`, `render-scheduler-data.py`, `render-research-index.py`. Pattern confirmed in `render-stats-data.py:519 merge_into_run()` — read data.json, set one key, atomic write.

**Verified: every block is real data, no stubs.**
- `reports/billboard/data.json`: `active_goals=13`, `paused_goals=3`, `audits=72`, `recent_changes=20`, `northern_star` populated, `guiding_lights=6`, `plans_by_goal=16`, `unscoped_plans=35`.
- `reports/timelapse/2026-05-25-overnight/data.json` keys: `asks, bg_launch_pattern, cluster, meta_loop_eval, meta_monitoring, skill_metrics, stats, waiting_on_you` — **all 11 operational blocks present and merged.**
- The only "empty states" are JS-side `bodyEl.innerHTML = 'No data yet…'` fallbacks (e.g. asks line 5237, meta line 5514, skill-health line 5765) — those are graceful-degradation messages, not unwired stubs. The panels ARE wired to real data.

### 2. View layer — the problem

**`scripts/timelapse/template.html`** (959 lines, 46 KB): renders the audit tab only. Grep for `billboard|chamber|northern|guiding|asks|stats|cluster|bug|goal|monkey` → **0 hits.** It has heatmap, reels, findings, scratchpad-rail. Nothing operational.

**`reports/timelapse/2026-05-25-overnight/index.html`** (9,017 lines, 769 KB): the live dashboard.
- `git check-ignore` → **IGNORED**; `git ls-files` → **NOT tracked.** It is a build artifact / hand-edited blob living only on disk.
- 7 tabs via `data-page` nav buttons (lines 2957–2981) + `.dashboard-page.active` CSS toggle (line 1764–1765): `page-timelapse, page-billboard, page-scheduler, page-subagents, page-monkey, page-improvement, page-stats` (section ids at lines 3086/3222/3386/3428/3492/3529/3555).
- Inline `<style>`: **3,159 lines.** Inline panel-render JS: ~5,000 lines.
- It `fetch()`es `/data.json` and `billboard/data.json` at runtime (20+ fetch sites, e.g. lines 3229, 4544–4547, 4773) AND embeds `const DATA = …` inline (line 4040) for the heatmap tab. So it's a **hybrid**: heatmap data baked in at generate-time, operational data fetched live.

**No script emits the operational tab markup.** `grep -rln "page-billboard|page-stats|page-monkey|…" scripts/` → empty. Confirmed: the 6 operational tabs have **no generator**. They were hand-written into the artifact.

---

## Where the UX rot is rooted in code

### Finding 1 — The artifact has no source template (THE root cause) 🚨
The operational dashboard is un-regenerable. `generate.py` will happily overwrite `reports/timelapse/<run>/index.html` from `template.html` (which lacks the 6 operational tabs) — meaning **re-running the documented generator would DESTROY the operational dashboard.** The live file is only safe because nobody re-runs `generate.py --run-id 2026-05-25-overnight` against it. This is a latent data-loss trap and the reason the dashboard can't be cleanly redesigned. *Cite: `generate.py:543` writes `index.html`; `template.html` has 0 operational hits; live artifact is gitignored + untracked.*

### Finding 2 — "Wall of text / no hierarchy" has TWO distinct roots
1. **Audit findings panel** (still raw): `generate.py:332-357 render_html()` dumps `consolidated-findings.md` into `<pre class="findings-pre">` (line 345), truncated at 8,000 chars (line 340), with only `&<>` escaped — **no markdown formatting, no hierarchy.** The live artifact bakes this in at line 3135 (`<pre class="findings-pre"># Consolidated findings…`). This IS the A57 wall-of-text mechanism, and for the findings panel it is **still unfixed** — it's a monospace pre-block dump.
2. **Chamber / billboard prose** (fixed): the A57 fix added a "minimal vanilla-JS markdown renderer" at live-artifact line 5921 (`A57 Track B`). Chamber free-text now goes through that, and an `escapeHtml` helper (line 4475) + per-field escaping (billboard lines 4937–4996) is in place. So the **chamber** wall-of-text was patched, but the **findings `<pre>`** still has the original raw-dump shape.

**Net:** the A57 incident was a *content/layout* root cause (raw markdown into a `<pre>`, no length caps, no whitespace/hierarchy handling), **NOT an escaping/XSS bug** — escaping was present. A redesign must replace the findings `<pre>` with the same markdown renderer the chamber now uses, and apply the length caps from `scripts/dashboard-content-lint.py`.

### Finding 3 — Themeability is *better than expected* but trapped in the artifact ✅⚠️
There are **41 CSS custom properties** on `:root` in the live artifact (colors `--bg/--fg/--accent/severity ramp`, plus a real **type scale** `--type-display/-body/-meta/-mono/-tile-label/-micro` and `--line-tight`). A re-skin's *color + type* surface is genuinely centralized — flipping the palette is a ~41-line change. **BUT** those vars live inside the 3,159-line inline `<style>` block of an un-regenerable artifact, and `template.html` carries a **separate, drifted copy** of the same vars (only ~18, no type scale — template.html:8-27). So there are two divergent CSS-var sources and the richer one isn't in any template. Re-skin difficulty is *low for tokens, high for structure*.

### Finding 4 — Reload/jump coupling (Stream B "anti-jump") — two sources
1. **Vite/Astro HMR:** `public/audit-timelapse → ../reports/timelapse` is a **symlink** (confirmed). When served via Astro dev, Vite watches that tree; any `render-*-data.py` writing `data.json` (which the PostToolUse hook `billboard-data-refresh.sh` fires on *every* goal/plan/queue edit) triggers an **HMR full-reload** of the open dashboard. This is the documented "dashboard jumps on every edit" annoyance. The dashboard's own auto-refresh is polite `setInterval` polling (10s billboard / 30s stats — lines 4687, 4710, 4890, 5188), but **HMR sits underneath it and yanks the whole page.** Decoupling = serve the dashboard outside Vite's watch root, or move data.json out of the symlinked tree and fetch by absolute path.
2. **Self-inflicted `location.reload()`:** the monkey-inbox "clear stuck queue" path calls `setTimeout(() => location.reload(), 800)` (line 6503). Minor (user-triggered only), but it's a hard reload in a SPA-ish dashboard.

### Finding 5 — Monolith maintainability / separation of concerns ❌
- **One file, three concerns fused:** 3,159 lines CSS + ~2,800 lines markup + ~3,000 lines JS, all inline, no imports, no components. A change to (say) the Stats tab requires editing a 9k-line file by hand and hoping you don't break tab 3.
- **No build step for the operational view** means no minification, no CSS scoping, no dead-CSS detection. CSS selectors for all 7 tabs share one global namespace.
- **The data→render contract is actually clean** (JSON blocks with documented schemas, e.g. `render-stats-data.py:12-75` schema docstring). The entanglement is NOT data↔view — it's that the *view itself* is a hand-built artifact. **Good news for redesign:** because the contract is clean, a rebuilt view (Astro components / a real templated generator) can consume the *exact same* `data.json` blocks unchanged. The data layer does not need rewriting.

### Finding 6 — Misnamed / dead code (cleanup, low-risk)
`render-billboard-page.py` carries ~200 lines of dead `render()` + `CSS` (lines 179–385) that `main()` never calls (writes a redirect stub instead, line 431). The file is a *data extractor* but named like a *page renderer*, which actively misleads anyone hunting for "where the billboard HTML is generated" (it isn't — that's the hand-built artifact). Rename → `extract-billboard-data.py` and delete the dead render path.

---

## Redesign-difficulty assessment

**Verdict: MEDIUM-HARD, but the hard part is bounded and the data layer is free.**

| Dimension | Difficulty | Why |
|---|---|---|
| Re-skin colors/type | **LOW** | 41 `:root` vars already centralize the token surface. ~1hr to re-palette — *once the vars live in a real template.* |
| Re-skin layout/structure | **HIGH** | 2,800 lines of hand markup with no components; every tab is bespoke. |
| Keep data logic intact | **TRIVIAL** | Data layer (12 scripts) is decoupled + schema-documented; a new view consumes the same `data.json` blocks. **Zero data rewrite needed.** |
| Avoid regressions | **MEDIUM** | No tests on the view; the artifact is gitignored so no diff history to lean on. |
| Stop the reload-jump | **LOW-MEDIUM** | Move data.json out of the Vite-watched symlink tree (Finding 4). |

**Cleanest code-path to a re-skin (recommended):**
1. **Promote the view to a real template.** Pull the 769 KB artifact's `<style>` (41 vars + rules) and per-tab markup into a templated source under `scripts/timelapse/` (or, better, Astro components under `src/pages/dashboard/`). The operational tabs need a *generator* the way the heatmap tab has `template.html`.
2. **Single CSS-token file.** Extract the 41 `:root` vars into one `dashboard-tokens.css` (or `:root` partial) imported by both the heatmap template and the operational view — kills the template.html-vs-artifact drift (Finding 3).
3. **Reuse the A57 markdown renderer for the findings panel** (Finding 2.1) so the last raw-`<pre>` wall-of-text is gone; wire `dashboard-content-lint.py` caps in.
4. **Decouple from Vite HMR** (Finding 4): serve dashboard data from a path Vite doesn't watch, or fetch data.json over an absolute URL outside the symlinked `public/audit-timelapse` tree.
5. **Rename + de-dead-code `render-billboard-page.py`** → `extract-billboard-data.py`; delete lines 179–385 (Finding 6).
6. **Add the operational view to the generator pipeline** so `generate.py` (or a new `render-dashboard.py`) can *rebuild the whole dashboard from data + template*, eliminating the un-regenerable-artifact trap (Finding 1) and the destroy-on-regen hazard.

**The single highest-leverage move:** Finding 1 — give the operational tabs a source template. Everything else (re-skin, anti-jump, findings-fix) becomes easy *after* the view is regenerable. Until then, any redesign is hand-surgery on a 9k-line untracked blob.

---

## Cross-references
- A57 (`A57-dashboard-content-quality-and-uxr.md`) — the wall-of-text incident; this audit confirms its root was content/layout not escaping, and that the findings `<pre>` is the one surface A57 did NOT fix.
- `dashboard-overhaul-plan.md` Stream B (anti-jump) — confirmed: HMR-via-symlink is the coupling (Finding 4). Stream C (flesh the stubs) — **good news: there are no data stubs; all 11 blocks carry real data.** The "meaningfulness" gap is a *view-design* problem, not a data-wiring one.
- `scripts/dashboard-content-lint.py` — exists; should be wired into the redesigned findings renderer.
- `feedback_dashboard-content-quality.md` (5 pillars) + `feedback_dashboard-centerpiece.md` — the redesign should re-apply these against the rebuilt view.

BG-COMPLETE-SENTINEL