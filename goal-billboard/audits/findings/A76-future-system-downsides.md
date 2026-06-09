---
audit_id: A76
finding_set: BG-C — future-system downside hunt
title: Adversarial downside analysis — does a semantic-index MCP server create regret across our PLANNED future systems?
created: 2026-05-28T23:55Z
status: complete
verdict: ADOPT-WITH-FENCES — no hard blocker found, but the naive "1-hr spike, point it at everything" framing carries 3 real future-regret traps. All are avoidable IF scoped correctly from day one. The conditional nod should be granted ONLY with the day-1 fences below baked in.
scope_evaluated:
  - G17 multi-orchestrator umbrella (M4 + M2 + Pi #1 + Pi #2)
  - multi-orchestrator-new-subscription-plan.md (M2 as 2nd orchestrator)
  - m2-ownership-tradeoff-analysis.md (Model C hybrid)
  - RH-019 Pi sub-cluster (8GB ARM Linux nodes)
  - G12 local-AI single-Mac (MLX/Ollama) + G13 cluster swarms
  - G7 cluster true parallelization
  - design-import pipeline (.claude/workflows/design-import.js)
  - heterogeneous-cluster-design.md skill ref
the_candidate: claude-context-local (FarhanAliRaza) — local-only, EmbeddingGemma model, tree-sitter/AST chunking, embeddings stored locally, MCP transport, zero API cost
---

# A76 BG-C — Future-system downside hunt for semantic-index MCP adoption

## How to read this

The user's bar: grant the nod ONLY if a thorough hunt finds **no downsides** across the current system AND **any version of our yet-unmade-but-planned future system**. I was adversarial — I tried to break it against M2, the Pi sub-cluster, local-AI (G12/G13), the cluster (G7/G17), and the design-import pipeline.

**Bottom line up front:** I found **no absolute blocker** — but I found **three genuine future-regret traps** that the RH-020-p3 "1-hour spike, adopt it" framing glosses over. None of them kill adoption; all of them are dodge-able **only if the fences are set on day one**, because retrofitting them later is exactly where the regret lives. I also caught **one load-bearing factual error** in the slam-dunk source that changes the value calculus. So the honest verdict is **ADOPT-WITH-FENCES, not the clean "no downsides" GO** the naive framing implies.

I rank findings by regret-severity. Each is tagged **[BLOCKER]** (must resolve before nod), **[FENCE]** (a day-1 constraint that prevents future regret — cheap now, expensive later), or **[CAVEAT]** (manageable, document and move on).

---

## The single grounding fact that drives everything

I measured the corpus the index would point at:

| Target | Files | Size | Git status | Index suitability |
|---|---|---|---|---|
| `src/` | **42** | ~7.4K LOC | tracked | ✅ trivial, safe, the obvious target |
| `context/` (whole) | **27,412** | **6.0 GB** | mixed | ❌ DANGER — see below |
| `context/markdowns/` | (the work corpus: 119 refs + 100+ plans + 68 audits) | moderate | tracked | ⚠️ the tempting high-value target |
| `context/codebases/` | 4 reference repos incl. **a DIFFERENT client (`the reference design*`)** | large | **gitignored** | ❌ cross-client contamination |
| `context/Evium-Charging-Google-Drive-AllAvailable-Data/` | client Drive export: `Legal`, `Due Diligence`, `Sales`, `Operations`, `Branding`… | large | **gitignored** | ❌ sensitive client data |

The slam-dunk's "growth curve" thesis — *"point it at `context/` and it becomes the single best way to answer 'have we already decided this?'"* — is pointing at a **6 GB, 27K-file directory that contains another client's source code and the client's legal/sales/due-diligence files.** That is the crux of nearly every future-regret finding below. The `src/`-only index is boring and safe; the value-add index is the one that's dangerous, and it's dangerous in ways that get *worse* under the multi-node future.

---

## Ranked findings

### 1. [FENCE] `context/`-index × multi-node sync = client-data + cross-client leak across machines

**The trap.** RH-020-p3 wants the high-value index pointed at `context/`. But `context/` holds (a) the client's Google Drive export — `Legal`, `Due Diligence`, `Sales` — and (b) `context/codebases/the reference design*`, a *different client's* codebase used as a style reference. Both are **gitignored** today precisely because they're not ours to publish.

Now layer the planned future on top:
- **G17 / multi-orchestrator** puts a second machine (M2) and two Pis into play, coordinating via "Git is the bus" (multi-orchestrator-plan §Shared filesystem coordination, Option A).
- The embeddings store is a derived artifact. Where does it live? Two bad outcomes:
  - **If the index store is committed** (to share across nodes the "Git is the bus" way) → you've just pushed **vector embeddings of the client's legal/sales files and another client's source** into the git remote. The whole reason context/codebases + the Drive export are gitignored is defeated, laundered through a `.index/` directory nobody thinks to gitignore.
  - **If the index store is per-node and rebuilt locally** → it doesn't leak via git, but now M2 and each Pi each carry a **local copy of embeddings of sensitive client data** on machines with weaker physical/disk security (a Pi on a shelf, an SD card). The Drive export was *one* copy on M4; now it's a derived copy on 4 machines.

**Why it's future-regret not present-regret:** today there's one machine. The leak surface is 1×. The moment G17 lands, the leak surface is 4× and the "just gitignore the index" reflex may already have failed (if someone committed it once, it's in history forever).

**Verdict: FENCE, not blocker.** Entirely avoidable, but ONLY with a day-1 rule:
- **Index `src/` ONLY at adoption.** `src/` is 42 files, tracked, ours, non-sensitive — and is the *only* thing the design-import pipeline + ripple-hook + discovery actually need.
- **If `context/` indexing is ever pursued, it is a SEPARATE, explicitly-scoped index that hard-excludes `context/codebases/` and `context/Evium-Charging-Google-Drive-AllAvailable-Data/`** — and its store must be added to `.gitignore` *in the same commit it's created* and declared a per-node-rebuild artifact (never synced). This is a `feedback_decision-handling-discipline` high-risk-gated action (touches client data scope), not an agent's call.
- The `.index/` store path must be in `.gitignore` before the server's first run, full stop. (Verified today: no `.mcp.json` exists yet and nothing in `.gitignore` anticipates an embeddings store — so this is a genuine gap, not already-covered.)

This is the finding the A76 frontmatter already half-flagged; BG-C's contribution is showing it **multiplies by node count** under G17 and that "Git is the bus" is the specific mechanism that turns it into a remote-history leak.

---

### 2. [CAVEAT→FENCE] The "retires the baseline-manifest staleness hook" claim is FALSE — and matters for what the index is sold as

**Finding (factual correction).** RH-020-p3 repeatedly claims adoption *"structurally retires our hand-maintained baseline-manifest staleness problem"* and that `snapshot-baseline-staleness-check.sh` exists to warn about `baseline-manifest.json` drift. **I read the actual hook. That is not what it does.**

`scripts/hooks/snapshot-baseline-staleness-check.sh` is a PreToolUse-on-`test:e2e` hook that warns when the **Phase-5 visual-snapshot baselines** (Playwright PNG baselines) may legitimately fail because open fixes are in flight. It reads the Phase-5 fix queue from `consolidated-findings.md`. It has **nothing to do with `baseline-manifest.json`** (the hand-extracted code map). A semantic code index over `src/` does **not** retire this hook — visual-regression baseline drift is a completely different concern from "is the code map current."

**Why this matters for the decision:** one of the three pillars the slam-dunk uses to justify "★★★★ leverage now" is bogus. The index's real value today is narrower: (a) semantic matches grep misses, (b) subagents not burning turns on discovery. At **42 files**, both are *moderate* wins (the slam-dunk itself concedes "grep is usually fine at 42 files"). So we're adopting a new always-on subsystem for a moderate present win + a future-growth bet whose growth path (finding #1) is the dangerous one.

**Verdict: CAVEAT (don't adopt it expecting it to retire a hook it can't touch), trending FENCE** — because it means the honest ROI is "modest now, and the big-payoff path is fenced off by finding #1." That's fine — modest wins are still wins — but the nod shouldn't be granted on a false premise. If we DON'T index `context/` (finding #1's fence), the leverage is permanently ★★★ not ★★★★★, and that should be priced in.

---

### 3. [FENCE] Multi-orchestrator index ownership + freshness divergence (the user's question #1, answered: yes it diverges, and silently)

**The trap.** The user asked directly: *does each node need its own index? Do indexes diverge between nodes? Is a stale index a source of cross-agent inconsistency? Who owns it?*

Answer, traced through the actual architecture:

- **The index is a per-machine derived artifact** (embeddings + vector store on local disk). It is NOT in the role-stable file-zone table of `heterogeneous-cluster-design.md` — it's an *unmodeled* piece of state. That's the gap.
- **Yes, it diverges.** The merkle-tree freshness that makes it "self-refreshing" works **per-checkout**. M4's index reflects M4's working tree; M2 works on `m2/*` feature branches (multi-orchestrator-plan §Git strategy) that M4 hasn't merged yet. So **M2's index sees code M4's index doesn't, and vice-versa.** The CAP-tradeoffs section of the cluster ref explicitly accepts **eventual consistency** ("M2 might be N seconds behind M4") — for a semantic index that means **M2 and M4 can return different answers to the same "where do we handle X?" query**, with no warning that they disagree.
- **Cross-agent inconsistency is the real risk.** Today our cross-session survival relies on *files* being the source of truth (git, append-only logs). A semantic index is a *derived cache*. If an M2 subagent retrieves a stale chunk (code that M4 already refactored on `main` but M2 hasn't pulled), it can act on code that no longer exists — a class of bug our grep-based discovery *cannot* have, because grep always reads the live tree.

**Why future-regret:** none of this exists today (one node, one tree). It's born the instant M2 comes online under G17-P2. And it's insidious because the index *looks* authoritative — a confident semantic hit on stale code is worse than a grep miss.

**Verdict: FENCE.** Cheap to prevent, must be decided up front:
- **The index is explicitly declared a per-node, non-authoritative cache.** Add it to `heterogeneous-cluster-design.md`'s anti-pattern list: *"the index is a discovery accelerator, never a source of truth; verify a hit against the live file before acting on it."* (The slam-dunk source already says "retrieval is for finding; subagents still do the understanding" — this just hardens that into a multi-node rule.)
- **Pre-task `git pull` discipline (already in the M2 plan as a pre-pull hook) gates index freshness too** — M2 reindexes after pulling. Document the ordering: pull → reindex → work.
- **No attempt to share/sync one index across nodes.** Each node rebuilds its own from its own tree (this also resolves finding #1's git-leak vector). The merkle-tree makes per-node rebuild cheap, so this costs little.

---

### 4. [CAVEAT] Pi sub-cluster: the Pis structurally CANNOT run this index — and that's fine (capability-match, not a break)

**The user's question #2, answered.** Can an 8GB ARM Pi run EmbeddingGemma + a vector store, or does adopting an index create a node Pis can't participate in?

- **Embedding inference on a Pi is technically possible but a poor fit.** EmbeddingGemma is small enough to *load* on 8GB, but the Pis' assigned roles (heterogeneous-cluster-design + RH-019) are **Chromium-shard farm, autonomic daemons, heartbeat** — all light, always-on, low-RAM. Adding an embedding-model + vector-store refresh load competes with exactly those roles on a RAM-constrained node. RH-017 sub-B notes a Pi can run a 3B model *only with the $130 AI HAT+ 2 accelerator* — i.e., not out of the box.
- **But here's the key insight: the Pis don't NEED the index.** Per `heterogeneous-cluster-design.md`'s capability-match model, the index is a capability the *orchestrator nodes* (M4, M2 — the ones running Claude Code agents that do discovery) need. The Pis run *scripts*, not agents (anti-pattern #3: "don't run Anthropic-billed workloads on Pis"). Scripts don't do semantic code discovery. So a Pi lacking the index is **capability-correct, not a capability-break** — exactly like a Pi lacking WebKit or an Anthropic account.

**Verdict: CAVEAT, explicitly NOT a blocker.** The heterogeneous-cluster model already handles "node X lacks capability Y" gracefully. The index simply becomes another capability tag (`SEMANTIC_INDEX`) that M4+M2 have and Pis don't. The ONLY regret would be if some future workload-dispatch logic *assumed* every node could answer a semantic query — so the fence is: **tag the index as an orchestrator-only capability in the capability set; never dispatch index-dependent work to a Pi.** One line in the capability table.

---

### 5. [FENCE] Lock-in / rip-out cost vs G12/G13 local-AI — adopt-then-replace is a real risk, removability is GOOD but the embedding-model choice is the entanglement

**The user's questions #3 + #5, answered together.** Does the index conflict with / duplicate the planned G12/G13 local-AI (MLX/Ollama) work? Would we adopt now then rip it out when G12 lands? Is it cleanly removable?

Two layers, opposite answers:

**(a) The MCP server wiring is cleanly removable.** It's an MCP entry + a local store dir. Removing it = delete the `.mcp.json` entry + `rm -rf` the store. No src/ entanglement, no hook entanglement (it's a retrieval tool the agent calls, not a substrate other code depends on — *provided* we never let the design-import pipeline or ripple-hook hard-depend on it; see finding #6). So on the *integration* axis, low lock-in. ✅

**(b) The embedding-model + store choice DOES collide with G12/G13's roadmap, and this is the genuine regret risk.** `claude-context-local` ships its **own** embedding model (EmbeddingGemma) and its **own** vector store, managed by the MCP server. Meanwhile:
- G12's whole thesis is *"one Mac running MLX/Ollama; multiple specialist models; token-free heavy lifting"* and its Phase-2 architecture explicitly designs an **event-bus + LiteLLM-style router** (RH-017 sub-B names LiteLLM as "the keystone… the routing brain").
- RH-017 sub-B even lists *"shared RAG vector DB on Pi-readable hardware"* / *"doubles as model + dataset + RAG vector store backing"* as part of the planned local-AI rig.

So **G12/G13 plan to own an embedding+vector-store layer themselves**, routed through LiteLLM/Ollama. Adopting `claude-context-local` now stands up a **second, parallel, independently-managed** embedding model + vector store that G12 will later want to consolidate. That's not "rip out the MCP entry" — that's "we now have two RAG stacks (the MCP server's EmbeddingGemma + the G12 Ollama/MLX one) and have to decide whether to migrate the index onto the G12 stack or keep them split." Re-embedding a `context/`-scale corpus onto a different model = a real (if one-time) migration cost, and running two embedding models concurrently = the kind of duplicated-substrate the `feedback_bottleneck-restriction-chant` warns against.

**Why future-regret:** today there's no local-AI stack, so there's nothing to collide with — the index is the only embedding game in town. The collision is *born* when G12 Phase 5 lands. The user has **explicitly deferred local AI until after Evium**, so G12 is genuinely future — which is exactly the "planned future system" the nod is gated on.

**Verdict: FENCE (and a deferral-decision the user should make eyes-open).** Mitigations:
- **`src/`-only scope (finding #1's fence) makes this nearly moot** — re-embedding 42 files onto G12's stack later is trivial; re-embedding a 6GB corpus is not. This is a *third* independent reason to keep the index `src/`-only until G12 exists.
- **When G12 lands, the explicit migration question is: fold the code-index into the G12/LiteLLM RAG stack, or keep `claude-context-local` as a separate purpose-built tool?** Document this now as a known G12-phase decision (cross-ref into G12's plan) so it's not a surprise. It is NOT a reason to block adoption — it's a reason to keep scope small so the future migration is cheap.
- **Do NOT let the index's embedding model become a dependency of anything else** (don't build the ripple-hook to call *this server's* embeddings specifically; have it call an abstraction). That keeps option (a)'s clean-removability intact.

---

### 6. [FENCE] design-import pipeline: index HELPS the DIFF phase, but must not become a hard dependency

**The user's question #4, answered.** Does the index help or interfere with the inventory→diff→stage flow?

I read `design-import.js`. The DIFF phase (Phase 2) is the relevant one: each agent must *"Find the current Evium counterpart. Search src/pages/ and src/components/…"* — this is **literally a semantic discovery task done by grep/glob today.** A "hero" element → `src/components/Hero.astro`; a section → a `<section>` block in a page. So:

- **The index HELPS here** — and this is actually the design-import pipeline's *best* use case for it. Semantic retrieval over `src/` (42 files, exactly the safe scope) is a better front-end for "find the Evium equivalent of this design element" than keyword grep, especially for fuzzy matches (a design "stats band" → an Evium section named something else). This is a genuine synergy, fully inside the safe `src/`-only scope. ✅
- **The interference risk is dependency, not function.** design-import is a flagship **read-only homogeneous fan-out** (per `dynamic-workflow-adoption.md`, uncapped, no A21 5-cap). If the DIFF agents are rewritten to *require* the index MCP server to find counterparts, then: (a) the pipeline breaks on any node where the index isn't running (every Pi; M2 before its index warms; M4 if the server is down), and (b) it couples a clean read-only workflow to a stateful external service. The pipeline currently has zero external-service dependencies — that's a feature.

**Verdict: FENCE.** The index should be an *optional accelerator* for the DIFF phase, never a requirement:
- DIFF agents try the index if available, **fall back to grep/glob if not** (they already know how — that's the current method). One conditional, preserves the pipeline's run-anywhere property.
- This keeps design-import working under G17 even on nodes without the index, and keeps the clean-removability from finding #5(a) intact.

---

### 7. [CAVEAT] Cross-session / cross-node freshness — the index does NOT break cross-session survival, but it's a NEW thing that needs the discipline

**The user's question #6, answered.** Our substrate prides itself on cross-session survival — does an index break or complicate that?

- **It doesn't break it.** Cross-session survival lives in *files* (git, append-only JSONL, the goal/bug billboards, MEMORY.md + mirror). The index is a *derived cache* of one of those file sets (`src/`). If the index is deleted, lost, or stale, **nothing of record is lost** — it rebuilds from the live tree. So on the "did we lose work?" axis, the index is strictly safe: it's a performance cache, not a system-of-record. ✅
- **The complication is the discipline-surface, not the data.** It adds one more piece of per-node state that has a freshness contract (findings #3). Our cross-session discipline is already heavy (per A40/A44/A71 etc.); the index adds a small new rule ("it's a cache; verify hits; rebuild after pull"). That's a manageable addition, not a break — but it IS a new line item in the substrate's mental model, and the `feedback_bottleneck-restriction-chant` lens applies: the index-refresh CPU/RAM must be measured (true-time discipline) so it never becomes the thing that bogs a node down. RH-020-p3 itself flags monorepo-scale indexing has hit 100GB RAM / 10-min builds — irrelevant at `src/`-scale (the safe scope), but a screaming klaxon against the `context/`-scale index (finding #1 again).

**Verdict: CAVEAT.** No cross-session-survival regret as long as the index is treated as a cache (which it is). Just measure its footprint before trusting it on the always-on/RAM-tight nodes.

---

## Synthesis — is adoption future-proof?

**It is future-proof IF AND ONLY IF it is fenced to `src/` and modeled as a per-node, non-authoritative, optional cache from day one.** The naive RH-020-p3 framing ("1-hour spike, adopt it, then point it at `context/` for the big win") is **NOT** future-proof — its own headline growth path (the `context/` index) is the source of the most severe future-regret findings (#1 client-data/cross-client leak multiplied by node count, #5 collision with the G12 RAG stack, #7's RAM klaxon). The slam-dunk also oversells the value with a false claim (#2 — it does not retire the staleness hook).

The good news: **every regret trap is dodge-able, and the dodge is the same small set of fences** — and those fences happen to also keep the *present* adoption trivially cheap and trivially removable. The fences aren't a tax on a good idea; they're the difference between a clean tool and a 4-machine client-data leak.

### The day-1 fences (grant the nod WITH these; they are not optional)

1. **Index `src/` ONLY.** 42 files, tracked, ours, non-sensitive. Do not index `context/` at adoption — it's 6GB / 27K files including another client's source (`the reference design*`) and the client's `Legal`/`Sales`/`Due Diligence` Drive export.
2. **Add the embeddings store path to `.gitignore` in the same commit the server is wired** (verified gap: nothing anticipates it today). The store is a per-node-rebuild artifact, never committed, never synced via "Git is the bus."
3. **Declare the index a per-node, non-authoritative discovery cache** in `heterogeneous-cluster-design.md` (anti-pattern entry + capability tag `SEMANTIC_INDEX` = orchestrator-only). Verify any hit against the live file before acting; rebuild after `git pull`.
4. **Keep it optional everywhere it touches the substrate** — design-import DIFF phase and any future ripple-hook try the index, fall back to grep. No hard dependency → preserves clean-removability + run-anywhere.
5. **Treat any future `context/` index as a separate, explicitly-scoped, user-gated decision** (`feedback_decision-handling-discipline` high-risk: touches client-data scope) that hard-excludes `context/codebases/` + the Drive export — AND fold it into the G12-phase "consolidate onto the LiteLLM/Ollama RAG stack?" decision rather than standing up a second permanent embedding stack.
6. **Don't sell it on the false premise** (#2): it does NOT retire `snapshot-baseline-staleness-check.sh`. Price the value as "modest discovery win at 42 files + optional DIFF-phase accelerator," not "★★★★★ + retires a hook."

### Honest one-liner verdict

**Not the clean "no downsides" the conditional nod asked for — but no blocker either.** A `src/`-scoped, fenced adoption is future-proof and a fine modest win. The dangerous version (the `context/` "growth curve" the slam-dunk is actually excited about) collides with client-data privacy across the M2/Pi fleet AND with the G12/G13 local-AI RAG roadmap. So: **grant the nod for the `src/`-only fenced tool; explicitly withhold it from `context/` until G12 forces the RAG-consolidation decision.** The install remains the user's gesture regardless.

## Cross-references

- A76 master audit — `goal-billboard/audits/A76-semantic-index-adoption-downside-analysis.md` (this is BG-C)
- RH-020 part 3 — the slam-dunk source (and the source of the #2 false claim + the `context/` growth thesis that drives the regret)
- `feedback_bottleneck-restriction-chant` — index-refresh CPU/RAM must never become a node bottleneck (#7)
- `feedback_decision-handling-discipline` — `context/`-indexing is a high-risk, user-gated, client-data-scope decision (#1, #5)
- `reference_context-folder` memory — confirms context/ holds the client Drive export + reference codebases (the privacy substrate of #1)
- `heterogeneous-cluster-design.md` — where the index needs a capability tag + anti-pattern entry (#3, #4)
- G12/G13 + `local-ai-single-mac-mass-wielding-plan.md` + RH-017 sub-B — the planned LiteLLM/Ollama RAG stack the index collides with (#5)
- `design-import.js` — DIFF phase is the index's best `src/`-scoped use case, must stay optional (#6)
- `scripts/hooks/snapshot-baseline-staleness-check.sh` — read in full; does NOT do what RH-020-p3 claims (#2)

BG-COMPLETE-SENTINEL
