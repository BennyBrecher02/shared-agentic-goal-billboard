---
audit_id: A76
title: Semantic code-index adoption — thorough downside analysis (current + all planned-future systems) before the user's conditional nod
status: verified_complete
catalogued: 2026-05-28T23:45Z
priority_when_run: P1
trigger: 2026-05-28 — user CONDITIONAL nod on the RH-020-part-3 #1 slam-dunk (adopt a local-only semantic-index MCP server): *"as long as there are no downsides after a thorough consideration of all the possible times this could be a problem for our current system or any version of our yet unmade but planned future system to this then id give you the nod."*
deferral_reason: NONE — the nod is gated on THIS analysis coming back clean. Adversarial downside hunt required before proceeding. Even if clean, the MCP-server INSTALL is the user's gesture.
serves_northern_star: G2
belongs_to_goal: G4
related:
  - context/markdowns/research/rabbit-holes/RH-020-cursor-integration/part-3-cursor-does-better-slam-dunks.md (the slam-dunk source)
  - candidate servers: claude-context-local, claude-context (local-only, zero-API-cost, MCP)
findings:
  - already-suspected-downside: our context/ is gitignored AND holds the client's Drive export (sensitive). A semantic index that embeds context/ could embed sensitive client data into a local store — and on the M2/Pi multi-orchestrator, embeddings might sync across nodes. Privacy + scope-of-indexing is a real consideration to resolve.
---

# A76 — Semantic-index adoption downside analysis

## The bar the user set

The nod is CONDITIONAL: proceed ONLY if a thorough hunt finds **no downsides** across (a) the current system AND (b) **any version of our yet-unmade-but-planned future system**. So this is an adversarial investigation — try to BREAK the idea. If real downsides surface, report them + do NOT proceed. If genuinely clean, report + the install remains the user's gesture.

## What "it" is

Adopt (not build) a local-only, zero-API-cost semantic code-index MCP server (e.g. `claude-context-local`) so our agent + subagents get `@codebase`-style semantic retrieval over `src/` AND our huge doc corpus (119 skill refs + 100+ plans + 68 audits). RH-020-p3 ranked it the #1 slam-dunk + claimed it retires the `snapshot-baseline-staleness` problem.

## Investigation (3 adversarial BGs)

- **BG-A — the server itself**: verify the local-only / zero-API-cost / no-telemetry claims; footprint; dependencies; maintenance/abandonment risk; security (does it run arbitrary code? what does it index — could it ingest secrets/.env?).
- **BG-B — current-system downsides**: conflicts with hooks / scheduler / memory / BG dispatch / matrix / gitignore / settings; the bottleneck-restriction chant (does it become a daemon/resource bottleneck?); the context/ client-Drive PRIVACY angle; does it duplicate/undermine existing retrieval (grep/glob/baseline-manifest)?
- **BG-C — future-system downsides**: multi-orchestrator (M2 + Pi) — per-node index? sync/divergence? heterogeneous compute (can a Pi even run the embeddings)? the design-import pipeline; the local-AI goals (G12/G13); cluster (G7/G17); lock-in / migration cost if we adopt now then pivot.

## Verdict

_(synthesized after all 3 BGs return — GO only if all clean; else report the blocking downsides)_

## Cross-references

- RH-020 part 3 (the slam-dunk)
- `feedback_bottleneck-restriction-chant` (the no-new-bottleneck constraint)
- `reference_context-folder` memory (context/ holds the client Drive export — privacy)
- G7/G12/G13/G17 (the planned future systems to stress-test against)
- `snapshot-baseline-staleness-check.sh` (the thing it claims to retire)

## Verdict (synthesized from all 3 adversarial BGs, 2026-05-29)

**The conditional nod's bar ("no downsides after thorough consideration") is NOT met. DEFER recommended.** The adversarial hunt found real downsides — most critically a client-data privacy risk — and falsified one of RH-020's three justifications.

### What the skeptic BGs found

1. **The only viable server has a privacy landmine.** Of the two candidates, `zilliztech/claude-context` is REJECT (paid API + cloud upload). The real local one, `claude-context-local`, has confirmed-true local/zero-cost/no-telemetry claims BUT: **no `.gitignore` parsing**, markdown is an embeddable language, and the README literally tells you to say "index this codebase" — which would ingest `context/markdowns/` AND the client's Drive `.md` files into a local vector store. No exclude param; mitigable only by always passing a narrowing include-glob (opt-in discipline the UX discourages). Plus solo-maintainer / 6.5mo-stale / beta / unpinned-deps / ~1GB RAM per session / subagent-reach-not-guaranteed.

2. **RH-020's #1 justification is FALSE.** It claimed the index "retires `snapshot-baseline-staleness-check.sh`." That hook is about Playwright VISUAL PNG baselines, not the code map — RH-020 conflated two unrelated "baselines." One of its three ★★★★ pillars is bogus. The index retires zero existing staleness and ADDS an embedding-staleness window.

3. **The high-value use IS the risky one.** `src/` is 42 files / 520KB — where "grep is usually fine" (RH-020's own words). The compounding payoff requires indexing `context/` = 6GB incl. the client Drive export + a DIFFERENT client's source in `codebases/`. So: safe scope = low value; high value = privacy-risky.

4. **Future fleet multiplies the risk ×4.** Under G17 (M2 + 2 Pis), a `context/`-index either pushes client-data embeddings into git history ("git is the bus") or rebuilds them on 4 machines (incl. a Pi on a shelf). Multi-orchestrator indexes also diverge silently (M2 vs M4 return different answers — a confident-wrong-hit bug class grep can't have).

5. **Lock-in vs the local-AI roadmap.** G12/G13/RH-017 plan a LiteLLM/Ollama RAG stack that will OWN embeddings. Adopting `claude-context-local` now stands up a SECOND parallel RAG stack G12 must later consolidate. `src/`-only keeps the future migration trivial; `context/`-scale makes it 6GB-expensive.

### Recommendation: DEFER (don't adopt now)

- The src/-only version is low-value (grep handles 42 files fine; modest help only on the design-import DIFF phase).
- The high-value version (index context/) is privacy-risky across the planned fleet.
- G12 local-AI will own embeddings anyway — adopting now = a 2nd RAG stack to later rip out.
- **Cleanest call: revisit when G12 forces the RAG-stack decision.** At that point, build it INTO the local-AI stack with proper scoping, instead of bolting on a solo-maintainer beta tool now.

If the user still wants a taste: `src/`-only, hard-exclude `context/`, `HF_HUB_OFFLINE=1`, treat as disposable non-authoritative tooling, 6 day-1 fences (in the findings files). Install remains the user's gesture — but my honest advice is **not yet.**

status → verified_complete (verify criterion = thorough downside hunt across current+future, verdict delivered)
