---
idea_id: I-012
title: Token-saving adoptions (the 5 RH-011 candidates) — protect the weekly limit without altering workflows
lifecycle: greenlit
serves_northern_star: G17   # peak-throughput / cost infrastructure; advisory
source: floated-ideas-audit-2026-06-07
source_ref: context/markdowns/goal-billboard/audits/findings/floated-ideas-audit-2026-06-07.md
proposed_as: context/markdowns/goal-billboard/proposals/token-savings-adoption-2026-06-08.md  # reciprocal of that proposal's source_idea (B1 backfill — derived from the proposal's explicit source_idea: this idea)
leverage: high
related: research/rabbit-holes/token-cheap-out-pass-2/ (RH-011) · scripts/hooks/token-budget-check.sh (the only piece built so far)
created: 2026-06-07T00:00Z
updated: 2026-06-07T00:00Z
---

# I-012 — Token-saving adoptions: the 5 RH-011 candidates (greenlight-ready)

> **The user's words (0d739f65, repeated 2 sessions):** *"how did they maximize token cost saves because my
> weekly limit is something i wanna consider without altering our workflows."* This was a **stated motive**,
> not idle curiosity — and the research landed while the adoptions it surfaced never got built.

## What it is

The RH-011 token-cheap-out research (`context/markdowns/research/rabbit-holes/token-cheap-out-pass-2/`)
surfaced **5 concrete adoption candidates** to protect the weekly token limit *without changing how we work*:

1. **Batch API for recurring BGs** — route non-interactive recurring background jobs through the Anthropic
   Batch API (the #1 saver; recurring BGs are the dominant token sink).
2. **Tool-result compression** — trim/summarize large tool outputs before they re-enter context on the next
   turn (the cached prefix is taxed every turn; bloated tool results are pure waste).
3. **Stable-prefix audit** — verify the cached prefix (auto-memory + pinned protocols) stays byte-stable so
   prompt-caching actually hits instead of silently re-billing on prefix churn.
4. **Multi-model tier hint** — a routing hint that lets cheap/volume work land on a cheaper tier while hard
   reasoning escalates (respecting the model-routing policy — subagents inherit by omitting `model`).
5. **Weekly-window visibility surface** — a dashboard/CLI readout of weekly-window token burn so the limit
   is *seen* before it's hit (today only `plans/TODO-after-weekly-limit-resets.md`, a parking note, exists).

## Why it matters

- It serves the user's **explicitly stated constraint** (the weekly limit) — the highest-intent class of
  idea: he told us the motive directly.
- **The research is done; the adoption is not.** Beyond the pre-existing pre-flight check
  (`scripts/hooks/token-budget-check.sh`), **none of the 5 are built** — no Batch API integration, no
  tool-result compression script, no stable-prefix audit, no weekly-window surface. The savings he wanted
  remain trapped as research.
- The constraint "without altering our workflows" is satisfiable: all 5 are infrastructure that sits beneath
  the workflow (batching, compression, caching, routing, visibility) rather than changing how the agent
  works.

## Scope (a SERIES, sequenced by ROI/risk)

Treat as a series, not one big build. Sequence: **#1 Batch API → #5 weekly-window surface → #2 tool-result
compression → #3 stable-prefix audit → #4 multi-model tier hint.** #1 is the biggest saver; #5 is
low-risk/high-visibility; #2–#4 follow. Each adoption is independently greenlightable. Before building #1,
read the `claude-api` skill (model ids, Batch API mechanics, pricing) — the canonical Claude/Anthropic
reference — rather than coding from memory.

## Concrete first step (the exact next action — greenlight-ready)

Scope **Batch API for recurring BGs (#1)** as a goal-billboard **proposal**: read the `claude-api` skill for
Batch API mechanics + pricing, enumerate which recurring BGs (the daemons / research-furthering refills /
nightly jobs) are non-interactive enough to batch, estimate the weekly-token delta, and write the proposal
with that ROI number. It's the #1 saver and recurring BGs are the dominant sink, so it's the right first
domino; the other 4 follow as their own proposals.

**Leverage: HIGH** — directly serves the user's stated weekly-limit motive; research already paid for.
**Awaiting greenlight** (surface-only by contract; the user owns graduation to a build).
