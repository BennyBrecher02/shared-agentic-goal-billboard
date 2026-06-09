---
belongs_to_goal: G20            # repo-decouple / kit↔claude-os split (the one-time extraction)
serves_northern_star: G2        # Mission spine
cluster: portable-os-extraction
siblings:
  - ../../../research/systems/extraction-mechanics-2026-06-07.md       # the LOCKED runbook (what moves where)
  - ../../../research/systems/cross-env-skill-consistency-2026-06-07.md # the memory-mirror break + endpoint-injection fix (the REFERENCE PATTERN)
  - ../../../cursor-composer-handoff/extraction-build-2026-06-07.md     # PHASE-C layout (git mv scripts/dashboard -> claude-os/dashboard)
  - ../../../plans/post-split-dashboard-builds.md                      # the post-split dashboard build queue
kind: READ-ONLY split-readiness audit (NOTHING executed; no git ops)
date: 2026-06-08T01:39:53Z
origin: user — "will the DASHBOARD still work AFTER the kit/claude-os extraction? … MAP every data source,
  mark kit vs claude-os, find the SPLIT-BREAKERS, write the fix each (mirror the memory-mirror endpoint-injection fix)."
status: COMPLETE
---

# Dashboard split-readiness audit — does the dashboard survive the kit↔claude-os extraction?

**Type:** READ-ONLY. Nothing executed; no git ops; this is a *findings + fix-each* doc. Mirrors the
memory-mirror break class (`cross-env-skill-consistency-2026-06-07.md` §3) applied to the dashboard.

**The split (per the LOCKED runbook `extraction-mechanics-2026-06-07.md`):**
- **`context/markdowns/` (2259 files — the WHOLE journal: goal-billboard, plans, research, ideas, bug-billboard,
  audits, notes, dev-handoff-queue, multi-change-queue) → EXTRACTS to the kit** (`agent-dev-env-kit`).
- **`scripts/dashboard/` (the dashboard) + the `render-*.py` mergers + `scripts/timelapse/` + `reports/` +
  `public/` + `.claude/cache/` + the Evium site → STAY in claude-os.** (PHASE-C does `git mv scripts/dashboard →
  agentic-os-orchestrators/claude-os/dashboard`.)
- **claude-os consumes the kit as a submodule at `vendor/agent-dev-env-kit/`** (recommended PHASE-C answer).

## TL;DR — the verdict

**The dashboard does NOT survive the split as-is. It breaks in TWO layers, both the same root cause** (a path
derived from the script's own repo-root resolving to claude-os, where the journal no longer lives — the exact
memory-mirror break class). It is **EMPTY/STALE-by-construction post-split for every journal-fed surface.**

- **SPLIT-BREAKERS: 16 scripts across 2 layers** all reading `REPO / "context/markdowns/…"` where `REPO =
  Path(__file__).resolve().parents[N]` resolves to **claude-os** post-split, but `context/markdowns/` has
  **moved to the kit**.
  - **Layer 1 — 6 dashboard scripts** read the journal directly (parsers + correlate_stats + the keystone).
  - **Layer 2 — 8 `render-*.py` mergers** (the TRANSITIVE break) that produce the `reports/*/data.json` the
    dashboard reads. Even if Layer 1 is fixed, these silently emit **empty/stale** `data.json` because they
    can't reach the journal → the dashboard renders cleanly but with **dead data**. This is the nastier half:
    no crash, no 404 — just a hollow dashboard. (Biggest breaker.)
- **+ 1 dangling cross-boundary symlink:** `public/MANIFESTO-CHECKLIST.md → ../context/markdowns/MANIFESTO-CHECKLIST.md`.
- **WORKS post-split (no change needed):** `serve-dashboard.sh`, the HTML output, `data.json` output location,
  the port source, `timelapse_run.py`, the dead-detection / push-state, `parse_services`, the `reports/`-only and
  `.claude/cache`-only `public/` symlinks, and `parse_capture_index` (its data lives in `reports/` + `public/`).
- **AUDIT-PREMISE CORRECTION:** the brief listed `.claude/cache/alignment/verdict.jsonl` as a dashboard source.
  **It is NOT** — `grep` shows **zero** dashboard scripts read `verdict.jsonl`. It is alignment *write-side*
  (what this very audit feeds), not a dashboard read. No break there.

**THE FIX (one pattern, applied to all 14 script breakers): repo-root → journal-root injection**, identical in
shape to the memory-mirror `MIRROR_DIR` endpoint-injection fix. Replace the self-derived `REPO /
"context/markdowns"` with an **injected** `JOURNAL_ROOT` env var (claude-os supplies the submodule path;
self-derived fallback keeps single-repo dev working). Details in §FIX.

---

## §1 — SOURCE MAP (every path the dashboard touches; kit vs claude-os)

Legend: **WORKS** = resolves correctly post-split, no change. **BREAKER** = resolves to a path that no longer
holds the data → empty/broken. Layer-1 = a dashboard script; Layer-2 = a merger feeding a dashboard source.

### A. Direct journal reads — the dashboard scripts (`scripts/dashboard/`) — Layer 1

All derive `REPO = Path(__file__).resolve().parents[2]` (today = repo root; **post-PHASE-C `git mv` to
`claude-os/dashboard/` the depth also shifts — see §3**), then read `REPO / "context/markdowns/…"`.

| Script | Path constant | Reads (journal subdir) | Lives post-split | Verdict |
|---|---|---|---|---|
| `parse_ideas.py` | `IDEAS_DIR = REPO/"context/markdowns/goal-billboard/ideas"` | **kit** (ideas lane) | claude-os | **BREAKER** |
| `parse_pins.py` | `PINS_DIR = REPO/"context/markdowns/goal-billboard/pins"` | **kit** (pins) | claude-os | **BREAKER** |
| `parse_bug_master.py` | `BUG_MASTER = REPO/"context/markdowns/bug-billboard/master.md"` | **kit** (bug-billboard) | claude-os | **BREAKER** |
| `parse_handoff_queue.py` | `QUEUE_DIR = REPO/"context/markdowns/dev-handoff-queue"` | **kit** (handoff queue) | claude-os | **BREAKER** |
| `parse_unified_board.py` | `RESEARCH_DIR = REPO/"context/markdowns/research"`; `pins_dir = …/goal-billboard/pins` | **kit** (research + pins) | claude-os | **BREAKER** |
| `correlate_stats.py` | `AUDITS_DIR = REPO/context/markdowns/goal-billboard/audits`; `FINDINGS_DIR`; `VERIFIED_DIR = …/multi-change-queue/verified` | **kit** (audits/findings + verified) | claude-os | **BREAKER** |
| `build_render_model.py` | `MATRIX_RUNS_MD = REPO/"context/markdowns/agent-recommendations/timing/matrix-runs.md"`; mtime probes on `…/goal-billboard/pins/_index.md` + `…/dev-handoff-queue/inbox` | **kit** | claude-os | **BREAKER** (matrix-runs + 2 freshness mtimes) |

> Note `build_render_model.py` *also* imports several Layer-1 parsers (parse_pins, parse_bug_master, …) via
> `sys.path`, so it inherits their breaks even where its own constants look clean.

### B. The TRANSITIVE break — `render-*.py` mergers (`scripts/`) that feed `reports/*/data.json` — Layer 2

These are **NOT** in the §5 portable list → they **stay in claude-os**. They derive `REPO =
Path(__file__).resolve().parents[1]` (i.e. `scripts/..` = repo root) and write to `reports/timelapse/<run>/`
or `reports/*/`. The dashboard reads those `reports/*/data.json` (claude-os-side → the *read* WORKS) — **but
the data is only fresh if the merger can read the journal**, which it can't post-split.

| Merger | Reads (journal) | Writes | Verdict |
|---|---|---|---|
| `render-asks-data.py` | `…/goal-billboard/asks-log.md`, `…/notes/steering-decisions-log.md` | `reports/timelapse/<run>/data.json` (the `asks` + `stats` blocks `build_render_model` consumes) | **BREAKER (transitive)** |
| `render-billboard-page.py` | `…/goal-billboard/{active,paused,audits,archived}`, `…/plans/**`, `…/multi-change-queue/verified` | `reports/billboard/data.json` (`BILLBOARD_JSON`) + `billboard.html` | **BREAKER (transitive)** — the single richest source |
| `render-research-index.py` | `…/research` | research index data | **BREAKER (transitive)** |
| `render-subagent-data.py` | `…/research/systems` | `reports/subagents/data.json` (`SUBAGENTS_JSON`) | **BREAKER (transitive)** |
| `render-stats-data.py` | `…/research/systems` | stats data | **BREAKER (transitive)** |
| `render-scheduler-data.py` | `…/multi-change-queue` | `reports/scheduler/data.json` (`SCHEDULER_JSON`) | **BREAKER (transitive)** |
| `render-cluster-data.py` | `…/plans/automation` | cluster data | **BREAKER (transitive)** |
| `render-meta-monitoring-data.py` | `…/goal-billboard/failure-class-registry.md` | meta-monitoring data | **BREAKER (transitive)** |

> **Why this is the biggest breaker:** Layer 2 fails **silently**. Fixing only the dashboard parsers (Layer 1)
> gives you a dashboard that *renders fine* but whose billboard/asks/stats/research/scheduler tiers are **empty
> or frozen at the last pre-split run**, because the upstream `data.json` producers can't see the journal. Any
> split-readiness fix MUST cover both layers or the dashboard looks healthy and lies.

### C. claude-os-side sources — WORK post-split (co-located, no change)

| Source | Where it lives | Verdict |
|---|---|---|
| `reports/billboard/`, `reports/timelapse/<run>/`, `reports/subagents/`, `reports/scheduler/`, `reports/screenshot-monkey/`, `reports/dashboard-build/`, `reports/git-rerender/`, `reports/vq-captures/`, `reports/mobile-sim/`, `reports/stats/`, `reports/improvement-tags.json` | **claude-os** (`reports/` is **gitignored, absent from history** → stays in working tree; mergers that fill it stay too) | **WORKS** (the *read*; freshness gated on §B) |
| `.claude/cache/scheduler-events`, `.claude/cache/services-*` (`parse_services`) | **claude-os** (`.claude/cache/` gitignored runtime cache) | **WORKS** |
| `parse_git_timeline.py` — push-state, dead/ahead-of-origin detection (`origin/main` ancestor checks over `src/`+`public/`) | **claude-os** keeps `.git` + `src/` + `public/` + the remote | **WORKS** |
| `parse_capture_index.py` — vq-captures / audit-timelapse / mobile-sim thumbs | reads `reports/` + `public/audit-timelapse` (both claude-os) | **WORKS** |
| `parse_commit_diffs.py`, `parse_correlations.py`, `parse_improvement_tags.py` | `reports/` + git (claude-os) | **WORKS** |

### D. serve / HTML / data.json / port (§3 of the brief) — WORK post-split

| Piece | Detail | Verdict |
|---|---|---|
| `scripts/serve-dashboard.sh` | `REPO=$(dirname BASH_SOURCE)/..`; serves web-root `$REPO/public` (site, claude-os); `source project-config.sh`; `PORT=${PORT_DASHBOARD:-43702}`; finds the run under `$REPO/reports/timelapse`. All claude-os-side. | **WORKS** |
| Port source | `scripts/project-config.json` → `ports.dashboard = 43702` (project-config **stays claude-os**) | **WORKS** |
| Dashboard HTML | `generate_dashboard.py` writes `HTML_OUT = REPO/"reports/dashboard-build/dashboard.html"` (claude-os) — self-contained, 0-fetch | **WORKS** (output path) |
| `data.json` files | written into `reports/` by the mergers (claude-os) | **WORKS** (location); **content gated on §B** |
| `generate_dashboard.py` inbox port read | `from project_config import PORT_AGENT_INBOX` (project-config stays) | **WORKS** |

### E. The just-fixed pieces (§4 of the brief) — WORK post-split

| Piece | Detail | Verdict |
|---|---|---|
| Timelapse resolver `scripts/timelapse_run.py` | `REPO = parents[1]` (`scripts/..`); `TIMELAPSE_DIR = REPO/"reports/timelapse"`; resolves `CURRENT_RUN`. Reads only `reports/` (claude-os). `build_render_model` imports `current_run_id` via `sys.path REPO/"scripts"` (claude-os). | **WORKS** |
| Dead-detection / event-bus reads | `parse_git_timeline` (push/ancestor over the kept `.git`+`src/`+`public/`) + `SCHEDULER_EVENTS_DIR = REPO/".claude/cache/scheduler-events"` (claude-os). | **WORKS** |
| `verdict.jsonl` reads | **Dashboard does not read it** (premise correction — §TL;DR). | **N/A (no break)** |

### F. Cross-boundary assets / symlinks (§5 of the brief)

| Link | Target | Verdict |
|---|---|---|
| `public/MANIFESTO-CHECKLIST.md` | `../context/markdowns/MANIFESTO-CHECKLIST.md` → **kit** | **BREAKER (dangling symlink)** — claude-os `public/` symlink into the kit's journal; dangles post-split. |
| `public/billboard`, `public/audit-timelapse`, `public/scheduler`, `public/dashboard-build`, `public/git-rerender`, `public/audit-comparisons`, `public/dashboard-demos` | `../reports/*` → **claude-os** | **WORKS** |
| `public/cache`, `public/scheduler-events` | `../.claude/cache*` → **claude-os** | **WORKS** |

---

## §FIX — repo-root → journal-root injection (mirrors the memory-mirror `MIRROR_DIR` fix)

**Root cause (identical to the memory-mirror break):** every breaker assumes *"the journal sits under my own
repo root"* (`REPO / "context/markdowns"`). Post-split that assumption fails — the journal is in the kit, the
script is in claude-os. The memory-mirror fix was to stop self-deriving and let the orchestrator *inject* the
endpoint (`MIRROR_DIR` / `PROJECT_MEMORY_DIR`). Apply the same posture here.

**The fix — one shared resolver, injected, with a self-derived fallback for single-repo dev:**

1. **Introduce a `JOURNAL_ROOT` resolver** (one helper, imported by all 14 breakers — Python and the bash
   `serve` if ever needed):
   ```python
   # journal-root = the kit's context/markdowns, INJECTED by claude-os; falls back to self-derived for
   # single-repo / pre-split dev (keeps today working unchanged).
   JOURNAL_ROOT = Path(os.environ.get("AOS_JOURNAL_ROOT",
                       REPO / "vendor" / "agent-dev-env-kit"))   # submodule mount (PHASE-C recommended path)
   MARKDOWNS = JOURNAL_ROOT / "context" / "markdowns"
   ```
   - **Pre-split / single-repo dev:** `AOS_JOURNAL_ROOT` unset AND no submodule → fall back to `REPO/"context/markdowns"`
     (the resolver should try `REPO/"context/markdowns"` first if `vendor/agent-dev-env-kit` is absent, so the
     literal default never breaks the current monorepo). Today keeps working with zero behavior change.
   - **Post-split:** claude-os sets `AOS_JOURNAL_ROOT=<repo>/vendor/agent-dev-env-kit` (the submodule), OR the
     resolver auto-detects the submodule mount. The 16 constants become `MARKDOWNS / "goal-billboard/…"` etc.
2. **Each breaker's one-liner edit:** swap `REPO / "context/markdowns/X"` → `MARKDOWNS / "X"`. (`IDEAS_DIR`,
   `PINS_DIR`, `BUG_MASTER`, `QUEUE_DIR`, `RESEARCH_DIR`, `AUDITS_DIR`/`FINDINGS_DIR`/`VERIFIED_DIR`,
   `MATRIX_RUNS_MD`, the 2 mtime probes — Layer 1; and the 8 merger constants — Layer 2.)
3. **Symlink fix:** repoint `public/MANIFESTO-CHECKLIST.md` → the submodule
   (`../vendor/agent-dev-env-kit/context/markdowns/MANIFESTO-CHECKLIST.md`), OR drop it if the checklist
   surface is retired post-split.
4. **Single source of the injection:** put `AOS_JOURNAL_ROOT` in `project-config.json`/`project_config.py`
   (alongside `PROJECT_ROOT`) so the **derivation lives in one place** — the same "config is data, injected,
   never re-baked into scripts" posture as ports and `ENV_PREFIX`. claude-os derives it from the submodule
   path; the kit (run standalone) sets it to its own `context/markdowns`.

**Why injection (not re-deriving `parents[N]`):** bumping `parents[2]→[3]` would "fix" the depth shift from the
PHASE-C `git mv` but would STILL point inside claude-os, where the journal isn't — so depth-fixing alone does
**not** solve the break (it only addresses §3, not the kit-relocation). The journal genuinely lives in a
*different repo*; only injection (or submodule auto-detect) crosses that boundary. This is exactly why the
memory-mirror fix chose injection over re-derivation.

**Invariants preserved (same as memory-mirror fix):** read-only stays read-only; the mergers remain the single
writers of their `reports/*/data.json`; claude-os remains the dashboard's owner; the kit is a pure read source
(consumed via submodule). Single-repo dev keeps working via the fallback.

---

## §3 — the secondary depth hazard (PHASE-C `git mv`)

Independent of the kit-relocation: PHASE-C does `git mv scripts/dashboard → claude-os/dashboard` and
`git mv scripts/<merger>.py → claude-os/<merger>.py`. The dashboard scripts use `parents[2]` (expecting
`scripts/dashboard/X → repo`); the mergers use `parents[1]` (expecting `scripts/X → repo`). After the move the
directory depth changes (e.g. `claude-os/dashboard/X`), so **even the claude-os-side `REPO` derivation for
`reports/` shifts** and must be re-anchored. **Fix:** re-derive `REPO` from `git rev-parse --show-toplevel` (as
`project-config` already does) rather than a hardcoded `parents[N]`, OR adjust each `parents[N]` to the new
layout. This is a *claude-os-internal* correction (separate from the §FIX journal injection) and affects the
`reports/` paths too — fold it into the same PHASE-C pass.

---

## §4 — feed to the alignment watchdog

The confirmed breakers were appended to `.claude/cache/alignment/verdict.jsonl` (`lobe: split-breaker`,
`area: dashboard`) — see the entries dated 2026-06-08.

## §5 — counts

- Sources mapped: **~30** (8 journal subdirs × Layer-1 readers, 8 mergers, ~12 reports/cache/git sources,
  serve+HTML+port+data.json, 3 just-fixed pieces, 10 symlinks).
- Kit-side sources (the journal subdirs that move): **8** distinct journal areas
  (goal-billboard{ideas,pins,active,paused,audits,archived,asks-log,failure-class-registry}, bug-billboard,
  dev-handoff-queue, research{/systems}, multi-change-queue/verified, plans{/automation}, notes, agent-recommendations).
- Script breakers: **16** (6 Layer-1 dashboard + 8 Layer-2 mergers + counting build_render_model's matrix +
  2 freshness mtimes as part of its one entry) — call it **14 distinct files** + **1 dangling symlink**.
- Biggest breaker: **the 8 Layer-2 `render-*.py` mergers** — they fail SILENTLY (dashboard renders, data is
  empty/stale), so they're the trap a naive "fix the parsers" pass misses.
