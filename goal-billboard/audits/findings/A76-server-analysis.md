# A76 — Semantic-Index MCP Server: Adversarial Downside Analysis

**Audit:** A76 (server-itself sub-analysis)
**Date:** 2026-05-28T23:45Z → 23:48Z (UTC, true-time sourced)
**Mandate:** The user adopts ONLY if there are NO downsides. My job was to try to find reasons NOT to. Skeptic stance.
**Scope:** The candidate server(s) themselves — not the broader "should we have semantic search" question.

---

## TL;DR verdict

Two distinct candidates, NOT interchangeable:

| | `zilliztech/claude-context` | `FarhanAliRaza/claude-context-local` |
|---|---|---|
| Stars | 11.6k | 231 |
| Embeddings | **Cloud API (OpenAI default)** | **100% local (EmbeddingGemma-300m)** |
| Vector DB | **Cloud (Zilliz/Milvus)** | **Local (FAISS file)** |
| API cost | **Per-token, ongoing** | **Zero** |
| Data egress | **Code leaves machine** | **None** |
| Maintainer | Company (Zilliz) | **Solo dev** |

**`zilliztech/claude-context` FAILS the "no downsides" bar outright** — it requires a paid embedding API key (OpenAI/Voyage/Gemini) AND a cloud vector database (Zilliz Cloud) by default. Per-token cost + your code uploaded to OpenAI + your vectors stored on Zilliz's cloud. That is a hard violation of both the zero-cost goal and the bottleneck-restriction/privacy posture. Its Ollama+self-hosted-Milvus "local mode" exists but is a poorly-documented afterthought (no quickstart, all examples are cloud). **Do not adopt the Zilliz one.** Everything below is about the LOCAL one.

**`FarhanAliRaza/claude-context-local` is genuinely local and genuinely zero-API-cost — that part is CONFIRMED in source, not just marketing.** BUT it is **not "no downsides."** It carries **4 real downsides** (one of them serious for THIS user's environment) plus several smaller ones. Verdict: **clean on cost/privacy/telemetry; NOT clean on (a) secret/client-data ingestion scope, (b) maintenance/abandonment risk, (c) supply-chain hygiene, (d) gated-model friction.** Adoptable only with explicit scoping discipline (always pass a narrowing `file_pattern`, never point it at the repo root bare).

---

## Claim-by-claim verdict (the LOCAL server)

### 1. "Local-only" embeddings — **CONFIRMED**
- `pyproject.toml` deps: `sentence-transformers`, `faiss-cpu`, `huggingface-hub`, tree-sitter. **No `openai`, no `voyageai`, no `anthropic`, no network embedding client.** ([pyproject.toml](https://github.com/FarhanAliRaza/claude-context-local/blob/main/pyproject.toml))
- `embeddings/embedder.py` runs `google/embeddinggemma-300m` via `SentenceTransformer(...).encode(...)` locally; `device="auto"` → CUDA→MPS→CPU. On the user's Apple Silicon it uses MPS. No API call in the embed path. ([embedder.py](https://github.com/FarhanAliRaza/claude-context-local/blob/main/embeddings/embedder.py))
- Model is downloaded ONCE from HuggingFace, cached under `~/.claude_code_search/models`, then `HF_HUB_OFFLINE=1` capable. The only network call is the one-time model pull.
- **This is the real deal.** Unlike the Zilliz tool, "local" is not a lie here.

### 2. "Zero API cost / forever free" — **CONFIRMED (no hidden paid dependency)**
- No paid SaaS in the dependency tree. No embedding API, no vector-DB-as-a-service. FAISS is an in-process C++ lib writing a local file. ([README](https://github.com/FarhanAliRaza/claude-context-local/blob/main/README.md))
- **One asterisk that is NOT a cost:** there is a *token-savings* claim ("fewer tokens in Claude Code"). That's the upside, out of scope here. The cost claim itself is clean.

### 3. Telemetry / phone-home — **CONFIRMED clean (no telemetry found)**
- No analytics/telemetry SDK in deps. No `requests`/`httpx` posting usage anywhere. The only outbound traffic is: (a) one-time HF model download, (b) `astral.sh`/HF during install. ([install.sh](https://github.com/FarhanAliRaza/claude-context-local/blob/main/scripts/install.sh))
- **UNVERIFIABLE-but-low-risk caveat:** I read the primary code paths (embedder, indexer, merkle, mcp server, chunker) — none phone home. I did not byte-audit every transitive dep (sentence-transformers / HF hub do their own update checks; HF hub can ping for model revisions unless `HF_HUB_OFFLINE=1`). Setting `HF_HUB_OFFLINE=1` after first download closes even that. **Effectively clean; pin offline to be certain.**

### 4. Footprint — **CONFIRMED; modest, does NOT inherently violate the bottleneck chant — with one caveat**
- **Disk:** model ~1.2–1.3 GB (one-time, shared across all projects) + per-repo FAISS index `code.index` + SQLite `metadata.db` + `stats.json`. README budgets "1–2 GB free." Index size scales with chunk count; for a site repo this is tens-to-low-hundreds of MB, not GB. ([README Storage section](https://github.com/FarhanAliRaza/claude-context-local/blob/main/README.md))
- **RAM:** EmbeddingGemma-300m is a 300M-param model → roughly **0.6–1.2 GB resident** while loaded (fp32/fp16). Loaded into memory during indexing and during a search session.
- **Daemon? — IMPORTANT NUANCE.** It is registered as an **stdio MCP server** (`claude mcp add code-search -- uv run ... server.py`). It is spawned by Claude Code as a child process and lives **as long as the Claude Code session holds the MCP connection** — i.e. it is NOT a system-wide always-on LaunchAgent, but it IS a persistent process for the duration of every Claude session that has it registered, holding the embedding model in RAM. With `--scope user`, that's **every** Claude Code session on the machine.
  - **Bottleneck-chant risk (real but bounded):** ~1 GB RAM held per active Claude session for the model + a `uv`/python process. On a multi-session / multi-worktree day (which this user runs — dual-worktree, BG dispatch), that's N× ~1 GB. Lazy-loading mitigates (model may not load until first index/search call — needs runtime confirmation), but worst case this is a steady ~1 GB+ resident tax. **Flag for the bottleneck-detect daemon if adopted.**
- **CPU:** Spikes during indexing (chunk + embed). Incremental Merkle indexing means only changed files re-embed after the first pass, so steady-state CPU is low. First full index of a large tree is a one-time multi-minute burn.
- **HTTP port:** server supports `--transport http --port 8000`, **but default is `stdio`** and the registration command uses stdio → **no open network port by default.** Clean unless someone opts into HTTP.

### 5. Dependencies — **CONFIRMED, with hygiene downsides**
- Runtime: `faiss-cpu`, `sentence-transformers`, `huggingface-hub`, `fastmcp`, `mcp`, `click`, `rich`, `sqlitedict`, `tree-sitter` + 11 tree-sitter grammar packages. ([pyproject.toml](https://github.com/FarhanAliRaza/claude-context-local/blob/main/pyproject.toml))
- **DOWNSIDE — test deps shipped as runtime deps:** `pytest`, `pytest-asyncio`, `pytest-cov`, `pytest-mock` are listed under `[project.dependencies]`, not a dev/optional group. Sloppy packaging → bloats every install with test machinery. Minor, but a code-smell about maintenance rigor.
- **DOWNSIDE — `uv.lock` was DELETED** in the most recent commit (2025-11-13, "Remove the `uv.lock` file"). No pinned/locked dependency versions → installs float to latest, which is a **reproducibility + supply-chain fragility** risk (a bad upstream sentence-transformers/HF release can silently break or alter behavior). ([commits](https://github.com/FarhanAliRaza/claude-context-local/commits/main))
- Pulls in a large transitive ML stack (torch via sentence-transformers, etc.) — heavy first install.

### 6. Maintenance / abandonment — **DOWNSIDE (real)**
- **Solo project.** 3 contributors but the distribution is `FarhanAliRaza: 20 commits, GreenWizard2015: 3, DevElCuy: 1`. Effectively one person + two drive-by PRs. ([contributors](https://github.com/FarhanAliRaza/claude-context-local/graphs/contributors))
- **Last push: 2025-11-13.** As of 2026-05-28 that is **~6.5 months with zero commits.** Self-labeled "🚧 Beta Release," "Benchmarks coming soon" (never arrived). 10 open issues.
- The README badge is literally **"🌍 Actively Seeking Remote Work"** linking the maintainer's email — i.e. the author is job-hunting; sustained maintenance is not promised.
- **Exposure if it dies:** It's an MCP server, not a library we build against — if abandoned, blast radius is contained: the index goes stale or a dep break stops it from starting, and we lose the search feature. No data loss (vectors are local files), no runtime coupling to the site build. **Recoverable, but it means treating this as disposable tooling, not infrastructure.** We'd also own any security patching ourselves (see §7).

### 7. Security — **MIXED. One CONFIRMED serious downside for THIS environment.**
- **Arbitrary code execution during indexing? — CONFIRMED NO.** Indexing only `open(file, 'rb')`s + SHA-256 hashes files (`merkle/merkle_dag.py: hash_file`) and runs tree-sitter *parsers* (parse, not eval) over supported extensions. No `eval`/`exec`/`import` of target code. `trust_remote_code` not set on the embedder. Parsing untrusted code is not executing it. ([merkle_dag.py](https://github.com/FarhanAliRaza/claude-context-local/blob/main/merkle/merkle_dag.py))
- **Install vector:** `curl -fsSL .../install.sh | bash` and a nested `curl .../astral.sh/uv/install.sh | sh`. Standard for the ecosystem but it IS remote-code-into-shell; pulls + `git clone` + `uv sync` + downloads a **gated Google HF model**. Recommend: clone the repo, read `install.sh`, run pinned — don't pipe to bash. (`set -euo pipefail` is present, which is good.) ([install.sh](https://github.com/FarhanAliRaza/claude-context-local/blob/main/scripts/install.sh))
- **🚨 SECRET / CLIENT-DATA INGESTION — the serious one.** I read the actual walk + filter logic:
  - **There is NO `.gitignore` parsing anywhere.** The only ignore mechanism is a **hardcoded `Set[str]` of basenames** in `merkle/merkle_dag.py` (`__pycache__`, `.git`, `.venv`, `node_modules`, `build`, `dist`, `out`, `public`, `.next`, `.astro`, … + `*.pyc`/`.DS_Store`). `change_detector.py` only adds the snapshot dir. **Your `.gitignore` is ignored by the indexer.** ([should_ignore + ignore set, merkle_dag.py lines 56–95](https://github.com/FarhanAliRaza/claude-context-local/blob/main/merkle/merkle_dag.py))
  - **`context/` is NOT in that hardcoded set.** Your gitignored client Drive export and the entire `context/markdowns/` workspace are **NOT excluded** by the tool's defaults.
  - **Two-stage filter limits but does NOT eliminate the blast radius:** every non-ignored file gets **walked and SHA-256 hashed** (including `.env`, `.pem`, `.key`, PDFs, CSVs — their *paths and hashes* enter the Merkle snapshot), but only files whose extension is in the supported set get **chunked + embedded into the vector store.** Supported set = the 15 code extensions **+ Markdown** (confirmed: `tree-sitter-markdown` is wired in `chunking/available_languages.py` lines 84–87, even though the README's language table omits it). ([available_languages.py](https://github.com/FarhanAliRaza/claude-context-local/blob/main/chunking/available_languages.py))
  - **Net exposure for this user:** (a) **Markdown is embeddable** → if you index the repo root, the *contents* of `context/markdowns/` (audits, plans, billboards) AND any `.md` in the gitignored client Drive export get embedded into the local FAISS index. (b) A hardcoded secret living inside a `.py`/`.ts`/`.js` file would be chunked + embedded. (c) `.env`, `.json`, `.yaml`, `.pem`, images, PDFs are NOT embedded (not supported extensions) — their content is safe, only path+hash is recorded. The index is local, so this is not *egress* — but it's still unwanted client-data ingestion into a tool whose abandonment/patch status you don't control.
  - **No MCP exclude parameter.** The exposed tools (`index_directory`, `search_code`) take a `file_pattern` **include-glob** (e.g. `"*.py"`) and a `project_name` — there is **no exclude/ignore arg**, and the hardcoded ignore set is not user-extensible via MCP. ([strings.yaml tool defs](https://github.com/FarhanAliRaza/claude-context-local/blob/main/mcp_server/strings.yaml))
  - **Mitigation EXISTS but is opt-in discipline, not a safety default:** always invoke `index_directory("<repo>/src", file_pattern="*.astro")` (or point at `src/` only, never the repo root). Because `file_pattern` is an allowlist, a tight include-glob effectively fences out `context/`. But this relies on the operator (or the agent) never running a bare `index_directory(".")` — and the README's own quickstart literally says *"say: index this codebase"* with no scoping, which would do the wrong thing. **This is the single biggest catch.**

### 8. MCP integration reality — **PARTIALLY CONFIRMED; a likely limitation**
- Exposed tools (from `strings.yaml`): `search_code`, `index_directory`, `find_similar_code`, `get_index_status`, `list_projects`, `switch_project`, `clear_index`, `index_test_project`. Reasonable surface. ([strings.yaml](https://github.com/FarhanAliRaza/claude-context-local/blob/main/mcp_server/strings.yaml))
- **Subagent access — UNVERIFIABLE from the server; it's a Claude-Code-side concern.** The server just speaks MCP over stdio. Whether *subagents/BG dispatches* get the tool depends on how Claude Code propagates registered MCP servers to spawned agents — the server has no say. **Known reality in this harness:** MCP tools are frequently NOT exposed to Task/BG subagents the same way they are to the main agent (subagents get a curated/different tool set). So the realistic expectation is **main-agent retrieval, not guaranteed subagent retrieval.** Given this user's heavy fan-out/BG model, that materially caps the value: the agents doing parallel grunt work may not be able to call it. **Needs a live test before counting on subagent reach — do not assume it.**

---

## Downsides found (the skeptic's list — even the small ones)

**Serious (environment-specific):**
1. **No `.gitignore` respect + no exclude param + Markdown is embeddable** → bare indexing ingests `context/` (client Drive export + markdown workspace) into the local vector store. Mitigable only by always passing a narrowing `file_pattern`/scoped root — which is opt-in discipline the README actively discourages.

**Moderate:**
2. **Abandonment risk** — solo maintainer, ~6.5mo since last commit, "beta," author job-hunting. If a dep break kills startup, we patch it ourselves or lose the feature.
3. **No `uv.lock`** — unpinned deps → reproducibility/supply-chain fragility; latest upstream can silently break it.
4. **Subagent reach not guaranteed** — likely main-agent-only in practice; caps value for this user's fan-out workflow until proven otherwise.

**Minor / hygiene:**
5. Test deps (`pytest*`) shipped as runtime deps — packaging smell.
6. **Gated HF model** — EmbeddingGemma requires accepting Google's terms on HuggingFace and possibly an `HF_HUB_TOKEN` to download; not a frictionless `uv sync`.
7. **License ambiguity** — pyproject + README both say **GPL-3.0** (copyleft), but the GitHub API reports **no detected LICENSE file** (`license: None`). We don't redistribute, so GPL is low-risk for internal use, but the missing-file/claimed-license mismatch is a real ambiguity.
8. `curl | bash` install vector (standard, `set -euo pipefail` present, but still remote-shell-exec; clone-and-read recommended).
9. ~1 GB RAM resident per active Claude session while model is loaded → multiply across dual-worktree/multi-session days (bottleneck-chant watch item).
10. **`from chunking.languages import LANGUAGE_MAP`** appears to reference a module path that 404s on `main` (the real package is `chunking/languages/` dir + `available_languages.py`) — either a branch/packaging inconsistency or latent import fragility. Another rigor signal.

---

## Footprint estimate (the local server, on this Mac)

- **One-time disk:** ~1.2–1.3 GB model (shared) + heavy torch/ML transitive stack in the venv (~several hundred MB).
- **Per-repo disk:** FAISS `code.index` + SQLite metadata — tens-to-low-hundreds of MB for a marketing-site repo; grows with chunk count (and balloons if `context/markdowns/` gets indexed).
- **RAM:** ~0.6–1.2 GB resident per active session while the embedding model is loaded.
- **CPU:** burst on first/full index (minutes); near-idle steady-state thanks to Merkle incremental.
- **Network:** one-time gated-model download from HF; zero thereafter with `HF_HUB_OFFLINE=1`. No telemetry, no per-query egress, no open port (stdio default).

---

## Final verdict

- **`zilliztech/claude-context`: REJECT.** Cloud embeddings (paid API key) + cloud vector DB + code egress by default. Fails the no-cost and privacy bars on contact. Not a candidate.
- **`FarhanAliRaza/claude-context-local`: the cost/privacy/telemetry claims are CONFIRMED TRUE — it is honestly local and honestly free.** But it is **NOT "zero downsides."** It carries one serious environment-specific catch (no-gitignore + Markdown-embeddable → client-data/secret ingestion into the index, fixable only by disciplined `file_pattern` scoping the tool's own UX discourages) plus real abandonment + supply-chain-pinning + likely subagent-reach limitations.
- **Honest bottom line for a user who adopts "only if NO downsides":** the *cost/privacy* promise is clean, but the **server is not categorically clean** — there is a catch, and it lands precisely on this user's crown-jewel risk (the gitignored client Drive data + the `context/` workspace). It is **adoptable with guardrails** (scoped-index-only, offline-pin, treat as disposable tooling, verify subagent reach), **not adoptable as a fire-and-forget "index this codebase."** Surface the catch loudly; do not let the "100% local, no downsides" marketing line stand unqualified.

---

## Sources
- [FarhanAliRaza/claude-context-local — repo](https://github.com/FarhanAliRaza/claude-context-local)
- [README.md](https://github.com/FarhanAliRaza/claude-context-local/blob/main/README.md)
- [pyproject.toml (deps + GPL classifier)](https://github.com/FarhanAliRaza/claude-context-local/blob/main/pyproject.toml)
- [merkle/merkle_dag.py (hardcoded ignore set + walk/hash)](https://github.com/FarhanAliRaza/claude-context-local/blob/main/merkle/merkle_dag.py)
- [merkle/change_detector.py (no gitignore parse)](https://github.com/FarhanAliRaza/claude-context-local/blob/main/merkle/change_detector.py)
- [chunking/multi_language_chunker.py (extension allowlist)](https://github.com/FarhanAliRaza/claude-context-local/blob/main/chunking/multi_language_chunker.py)
- [chunking/available_languages.py (Markdown wired in)](https://github.com/FarhanAliRaza/claude-context-local/blob/main/chunking/available_languages.py)
- [embeddings/embedder.py (local SentenceTransformer, device=auto)](https://github.com/FarhanAliRaza/claude-context-local/blob/main/embeddings/embedder.py)
- [mcp_server/server.py (stdio default, optional http/port)](https://github.com/FarhanAliRaza/claude-context-local/blob/main/mcp_server/server.py)
- [mcp_server/strings.yaml (tool surface; file_pattern include-only, no exclude)](https://github.com/FarhanAliRaza/claude-context-local/blob/main/mcp_server/strings.yaml)
- [scripts/install.sh (curl|bash, uv, gated model pull)](https://github.com/FarhanAliRaza/claude-context-local/blob/main/scripts/install.sh)
- [contributors graph (solo maintainer)](https://github.com/FarhanAliRaza/claude-context-local/graphs/contributors)
- [commits (last push 2025-11-13; uv.lock removed)](https://github.com/FarhanAliRaza/claude-context-local/commits/main)
- [zilliztech/claude-context — repo (cloud-first comparator)](https://github.com/zilliztech/claude-context)
- [zilliztech/claude-context MCP README (OPENAI_API_KEY + MILVUS_ADDRESS required)](https://github.com/zilliztech/claude-context/blob/master/packages/mcp/README.md)
- [@zilliz/claude-context-mcp on npm](https://www.npmjs.com/package/@zilliz/claude-context-mcp)
- GitHub REST API: repo meta, contributors, commits (queried 2026-05-28T23:45Z)

BG-COMPLETE-SENTINEL
