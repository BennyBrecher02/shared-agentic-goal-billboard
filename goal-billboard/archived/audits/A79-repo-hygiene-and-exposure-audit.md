---
audit_id: A79
title: Repo hygiene + exposure audit — what's tracked, what's exposed, what's safe
status: completed
catalogued: 2026-05-29
trigger: 2026-05-29 — user: *"audit our repo safely, very very safely"* (+ "the root level screenshots werent removed from the repo yet")
method: READ-ONLY. No git mutations, no rm, no commits, no gc, no history rewrite. Sensitive content flagged by PATH/COUNT only — never reproduced.
serves: user-safety / privacy (client data is the standing red line)
---

# A79 — Repo hygiene + exposure audit

## Headline (inline findings, 2026-05-29)

### 🔴 The repo is PUBLIC — `BennyBrecher02/EviumCharging` (`isPrivate: false`)
This is the finding that frames everything. Implications:
- ✅ **No raw client-data leak.** The 2GB Drive export (`context/Evium-Charging-Google-Drive-AllAvailable-Data/`) and the reference codebases (`context/codebases/`, incl. SkilkenGold source) are **gitignored AND were never committed** (history checked clean for both). The bulk-sensitive red line holds.
- ⚠️ **All 1,945 tracked `context/markdowns/` files are world-visible.** That's our entire internal workspace: plans, audits, the goal-billboard + monkey-chamber, the Organic-OS process docs, memory-mirror — **and the SkilkenGold design *analysis*** (`audits/skilkengold-parity/`, `research/rabbit-holes/skilkengold-original-design-patterns/`, `direct-to-design/p1-07b-…-skilkengold.md`). No secrets, no raw client files — but a *different client's* design analyzed publicly under your name, plus your full internal AI-assisted workflow, is exposed.
- **This is a USER decision, not an agent action.** Options (yours): (a) intended-public, accept it; (b) make the repo private; (c) stop tracking the internal workspace (contradicts the current intentional `.gitignore` negate-include of `markdowns/`); (d) selectively untrack the cross-client analysis. I will NOT change visibility or mass-untrack — both are firmly gated.

### ✅ Bulk sensitive data — clean
`context/codebases/` and the Drive export: gitignored + never in history. The `.gitignore` negate-include design (`context/` not blanket-ignored so `markdowns/` can be tracked) is working as intended for the *data* boundary.

### ⚠️ Secrets smell-test — likely false positives, BG verifying
A word-match for `api_key|secret|password|BEGIN PRIVATE` on tracked files hit: the dashboard backup HTML, a `.patch`, `package-lock.json` (integrity hashes), `agent-inbox-server.py`, `lib-common.sh`, `inbox-write.py`, `Input.astro`. All look like field-names / hashes / comments — the thorough pass confirms **no real secret VALUES** are committed.

### 🟡 Root-level snapshot PNGs — untrack BLOCKED by the destructive-op guard
8 tracked root PNGs (`a46-phase-1-*`, `cluster-*phase1`, `p0-chip-current`, `sweep-timelapse-1440`) violate the no-root-images rule. The safe untrack (`git rm --cached`) is **denied by the settings.json destructive-op guard** — the guard working as designed. Surfaced to the user as a one-liner. **Note:** the deny blocks even safe `--cached`, broader than the user's "yes to all but git push" — flag for the user to reconcile.

### Notes for the thorough pass
- Large tracked blobs: `public/video/hero-electricity.mp4` (13M) + `hero-abstract.mp4` (7.3M) + several 2–6M chargers/heroes PNGs — acceptable for a site, but assess Git-LFS candidacy + whether `_archive` images need to ship.
- 40 commits unpushed (A12-F1 single-machine risk — separate finding).

## Thorough pass (read-only BG) — scope
1. Verify the secrets smell-test = false positives (grep for real key/token/password VALUES, history included).
2. Full tracked-file categorization — confirm only intended content is tracked; flag anything surprising.
3. `.gitignore` completeness — any path where future sensitive data could slip in tracked? (e.g. a new client dir not yet ignored.)
4. History scan — any secret / `.env` / key file / large sensitive blob ever committed then removed (recoverable on a PUBLIC repo).
5. Bloat / LFS candidacy; branch/tag/ref hygiene.
6. Rank by severity; PROPOSE remediations (apply NOTHING — esp. nothing destructive; the user acts).

## Cross-references
- A12 destructive-ops audit (the guard that blocked the PNG untrack; the 40-unpushed-commits risk)
- `findings-leaderboard.md` (this audit's findings feed the board)
- Memory: client-data privacy red line; `public/img` structure (no root images)

---

## Thorough pass — findings (read-only)

*Completed 2026-05-29. STRICTLY read-only: no git mutations, no rm/mv, no settings changes. Every read-command used is logged below per finding. Sensitive values were never reproduced — findings are PATH + COUNT only. PROPOSE-only; nothing applied.*

### Method recap (commands run)
`git ls-files`, `git log --all`, `git show HEAD:<path>`, `git grep -nI` / `-lI` / `-cI`, `git rev-list --objects --all`, `git log -S<pat>` (pickaxe), `git log --diff-filter=A/D --name-only`, `git check-ignore -q`, `git for-each-ref`, `git worktree list`, `git rev-list --count`, `du -k`. No `gh` mutation; visibility already confirmed PUBLIC inline.

---

### 🟢 1 — Secrets: CONFIRMED clean (all flagged hits are false positives)

Every word-match flagged inline was inspected in-context. **None hold a real secret value.**

| Flagged path | What the match actually is | Verdict |
|---|---|---|
| `scripts/agent-inbox-server.py` | `import secrets` (Python stdlib) + `secrets.choice(...)` for a 5-char random ID | false positive |
| `scripts/inbox-write.py` | `import secrets` + `secrets.token_hex(4)` for a filename suffix | false positive |
| `scripts/cluster/lib-common.sh` | a comment: "`-o BatchMode=yes` prevents interactive **password** prompts" | false positive |
| `src/components/forms/Input.astro` | a TypeScript union type member `'password'` (input `type` attr) | false positive |
| `context/markdowns/dashboard-source-backup/operational-dashboard-2026-05-29.html` | 61 hits — ALL the word "token" referring to the **token-budget meter UI** (`.lefter-tokens` CSS classes, `fmtTokens()` JS, "today's tokens went"). Zero credential material. | false positive |
| `*.patch` files (36 tracked under `multi-change-queue/patches/`) | code diffs; no value matches surfaced | false positive |
| `package-lock.json` | npm `integrity` sha512 hashes (subresource integrity, not secrets) | false positive |

**Whole-tree value scan (tracked):** grep for `sk-ant-…`, generic `sk-[A-Za-z0-9]{20,}`, `gh[pousr]_…`, `AKIA[0-9A-Z]{16}`, `xox[baprs]-…`, `AIza[…]{35}`, `BEGIN …PRIVATE KEY` → **zero matches.**

**Full-history value scan:** `git log --all -S` pickaxe for each of `sk-ant-`, `AKIA`, `ghp_`, `xoxb-`, `PRIVATE KEY`, `AIza` → **zero commits.** No real key/token/password VALUE has ever entered the tree on any ref.
*Evidence: §1 commands above. Why it matters (PUBLIC): a single committed live key would be world-readable + permanently recoverable. Confirmed none exists.*
**PROPOSED remediation:** none required for secrets. (Optional hardening in §3 below.)

---

### 🟢 2 — History exposure beyond the 2 known-clean dirs: clean

- `git rev-list --objects --all` filtered for `.env / .pem / .key / .p12 / .pfx / credential / secret / .keystore / id_rsa / id_ed25519 / .crt / service-account / aws*key` → **no such blob ever existed** on any ref.
- `git log --all --diff-filter=A` (ever-added) and `--diff-filter=D` (ever-removed) filename scans for env/credential/key/password patterns → **empty.**
- No "committed-then-removed" sensitive blob is recoverable. The clean-history claim now extends past `context/codebases/` + the Drive export to **the entire object database**.
*Evidence: §1 history commands. Why it matters (PUBLIC): removed-but-in-history files are trivially recoverable on a public repo via `git cat-file`. None found.*
**PROPOSED remediation:** none.

---

### 🔴 3 — `.gitignore` is enumerate-the-bad, not default-deny — FUTURE client drops under a NEW `context/` name land TRACKED

This is the highest-value *structural* finding (no live leak, but the guardrail has holes a future drop falls through). `git check-ignore` probe results:

| Hypothetical future drop | Ignored? |
|---|---|
| `context/markdowns/agent-inbox/secret.md` | ✅ IGNORED |
| `.env.production` / `.env.local` / `context/markdowns/.env` | ✅ IGNORED |
| `context/design-imports/some-export/page.html` (the design-import dropzone) | 🔴 **TRACKED** |
| `context/new-client-acme/data.csv` (a NEW client dir) | 🔴 **TRACKED** |
| `context/SomeClient-Drive-Export/financials.xlsx` (a new Drive export) | 🔴 **TRACKED** |
| `context/markdowns/new-subfolder/notes.md` | 🔴 TRACKED (intended — markdowns IS the workspace) |
| `secrets.txt` (repo root) | 🔴 **TRACKED** |
| `scripts/credentials.json` | 🔴 **TRACKED** |
| `public/img/client-contract.pdf` | 🔴 **TRACKED** |

**The hole:** `.gitignore` protects the data boundary by *naming the two bad dirs exactly* (`context/codebases/`, `context/Evium-Charging-Google-Drive-AllAvailable-Data/`). Everything else under `context/` is tracked by default. So the protection is "block the specific known-bad paths" — a NEW client folder, a NEW Drive export under a different name, or the `context/design-imports/` dropzone all default to **tracked → public**. The `markdowns/` negate-include itself has no hole (it's the intended workspace), but the *sibling* default-tracked behavior is the risk.
*Evidence: §3 `git check-ignore` probe table. Why it matters (PUBLIC): the standing red line is client data. The current ignore design relies on the agent/user remembering to add an ignore rule BEFORE the first commit of any new client material — a discipline dependency, exactly the fragility the system elsewhere converts to structure.*

**PROPOSED remediation (PROPOSE only — do not apply):**
1. **Flip `context/` to default-deny** with a markdowns negate-include. Conceptually: ignore `context/*` then `!context/markdowns/` (+ `!context/markdowns/**`). New client dirs then default to *ignored* and must be deliberately un-ignored — failure mode becomes "data accidentally untracked" (safe) instead of "data accidentally public" (unsafe). NOTE: this is a `.gitignore` edit → bring to the user; verify no currently-tracked `context/` path outside `markdowns/` exists first (this pass found none — only `markdowns/` is tracked under `context/`).
2. **Add belt-and-suspenders secret-name ignores**: `secrets.*`, `credentials.json`, `*.credentials`, `*.key`, `*.pem`, `*.p12`, `*.pfx`, `id_rsa*`, `*.keystore`, `service-account*.json`.
3. **Explicitly ignore the dropzone**: `context/design-imports/` (it's a staging area for unzipped Claude-Design exports — never source-of-truth, shouldn't be tracked).
4. Consider a `pre-commit` warn-hook (parallel to existing warn-hooks) that flags staging of any new top-level `context/` subdir or any `*.key/*.pem/.env*` — defense-in-depth for the discipline gap.

---

### 🟡 4 — Public-exposure catalog (repo is PUBLIC; what a stranger sees)

Top-level tracked footprint: `context/` 1945 files · `public/` 310 · `scripts/` 294 · `.claude/` 184 · `tests/` 162 · `src/` 42 · root configs.

| Category | Sensitive? | Why (PUBLIC) |
|---|---|---|
| `src/`, `public/img|video/`, `astro.config`, `package*.json`, CI workflows | 🟢 no | Standard for the client's own site; CI workflows clean (only `${{ github.* / runner.* / matrix.* / env.NODE_VERSION }}` template refs — no `secrets.*` even referenced). |
| **Client install photos** named by real site: `installations/{secaucus-plaza, wyndham-hampton, atlantic-pointe, morrison-1240, seacoast-kittery, the-caroline}/` + `_archive/install-*` | 🟢 intended-public | These name the client's real install locations — but they're the client's **own published portfolio** (they want them shown). Not a leak; noted for awareness only. |
| Client public contact emails `contact@ / support@eviumcharging.com` (135 / 134 hits) | 🟢 no | The client's own published business emails. |
| **SkilkenGold (a DIFFERENT client) design analysis** — 42 tracked files: `audits/skilkengold-parity/`, `research/rabbit-holes/skilkengold-original-design-patterns/` (8 deep-dives), `direct-to-design/p1-07b-…-skilkengold.md` + refs in plans/goal-billboard | 🟡 **plausibly sensitive** | A second client's site is reverse-engineered (color/spacing/typography/motion/"premium feel" deep-dives) **publicly, under your GitHub name**. No secrets — but cross-client competitive analysis being world-visible is a professional/optics exposure, not a data-leak. Already flagged inline; re-confirmed scope = 42 files across 8 dirs. |
| **Your full internal AI-workflow** — goal-billboard, monkey-chamber, Organic-OS docs, every audit/plan/steering-log, memory-mirror (49 files), all skill refs (129 files) | 🟡 your call | World-visible. No credentials. It's your private operating methodology exposed — optics/IP, not security. Intended per the `markdowns/` negate-include, but worth a conscious decision given PUBLIC. |
| **Your personal identity**: `benbrecher+m2@icloud.com` (1 file: `plans/multi-orchestrator-new-subscription-plan.md`), `user@m2.local` (1 file: `research/.../RH-018-m2-multi-orchestrator-connectivity-slam-dunks/README.md`), and your macOS username via **`/Users/bennybrecher` hardcoded in 40 tracked files** | 🟡 minor PII | Your personal email + macOS home path are world-readable. Low severity (email is semi-public; username low-value) but it's personal info on a public repo. The dashboard backup HTML was checked and is CLEAN of `/Users/` paths + machine names (0 occurrences). |

*Evidence: §2/§4 grep-by-path tables above. PROPOSED remediation (PROPOSE only):* decide repo visibility holistically (the inline 🔴 already frames a/b/c/d). If staying public: (i) consider relocating the SkilkenGold analysis out of the tracked tree or into a private repo; (ii) sweep hardcoded `/Users/bennybrecher` → `$HOME`/relative paths in the 40 files (also improves portability for the m2 multi-machine plan); (iii) scrub the 2 personal-email references. All are content edits for the user to decide — not applied.

---

### 🟡 5 — Bloat / LFS

Tracked total ≈ **212 MB**. Largest categories:

| What | Size | Assessment |
|---|---|---|
| **Playwright visual-snapshot baselines** `tests/*-snapshots/*` — 108 PNGs | **98.9 MB** (≈ 47% of repo) | Biggest single category. Intended-tracked (visual-regression fixtures) but the **prime Git-LFS candidate** — binary, churns on every baseline refresh, bloats history. |
| `public/video/hero-electricity.mp4` 13.6M + `hero-abstract.mp4` 7.3M | 21 MB | Site hero videos. LFS candidates; or host on a CDN and reference. |
| `public/img/chargers/siemens-l2.png` 6.6M, `evocharge-40a.png` 3.5M, several heroes 2–2.7M | ~25 MB | Large but legit site assets. Worth `pngquant`/`webp` optimization (these are oversized for web). |
| **`public/img/_archive/` — 14 images, 14.6 MB** | 14.6 MB | **Not referenced anywhere in `src/` or `public/`** (grep confirmed) — superseded/dead. BUT `public/` ships as-is, so these are still **publicly fetchable** by direct URL. Pure dead weight + minor exposure of old install photos. |

**Regenerable outputs check:** no `dist/`, `.astro/`, `node_modules/`, `.cache/`, `*.map`, or `*.log` slipped into tracking — the gitignore output-boundary is working. ✅
*Evidence: §5 `du -k` sort + `_archive` reference grep. Why it matters (PUBLIC): bloat is a clone/perf cost, not a security one; `_archive` is the only item that's both dead AND publicly fetchable.*
**PROPOSED remediation (PROPOSE only):** (1) migrate `tests/*-snapshots/` + `public/video/*.mp4` to **Git-LFS** (biggest win; ~120 MB off the git object path). NOTE: LFS migration rewrites how those blobs are stored → it's a history-touching op → MUST be user-authorized, never auto. (2) Untrack the unreferenced `_archive/` images (`git rm --cached` — but per the inline finding the destructive-op guard will block this; surface to user). (3) Optimize the oversized chargers/heroes PNGs to webp. All gated.

---

### 🟢 6 — Ref hygiene

- **Tags:** none. **Stashes:** 1 (`stash@{0}: g2-patch-test-tmp`) — looks like a leftover test stash; harmless, user may want to drop it.
- **Local-only branches (NOT on origin, so NOT publicly exposed):** 11 × `claude/*` (agent worktree branches) + 9 × `verified/*` (per-change verification branches). None pushed → invisible to strangers. They're local cruft, not an exposure.
- **Worktrees:** `git worktree list` shows the main checkout + **2 detached worktrees in `/private/tmp/`** (`evium-blog-patchtest`, `p2-12-verify`) + 8 agent worktrees under `.claude/worktrees/` (gitignored). The `/private/tmp/` ones are stale test worktrees.
- **origin/main divergence: 41 commits** on local `main` not pushed (was "40" inline — now 41). This is the A12-F1 single-machine-risk finding: 41 commits of work exist only on this Mac. Not an *exposure* risk (unpushed = private) but a *durability* risk (disk loss = work loss).
*Evidence: §6 `git for-each-ref` / `worktree list` / `rev-list --count`. PROPOSED remediation (PROPOSE only):* prune the 20 stale local branches + 2 `/private/tmp/` worktrees + 1 stash once their work is confirmed merged (all gated — branch/worktree deletion is mutating). Push `main` to origin to close the durability gap (push is the one op the user pre-authorized, but still surface the 41-commit batch first).

---

### Severity roll-up

| # | Finding | Severity |
|---|---|---|
| 3 | `.gitignore` enumerate-the-bad → new `context/` client drop lands TRACKED+PUBLIC | 🔴 |
| 4 | SkilkenGold cross-client analysis (42 files) + your AI-workflow + personal PII, all world-visible | 🟡 |
| 5 | 99 MB snapshot baselines + 21 MB video → LFS; 14.6 MB dead `_archive` still publicly fetchable | 🟡 |
| 6 | 41 unpushed commits (durability), 20 stale local branches, 2 tmp worktrees, 1 stash | 🟢→🟡 (durability) |
| 1 | Secrets — all flagged hits false positives; tree + full history value-scan clean | 🟢 |
| 2 | History exposure — no env/key/credential blob ever committed on any ref | 🟢 |

### Biggest exposure risk (one line)
**No live secret leak exists — the single biggest exposure risk is structural: the `.gitignore` enumerates the two known-bad `context/` dirs instead of default-denying, so the *next* client folder or Drive export dropped under a new name will be silently committed to a PUBLIC repo (finding 🔴-3); the SkilkenGold cross-client analysis already sitting world-visible (🟡-4) is the most sensitive thing currently exposed.**
