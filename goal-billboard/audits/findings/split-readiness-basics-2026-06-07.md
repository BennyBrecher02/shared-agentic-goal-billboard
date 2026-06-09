---
title: "EXTRACTION SPLIT-READINESS BASICS — every basic the kit/claude-os split would BREAK"
kind: audit-findings
belongs_to_goal: G20            # repo-decouple / new-home (the one-time extraction)
serves_northern_star: G2
created: 2026-06-08
audit_window: 2026-06-07 (read-only; no repo mutation, no commit, no spawn)
sources_audited:
  - context/markdowns/cursor-composer-handoff/extraction-build-2026-06-07.md   # the BUILD doc (what was extracted + the gated steps)
  - context/markdowns/cursor-composer-handoff/decoupled-architecture.md        # the LOCKED architecture
  - context/markdowns/research/systems/extraction-mechanics-2026-06-07.md      # the filter-repo runbook
  - the ACTUAL repo at /Users/bennybrecher/Claude/Code/agentic-organic-os
the_extraction: >
  pull `agent-dev-env-kit` (.claude/skills + .claude/memory-mirror + work-tools + context/markdowns +
  AGENTS.md + 13 portable scripts) into its own repo via git-filter-repo; the remainder becomes
  agentic-os-orchestrators/claude-os/ (the Claude machinery: hooks, scheduler, daemons, settings, site);
  the kit is consumed by claude-os via git submodule (vendor/agent-dev-env-kit) + deploy-copy.
the_precedent: >
  The memory-mirror sync break (sync-memory.sh derives BOTH endpoints from its own repo-root → after the
  kit/claude-os split it mis-points) was a BASIC the split design should have front-loaded, caught only by
  luck. This audit hunts EVERY OTHER basic of that same class that the extraction would break un-verified.
---

# Extraction split-readiness basics — the SPLIT-BREAKERS

**Bottom line.** I checked 10 basic classes plus 4 extra ones I spotted. **9 are SPLIT-BREAKERS** (the
extraction breaks them and the plan does NOT handle the break), **3 are HANDLED**, **2 are partial /
latent**. The biggest breaker is **NOT the slug** as the prompt guessed — it is the **mass of intra-kit
references to claude-os machinery** (skills → `scripts/`, AGENTS.md → site/settings, the kit-asset
*managers* left in claude-os). The slug IS a real breaker, but it is rank #3.

The recurring root cause is the SAME one that broke `sync-memory.sh`: **the split was drawn as a clean
path-union (filter-repo `--path` list), but the CONTENTS that ride along still point at the OTHER half.**
filter-repo carries files faithfully; it does nothing about the references *inside* them.

---

## SEVERITY-RANKED BREAKERS

### 🔴 BREAKER #1 — Skills (kit) reference `scripts/…` paths that STAY in claude-os — and only 13 of ~165 scripts go to the kit
**Class 3 (skill→script refs). SPLIT-BREAKER. Highest severity — largest blast radius.**

`grep -rln "scripts/" .claude/skills/` → **77 skill files** reference `scripts/` paths. The kit takes only
**13 portable scripts**. The referenced scripts are overwhelmingly the **machinery that stays in claude-os**:

- `agentic-device-testing/SKILL.md` + refs → `scripts/ios-simulator-capture.sh`, `scripts/android-emulator-capture.sh` (NOT portable; stay in claude-os).
- `agentic-flutter-development/.../flutter-on-our-matrix.md` → the same two capture scripts.
- `agentic-page-scrutiny/SKILL.md` → `scripts/measure-asset-size.py` (not in the 13).
- `agentic-script-design/references/*` → `scripts/scheduler/resources/`, `scripts/scheduler/shards/`, `scripts/cluster/`, `scripts/dispatch-bg.sh`, `scripts/post-to-chamber.py`, `scripts/daemon-graduation-check.sh`, `scripts/event-bus/consumers/`, `scripts/hooks/*` — essentially the entire machinery catalog, used as worked SOLID/OCP examples.
- Every SKILL.md carries an `AUTO-SIBLINGS:START — managed by scripts/hooks/skill-cross-link-rebuild.sh` block (see Breaker #2).

**Why it breaks:** in the standalone kit, every one of those `scripts/…` paths is a **dangling reference** —
the file does not exist in the kit. A reader (human or agent) following the device-testing skill to
`scripts/ios-simulator-capture.sh` finds nothing. The skills are the kit's crown jewel and they are wired to
machinery they no longer ship with.

**Plan coverage:** NOT handled. The build doc verifies the *13 portable scripts have zero Claude-coupling*
(correct, narrow), but never checks the **inverse** — that the **kit's other assets reference 150+ scripts
that do NOT come along.** This is the exact `sync-memory` blind spot at 10× scale.

**Fix:**
1. Decide the contract: either (a) the capture/measure scripts the skills DEPEND ON are *also* portable and move to the kit (re-scope the portable set beyond 13 — `ios-simulator-capture.sh`, `android-emulator-capture.sh`, `measure-asset-size.py`, `dispatch-bg.sh` are arguably tool-neutral), OR (b) skills reference scripts via a documented `claude-os/` / `vendor`-relative prefix and accept that those examples are claude-os-specific.
2. Add a CI guard in the kit: `grep -rl 'scripts/' .claude/skills/` cross-checked against the kit's actual `scripts/` tree → fail on any dangling path. (Mirror of the no-client-name test.)
3. The SOLID/OCP *illustrative* refs in `agentic-script-design` are fine as prose examples IF the ref says "in the claude-os machinery" — but that re-framing must be a deliberate edit, not left implicit.

---

### 🔴 BREAKER #2 — The kit-asset MANAGERS live in claude-os; the kit can't maintain its own skills/memory
**Class 2 (cross-repo path derivation) + Class 3. SPLIT-BREAKER. Second-highest — the kit becomes un-maintainable standalone.**

The scripts that **write into / validate the kit's own assets** all live under `scripts/` (→ claude-os),
and they derive their target from their OWN repo-root — the identical mechanism that broke `sync-memory.sh`:

| Manager (stays in claude-os) | Mutates / validates (kit asset) | Post-split failure |
|---|---|---|
| `scripts/hooks/skill-cross-link-rebuild.sh` | the `AUTO-SIBLINGS` blocks inside **every** `.claude/skills/*/SKILL.md` | runs against claude-os's (now skill-less) `.claude/skills/` → no-op or error; kit's sibling blocks go stale forever |
| `scripts/lint-skill-staleness.sh`, `run-skill-staleness-scan.sh`, `run-skill-usage-count.sh`, `render-skill-metrics-data.py` | the kit's skills | scan an empty dir in claude-os |
| `scripts/tests/test-memory-within-limit.sh` | the kit's `MEMORY.md` (via the slug) | (see Breaker #3 — slug) |
| `scripts/sync-memory.sh`, `verify-memory-sync.sh` | the kit's `.claude/memory-mirror/` | **the known precedent break** — derives DST=`<own-root>/.claude/memory-mirror`, which in claude-os no longer holds the mirror |
| `scripts/bug-billboard-consolidate.sh` (IS portable, in the 13) | `context/markdowns/bug-billboard/` (kit) | OK direction (manager+target both in kit) — the ONE that was got right |

**Why it breaks:** the kit owns the *data* (skills, memory-mirror, billboards) but claude-os owns the
*tools that maintain that data*. After the split, claude-os's maintainers point at claude-os's own (empty)
`.claude/` and `context/markdowns/`, so the kit's skills/memory silently rot — no cross-link rebuild, no
staleness lint, no bloat guard, no sync.

**Plan coverage:** NOT handled. The build doc explicitly leaves `sync-memory.sh`/`verify-memory-sync.sh` in
claude-os ("they manage the Claude auto-memory mirror — machinery") — but the *mirror they sync* moves to
the kit. The §c restructure even files them under `claude-os/memory-sync/`, cementing the cross-repo break.

**Fix:** for each manager, pick one:
- Move the manager to the kit too (skill-cross-link-rebuild, lint-skill-staleness, the skill-metrics renderers are tool-neutral skill-maintenance — they belong WITH the skills). This is the cleanest.
- OR keep it in claude-os but make it operate on the **submodule path** `vendor/agent-dev-env-kit/.claude/...` (an explicit override var, NOT its own repo-root). This is the generalized fix the sync-memory break demands and it must be applied to the whole cohort, not just sync-memory.
- The memory bloat-guard / `test-memory-within-limit.sh` derive the slug from `git rev-parse` → they will validate the KIT's slug when run inside the kit (correct) but SKIP (no MEMORY.md at the kit slug) — tie this to Breaker #3.

---

### 🔴 BREAKER #3 — THE SLUG CHANGE: the brain is orphaned, and the migration is documented-but-NOT-WIRED
**Class 1 (the slug). SPLIT-BREAKER. The prompt's prime suspect — real, but rank #3, and the fix EXISTS yet is not in the steps.**

The canonical auto-memory is `~/.claude/projects/<SLUG>/memory/`, `<SLUG>` = repo abs-path with `/`→`-`.
Today's slug = `-Users-bennybrecher-Claude-Code-agentic-organic-os`. The plan moves/renames the repo
(→ `agentic-os-orchestrators`, + the new `agent-dev-env-kit` repo). **Both new repos get NEW slugs → the
existing 49-topic brain is ORPHANED at the old slug, and a fresh session reads an empty memory dir.**

**Good news:** the move-handler already EXISTS and is correct — `scripts/migrate-memory-on-move.sh` does
exactly this (cp -R old-slug/memory → new-slug/memory, dry-run by default, `--auto`, parity-verify, never
deletes the old). It was the flagged memory-gap; it is now real and solid.

**The break:** it is **NOT wired into the gated move steps.** Evidence:
- The build doc references `migrate-memory-on-move.sh` **once** — only in §c as a *file to `git mv` into
  `claude-os/memory-sync/`* (line 248). Never as a STEP the user runs at G2 (the `mv` to `agent-dev-env-kit`)
  or G4 (the rename of `$SRC` → `agentic-os-orchestrators`).
- The runbook (`extraction-mechanics`) explicitly punts: "renaming the repo directory… Claude Code will look
  under a *new* slug and the old memory won't follow… we do not solve it here. (Mitigation when it happens:
  copy …/memory)" — i.e. a parenthetical, not a step.
- G3 of the build doc says of the canonical memory "leave it exactly where it is" — TRUE for a non-renaming
  redeploy, but **G2 and G4 DO change the path**, so "leave it" is wrong for those steps.

**Subtle compounding factor:** which repo even *owns* the brain? The memory describes the WHOLE system
(skills, machinery, goals). After the split it has no single home. The slug will track whichever repo the
user opens — likely **claude-os** (that's where they'll drive Claude). So the migration target should be
the claude-os slug, NOT the kit slug — but the kit's own memory-managers (Breaker #2) then look at the kit
slug and find nothing. The plan resolves none of this.

**Fix:**
1. Wire `migrate-memory-on-move.sh --auto <new-path> --apply --with-transcripts` as an **explicit gated
   step** in BOTH G2 (if the kit is ever opened as a Claude project) and G4 (the `$SRC` rename) — run it
   BEFORE the `mv`, per its `--auto` design.
2. Decide + DOCUMENT the brain's single home (recommend: **claude-os**, since the orchestrator drives Claude
   and the machinery the memory mostly describes stays there). The kit ships the memory-MIRROR (in-repo,
   subagent-readable) but the live auto-memory tracks claude-os's slug.
3. Add the post-move VERIFY the script already prints (pulse MEMORY vital sign non-zero at the new slug) as
   a checklist line in the gated steps.

---

### 🟠 BREAKER #4 — AGENTS.md (kit) is the universal entry but describes claude-os's site + settings + scripts
**Class 5 (AGENTS.md/CLAUDE.md). SPLIT-BREAKER. The kit's front door lies about the kit.**

`AGENTS.md` moves to the kit (it's in the path-union). But its CONTENT is written for the *combined* repo:
- L10–11: "plus the **Evium Charging** website (Astro + Tailwind v4) it builds. Site source is at the repo
  root (`astro.config.mjs`, `src/`, `package.json`)." — **none of that is in the kit** (site stays in
  claude-os/site).
- L23: GATED `.claude/settings.json` — **not in the kit.**
- L26–28: "Ports… from `scripts/project-config.json`… `astro dev` binds 43700 via `astro.config.mjs`." —
  **project-config.json + astro.config.mjs stay in claude-os.**
- L16: `scripts/hooks/alignment-sweep.sh` — **stays in claude-os.**
- L34: bug-billboard at `context/markdowns/bug-billboard/` — OK (that path IS in the kit).

**Why it breaks:** AGENTS.md is the harness-agnostic entry every non-Claude agent reads (Cursor, Composer).
In the standalone kit it tells the reader the repo contains a website + settings.json + a scripts/ substrate
that aren't there → instant confusion + dangling pointers, on the FIRST file an agent loads.

**Plan coverage:** PHASE-A scrub (G0) only swaps client-name *identity literals* (Evium→AgenticOS). It does
NOT re-scope AGENTS.md's structural claims about what the repo contains.

**Fix:** AGENTS.md needs a kit-vs-orchestrator split of its own:
- The **kit's** AGENTS.md describes ONLY the kit (skills, journal, work-tools, portable scripts, conventions).
- A **separate** AGENTS.md in `agentic-os-orchestrators/` (or claude-os/) describes the machinery + site.
- The "Site source / astro / settings.json" prose moves to the claude-os entry, not the kit's.

---

### 🟠 BREAKER #5 — CLAUDE.md is NOT in the kit path-union, and its `@AGENTS.md` import will dangle
**Class 5 (AGENTS.md/CLAUDE.md). SPLIT-BREAKER. Two coupled problems.**

`CLAUDE.md` (repo root) is **not** in the filter-repo `--path` union → it stays ONLY in the remainder
(claude-os). Two breaks:
1. **The import dangles.** `CLAUDE.md` line 15 is `@AGENTS.md`. After the split, `AGENTS.md` moves to the
   KIT, but `CLAUDE.md` stays in claude-os. The relative `@AGENTS.md` import resolves against CLAUDE.md's
   own dir (claude-os root), where AGENTS.md **no longer exists** → the import silently loads nothing, so
   claude-os loses the entire convention bridge at startup.
2. **The kit has no Claude entry point.** If the kit is ever opened directly as a Claude Code project (e.g.
   to maintain skills), there's no CLAUDE.md → no compaction-survival instructions, no memory pointer.

**Plan coverage:** NOT handled. CLAUDE.md is never mentioned in the build doc's path decisions; the §c tree
omits it entirely.

**Fix:**
1. claude-os keeps a CLAUDE.md, but its `@AGENTS.md` must point at the submodule: `@vendor/agent-dev-env-kit/AGENTS.md` (or the deploy-copied path) — OR claude-os keeps its OWN AGENTS.md (per Breaker #4's split) and imports that.
2. The kit ships its own minimal CLAUDE.md (or relies on AGENTS.md alone) so it's self-describing when opened directly.
3. Whoever owns the canonical memory pointer (Breaker #3) — that CLAUDE.md must live in the same repo as the slug-tracked brain.

---

### 🟠 BREAKER #6 — `public/MANIFESTO-CHECKLIST.md` symlink crosses the split boundary
**Class 7 (symlinks). SPLIT-BREAKER. Confirmed by `find -type l`.**

`public/MANIFESTO-CHECKLIST.md -> ../context/markdowns/MANIFESTO-CHECKLIST.md`. After the split:
- `public/` stays in claude-os (site).
- `context/markdowns/` (the target) moves to the KIT.
→ The symlink in claude-os points at `../context/markdowns/MANIFESTO-CHECKLIST.md`, which **no longer
exists in claude-os** → **dangling symlink**; the site can't serve the manifesto, and the
[user-owns-"done"] discipline (`MANIFESTO-CHECKLIST.md` is where the USER checks items solved) loses its
served copy.

The other 9 `public/*` symlinks (→ `../reports/…`, `../.claude/cache/…`) all stay WITHIN claude-os, so they
survive. The `reports/vq-captures/*/…` symlinks are internal to claude-os too. Only the MANIFESTO one
crosses the boundary. (The `context/Cursor-Work-Unzipped` + `context/codebases` node_modules symlinks are in
gitignored reference dirs — not tracked, irrelevant.)

**Plan coverage:** NOT handled. No symlink audit in any of the three plan docs.

**Fix:** decide where MANIFESTO-CHECKLIST.md lives. It's user-facing project state → arguably belongs with
claude-os (the orchestrator/site), not the portable kit. Either (a) keep MANIFESTO-CHECKLIST.md in claude-os
and exclude it from the kit's `context/markdowns/` path (a filter-repo `--invert-paths` carve-out), or (b)
leave it in the kit and recreate the symlink in claude-os pointing at the submodule path
`../vendor/agent-dev-env-kit/context/markdowns/MANIFESTO-CHECKLIST.md`.

---

### 🟠 BREAKER #7 — The test suite spans both halves; `run-tests.sh` + hook tests stay in claude-os but exercise kit assets
**Class 10 (test suite). SPLIT-BREAKER.**

`run-tests.sh` is NOT in the 13 portable scripts → stays in claude-os. It hard-codes
`TESTS_DIR="$REPO_ROOT/tests"`, `SCHEDULER_DIR="$REPO_ROOT/scripts/scheduler"`, and wraps
`scripts/test-affected.sh` + `scripts/test-cache.sh` (those two ARE in the 13 portable set → move to the
kit). So **the dispatcher (claude-os) calls its affected/cache substrate (kit)** — broken unless the kit is
vendored AND the paths are re-pointed.

Conversely, **12 test files reference kit assets** while living in `tests/` (claude-os):
- `tests/hooks/test_skill_cross_link_rebuild.py`, `test_sync_memory.py`, `test_verify_memory_sync.py`,
  `test_lint_skill_staleness.py`, `test_memory_sync_post.py` — all test managers that touch the kit's
  skills/memory-mirror (Breaker #2). They'll run against claude-os's empty `.claude/`.
- The `.spec.ts` visual/a11y/rail tests reference `skills/` paths in comments (low impact).

**Plan coverage:** NOT handled. The §c tree never assigns `tests/` or `run-tests.sh` to either repo, nor
addresses the affected/cache split (dispatcher in claude-os, primitives in kit).

**Fix:**
1. Keep `tests/` + `run-tests.sh` in claude-os (they mostly test machinery), but the two portable test
   primitives (`test-affected.sh`, `test-cache.sh`) either stay in claude-os too (they're test-infra, not
   obviously kit-portable — reconsider their inclusion in the 13) OR `run-tests.sh` references them via the
   submodule path.
2. The kit-asset hook tests (skill-cross-link, sync-memory, staleness) follow their managers (Breaker #2):
   if the manager moves to the kit, its test moves with it; the kit gets its own `tests/`.
3. Re-audit the 13 portable scripts: `test-affected.sh`/`test-cache.sh`/`test-cache-wrap.sh`/`test-affected-run.sh`
   are test-INFRASTRUCTURE — they belong wherever `run-tests.sh` + `tests/` live (claude-os), not the kit.
   **This is a mis-classification in the current portable set.**

---

### 🟡 BREAKER #8 — `.gitignore` is inherited wholesale; each half carries the other half's rules
**Class 6 (.gitignore staleness). SPLIT-BREAKER (low-severity but guaranteed).**

The kit (via filter-repo, which does NOT rewrite `.gitignore` — and `.gitignore` isn't even in the
path-union, so the kit gets NO `.gitignore` unless one is added). Two symmetric problems:
- **The KIT has no `.gitignore`** (it's not a kept path) → `__pycache__/`, `.DS_Store`, editor cruft,
  `.obsidian` workspace state under `context/markdowns/` land untracked-but-exposed, and there's no
  default-deny for any future sensitive folder. The kit needs its OWN trimmed `.gitignore`.
- **claude-os keeps the full `.gitignore`** which still ignores `context/markdowns/agent-inbox/*.md`,
  `context/markdowns/agent-recommendations/script-candidates.md`, the `.obsidian` `.DS_Store` rules, the
  `portable-kit/universal-memory/` rule, and the `!context/markdowns/` re-include — all of which reference
  the journal that MOVED to the kit. These become dead rules in claude-os.

**Plan coverage:** NOT handled. No `.gitignore` strategy in any plan doc.

**Fix:**
- Author a fresh kit `.gitignore`: keep `node_modules/`, `__pycache__/`, `*.pyc`, `.DS_Store`, editor cruft,
  the secret-name default-deny, and a `context/markdowns/agent-inbox/*.md` rule if the inbox journal rides
  along. Drop everything site/machinery (`dist/`, `.astro/`, `/reports/`, `public/*` symlink rules,
  `bls-*.png`, scheduler/worktree rules).
- Trim claude-os's `.gitignore`: drop the journal-specific re-includes once `context/markdowns/` is a
  submodule (the submodule has its own ignore semantics).

---

### 🟡 BREAKER #9 — settings.json hooks invoke `bash scripts/hooks/X.sh` — fine WITHIN claude-os, but the machinery+kit submodule path is unaddressed
**Class 9 (settings.json hook paths). PARTIAL — mostly handled by colocation, but two real edges.**

All ~80 hook commands use repo-relative `bash scripts/hooks/X.sh`. Since BOTH `settings.json` AND
`scripts/hooks/` stay in claude-os, the hooks themselves resolve fine post-split (this is the *good* case —
they're colocated). BUT:
- **Edge A — hooks that read kit assets.** Hooks like `skill-cross-link-rebuild` (if wired), `sync-memory`,
  any hook that reads `.claude/skills/` or `.claude/memory-mirror/` or `context/markdowns/` will, from
  claude-os, read claude-os's now-empty copies — the kit's real assets are under
  `vendor/agent-dev-env-kit/`. The hooks do NOT account for the submodule path. (Overlaps Breaker #2.)
- **Edge B — `migration-drift-sentinel.sh` + the deploy drift sentinel (§d).** The plan's §d adds a
  SessionStart drift sentinel templated on `git-drift-warn.sh`; it must watch the SUBMODULE pin + the
  deploy-copied configs, a path relationship that doesn't exist pre-split.

**Plan coverage:** the *submodule consumption* is named (vendor/agent-dev-env-kit + deploy-copy) but the
**hook→kit-asset path rewrite** is never enumerated. The colocated-hook case is implicitly fine.

**Fix:** enumerate every hook that touches `.claude/skills`, `.claude/memory-mirror`, or `context/markdowns`
and give it a kit-root override (env var → `vendor/agent-dev-env-kit/…`), defaulting to in-repo for the
pre-split / in-place case. Same generalized override the sync-memory fix needs.

---

## HANDLED (the plan covers these — credit where due)

### ✅ HANDLED — substrate/ → work-tools/ history preservation (Class adjacent)
The build doc's `--path-rename substrate/:work-tools/` is correct and VERIFIED (6 pre-rename phase-2a..2d
commits ride along; `node --test` = 20/20). No dangling `substrate/` refs in tracked machinery — the only
hits are in `work-tools/README.md` + `work-tools/rest.mjs` (self-historical, fine) and two journal
briefing docs (historical narration). **Not a breaker.**

### ✅ HANDLED — work-tools has NO runtime dependency on claude-os
**Class 8 (work-tools → claude-os deps).** `grep` across `work-tools/` for `scripts/|settings.json|project-config|CLAUDE_PROJECT_DIR`
→ a SINGLE hit, and it's a **comment** in `rest.mjs` ("port from scripts/project-config.json, 437xx band")
plus a matching error string. No `import`, no `source`, no exec of any claude-os file. work-tools reads its
port from `OS_API_PORT` env. **The kit's CLI/MCP/REST core is genuinely standalone.** Only nicety: soften
the comment so it doesn't point a kit reader at a file that isn't in the kit (cosmetic, NOT a breaker).

### ✅ HANDLED — filter-repo safety (remotes auto-detached, fsck clean, leak-check empty)
The build doc's CHECKPOINT-4 leak check + `git remote rm origin` forcing-function + `fsck --full` are
correct and verified. The mechanical extraction is sound; the breakers above are all about *reference
integrity across the boundary*, not the cut itself.

---

## PARTIAL / LATENT (watch, not blockers)

### 🟡 PARTIAL — project-config.json stale-slug cache (Class 1 adjacent)
`project-config.json` still caches `PROJECT_ROOT=/Users/bennybrecher/Claude/Design/EviumOverhaul` +
`PROJECT_SLUG=…-EviumOverhaul`. project-config.sh/.py DERIVE the live value (git toplevel wins), so it's
**latent, not active** — the derived value self-heals on a move. BUT it stays in claude-os (correct, per
runbook), and the no-git/no-harness fallback would read the stale literal. Documented in the runbook as a
known latent. **Not a fresh breaker — already flagged; tighten by refreshing the cache during G4.**

### 🟡 PARTIAL — `portable-kit/` reconciliation (Class adjacent)
`portable-kit/` (deploy-kit.sh + cursor-rules + templates + universal-memory) is NOT in the kit path-union
→ stays in claude-os. §d's plan to `mv portable-kit/deploy-kit.sh → claude-os/deploy` is therefore
internally consistent (both in claude-os). The risk is duplication: `portable-kit/universal-memory/` is a
SECOND copy of memory content that now diverges from the kit's `.claude/memory-mirror/`. The runbook flags
"reconcile not run in parallel." **Not a hard breaker; a known cleanup (retire portable-kit's bundler,
keep its allow-list).**

---

## The meta-lesson (why these were missable)

Every breaker except #3 (slug) shares the **`sync-memory.sh` signature**: a file is cleanly *assigned* to one
half by the path-union, but its **CONTENTS reference the other half** — and filter-repo + `git mv` move files,
never fix references. The plan verified the *cut* (leak-check, history, fsck) and the *one* asset's
self-containment (work-tools) thoroughly, but never ran the **inverse reference-integrity check**: "for every
file in repo A, does it point at anything that lives in repo B?" That single grep class — run in BOTH
directions — would have surfaced #1, #2, #4, #5, #6, #7, #8, #9 at once.

**Recommended pre-flight gate (add to the runbook before the GATED push):**
```
# In the rewritten kit clone: nothing should reference claude-os-only machinery.
grep -rlnE 'scripts/(hooks|scheduler|daemons|cluster|event-bus)/|\.claude/settings\.json|astro\.config|src/pages' \
  .claude/skills AGENTS.md context/markdowns/MANIFESTO-CHECKLIST.md   # → must be empty or knowingly-prose
# In claude-os after the carve-out: no dangling import / symlink into the departed journal/skills.
find . -type l -xtype l        # broken symlinks (catches the MANIFESTO link)
grep -rn '@AGENTS.md' CLAUDE.md  # import target must still resolve
```

---

## Confirmed breakers fed to `.claude/cache/alignment/verdict.jsonl` (lobe: split-breaker)
#1 skills→scripts (high), #2 kit-asset-managers-in-claude-os (high), #3 slug-migration-not-wired (high),
#4 AGENTS.md-describes-claude-os (medium), #5 CLAUDE.md-import-dangles (medium), #6 MANIFESTO-symlink-crosses
(medium), #7 test-suite-spans-both (medium), #8 gitignore-inherited (low), #9 hook→kit-asset-paths (low).
