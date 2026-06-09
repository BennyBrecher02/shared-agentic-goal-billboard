---
title: "Evium-in-the-Machinery: Agnosticism-Debt Audit + Safe Rename Plan"
kind: audit/findings-and-plan
status: findings-and-plan
mode: READ-ONLY — inventory + plan only, NO renames/edits performed
created: 2026-06-07
scope: OS MACHINERY ONLY (scripts/, .claude/settings*, tests/, ~/Library/LaunchAgents/, plist templates)
excludes: the Evium WEBSITE src/ (legit client content), client-site docs, project_evium-overhaul memory
cluster: portability
siblings:
  - hardcoded-path-portability-audit.md   # the PATH dimension — DONE; this is the NAMING dimension it deferred (§7)
  - repo-decouple-ownership-transition-plan.md
trigger: >
  User: "i thought you made our system project agnostic? why do we have so many references in
  naming to evium? ... it shouldnt be this deeply baked into our code dna, audit that and safely
  plan to fix." Evium is a CLIENT PROJECT; its name leaked into the OS's OWN machinery DNA.
headline: >
  The portable SKILLS + the website src/ are clean. The leak is concentrated in the MACHINERY
  NAMESPACE: ~221 distinct EVIUM_* env vars (79 of them kill-switches) hardcoded inline across
  120 script/test/settings files, 6 LIVE com.evium.* launchd daemons + their templates, and the
  EviumOverhaul worktree-prefix that the settings.json safety patterns hardcode literally. None
  break on move (path-portability already fixed that); this is purely BRANDING-DNA debt. The
  rename is a literal find/replace, NOT a config flip — ENV_PREFIX is recorded but UNREAD.
---

# Evium-in-the-Machinery: Agnosticism-Debt Audit + Safe Rename Plan

> **READ-ONLY deliverable.** No file was renamed or edited; no plist was touched; no launchctl
> command was run. This is the inventory + the gated, sequenced, compat-shimmed rename plan.

---

## 0. TL;DR — the diagnosis in five lines

1. **The scrub was half-done.** The *portable content* (skills, the website `src/`) is **100% clean**
   — `src/` has **0** hits for `EVIUM_`, `com.evium`, or `EviumOverhaul`. The leak is entirely in
   the **OS's own machinery namespace**.
2. **Three coupled namespaces carry the client name:** `EVIUM_*` env vars (the kill-switches +
   overrides), `com.evium.*` launchd labels (the daemons), and `EviumOverhaul` (the worktree-path
   prefix). Counts + blast-radius in §1.
3. **Nothing breaks today.** The companion path-portability audit (`hardcoded-path-portability-audit.md`)
   already DERIVES the repo path/slug live, so a move self-heals. These names are *branding*, not
   *paths* — they were that audit's **explicitly deferred §7 tail** ("the held part of manifesto §4").
   This doc is that deferred dimension, now requested.
4. **The rename is a literal find/replace, not a one-line config flip.** `project-config.json` records
   an `ENV_PREFIX: "EVIUM"` key as the intended single source — but **nothing reads it to build a var
   name** (verified: zero dynamic `${PREFIX}_X` construction). All 221 vars are hardcoded string
   literals. The compat-shim phase is the chance to finally *wire* that key so it's never re-baked.
5. **Most of it is mechanically safe; the dangerous slice is small + already has machinery.** Scripts,
   tests, templates, and docs are mechanically-safe edits. The GATED slice is exactly two things:
   `.claude/settings*.json` and the 6 LIVE launchd plists — and existing scripts (`reload-all.sh`,
   `stage-repointed-plists.sh`, `settings-proposed-sync.sh`) already implement the dances they need.

---

## 1. Inventory — counts by TYPE, with blast-radius

Noise excluded: `.git/`, `node_modules/`. "Machinery" = executable/config (`.sh` `.py` `.json`
`.plist` `.template`) + the live LaunchAgents dir. "Docs" = `context/markdowns/` + `.claude/skills/`
+ `.claude/memory-mirror/` (prose that *describes* the machinery and will follow the rename).

### 1.0 — Grand totals (raw, for scale)

| Token | Raw occurrences (repo) | Note |
|---|---:|---|
| `evium` (case-insensitive, all forms) | **5,580** | inflated by docs + the word "Evium" in client-site prose |
| `EVIUM_` (env-var prefix) | **1,267** | |
| `com.evium` (plist label) | **276** | |
| `EviumOverhaul` (path/worktree) | **296** | |

The raw 5,580 is **not** the action surface. The machinery surface is far smaller and is what follows.

### 1.1 — TYPE 1: `EVIUM_*` env-var prefix (the kill-switches + overrides)

| Metric | Count |
|---|---:|
| **Distinct `EVIUM_*` variable names** (the namespace to migrate) | **~221** |
| &nbsp;&nbsp;↳ of which **`EVIUM_*_OFF` kill-switches** | **79** |
| &nbsp;&nbsp;↳ remainder = `_MODE` / `_OK` / `_DIR` / `_REPO` / threshold/state overrides | ~142 |
| Total `EVIUM_` refs in `scripts/` (`.sh` + `.py`) | 481 |
| Total `EVIUM_` refs in `scripts/**/*.json` (settings-patch fragments) | 60 |
| Total `EVIUM_` refs in `tests/` | 231 |
| Total `EVIUM_` refs in `.claude/settings.json` | 52 |
| Total `EVIUM_` refs in `.claude/settings.proposed.json` (mirror of the above) | 52 |
| Total `EVIUM_` refs in `context/markdowns/` (docs — follow the rename) | 357 |
| **Distinct MACHINERY files touched** (scripts `.sh`/`.py`/`.json` + tests + `.claude/settings*`) | **120** |

**Location hot-spots (machinery):** `.claude/settings.json` + `.claude/settings.proposed.json`
(52 each, the wiring); `tests/spawned-task-result-surface-test.sh` (38),
`tests/findings-life-thread-surface-test.sh` (36), `tests/idea-gap-surface-test.sh` (32);
`scripts/hooks/findings-life-thread-surface.sh` (25), `scripts/hooks/idea-gap-surface.sh` (22),
`scripts/hooks/inbox-server-guard.sh` (18). `scripts/project-config.sh` (11) and
`scripts/project_config.py` carry the `ENV_PREFIX` plumbing.

**The COUPLING is 4-deep, not 2-deep (this is the load-bearing nuance).** The prompt called for
mapping settings.json↔script pairs; the reality is each kill-switch can appear in up to **four**
coupled sites that must move together or the switch silently breaks:

| # | Coupled site | Example for `EVIUM_HARVEST_OFF` |
|---|---|---|
| (a) | **settings.json hook command guard** | `"[ -z \"$EVIUM_HARVEST_OFF\" ] && bash scripts/harvest-spawned-tasks.sh"` (line 93) |
| (b) | **the script's own early-exit** | `harvest-spawned-tasks.sh` checks the same var (defensive double-gate) |
| (c) | **the settings-PATCH JSON fragment** | `scripts/hooks/settings-patches/harvest-spawned-tasks-wire.json` (the deployable source of the settings.json entry) |
| (d) | **the test** | a `tests/*-test.sh` asserting `EVIUM_HARVEST_OFF=1 → silent exit 0` |
| (e) | **`settings.proposed.json`** | a byte-mirror of (a) |

> **Why a one-sided rename silently breaks the switch:** if you rename only (a) the settings.json
> guard to `AOS_HARVEST_OFF` but leave (b) the script reading `EVIUM_HARVEST_OFF`, then setting the
> *new* var no longer reaches the script's internal gate — the hook still fires via (a) but the
> script ignores the off-switch (or vice-versa: the guard passes but the script's own gate, keyed to
> the old name that the user no longer sets, never trips). The off-switch becomes a no-op. **All
> sites of one var must flip atomically.** (See §4.2 for the atomic-per-var method.)

**The non-flip finding (most important for strategy):** `ENV_PREFIX: "EVIUM"` exists in
`project-config.json` and is read into `$PROJECT_ENV_PREFIX` (sh) / `ENV_PREFIX` (py), but a grep for
any dynamic construction `${PREFIX}_X` returns **empty** — *nothing* builds a var name from it. Every
`EVIUM_*` is a hardcoded literal. So the rename is a textual find/replace across 120 files, **not** a
config edit. (Upside: the shim phase can finally make this dynamic — see §4.2 Option B.)

### 1.2 — TYPE 2: `com.evium.*` launchd plist labels (the daemons)

| Metric | Count |
|---|---:|
| **LIVE plists installed** in `~/Library/LaunchAgents/` | **6** |
| &nbsp;&nbsp;↳ **all 6 currently LOADED** (`launchctl list` → status 0, clean) | ✅ |
| `com.evium` refs in `scripts/` (`.sh`/`.py`) | 46 |
| `com.evium` refs in plist **templates** (`.template`) | 18 |
| `com.evium` refs in `scripts/**/*.json` | 1 |
| `com.evium` refs in `context/markdowns/` (docs) | 207 |
| **Distinct MACHINERY files** (scripts + templates + live plists) | **22** |

**The 6 live, loaded labels (the GATED set):**
- `com.evium.heartbeat-independent` → `scripts/hooks/heartbeat-independent-tick.sh`
- `com.evium.daemon2` → `consumer-memory-consolidator-daemon2.sh`
- `com.evium.daemon3` → `consumer-audit-catalog-scan-daemon3.sh`
- `com.evium.daemon4` → `consumer-steering-log-compact-daemon4.sh`
- `com.evium.daemon5` → `consumer-bug-billboard-groom-daemon5.sh`
- `com.evium.daemon10` → `consumer-heartbeat-tier-low-daemon10.sh`

**Templates that mint NEW labels** (uninstalled but will reproduce the leak if rendered):
`scripts/heartbeat/heartbeat-launchd.plist.template` (→ `com.evium.heartbeat-independent`),
`scripts/heartbeat/launchd-template/com.evium.heartbeat.plist.template` (label `com.evium.heartbeat`,
also an evium-in-FILENAME leak), `scripts/daemons/research-furthering-launchd.plist.template`
(→ `com.evium.research-furthering`).

**Existing machinery the rename can REUSE (do not reinvent):**
- `scripts/daemons/reload-all.sh` — already globs `com.evium.daemon*.plist` and does the
  `enable`→`bootout`→`bootstrap` idempotent reload dance. Read-only on plist *content*.
- `scripts/stage-repointed-plists.sh` — already renders repo-pointed copies into a staging dir and
  prints the **user-gated** install line (`cp … ~/Library/LaunchAgents/ && reload-all.sh`); launchctl
  is correctly left to the user.
- `scripts/daemon-health.sh` + `scripts/tests/daemon-health.test.sh` — assert the labels are loaded
  (so they double as the post-rename verification AND are themselves in the blast radius).

> **The label LITERAL is a path-independent identity** (it's not a filesystem path), so it does NOT
> self-heal on move/clone — confirmed by the path audit §2.5. A rename here is a true rename, and
> because the agents are LOADED it requires the unload-old / load-new dance (§4.3), which is GATED.

### 1.3 — TYPE 3: `EviumOverhaul` worktree/path prefix

| Metric | Count |
|---|---:|
| `EviumOverhaul` refs in `scripts/` (`.sh`/`.py`) | 56 |
| `EviumOverhaul` refs in `scripts/**/*.json` (`project-config.json`) | 7 |
| `EviumOverhaul` refs in `.claude/settings.json` + `.proposed.json` | 16 (8 each) |
| `EviumOverhaul` refs in `tests/` | 9 |
| `EviumOverhaul` refs in `context/markdowns/` (docs) | 193 |
| **Distinct MACHINERY files** (scripts + tests + settings) | **27** |

Two sub-classes, with very different handling:

**(3a) The worktree-name PREFIX — already config-derived (mechanically safe).** The scheduler builds
worktree paths as `<repo>/../<PROJECT_NAME>-change-<id>` and `<...>-calibration-<stamp>`. `PROJECT_NAME`
comes from `project-config.json` (`"PROJECT_NAME": "EviumOverhaul"`) and ~5 scripts read
`${PROJECT_NAME:-EviumOverhaul}` (`scripts/scheduler/worktree.py`, `scheduler/__main__.py`,
`calibration-cleanup.sh`, `sample-resources.sh`, `hooks/worktree-remove-safety.sh`). **Changing the
one JSON value re-points all of them** — this half is a clean config edit.

**(3b) The settings.json safety PATTERNS — hardcoded literals (GATED + coupled to 3a).** These do
NOT derive from `PROJECT_NAME`; they bake the literal string:
- `.claude/settings.json:540-544` — `Bash(git worktree add/remove ../EviumOverhaul-change-*)` and
  `-calibration-*` (the **allow** patterns the scheduler relies on).
- `.claude/settings.json:645-646` — `Bash(git worktree remove ../EviumOverhaul[/])` (deny/safety).
- `.claude/settings.json:666` — `Bash(mv * ~/.Trash/EviumOverhaul)` (the Trash grace path).

> **THE COUPLED-PAIR TRAP for worktrees (call out):** `PROJECT_NAME` (3a) and these literal patterns
> (3b) are a coupled pair. If you rename `PROJECT_NAME` → `AgenticOS` in the JSON but leave the
> settings.json patterns matching `../EviumOverhaul-change-*`, then the scheduler will try to create
> `../AgenticOS-change-<id>` — which **no longer matches the allow-pattern** → the Bash-tool permission
> layer **blocks the worktree creation**, AND `worktree-remove-safety.sh` (which derives from the new
> `PROJECT_NAME`) will guard a different name than settings.json allows. The multi-change scheduler
> silently stops being able to make worktrees. **3a and 3b must flip in the same change.**

**Note — the OTHER `EviumOverhaul` refs are PATH literals, not naming**, and are the
already-planned path-portability surface (`/Users/.../EviumOverhaul` byte-identical floors in
`project_config.py` `_DEFAULTS`, the analyzers' except-branches, etc.). Those are *out of scope here*
(they're `hardcoded-path-portability-audit.md` §6.1's job and self-heal via derivation). This audit
counts only the **naming** use of the bare `EviumOverhaul` token as a worktree/identity prefix.

### 1.4 — TYPE 4: hardcoded strings / comments / config keys

- **`project-config.json`**: `"PROJECT_NAME": "EviumOverhaul"` (the 3a source) + `"ENV_PREFIX":
  "EVIUM"` (the recorded-but-unread rename anchor) + the `_doc` prose that says "Evium-specific
  identity" and "the ~90 EVIUM_* … hardcode this prefix inline. Renaming them system-wide is the held
  part of manifesto §4." **The config file already documents this exact debt.**
- **`.claude/launch.json:5`**: `"name": "evium"` — a launch-profile display name (cosmetic, mechanically safe).
- **`scripts/heartbeat/launchd-template/com.evium.heartbeat.plist.template`** — `evium` in the
  **filename** itself (only TYPE that's a filename rename, not just content).
- **Comments/docs in machinery**: `reload-all.sh` ("reload every evium autonomic-daemon"),
  `stage-repointed-plists.sh`, dozens of `# EVIUM_X_OFF` kill-switch doc-comments inside hooks.

### 1.5 — EXCLUDED as legitimate (NOT debt — do not touch)

Per the prompt's exclusion list, these are the client project legitimately named and stay:
- **The Evium website `src/`** — 0 machinery hits; any "Evium" there is brand copy. ✅ leave.
- **`.claude/memory-mirror/project_evium-overhaul.md`** + the canonical `project_evium-overhaul.md`
  memory topic — the client-project tracker. Legit.
- **Client-site docs** that discuss the Evium overhaul (design handoffs, the overhaul phase plans,
  audits OF the site). These *describe the client*, not the OS.
- **The ~170 doc files** that mention a machinery token are a *grey zone*: they describe the
  machinery, so they should follow the rename for correctness, but they don't execute → lowest
  priority, cosmetic, batchable last (§4.5). (The raw "1402 docs contain the word evium" figure is
  almost all client-content noise — only **170** docs contain a machinery token `EVIUM_`/`com.evium`/
  `EviumOverhaul`.)

---

## 2. RISK + GATED map

| Surface | Files | GATED? | Why / handling |
|---|---:|:--:|---|
| **`.claude/settings.json`** (52 `EVIUM_` + 8 `EviumOverhaul`) | 1 | **GATED** | A66 high-risk-always-gated; the settings-edit-guard PreToolUse hook BLOCKS agent Write/Edit. User applies, or agent stages a patch + `EVIUM_SETTINGS_EDIT_OK=1` override the **user** sets. |
| **`.claude/settings.proposed.json`** (byte-mirror) | 1 | **GATED-adjacent** | Synced to/from settings.json via `settings-proposed-sync.sh`; must move in lockstep or `migration-drift-sentinel.sh` / `test-settings-proposed.sh` will (correctly) flag drift. |
| **6 LIVE `com.evium.*` plists** in `~/Library/LaunchAgents/` | 6 | **GATED** | launchd install/`launchctl` is user-only (A66, CLAUDE.md §2). Renaming a LOADED label = unload-old + install-new + load-new (§4.3). |
| **`scripts/**` (`.sh`/`.py`)** — inline `EVIUM_*`, `com.evium` strings, `${PROJECT_NAME:-EviumOverhaul}` | ~110 | **SAFE** | Plain source edits. The `src-edit-guard` is scoped to the **website** `src/`, NOT `scripts/` — verified; `scripts/` edits are not gated. |
| **`scripts/hooks/settings-patches/*.json`** (the deployable settings fragments) | ~12 | **SAFE to edit** | They're source; but editing them does NOT change live settings.json until the user redeploys → safe to stage, gated to take effect. |
| **plist `.template` files** (mint `com.evium.*`) | 3 | **SAFE to edit** | Editing a template is a source edit; it only affects a FUTURE render+install (which is gated). |
| **`tests/` + `scripts/tests/`** (231 `EVIUM_` + 9 `EviumOverhaul`) | ~30 | **SAFE** | Test edits. 13 files hard-assert `EVIUM_*_OFF=1` behavior → they are BOTH the regression net AND in the blast radius; migrate them in the same per-var change so they keep guarding. |
| **`scripts/project-config.json`** (`PROJECT_NAME`, `ENV_PREFIX`) | 1 | **SAFE** | Plain config edit; the 3a worktree-prefix flip + the ENV_PREFIX anchor. |
| **`.claude/launch.json`** (`"name":"evium"`) | 1 | **SAFE** | Cosmetic display name. |
| **`context/markdowns/` + skills + memory-mirror docs** (~170 files) | ~170 | **SAFE** (mirror caveat) | Prose; cosmetic. **EXCEPTION:** never hand-edit `.claude/memory-mirror/` topic files (feedback_memory-mirror-sync) — edit the canonical auto-memory copy + let `sync-memory.sh` propagate. |

**GATED total: 8 surfaces** — `settings.json` (1) + `settings.proposed.json` (1) + 6 live plists.
Everything else is mechanically safe source/test/doc editing.

---

## 3. Neutral-prefix options (USER PICKS — not decided here)

Naming is the user's call. Three internally-consistent option sets; pick ONE row and it applies across
all three TYPEs so the system reads coherently:

| Option | Env-var prefix | launchd label namespace | Worktree prefix (`PROJECT_NAME`) | Feel |
|---|---|---|---|---|
| **A — `AOS`** | `AOS_*` (e.g. `AOS_HARVEST_OFF`) | `com.agentic-os.*` | `agentic-os` → `agentic-os-change-<id>` | Ties to the repo name "agentic-organic-os"; most self-descriptive. |
| **B — `ADE`** | `ADE_*` (Agentic Dev Env) | `com.agentic-dev.*` | `agentic-dev` | Emphasizes "general agentic *development* system" (manifesto §4 language). |
| **C — `OS`** | `OS_*` (e.g. `OS_HARVEST_OFF`) | `com.organic-os.*` | `organic-os` | Shortest; matches "Organic-OS" branding. Caveat: `OS_` is a slightly generic prefix (small collision risk with system env). |

Recommendation framing (not a decision): **A (`AOS`)** has the cleanest property — the worktree prefix
can then be a pure derivation of the repo directory name (`basename $(git rev-parse --show-toplevel)` =
`agentic-organic-os`), eliminating even the `PROJECT_NAME` literal. **C (`OS_`)** is the only one with a
mild namespace-collision caveat (some shells/tools export `OS`-ish vars). All three avoid the retired
AI-default ports issue (irrelevant here) and are equally mechanical to apply.

> The worktree-prefix option also opens a **simplification**: instead of any literal, derive it from
> the repo folder name. That makes a future second rename free (rename the folder → prefix follows).
> Flag for the user as a bonus, contingent on the prefix choice.

---

## 4. SAFE SEQUENCED MIGRATION PLAN

Ordering principle (from the path audit's "land in-tree → verify green → THEN touch gated surfaces"):
do every **mechanically-safe** edit first behind the existing kill-switches, prove the system still
works with a **compat-shim** that honors BOTH old and new names, and only flip the **gated** surfaces
(settings.json + live plists) as the final, user-driven step. Order TYPEs so the riskiest (env-var
coupling) gets the shim, and the gated set is last.

### 4.0 — Phase 0: Decision + freeze (S, no risk)

1. User picks a prefix row from §3 (env prefix + label namespace + worktree prefix).
2. Record the choice in `project-config.json` `ENV_PREFIX` (+ a new `LAUNCHD_NAMESPACE` key and reuse
   `PROJECT_NAME` for the worktree prefix). **This is the moment to make these keys actually READ
   (today they're recorded but inert).**
3. Snapshot the live state for rollback: `launchctl list | grep com.evium > /tmp/evium-labels.before`,
   `cp .claude/settings.json .claude/settings.json.pre-rename`.

### 4.1 — Phase 1: TYPE 3b/3a worktree prefix (S–M, mostly SAFE + one GATED edit)

The smallest coupled pair; do it first to exercise the atomic-pair discipline on a 6-line surface.
- **SAFE:** flip `project-config.json` `"PROJECT_NAME"` → new prefix. The ~5 `${PROJECT_NAME:-…}`
  readers auto-follow (and update their inline `:-EviumOverhaul` fallback literals to the new default).
- **GATED (atomic with the above):** the user edits `.claude/settings.json` lines 540-544/645-646/666
  to match the new prefix (`../<new>-change-*`, `../<new>-calibration-*`, `~/.Trash/<new>`). Mirror to
  `settings.proposed.json`.
- **Verify:** `worktree-remove-safety.sh` self-test + a dry `git worktree add ../<new>-change-test`
  (must be ALLOWED), then remove. Confirm `test_worktree.py` / `test_worktree_remove_safety.py` pass
  with the new prefix.
- **Effort:** S. **Risk:** LOW-MED — the only failure mode is the coupled-pair trap (§1.3); doing
  both edits in one change eliminates it.

### 4.2 — Phase 2: TYPE 1 env-var rename WITH a compat-shim (L, SAFE edits + GATED settings flip last)

This is the big one (221 vars × up to 5 sites × 120 files). The shim makes it non-breaking mid-flight.

**The compat-shim (read BOTH old + new during transition).** Two viable mechanisms — present both,
user/impl picks:

- **Option A — per-switch OR-guard (mechanical, no architecture change).** Change each settings.json
  guard from `[ -z "$EVIUM_X_OFF" ]` to `[ -z "$EVIUM_X_OFF" ] && [ -z "$AOS_X_OFF" ]` (fires only if
  NEITHER off-var is set), and each script's internal early-exit to
  `[ -n "${EVIUM_X_OFF:-}${AOS_X_OFF:-}" ] && exit 0`. During the window, setting EITHER name silences
  the hook → nothing breaks whichever name a caller uses. After the window, drop the `EVIUM_` half.
  *Pro:* dead-simple, per-switch, no new code path. *Con:* doubles the guard text temporarily.

- **Option B — central shim that re-exports old→new (architectural, the better end-state).** Add to
  `project-config.sh` / `project_config.py` a loop that, for every known suffix, does
  `: "${AOS_X_OFF:=$EVIUM_X_OFF}"` (new inherits old if only old is set) and the reverse during
  transition. Since `project-config` is already sourced by the hooks, ONE shim block covers all 221
  vars without touching each guard. **This is also where you finally WIRE `ENV_PREFIX`:** build the
  var names from `$PROJECT_ENV_PREFIX` so the prefix is data, not 221 literals — re-baking becomes
  impossible. *Pro:* single point of change, fixes the root design defect (the unread `ENV_PREFIX`).
  *Con:* requires the shim to enumerate suffixes (or use `compgen -v | grep ^EVIUM_` to auto-map).

**Atomic-per-var sweep (the safe order within Phase 2):**
1. **SAFE first — rename the non-gated 4 of the 5 coupled sites per var, together:** the script
   early-exit (b), the settings-patch fragment (c), the test (d). Do it **one var (or one small
   cluster) at a time**, run that var's test (`EVIUM_X_OFF=1` → still silences via the shim; `AOS_X_OFF=1`
   → silences via new name). 13 test files already assert this — extend each to assert BOTH names
   during the window.
2. Land the **compat-shim** (Option A or B) so old-named callers (incl. the un-flipped settings.json)
   keep working.
3. **GATED last — flip settings.json (a) + settings.proposed.json (e)** to the new names, user-applied
   (or via a staged patch + the user setting `EVIUM_SETTINGS_EDIT_OK=1`). Because the shim honors both,
   this flip is non-urgent and reversible.
4. **De-shim** (a later phase, after a clean window): remove the `EVIUM_` half from guards/shim, delete
   the dual-name test assertions. `migration-drift-sentinel.sh` confirms no `EVIUM_` survives in
   settings.
- **Effort:** L (the bulk of the project). **Risk:** LOW *with* the shim (every intermediate state has
  both names live), MED without it (the coupled-pair trap, §1.1).

### 4.3 — Phase 3: TYPE 2 launchd label rename (M, SAFE template/script edits + GATED unload/load dance)

- **SAFE:** edit the 3 `.template` files + the `reload-all.sh`/`stage-repointed-plists.sh` globs +
  `daemon-health.sh`/`.test.sh` to the new `com.<ns>.*` namespace. Rename the one evium-in-filename
  template (`launchd-template/com.evium.heartbeat.plist.template` → `com.<ns>.heartbeat.plist.template`).
- **GATED — the unload-old / load-new dance (USER runs launchctl):** for each of the 6 live labels:
  ```
  # 1. render the new-labelled plist (reuse stage-repointed-plists.sh, now emitting com.<ns>.*)
  # 2. USER: launchctl bootout gui/$(id -u)/com.evium.daemonN        # stop old
  #          rm ~/Library/LaunchAgents/com.evium.daemonN.plist        # remove old file
  #          cp <staged>/com.<ns>.daemonN.plist ~/Library/LaunchAgents/
  #          launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.<ns>.daemonN.plist
  # 3. verify: launchctl list | grep com.<ns>   (6 labels, status 0)
  ```
  `reload-all.sh` (re-globbed to `com.<ns>.daemon*`) automates steps 2's enable/bootstrap once the
  files are in place; the user just runs it (launchctl stays user-gated).
- **Order safety:** do this AFTER Phase 2's shim is live but it's independent of env-vars — a daemon's
  label and its kill-switch are orthogonal. No daemon downtime beyond the per-label bootout→bootstrap
  (seconds); `daemon-health.test.sh` is the green-light.
- **Effort:** M. **Risk:** MED — a typo in a label leaves a daemon dark; mitigated by the
  before/after `launchctl list` snapshot (Phase 0.3) + `daemon-health.test.sh`.

### 4.4 — Phase 4: TYPE 4 strays (S, SAFE)

`launch.json` `"name"`, residual comments in `reload-all.sh`/`stage-repointed-plists.sh`,
`project-config.json` `_doc` prose, any stray machinery comment. Mechanical. **Effort:** S. **Risk:** none.

### 4.5 — Phase 5: docs follow the rename (S–M, SAFE, batchable, lowest priority)

~170 doc files mention a machinery token. A scripted `grep -rl … | xargs sed` over
`context/markdowns/` + `.claude/skills/` updates them in one batch. **Caveat:** do NOT hand-edit
`.claude/memory-mirror/` topic files — edit the canonical auto-memory copies and let `sync-memory.sh`
propagate (feedback_memory-mirror-sync). **Effort:** S–M. **Risk:** none (prose).

### 4.6 — Anti-recurrence (S, SAFE — make it un-re-bakeable)

After de-shim, add a test mirroring the path-audit's `test-no-hardcoded-paths.sh`:
`test-no-client-name-in-machinery.sh` — FAIL if `grep -rE 'EVIUM_|com\.evium|EviumOverhaul'` hits any
`scripts/`/`tests/`/`.claude/settings*` file (excluding the `project_evium-overhaul` client tracker +
client-site docs). Register in the static test tier. If Option B wired `ENV_PREFIX`, the env-var class
literally cannot recur (names are built from the config value). **Effort:** S. **Risk:** none.

---

## 5. Effort / risk summary per phase

| Phase | TYPE | Effort | Risk | Gated? |
|---|---|:--:|:--:|:--:|
| 0 — Decision + freeze | — | S | none | no |
| 1 — Worktree prefix (3a+3b) | 3 | S–M | LOW-MED (coupled pair) | 1 settings edit |
| 2 — Env-var rename + shim | 1 | **L** | LOW *with shim* / MED without | settings flip last |
| 3 — launchd labels | 2 | M | MED (dark-daemon typo) | 6-plist unload/load |
| 4 — Strays | 4 | S | none | no |
| 5 — Docs follow | docs | S–M | none | no (mirror caveat) |
| 6 — Anti-recurrence test | — | S | none | no |

**Total gated touch-points across the whole migration: exactly 2 kinds** — `settings.json` (+ its
proposed-mirror), edited in Phases 1 & 2; and the 6 live plists, re-loaded in Phase 3. Everything else
is mechanically safe and reuses machinery that already exists (`reload-all.sh`,
`stage-repointed-plists.sh`, `settings-proposed-sync.sh`, the 13 kill-switch tests, `daemon-health.test.sh`,
`migration-drift-sentinel.sh`).

---

## 6. Follow-ups / open questions for the user

1. **Pick a prefix row (§3).** Everything keys off this.
2. **Shim mechanism (§4.2): Option A (per-switch OR-guard) or Option B (central `ENV_PREFIX`-wired
   shim)?** B is more work up-front but kills the root defect (the unread `ENV_PREFIX`) so it can never
   recur. Recommend B if appetite allows.
3. **Bonus simplification:** derive the worktree prefix from the repo folder name (`basename` of git
   toplevel) instead of any literal — makes future renames free. In-scope?
4. **This is the held part of manifesto §4** ("EVIUM ↔ SYSTEM DECOUPLING", STATUS: PENDING). Landing it
   would let §4's research-DONE flip to execution-DONE. Worth a goal-billboard proposal? (Per
   feedback_goal-billboard, I do not add goals unilaterally — flagging for your call.)

---

BG-COMPLETE-SENTINEL: (refs_by_category_with_counts={EVIUM_env_vars: {distinct_names: ~221, kill_switches_OFF: 79, machinery_files: 120, settings.json: 52, settings.proposed: 52, scripts_sh_py: 481, tests: 231, docs: 357}; com.evium_labels: {live_loaded_plists: 6, machinery_files: 22, templates: 3, scripts_refs: 46, docs: 207}; EviumOverhaul_naming: {machinery_files: 27, settings.json_patterns: 8, PROJECT_NAME_readers: ~5, scripts_sh_py: 56, tests: 9, docs: 193}; type4_strays: {launch.json_name: 1, evium_in_filename_templates: 1, project-config_keys: 2}; client_website_src: 0_machinery_hits_CLEAN; distinct_doc_files_with_machinery_token: 170}, gated_count=8 surfaces [settings.json + settings.proposed.json + 6 live com.evium plists], plan_phases=7 [P0 decide/freeze, P1 worktree-prefix, P2 env-var+compat-shim, P3 launchd-labels-unload/load, P4 strays, P5 docs-follow, P6 anti-recurrence-test], status=AUDIT+PLAN COMPLETE — read-only, nothing renamed; ENV_PREFIX recorded-but-unread so rename is literal find/replace not config-flip; path-portability already DONE (this is the deferred §7 naming/branding dimension = held manifesto §4); rename is non-breaking via per-var atomic flip + compat-shim honoring old+new; awaiting user prefix-choice + shim-mechanism decision)
