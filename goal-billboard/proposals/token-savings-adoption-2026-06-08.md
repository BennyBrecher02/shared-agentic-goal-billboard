---
title: "Token-savings adoption — the 5 RH-011 candidates, costed & greenlight-ready"
status: proposed
serves_northern_star: G17   # peak-throughput / cost infrastructure; advisory to G2
source_idea: context/markdowns/goal-billboard/ideas/I-012-token-saving-adoptions-rh011.md
source_research: context/markdowns/research/rabbit-holes/token-cheap-out-pass-2/  (RH-011; synthesis.md is the payoff doc)
source_audit: context/markdowns/goal-billboard/audits/findings/floated-ideas-audit-2026-06-07.md  (Pass-4 HIGH-leverage drop #3)
claude_api_skill_read: true   # Batch API mechanics / prompt-caching / pricing pulled from the claude-api skill, not memory
logged: 2026-06-08
constraint: "workflows unchanged — every candidate sits BENEATH the workflow (substrate-side)"
weekly_limit_aware: true
gating: "NOT building now. Each candidate is independently greenlightable; the user owns graduation to a build."
---

# Token-savings adoption — the 5 RH-011 candidates (costed proposal)

> **The user's words (msg 0d739f65, repeated across 2 sessions):** *"how did they maximize token cost
> saves because my weekly limit is something i wanna consider **without altering our workflows** tho."*
> This was a **stated motive**, not idle curiosity. RH-011 did the research and surfaced 5 adoptions.
> **The research landed; zero were built** (beyond the pre-existing `scripts/hooks/token-budget-check.sh`
> pre-flight). This proposal costs each of the 5 so the user can greenlight builds. **It builds nothing.**

---

## The reframe RH-011 delivered (read this first)

The biggest weekly-limit lever is **not** squeezing the prompt — Anthropic auto-caches it generously.
The win is **routing the work**: recognizing which work doesn't need synchronous frontier-model treatment
and **shifting** it (to batch, to a cheaper tier, to off-peak) rather than compressing it. Two of the five
candidates (#1 Batch, #4 tier-hint) are pure *dispatch shifts*; the other three are caching/compression/
visibility hygiene. All five satisfy the binding constraint: **the user prompts the same way and sees the
same agent behavior** — only the substrate underneath changes.

### The structural fact that shapes every candidate (the honest lead)

Our recurring background work runs as **Claude Code agent sessions** — the 5 launchd daemons
(`com.agentic-os.daemon{2,3,4,5,10}`, wired in `scripts/daemons/` + `~/Library/LaunchAgents/`) and the
`research-furthering-daemon.sh`, plus in-session BG subagents. **None of these speak the raw Anthropic
Messages/Batches API.** The Batch API (`client.messages.batches.create`) is a *direct-API* surface: it takes
a list of self-contained `{custom_id, params}` request objects, runs them async at **50% off**, and returns
results in <24h (most <1h). You cannot point it at "a Claude Code BG" — you point it at concrete prompt
payloads.

**Implication:** adopting Batch (candidate #1) means standing up a **second, direct-API dispatch path**
alongside Claude Code — an API key in the environment, a small dispatch+poll+archive script, and a clear
rule for *which* recurring jobs are re-expressible as self-contained batch requests. That is the real effort
and the real risk (double-billing if a job runs in *both* paths), and it is why #1 is medium-effort, not
small. The savings math is clean; the plumbing is the work.

---

## LEAD CANDIDATE — #1 Batch API for recurring BG dispatches

**The audit's #1, and RH-011's highest-leverage lever.** Recurring reactionary/consolidation work is the
dominant token sink and is **24h-tolerant by nature** — a perfect batch fit.

### What it saves (the math, from the `claude-api` skill + RH-011 batch deep-dive)

- Batch API = **50% off ALL models, ALL token types**, <24h SLA, **separate rate-limit pool** (does NOT
  draw down our standard quota — directly relevant to the weekly limit), up to ~100k requests / 256 MB per
  batch. Results retrievable for 29 days.
- It **stacks multiplicatively with prompt caching**: `cache_read` is ~0.1× base, batch is 0.5× →
  **0.5 × 0.1 = 0.05× = 5% of base** on batch+cache-eligible tokens (a 20× reduction on that segment).
- **Rough weekly delta: 10–25%** of total spend (RH-011 HIGH confidence on the ~10% floor). The floor comes
  from: if ~20–30% of spend is batch-eligible recurring work, batch alone (50%) yields ~10–15%; cache
  stacking on the cacheable share pushes the upper bound. The single unknown is *how much* of our spend is
  batch-eligible — which the first build-step measures rather than guesses.
- **Dollar frame** (Opus 4.8 = $5 in / $25 out per 1M; the model the daemons inherit): every 1M batched
  input tokens drops from $5.00 → **$2.50**; with a cache-resident prefix the cached portion drops to
  ~$0.25. The save scales linearly with batched volume — concrete number pending the inventory.

### Effort: MEDIUM (~5–10h) — plumbing, not prompt work

A direct-API path is net-new: API-key handling (env var, never committed — gated, see below), a dispatch
script (`client.messages.batches.create`), a poll loop (`batches.retrieve` until `processing_status==ended`),
and result-archive into our existing substrate (memory-mirror / audit-findings / bug-billboard inbox).

### Risk: MEDIUM

- **Double-billing** — a job that runs in *both* the Claude Code daemon AND the batch path bills twice and
  wastes more than it saves. Mitigation: the batch dispatcher and the daemon for a given job are mutually
  exclusive (flip the daemon off when its work moves to batch).
- **Quality drift on delayed work** — batch results land hours later; only truly post-hoc work qualifies.
  Mitigation: the eligibility rule below is strict (post-hoc only; nothing in-session).
- **Poll discipline** — a dispatched-but-never-polled batch = wasted spend. Mitigation: poll-on-cron +
  archive-on-complete is part of the build, not optional.

### Which recurring jobs are batch-eligible (concrete, named against our real daemons)

| Daemon / job | Today | Batch-eligible? | Why |
|---|---|---|---|
| `daemon2` memory-consolidator | launchd | **YES** | post-hoc; <24h fine; nightly cadence already |
| `daemon3` audit-catalog-scan | launchd | **YES** | periodic triage; not interactive |
| `daemon4` steering-log-compact | launchd | **YES** | retrospective compaction |
| `daemon5` bug-billboard-groom | launchd | **YES** | inbox→master merge; post-hoc |
| `research-furthering-daemon` | launchd | **MAYBE** | only when its output isn't blocking the next session; idle-fill research is a strong fit |
| `daemon10` heartbeat-tier-low | launchd | **NO** | liveness signal; must run on schedule, not <24h |
| In-session reactionary BGs | CC BG | **NO** | feed back into ongoing work; user waiting |
| Live audit/scrutiny BGs | CC BG | **NO** | results wanted in <5 min |

### THE EXACT FIRST BUILD-STEP (greenlight-ready)

> **Build a batch-eligibility inventory + a one-job end-to-end pilot — NOT the full router.**
> 1. **Measure** (this answers the only open unknown): run `scripts/analyze-token-burn.py` /
>    `analyze-burn-workflows.py` to get the **share of weekly token spend attributable to the 4 YES daemons
>    above**. That number sets the real ROI (replaces the 10–25% estimate with a measured figure).
> 2. **Pilot ONE** — take `daemon2` (memory-consolidator), the cleanest post-hoc job. Re-express its work as
>    a self-contained `client.messages.batches.create` request (the consolidation prompt + the inputs it
>    reads). Dispatch it via a new `scripts/batch-dispatch.sh` (direct API), poll to completion, and write
>    the result through the existing memory-mirror sync path.
> 3. **Verify** the batched output matches what the daemon produces today (structured diff), and confirm the
>    daemon is OFF for that job while the batch path owns it (no double-bill).
>
> One measured ROI number + one working end-to-end batched job. The other 3 YES jobs follow as the same
> pattern once the pilot proves the plumbing. Spawns **A44** (batch-eligible BG inventory + dispatch design).

### Safe-to-build-now vs gated

- **Build-now:** the inventory/measurement (step 1) — read-only, zero risk.
- **Build-now:** `scripts/batch-dispatch.sh` + poll + archive (steps 2–3) — new script, no settings/launchd edit.
- **GATED (user-only):** putting the Anthropic **API key** into the environment (a credential decision the
  user owns); **disabling a launchd daemon** to hand its work to batch (touches `~/Library/LaunchAgents/` +
  `launchctl` — per the non-negotiables, the user applies launchd changes by hand). The script can be built
  and dry-run-tested against a test key before either gate is crossed.

---

## The other four (costed; sequence after #1)

### #2 — Tool-result compression discipline + auto-spill wrapper

- **Saves:** ~8–20% weekly (depends on tool-output share of session cost; the cached prefix is taxed every
  turn, so a bloated tool result re-bills on each subsequent turn until it scrolls out — compounding waste).
  SWE-agent's bounded-output ACI delivered a 3.3× task improvement on the same principle.
- **Effort:** SMALL–MEDIUM (~3–6h): a `tool-result-compression-discipline.md` skill ref (head-tail + token-cap
  pattern) + an optional Bash wrapper that auto-spills large outputs to a tempfile with a structured
  truncation marker so the agent knows it can re-fetch.
- **Risk:** LOW — the marker keeps it transparent; the agent reads output normally and re-fetches if truncated.
  Only risk is over-aggressive truncation hiding needed data (the marker mitigates).
- **First build-step:** run **A43** — measure tool-output token share first (sizes the ROI), then write the
  skill ref; the wrapper follows if the share justifies it.
- **Gating:** **build-now** (skill ref + script are non-gated). Our `head -200` discipline is the partial
  version this completes.

### #3 — Stable-prefix audit + cache-discipline skill-ref extension

- **Saves:** ~3–8% weekly, but **mostly defensive** — if we've accidentally broken byte-stable prefix order
  (a `datetime.now()` in a prefix, a varying tool set, non-deterministic JSON), prompt-caching silently
  re-bills the whole prefix every turn. The audit either finds a regression (real save) or confirms clean
  (no-regression insurance). Verify via `usage.cache_read_input_tokens` — if it's ~0 across repeated
  prefixes, an invalidator is at work.
- **Effort:** SMALL (~2–4h): audit the actual session prefix order (auto-memory → pinned protocols → AGENTS.md
  → tools) against Anthropic's stable-first guidance; document/recommend any reorder; extend the existing
  `prompt-cache-budget-discipline.md` ref with the 1-hour-TTL economics + breakpoint-placement math +
  the *Don't Break the Cache* empirical 41–80% range.
- **Risk:** LOW — read-only audit + a doc edit.
- **First build-step:** run **A40** — diff the live prefix against the stable-first ordering and
  measure cache-read rate from recent session JSONLs.
- **Gating:** **build-now** (audit + skill-ref edit, non-gated).

### #4 — Multi-model tier hint in BG dispatch

- **Saves:** ~5–15% weekly — Haiku 4.5 ($1 in / $5 out) is ~5× cheaper input than Opus 4.8 ($5/$25); routing
  known-narrow recurring work (consolidators, triagers) to a cheaper tier captures that delta.
- **Effort:** SMALL–MEDIUM (~2–5h): add a `model_tier_hint` to BG dispatch metadata + pilot 3–5 narrow BG
  types on the cheaper tier with structured-output validation + auto-escalate-on-failure.
- **Risk:** MEDIUM — **two unknowns.** (a) **Silent quality degradation** is a bug class we can't afford —
  tier routing MUST have structured-output validation + an escalation path (never silent auto-downgrade).
  (b) **Does Claude Code even honor a per-BG model hint?** *Unverified* (RH-011 open question). **Must
  reconcile with `feedback_model-routing.md` (A74):** our policy is that subagents inherit the session model
  by **OMITTING** `model` — never `model:"opus"`. A tier hint that *forces* a model fights that policy, so
  this candidate is scoped to **recurring direct-API/daemon jobs** (where we control the call), NOT in-session
  subagents. If Claude Code doesn't expose per-BG model selection, #4 collapses into #1 (tier-route via the
  direct-API batch path — they compound).
- **First build-step:** run **A41** — a 1-job pilot: route `daemon3` (audit-catalog-scan) to Haiku via the
  direct-API path from #1, validate output quality with a structured check, measure the delta.
- **Gating:** **build-now** for the pilot script; the hint never overrides the in-session model-routing policy
  (that boundary is a hard constraint, not a gate). Sequenced **after #1** because it rides #1's direct-API path.

### #5 — Weekly-window-aware visibility + BG deferral

- **Saves:** ~2–5% standalone (it's **visibility**, not a new cost lever — it *prevents accidental weekly
  overshoot* and surfaces the limit before it's hit). **Compounds with #1:** the deferral substrate (defer
  batch-eligible BGs to off-peak when projected >95% of the weekly cap) IS the batch-routing substrate.
- **Effort:** SMALL–MEDIUM (~4–8h): extend `scripts/claude-5hr-window.py` from a 5-hr to a **7-day rolling**
  window; add a P95 7-day projection to the dashboard; add a burn-rate alarm to the standing-protocols hook
  output (reusing `burn-rate-governor.py`). Today only `plans/TODO-after-weekly-limit-resets.md` (a parking
  note) and the 5-hr window exist.
- **Risk:** LOW — visibility is value-additive; **NO hard enforcement** (per our discipline: warn
  aggressively, never block — the user retains override; the audit's own DECLINE list rules out user-blocking
  quota enforcement).
- **First build-step:** run **A45** — extend the 5hr-window script to compute a 7-day rolling total + P95
  projection; surface it in the existing token-budget SessionStart digest.
- **Gating:** **build-now** (extends existing scripts + dashboard; non-gated). Deferral notices are
  informational ("BG queued for batch; results by HH:MM"), not workflow changes.

---

## Ranked summary (for fast scan)

| Rank | Candidate | Saves (weekly) | Effort | Risk | Build-now vs gated | Spawns |
|---|---|---|---|---|---|---|
| **1** | **Batch API for recurring BGs** | **10–25%** | MEDIUM | MEDIUM | script build-now; **API key + daemon-disable GATED** | A44 |
| 2 | Tool-result compression + wrapper | 8–20% | S–M | LOW | build-now | A43 + skill ref |
| 3 | Stable-prefix audit + ref ext | 3–8% (defensive) | SMALL | LOW | build-now | A40 + skill-ref ext |
| 4 | Multi-model tier hint | 5–15% | S–M | MEDIUM* | build-now (rides #1); never overrides A74 | A41 |
| 5 | Weekly-window visibility + deferral | 2–5% standalone | S–M | LOW | build-now | A45 |

\* quality-validation + escalation required; per-BG model control is *unverified* — must pilot.

- **Combined (compounded, not summed): ~15–40% weekly reduction.** HIGH-confidence ~15% floor (from #1
  alone). Realistic mid-case ~20–30%.
- **Total effort across all five: ~16–34h** (1–2 weeks of substrate work).
- **Recommended sequence (by ROI/risk):** **#1 Batch → #5 weekly-window → #2 tool-compression →
  #3 stable-prefix → #4 tier-hint.** #4 last because it rides #1's direct-API path and carries the
  quality-degradation risk. Each is independently greenlightable.

## Workflow-impact assessment (the binding constraint)

| Candidate | Same prompts? | Same visible agent behavior? | New surface the user sees |
|---|---|---|---|
| #1 Batch | YES | YES | "BG queued for batch (saves N; results by HH:MM)" — info |
| #2 Tool compression | YES | YES | truncation marker in tool output — informational |
| #3 Stable-prefix audit | YES | YES | none directly (audit artifact) |
| #4 Tier hint | YES | YES (quality must hold) | none directly (substrate metadata) |
| #5 Weekly visibility | YES | YES | dashboard widget + burn alarm in reports |

**All five satisfy "without altering our workflows."** Nothing changes how the user prompts or what the
agent does in-session; the changes live in the substrate (batching, compression, caching, routing, visibility).

## What NOT to adopt (RH-011 explicit rejects — don't let these creep into a build)

- **Speculative parallel execution to save cost** — speculation BURNS tokens for *latency*; it is the WRONG
  weekly-limit lever (raises spend). Use only when time is the binding constraint.
- **Hard quota enforcement that blocks the user** — violates "warn, never block" + the workflow-unchanged
  constraint.
- **Silent auto-downgrade to Haiku without validation** — the bug class we can't afford (gates #4).
- **Manual `cache_control` breakpoints across all prompts** — Anthropic's auto-caching already uses a slot;
  manual additions can collide. Reserve manual breakpoints for the direct-API/batch path only.

---

## Status

`proposed` — costed and ready for the user to greenlight builds, candidate-by-candidate. **Nothing built
here.** Lead with #1 (Batch); its measurement step (the token-share inventory) is safe-to-run-now and is the
cheapest way to replace the 10–25% estimate with a real number before any plumbing or gated step.

---
## GREENLIT 2026-06-08 (Ben)
Approved to build. Sequence: post-A/B/C (or its safe read-only first-step — measure batch-eligible token share via analyze-token-burn.py). Gated bits flagged inside: Anthropic API key in env + the one daemon-disable.
