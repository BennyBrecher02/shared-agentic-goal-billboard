# `~/.cursor/` full audit — did the OS's Jun 3–5 dump "mess us over"?

- **When:** 2026-06-08T11:22Z
- **Scope:** READ-ONLY audit of every entry under `~/.cursor/`. Nothing was modified, disposed, or committed.
- **Trigger:** user worry that a "lazy/blind code dump" into Cursor's files on Jun 3–5 was handled wrong.
- **Bottom line (the headline):** **No mess. The system is clean.** Every OS-deployed artifact is CURRENT
  and byte-identical to repo canon; the user's real Cursor work (`plans/`, `projects/`, `ai-tracking/`) is
  **untouched** (zero OS fingerprint inside it); vendor skills are **not shadowed**; **no secrets, no real
  hardcoded user paths, no dead-repo-slug pollution**. The *one* place the OS touched Cursor's OWN config —
  a 4-line `attribution` block added to `cli-config.json` — was **deliberate, correct, backup-protected, and
  trivially reversible**: it enforces the user's own no-AI-authorship rule. Reported in full below for
  transparency; **no action required**.

---

## Per-entry table

Classification key: **MINE** = OS transported/edited · **CURSOR** = vendor/user state (never touch) ·
**OS-NEITHER** = macOS/filesystem artifact.

| Entry | Mine / Cursor's | Current / Stale / Risk | Verdict |
|---|---|---|---|
| `cli-config.json` | **CURSOR (OS-edited 1 block)** | OS added native `attribution{attributeCommitsToAgent:false, attributePRsToAgent:false}` on Jun-3 (see RISK-a) | **KEEP — load-bearing & correct.** Enforces no-AI-authorship in Cursor. Backup taken. Revert = trivial. |
| `cli-config.json.bak-20260603T183906Z` | **MINE** (OS backup-before-mutate) | Current — the pre-edit snapshot | KEEP as-is; it's the revert source. Optional later cleanup only. |
| `argv.json` | CURSOR | Untouched (mtime 2025-09-17) | Clean. VS Code/Cursor crash-reporter config. Not touched. |
| `mcp.json` | CURSOR | Untouched (mtime 2026-02-18) | Clean. User's Figma-Desktop MCP. OS did **not** touch it. |
| `ide_state.json` | CURSOR | Untouched by OS (mtime Jun-4, written by Cursor) | Clean. Cursor's recently-viewed-files (a *different* project: `backlawrenceshul.org`). See RISK-c note. |
| `.gitignore` | CURSOR (managed block) | Untouched (mtime 2026-04-23) | Clean. Cursor-managed allowlist block. OS did not edit. |
| `.DS_Store` | OS-NEITHER | macOS Finder metadata (mtime Jan-16) | Ignore. Filesystem artifact, predates everything. |
| `skills/` (13 dirs) | **MINE** | **CURRENT** — content byte-identical to repo `.claude/skills/`; single clean deploy (`kit_commit bdd04dc`, `deployed_at 2026-06-08T02:07:44Z`) | **CLEAN.** No nesting, no dups, no Jun-3 leftovers. Only delta vs source = `.deploy-receipt.json` marker (expected). |
| `skills/*/.deploy-receipt.json` (12) | **MINE** | Current — all same kit_commit/deploy time | CLEAN. Confined to `skills/` only; nowhere else in tree. |
| `universal-memory/` (38 files) | **MINE** | **CURRENT** — byte-identical to repo canonical `portable-kit/universal-memory/` (diff exit 0) | **CLEAN.** Curated portable subset; "missing" topics vs memory-mirror are *by-design* (MIXED/project-specific excluded), not staleness. |
| `rules/` (4 `.mdc`) | **MINE** | **CURRENT** — all 4 MATCH repo `.cursor/rules/` source | **CLEAN.** Well-scoped, harness-agnostic. `house-conventions.mdc` (Jun-3 mtime) is content-identical to source → benign, not a leftover. |
| `hooks/` (3 `.sh`) | **MINE** | **CURRENT** — all 3 MATCH repo `scripts/hooks/` source | CLEAN. `artifact-landed-check`, `no-hardcoded-paths-guard`, `screenshot-janitor`. |
| `skills-cursor/` (15 vendor skills) | CURSOR | Untouched by OS; `canvas/sdk/*.d.ts` Jun-8 mtime = Cursor's OWN managed-skill re-sync (canvas ∈ `managedSkillIds`) | **CLEAN — not polluted.** Namespace disjoint from MINE (RISK-b). `.sync-manifest.json` is Cursor-managed. |
| `plans/` (160 files) | CURSOR (user's work) | Untouched by OS — zero OS fingerprint | **CLEAN — confirmed user's real Cursor plan-mode artifacts** (DALL-E, finance-API, OCR, Olumie). OS did not write here. |
| `projects/` (99 dirs) | CURSOR (user's work) | Untouched by OS — zero OS fingerprint | **CLEAN — confirmed Cursor per-workspace state** across many real projects. The `agentic-organic-os*` dirs hold only Cursor's native `canvases/mcps/terminals/` subdirs. |
| `ai-tracking/ai-code-tracking.db` | CURSOR | Cursor's own SQLite telemetry DB (written Jun-8 07:12 by Cursor) | CLEAN. Vendor AI-code attribution tracking. Not OS-written. |
| `extensions/` (15, 358M) | CURSOR | Vendor VS Code/Cursor extensions (anysphere.*, ms-python.*, redhat.java, github.*) | CLEAN. Cursor's install footprint. Large but not OS-transported. |
| `plugins/` (`plugins/local`, empty) | CURSOR | Empty vendor placeholder | Ignore. |

---

## RISK CHECK — what could actually "mess them over"

### (a) Did the OS edit Cursor's OWN config? — YES, exactly one block, and it's CORRECT
**Exact diff (`.bak` → live `cli-config.json`):** the OS APPENDED only this block; nothing else changed,
nothing was removed or overwritten:
```json
+  "attribution": {
+    "attributeCommitsToAgent": false,
+    "attributePRsToAgent": false
+  }
```
**Why this is correct, not a "mess":**
- `attribution.attributeCommitsToAgent` / `attributePRsToAgent` are **native Cursor CLI settings** (documented
  in Cursor's own vendor skill `skills-cursor/update-cli-config/SKILL.md` and Cursor's config schema). The OS
  did **not** invent a key or corrupt the schema.
- Cursor's CLI **defaults these to `true`** → it would stamp an AI attribution trailer on commits/PRs, a
  **direct violation** of the user's hard standing rule `feedback_commit-authorship.md` (commits carry **no**
  `Co-Authored-By`; the user owns history). Setting them `false` **enforces the user's own convention** in the
  Cursor harness.
- The OS followed **backup-then-mutate** discipline: it wrote `cli-config.json.bak-20260603T183906Z` first
  (same Jun-3 timestamp). **Revert is one `cp`.**
- The OS even documented this as load-bearing in `portable-kit/templates/cursor-permissions.proposed.json` and
  `CURSOR-APPLY.md` ("don't clobber your existing attribution=false") — so it's an intentional, recorded
  decision, not a blind dump.
- **`argv.json`, `mcp.json`, `ide_state.json` were NOT touched by the OS** (mtimes 2025-09, 2026-02, and a
  Cursor-written Jun-4 respectively; none carry OS edits). Confirmed.

**Verdict (a): defensible + backup-protected + reversible + aligns with the user's explicit rule. No action
needed; flagged purely for transparency since it is the lone OS write into Cursor's own config.**

### (b) Skill/rule shadowing of vendor skills — NONE
The MINE skill namespace (`agentic-*` + `reactionary`, 13) and the Cursor vendor namespace
(`automate babysit canvas create-* loop migrate-to-skills sdk shell split-to-prs statusline update-cli-config
update-cursor-settings`, 15) are **fully disjoint** — `comm -12` intersection is empty. No skill shadows or
overrides a vendor skill; the 4 rules are additive convention rules, not redefinitions of Cursor behavior.

### (c) Security — secrets / absolute user paths / other-project data — CLEAN
- **Secrets:** zero. A regex sweep (api-key/secret/password/token/bearer/AKIA/sk-/ghp_/xox*/PEM headers) across
  all MINE files returned only the English word "token" in prose (token-cost, design token). No credentials.
- **Hardcoded user paths in MINE bodies:** none real. The only `/Users/` literals are (1) deliberate generic
  teaching placeholders (`/Users/you/Code/your-project`) in `agentic-script-design` reference docs — portable,
  fine; and (2) inside `no-hardcoded-paths-guard.sh`, where the real identifiers appear **on purpose** as the
  `HAZARD_RE` pattern the guard scans FOR (a path-guard must name what it blocks).
- **Other-project data transported into a shared place:** none from the OS. (Note: `ide_state.json` references
  another of the user's projects — `Desktop/backlawrenceshul.org`, including `.env` *paths* — but that file is
  **Cursor's own** recently-viewed-files state, **not** OS-written, and contains only file paths, not secret
  values. Out of OS scope; flagged only for completeness.)

### (d) Dead/old repo-slug pollution — NONE (only intentional guard references)
The dead slug `Claude/Design/EviumOverhaul` appears **only** inside `no-hardcoded-paths-guard.sh` — correctly,
as the blocklist pattern that catches any reintroduction. The CURRENT slug
(`-Users-bennybrecher-Claude-Code-agentic-organic-os`) appears in `feedback_memory-mirror-sync.md` as a
documented drift-detection path. Both are intentional; neither is stale pollution. (`reference_repo-geography.md`
— which tracks the dead `~/agentic-organic-os` path — is *not* in the portable bundle, so it didn't leak here.)

### (e) Bloat — NONE attributable to the OS
`~/.cursor/` is 394M total, but **358M is `extensions/` (vendor)** and **22M is `ai-tracking/` (vendor DB)** —
neither is OS-transported. The OS footprint is tiny and proportionate: `skills/` 2.0M, `universal-memory/`
204K, `rules/` 16K, `hooks/` 16K. No duplicate/nested skill trees, no orphaned Jun-3 copies.

---

## Pollution proof (the user's specific worry)
- **Zero OS-deploy fingerprint** inside `plans/` or `projects/`: grep for `deploy-receipt`,
  `repo-canon .claude/skills`, `GENERATED by extract-universal-memory`, `portable-kit`, `memory-mirror` → no hits.
- **All 12 `.deploy-receipt.json` files are confined to `~/.cursor/skills/`** — nowhere else in the tree.
- The `agentic-organic-os` / `-src` workspace dirs under `projects/` contain only Cursor's native
  `canvases/`, `mcps/`, `terminals/` subdirs (created when the user opened the repo in Cursor) — no OS content.

**Conclusion: the OS deploy stayed inside its lane (`skills/`, `universal-memory/`, `rules/`, `hooks/`, + the
one `cli-config.json` attribution block). It did not pollute the user's real Cursor work.**

---

## RANKED remediation list

**Nothing is required. The system is clean.** The items below are optional hygiene only, ranked by (negligible)
value. Per house rules, deletions go through `scripts/dispose.sh "<reason>" <path>` (soft-delete to
disposal-bin), **never `rm`** — and **all of these touch Cursor's vendor dir or are gated, so leave them to
the user.**

1. **(Optional, P3) Prune the stale CLI-config backup once trust is established.** `cli-config.json.bak-20260603T183906Z`
   has served its purpose (the attribution edit is verified good). It's 223 bytes — harmless. If the user wants
   a tidy `~/.cursor`: `scripts/dispose.sh "cli-config backup no longer needed; attribution edit verified" ~/.cursor/cli-config.json.bak-20260603T183906Z`. **User's call — it lives in Cursor's dir.** No urgency.
2. **(Optional, P3) Re-confirm the attribution edit is desired.** It is correct per `feedback_commit-authorship.md`,
   but since it's the lone OS write into Cursor's own config, the user may want to eyeball it once. **To revert
   (not recommended):** `cp ~/.cursor/cli-config.json.bak-20260603T183906Z ~/.cursor/cli-config.json`.
3. **(No-op) `house-conventions.mdc` Jun-3 mtime.** Content matches live source exactly → no refresh needed.
   Listed only to preempt a "why is this older?" question.

**No refresh needed** (all MINE artifacts already byte-identical to canon). **No revert needed** (the one
config edit is correct). **No dispose needed** (no pollution, no dead copies).

---

## Clean-bill summary (what the user can stop worrying about)
- ✅ **Skills (`skills/`):** 13 dirs, CURRENT, byte-identical to repo canon, single clean deploy, no
  nesting/dups/leftovers.
- ✅ **Portable memory (`universal-memory/`):** byte-identical to canonical `portable-kit/universal-memory/`;
  curated-subset omissions are by-design.
- ✅ **Rules (`rules/`) + hooks (`hooks/`):** all match repo source; scoped, path-clean, convention-aligned.
- ✅ **Cursor's own config (`argv.json`, `mcp.json`, `ide_state.json`, `.gitignore`):** NOT touched by the OS.
- ✅ **`cli-config.json`:** the one OS edit is a correct, backup-protected, reversible enforcement of the
  user's no-AI-authorship rule using a native Cursor setting.
- ✅ **User's real Cursor work (`plans/` 160, `projects/` 99, `ai-tracking/`):** UNTOUCHED — zero OS fingerprint.
- ✅ **Vendor skills (`skills-cursor/`):** not shadowed (disjoint namespace), not polluted.
- ✅ **Security:** no secrets, no real hardcoded user paths, no other-project data transported by the OS, no
  dead-repo-slug pollution (only intentional guard references).
- ✅ **Bloat:** none attributable to the OS (the 358M is vendor extensions).

The Jun 3–5 dump worry is unfounded: it was a disciplined, in-lane, backup-protected deploy. **The system is
in good shape.**
