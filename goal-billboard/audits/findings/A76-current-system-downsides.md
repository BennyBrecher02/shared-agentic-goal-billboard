# A76 BG-B — Semantic-index MCP server: CURRENT-system downside hunt

**Audit:** A76 · **BG:** B (current-system conflicts) · **Written:** 2026-05-28 (UTC clock-sourced)
**Mandate:** adversarial. The user's nod is gated on *no downsides*. Try to break it.
**Scope:** how a local-only semantic-index MCP server (e.g. `claude-context-local`) would be a problem for our *current* system — hooks, bottleneck chant, retrieval duplication, the gitignored sensitive `context/`, staleness inversion, design-import, MCP registration latency.

**One-line verdict up front:** **NOT clean to adopt as-pitched.** Two **cautionary** downsides need an explicit decision before install (privacy scope of `context/`; one of RH-020's headline benefits is *false*), and four smaller risks need mitigation. None are physically un-fixable, but the user said "adopt ONLY if no downsides" — so by that bar this returns **CONDITIONAL, not GO**. Every downside below has a mitigation; the decision is whether the scoped-down version still clears the user's bar.

---

## Downsides found — ranked by severity

### 🔴 D1 (CAUTIONARY — privacy/contract) — Indexing `context/` would embed the client's Drive export into a local vector store

**The finding.** `context/Evium-Charging-Google-Drive-AllAvailable-Data/` is **2.0 GB / 989 files** of the *client's* Google Drive export — branding, sales pitches, customer segments, preliminary proposals, legal, due-diligence, customer-reviews spreadsheet (`reference_context-folder` memory + verified on disk). It is `git check-ignore`-confirmed gitignored. RH-020-p3's growth thesis is *explicitly* "point it at `context/` … becomes the single best way to answer 'have we already decided this'" — i.e. the slam-dunk's *upside* is literally indexing this corpus. The whole 6.0 GB `context/` also contains **3.9 GB of reference codebases** (`the reference site` = a real third party's production site source, byte-identical to live; plus the user's prior attempts).

**Why it's a downside even local-only.** "Local-only, no API cost" addresses *network exfiltration*, not *data-at-rest scope*. Embedding client material creates a **derived copy of sensitive client data** in a new on-disk artifact (the vector DB + chunk cache) that:
- sits *outside* the consented/contemplated location (the Drive export was given as source material, not as "feedstock for a derived searchable store");
- is **not gitignore-protected by default** — a new `.index/` or `~/.claude/.../embeddings/` dir would need explicit gitignore rules or it risks being committed (we already had to hand-add `__pycache__/`, `.claude/cache/`, `public/cache` — a new index dir is one more drift surface, see D5);
- on the **planned multi-orchestrator future (M2 + Pi, G7/G17)** could be rsync'd/synced across nodes — putting client-derived embeddings on a second physical device. (That's BG-C's turf, but it *originates* from the current decision to index `context/`.)
- reconstruction risk: chunk caches store **verbatim source text** (that's how Cursor's "cache by chunk content" works — confirmed in RH-020-p3 §2), so this isn't lossy embeddings-only; it's effectively a second verbatim copy of client docs in a new format.

**Mitigation (makes it adoptable):** **scope the index to `src/` + our own first-party docs ONLY.** Hard-exclude:
- `context/Evium-Charging-Google-Drive-AllAvailable-Data/` (client data — non-negotiable),
- `context/codebases/` (third-party source incl. `the reference site`),
- and treat `context/markdowns/` (OUR work product — 119 skill refs / 100+ plans / 68 audits) as a **separate, opt-in, second index** decided on its own merits.

The catch: doing this **forfeits the headline growth-curve upside** RH-020 used to rank it #1 (★★★★☆→★★★★★ "once context/ is indexed"). Scoped to `src/` (42 files / 520 KB) the remaining value is — by RH-020's *own* admission — "moderate, not huge … grep is usually fine at 42 files." So the safe version is the *low-value* version. That's the core tension for the verdict.

**Residual even after scoping:** an indexer that walks the repo root needs a correctly-configured ignore list; a misconfig (or a future "just index everything" convenience flag) re-introduces D1. The exclusion must be **enforced**, not advisory — ideally the index config lives in-repo and is itself reviewed.

---

### 🟠 D2 (CAUTIONARY — false premise) — The headline "retires the baseline-manifest staleness problem" benefit is **WRONG**. RH-020 conflated two unrelated "baselines."

**The finding.** RH-020-p3 §2 ranks this #1 partly because it claims the auto-fresh index "structurally retires the `snapshot-baseline-staleness-check.sh` hook's reason to exist." **I read the hook. It does not do what RH-020 says.**

`scripts/hooks/snapshot-baseline-staleness-check.sh` is a `PreToolUse(Bash on npm run test:e2e)` hook whose *entire job* is: when the **Phase-5 visual-fix queue** has open items on routes with **Playwright screenshot baselines**, warn that the next matrix run's *screenshot* failures are EXPECTED (not regressions). It reads `consolidated-findings.md`, not the code map. It is about **visual snapshot drift during fix work** — pixels, not source structure.

The thing RH-020 *thinks* it retires — `reports/evium-baseline/baseline-manifest.json` being "hand-extracted and goes stale" — has **no dedicated staleness hook at all** (`grep -rln "baseline-manifest" scripts/` → empty; `grep "src_unchanged" scripts/` → empty). Manifest freshness is tracked by **hand-maintained YAML fields inside the JSON** (`src_unchanged_capture_to_head`, `head_git_sha_at_write`) and the design-import skill's read-through, not by the named hook.

**Why it's a downside.** One of the two concrete, named justifications for the #1 ranking is **based on a misread of our own substrate**. A semantic index over `src/` would NOT let us delete `snapshot-baseline-staleness-check.sh` (that hook would still be needed for visual-fix-queue false-alarm suppression — totally orthogonal). So the *adopt* decision should be re-scored with that benefit struck. What's left is "faster discovery on a 42-file repo + semantic matches grep misses" — real but, again, "moderate."

**Mitigation:** none needed for *safety* — but the **leverage estimate must be corrected downward**: strike "retires staleness hook" from the pro column. Could the index help keep `baseline-manifest.json` fresh? Only indirectly (you could regen the manifest from index queries), and that's a *new build*, not a free byproduct. Net: the benefit ledger is thinner than RH-020 advertised.

---

### 🟠 D3 (CAUTIONARY — the constitutional chant) — A persistent re-index-on-change daemon is a textbook bottleneck-chant risk

**The finding.** `feedback_bottleneck-restriction-chant` is the *constitutional law* of the Immune system — verbatim user line repeated **25×**: *"i dont ever want testing becoming a time/tokencost bottleneck,"* generalized in A58 to **"no system substrate may become THE bottleneck."** A self-refreshing index is, by design, a background substrate that consumes time + CPU + disk + RAM:
- RH-020-p3's *own* caveat cites monorepo-scale indexing hitting **100 GB RAM / 10-min builds** [augmentcode], and says "local model = no token cost, but disk + RAM + index-refresh CPU are real; measure before trusting."
- The embedding model itself (EmbeddingGemma per `claude-context-local`) is a **multi-hundred-MB-to-GB download + load**; our disk is at **82% used, 79 GB free** — fine for `src/` but a `context/`-scale index (6 GB of source → chunked + embedded + cached) plus the model eats meaningfully into that, and disk pressure is its own bottleneck class.
- A **re-index-on-every-file-change** trigger, if wired as a hook, would tax our **already-69-hook chain** (see D6) and our `PostToolUse(Write|Edit)` matcher already runs 6 hooks. Adding "kick the indexer" there risks per-edit latency — the exact micro-bottleneck the chant forbids.

**Where our system already says how to do this right:** the **autonomic 7-gate safety contract** (`autonomic-system-discipline.md`) is *mandatory* for any daemon: (1) `--dry-run` default 24h, (2) wrapper-only invocation, (3) **cost gate: skip when burn >80% or no idle slots**, (4) pivot-signal respect, (5) idempotent, (6) CoT entry per fire, (7) steering entry per finding. An index-refresh daemon is a **new daemon** and must clear all 7 — including the 24h dry-run (A47 Rule 5). The MCP server itself (query-time) is fine; the **refresh mechanism** is the regulated part.

**Mitigation:** (a) NEVER wire reindex into the `PostToolUse(Edit|Write)` hook chain; run refresh as a **debounced/throttled daemon** (merkle-style "only changed files," like Cursor's ~10-min cadence), gated by the 7-gate contract, dry-run 24h first; (b) measure index-refresh wall-time + RAM on first run and put a number in steering (true-time discipline) before trusting; (c) scope = `src/` keeps refresh cost near-zero (42 files). The chant is *satisfiable*, but it converts "1-hour spike" into "1-hour spike + a gated daemon with a 24h dry-run" — i.e. RH-020's effort estimate ("LOW, server exists") is **optimistic**; the safe-deployment effort is MED.

---

### 🟡 D4 (MODERATE — drift / second source of truth) — A semantic index is a *new* retrieval source that can DRIFT from reality, partly duplicating grep/glob/manifest

**The finding.** We already have three retrieval mechanisms: **grep/glob** (ground truth, never stale — reads the file as-is), **explore-subagents** (reasoning-grade discovery), and **`reports/evium-baseline/baseline-manifest.json`** (hand-extracted code map) + **`scripts/_research-surface-parser.py`** (parses `research-index.md` for SessionStart). RH-020-p3 §0 Q4 itself names the disqualifier: *"Anything that would create a second source of truth … is disqualified regardless of leverage."*

A vector index is **inherently a cache** — it is correct only as fresh as its last refresh. Between refreshes it can return **confidently-wrong retrieval**: "where do we handle X" pointing at a function that was renamed/moved/deleted 8 minutes ago. grep cannot lie this way (it reads HEAD-of-working-tree every time). So the index adds a **staleness-inversion** risk (D4 = the answer to the audit's Q4): RH-020 claims it *removes* a staleness problem (D2 showed that claim is wrong) while *introducing a new one* — embedding staleness. For an agent that's been told grep is ground truth, a sometimes-stale semantic layer is a **trust-calibration hazard**: the agent may trust a stale hit and skip the grep that would've corrected it.

**Mitigation:** (a) treat the index as a **discovery front-end, never an authority** — RH-020-p3 §2 caveat agrees ("for *finding*; subagents still do the *understanding*"); enforce "index hit → confirm with a Read/grep before acting"; (b) the merkle/throttled refresh keeps the window small; (c) don't let it replace the baseline-manifest (which serves the design-import DIFF, see D5) — they answer different questions. Net: manageable with discipline, but it's *more* moving parts and a *new* freshness contract to maintain, against a substrate whose existing retrieval has **zero** freshness contract.

---

### 🟡 D5 (MODERATE — workflow interference) — The design-import baseline-manifest workflow is NOT replaced by the index, and could be confused for replaceable

**The finding.** `reports/evium-baseline/baseline-manifest.json` exists for a specific job (its own header): *"Structural 'before' baseline for the Claude-Design import diff pipeline."* The **design-import skill** (the OUTPUT side that ingests a Claude-Design folder and DIFFs each element against its current Evium `src/` equivalent) depends on this manifest as the "current state" anchor for its DIFF phase. The manifest is a **point-in-time snapshot at a pinned `git_sha`** with curated counts (61 sections, 26 generated routes, 16 product models, etc.) — a *frozen reference*, deliberately not auto-fresh.

**Why it's a downside.** A semantic index is **live/auto-fresh by design** — the opposite of what design-import needs (a *frozen before-state* to diff against). If we (or a future agent) read RH-020's "retires the manifest" claim literally and **delete/deprecate the manifest in favor of the index**, the design-import DIFF phase loses its anchor. The index answers "where is X *now*"; the manifest answers "what was the structure at the captured baseline." Different tools; the index cannot do the manifest's job.

**Mitigation:** explicitly record that the index **does not replace** `baseline-manifest.json`; design-import keeps using the manifest. (This is really a corollary of D2 — same conflation, different victim.) Low effort to mitigate (a documented boundary), but it *must* be documented or the conflation in RH-020 propagates into a real workflow break.

---

### 🟡 D6 (MODERATE — startup dependency / SessionStart) — MCP registration + a services-registry P0 entry add SessionStart surface and a new failure mode

**The finding.** Two startup interactions:
1. **MCP server registration.** We currently have **no project `.mcp.json`** and **no global `mcpServers`** (verified). Adding one means the Claude Code client spawns/handshakes the MCP server **at session start**. If the server's runtime is heavy (Python venv + EmbeddingGemma load), that's startup latency on *every* session; if it fails to spawn (venv broke, model missing, port conflict), it's a **new startup dependency that can fail** — and MCP handshake failures can surface as noisy client errors unrelated to our work.
2. **Our `required-services.json` + SessionStart digest.** `scripts/services-check.sh` + `sessionstart-services-digest.sh` probe every registered service's `health_check_url`; **≥1 P0 down → exit 1 + a `<system-reminder>` recovery block at SessionStart.** If we register the index/MCP as a service to monitor it (natural, by our own A45 discipline), then **any session where the index daemon isn't running shows a failure digest** — recurring false-alarm noise, itself a micro-bottleneck (D3 family). If we *don't* register it, it's an **unmonitored** service (against our own A45 services-health discipline — an inconsistency).

We already run **69 hooks** (counted in settings.json: 20 SessionStart + 11 UserPromptSubmit + PreToolUse cluster + PostToolUse + 18 Stop + …). SessionStart alone fires 20 hooks; adding an MCP spawn + (maybe) a services probe is incremental but non-zero, and the bottleneck chant explicitly lists "hook chains / SessionStart cost" as a watched substrate.

**Mitigation:** (a) register the MCP server but mark the index daemon **P2** (like `scheduler` — `auto_restart_authorized:false`, health-check no-op until it's a real supervised daemon) so a stopped index never trips a P0 SessionStart alarm; (b) lazy/on-demand MCP spawn if the server supports it, to avoid per-session model-load latency; (c) measure SessionStart delta before/after (true-time discipline) and keep it under the hook-budget. Manageable, but it's **+1 startup dependency** on a system that currently has a clean MCP slate, and it interacts with our own services-health discipline in a way that needs a deliberate P-tier call.

---

### ⚪ D7 (MINOR — supply chain / maintenance) — Adopting a third-party MCP server adds an unvetted dependency + abandonment risk

**The finding.** `claude-context-local` is a **tier-2 OSS** single-maintainer GitHub project (FarhanAliRaza). Adopting it means: (a) running **third-party code that walks our filesystem and reads source** (incl. whatever we let it index — see D1); we'd need to confirm it doesn't phone home, doesn't ingest `.env`-style secrets (we have `context/codebases/reference-site-template/.env.example` in-tree — harmless example, but an indexer with a bad default could chunk real secret files if any existed); (b) **abandonment risk** — if the maintainer stops, we own an orphaned MCP dep; (c) `npm install` / `pip install` of its deps is **gated** in our system (`Bash(npm uninstall*)` denied, installs are allow-listed but the MCP's Python deps are a new surface).

**Mitigation:** BG-A is chartered to verify exactly this (local-only / no-telemetry / what-it-indexes / arbitrary-code). This row just flags that *even if* BG-A clears it, "adopt a single-maintainer OSS server into our runtime" is a standing maintenance liability the current zero-MCP system doesn't have. Pin a version; vendor it if feasible; have a removal path.

---

## Answers to the audit's six specific questions

1. **Duplicates existing retrieval / 2nd source of truth that can drift?** — **Yes, partially (D4).** grep/glob is ground-truth and never stale; the index is a cache with a freshness contract we'd newly own. RH-020's own Q4 says a second source of truth is a disqualifier. It doesn't *replace* grep, but it adds a sometimes-stale parallel layer + a trust-calibration hazard.
2. **Re-index-on-change = hook-chain/CPU tax? Violates the chant?** — **Risk: yes if mis-wired (D3).** Never put reindex in the `PostToolUse(Edit|Write)` chain. As a throttled, 7-gate-gated daemon scoped to `src/`, the tax is near-zero and the chant is satisfiable — but that converts the "1-hr spike" into "spike + gated daemon + 24h dry-run."
3. **Privacy of indexing `context/` (client Drive) — acceptable?** — **No, not as-pitched (D1).** Embedding 2.0 GB of client Drive data into a local vector + verbatim chunk cache is a real data-at-rest scope expansion, gitignore-fragile, and sync-risky on the future cluster. **Safe scope = `src/` + first-party `context/markdowns/` only; hard-exclude `context/Evium-Charging-…/` and `context/codebases/`.** The catch: that forfeits the headline upside.
4. **Staleness inversion — new stale-embeddings problem?** — **Yes (D4), and worse: the *removal* it claims (D2) is false.** RH-020 says it retires `snapshot-baseline-staleness-check.sh`; that hook is about Playwright *visual* baselines, totally unrelated to the code map. So the index removes *no* existing staleness AND introduces a new embedding-staleness window. Net staleness change is *negative* (adds one, removes none).
5. **Interferes with design-import baseline-manifest workflow?** — **Risk of confusion, not direct breakage (D5).** The manifest is a deliberately-frozen before-state for the DIFF pipeline; the index is live. They're complementary. Danger is only if RH-020's "retires the manifest" framing leads someone to deprecate the manifest — which would break design-import's DIFF anchor. Mitigation: document the boundary.
6. **MCP registration → SessionStart latency / startup dependency that can fail?** — **Yes, incremental (D6).** We have a clean zero-MCP slate today; registration adds a per-session spawn/handshake (latency if the model loads eagerly) and a new failure mode. Registering it in `required-services.json` risks recurring P0 SessionStart false-alarms unless marked P2. Manageable with lazy spawn + P2 tier + measurement.

---

## Honest verdict

**The current system is NOT "clean to adopt" by the user's literal bar ("only if no downsides").** I found **7 downsides**: 1 cautionary-privacy (D1), 1 cautionary-false-premise (D2), 1 cautionary-chant (D3), three moderate (D4 drift, D5 workflow, D6 startup), and 1 minor supply-chain (D7). **Every one has a mitigation** — none is physically un-fixable — but two structural facts should reach the user before any install:

- **The #1-ranking justification is partly false.** RH-020 oversold it: the "retires the staleness hook" benefit is a **misread of our own hook** (D2), and the index actually *adds* a staleness window rather than removing one (D4). Re-scored honestly, the *safe* (src-only, no-context/) version is — by RH-020's own words — a "moderate, not huge" discovery win on a 42-file repo where "grep is usually fine."
- **The big upside requires the risky scope.** The growth-curve that justified ★★★★★ is *exactly* indexing `context/` — which is where the **client's 2.0 GB Drive export** lives (D1). You can have the safe scope OR the big upside, not both.

**Recommendation for the user-decision:** this is a **CONDITIONAL** — not a clean GO and not a hard block. If the user is willing to (a) accept `src/`-only scope (hard-exclude client Drive + reference codebases), (b) treat any `context/markdowns/` index as a separate opt-in decision, (c) deploy index-refresh as a 7-gate daemon with a 24h dry-run (not a per-edit hook), (d) keep the baseline-manifest + grep as authorities (index = discovery front-end only), and (e) accept the corrected, lower leverage estimate (the "retires staleness" benefit is struck) — then the residual risk is small and the install is reasonable. **If the user's bar is truly "zero downsides," the honest answer is it does not clear it** — the privacy-scope tradeoff and the falsified headline benefit are real, even if small. Surface both; let the user decide whether the scoped-down, lower-value version is still worth the 1-hour-plus-a-daemon.

*(BG-B scope only — current system. BG-A owns the server's own claims; BG-C owns the multi-orchestrator/cluster/local-AI-goal future. D1's cross-node sync angle and the Pi-can't-run-embeddings angle are flagged here but belong to BG-C for depth.)*

BG-COMPLETE-SENTINEL
