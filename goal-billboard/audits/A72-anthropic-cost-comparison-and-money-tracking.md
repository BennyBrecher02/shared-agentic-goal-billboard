---
audit_id: A72
title: Anthropic cost comparison (Team plan vs 2× individual Pro/Max) + money-tracking system design
status: in_progress
research_completed: 2026-05-28T16:20Z
recommendation: 2× Max 20x ($400/mo) on separate accounts — beats Team Premium ($500/mo min) on both cost AND total usage for 2-active-developer situation
catalogued: 2026-05-28T15:53:00Z
priority_when_run: P1
estimated_effort: small (research + comparison table + tracking-system proposal)
trigger: |-
  2026-05-28T15:50Z — user explicit while answering multi-orchestrator subscription path: *"im gonna create a new account but start a new audit for price comparison between team approach vs two full pro/max plans and also find some way to start keeping track of money spent in this system and maybe that can be kept track of in the dashboard somewhere"*
deferral_reason: NONE — user explicit catalog request
related_goals: [G2 (cost-efficient peak throughput)]
related_plans: [plans/multi-orchestrator-new-subscription-plan.md (parent), plans/m2-ownership-tradeoff-analysis.md (sibling)]
serves_northern_star: G2
related_refs:
  - feedback_verified-counts (token scale baseline)
  - plans/TODO-after-weekly-limit-resets.md (the throughput-fight plan)
findings:
  - team-vs-2x-pro-known-as-of-this-audit: Anthropic Team plan (per public pricing as of last refresh) is $30/seat/month for 5+ seats minimum; Pro/Max individual is $200/month each. 2× Pro/Max = $400/mo; Team-with-2-seats not offered (5-seat minimum). Verify with Anthropic billing before committing.
  - usage-pot-shape: Pro/Max has weekly limit; Team usage pools across seats but admins set per-seat caps. 2× individual gives 2 fully independent weekly limits; Team gives 1 shared pool with caps.
  - hidden-cost-overage: Current path (1 Pro + $70 overage) is the comparison baseline. Overage rates change; verify directly.
  - tracking-substrate-gap: No system today logs Anthropic spend. The dashboard renders token-cost (effective tokens via cache_create + in + out) but doesn't multiply by USD-per-token. Would need a tracking script + dashboard widget.
---

# A72 — Anthropic cost comparison + money-tracking system

## The user's actual question

> *"start a new audit for price comparison between team approach vs two full pro/max plans and also find some way to start keeping track of money spent in this system and maybe that can be kept track of in the dashboard somewhere"*

Two distinct deliverables:

1. **Cost comparison**: which subscription path is best?
2. **Money-tracking**: how do we capture USD spent, ideally in the dashboard?

## Part 1 — Cost comparison (RESEARCH UPDATE 2026-05-28T16:20Z)

### Verified current pricing (web research via 3 parallel queries)

| Option | Monthly cost | Pot size per seat (Pro-equivalent) | Min seats | Critical limitation |
|---|---|---|---|---|
| **Current** — 1× Max 20x + overage | $200 + ~$70 overage | 20× Pro / 1 person | 1 | Overage costs unbounded |
| **2× Max 20x** — separate accounts | **$400 ($200×2)** | 20× Pro / each account, fully isolated | 2 (one each) | Two logins |
| **Team Premium** — Claude Code requires Premium tier | **$500/mo annual ($100×5) / $625/mo monthly ($125×5)** | 6.25× Pro / seat | **5 seats MINIMUM** | 3 unused seats; usage DOES NOT pool across seats |
| Team Standard | $100/mo annual ($20×5) | 1.25× Pro / seat | 5 seats | **DOES NOT include Claude Code** — Premium tier required |

### The killer finding for your 2-person situation

**Total Pro-equivalent usage by option** (for 2 active developers):

| Option | Total Pro-equivalent | Cost | Cost-per-Pro-unit |
|---|---|---|---|
| 2× Max 20x | **40× Pro** (20 + 20) | $400 | $10/Pro-unit |
| Team Premium 5-seat (annual) | **31.25× Pro** (6.25 × 5; 3 idle) | $500 | $16/Pro-unit |
| Team Premium 5-seat (monthly billing) | 31.25× Pro | $625 | $20/Pro-unit |

**2× Max 20x wins on BOTH dimensions**: cheaper ($400 < $500) AND more total usage (40× > 31.25× Pro-equivalent).

### Where each option's limitations matter

**Pro/Max (separate accounts) limitations:**
- ❌ Two logins to manage (mild annoyance)
- ❌ No single invoice (tax/expense tracking adds work)
- ❌ Can't share limits — if one account caps, other can't help (BUT you're getting separate accounts SPECIFICALLY to get independent pots, so this is a feature not a bug)

**Team Premium limitations (the killers for your situation):**
- ❌ **5-seat minimum**: $500/mo MINIMUM even though you'd only use 2 seats → 60% of spend is on idle seats
- ❌ **Usage does NOT pool across seats** (per Anthropic support docs) — same isolation as 2 separate accounts, but you pay for 5 isolated pots and only use 2
- ❌ **Premium tier required** for Claude Code (Standard at $20/seat doesn't include Code — so the "Team Standard for $100/mo" idea is invalid for our use)
- ❌ Admin console overhead — minor but real for a 1-person org

**Team's actual advantages** (where it would win):
- ✅ Single invoice (matters for businesses with accounting; doesn't matter for personal billing)
- ✅ Centralized seat management (matters at 5+ developers; doesn't matter at 2)
- ✅ Usage credits can be enabled org-wide (matters for irregular bursts; 2× Max 20x equivalent already covers your current burn)

**Net: for 2 active developers on personal billing, 2× Max 20x is the unambiguous winner. Team plan only makes sense at 5+ active developers.**

### Definitive recommendation

**Buy a second Max 20x subscription on a new Anthropic account.** Specifically:
1. Create new account with separate email (recommend `<your-name>+m2@<domain>` if your email provider supports plus-aliases; otherwise a fresh address)
2. Subscribe to Max 20x ($200/mo) on the new account
3. Total monthly: $200 (existing main) + $200 (new M2) = **$400/mo** — $100/mo SAVINGS vs Team Premium at $500/mo minimum

Compared to current path ($200 + ~$70 overage = ~$270/mo), the new-account path costs an extra ~$130/mo BUT gives:
- Full Max 20x pot for M4 with no overage anxiety
- Full Max 20x pot for M2 (the parallel orchestrator)
- ~40× Pro-equivalent total usage (vs ~20× + overage-capped today)

### Honest caveats still apply

- Anthropic policy can change; verify current pricing at https://claude.com/pricing before paying
- Discount programs (annual prepay, startup credits, founder programs) may apply — ask Anthropic billing if eligible
- If you anticipate hiring 3+ collaborators in next 6 months, Team becomes cheaper at 5 active users — re-evaluate then

### Sources

- https://support.claude.com/en/articles/11145838-use-claude-code-with-your-pro-or-max-plan
- https://support.claude.com/en/articles/9266767-what-is-the-team-plan
- https://support.claude.com/en/articles/9267289-how-is-my-team-plan-bill-calculated
- https://claude.com/pricing
- https://lord.technology/2026/03/28/claude-team-premium-vs-max-plans-usage-limits-pricing-and-which-to-choose.html

## Part 2 — Money-tracking system design

User wants: "keep track of money spent in this system and maybe that can be kept track of in the dashboard somewhere."

### What we already track

The session-token tracker (`scripts/token-curve-tracker.sh` family) records per-session effective tokens (in + out + cache_create) in JSONL form. That's the **input** to USD calculation.

### What's missing

- USD-per-token conversion (token type × Anthropic price = USD)
- Per-subscription bucketing (which account's pot did this burn from?)
- Daily/weekly/monthly rollups
- Dashboard widget rendering the spend

### Proposed system (small)

**Substrate**:
- `.claude/cache/anthropic-pricing.json` — current per-million-token rates (input/output/cache_create/cache_read), refresh manually when Anthropic changes prices
- `scripts/cost-tracker.py` — reads existing token JSONL, multiplies by pricing, emits `.claude/cache/cost-ledger.jsonl` (per-session USD cost, timestamped)
- `scripts/cost-rollup.py` — aggregates ledger by day / week / month
- Dashboard widget — reads rollups, renders "Today: $X · This week: $Y · This month: $Z"

**Multi-orchestrator dimension** (post-M2):
- Each agent (main / m2) gets its own subscription_id field in ledger
- Per-account cost rollups separately
- Dashboard shows side-by-side: "Main account: $X / week (cap $Y) · M2 account: $A / week (cap $B)"

**Implementation tier**:

| Tier | Scope | Effort |
|---|---|---|
| **Tier 1 — minimal** | pricing.json + cost-tracker.py emitting daily totals; CLI-only readout | 1-2 hours |
| **Tier 2 — dashboard hooked** | + cost-rollup.py + dashboard widget | +2-3 hours |
| **Tier 3 — multi-account** | + subscription_id per session + per-account rollup | +1-2 hours (after multi-orchestrator goes live) |

### Recommendation: build Tier 1 now, Tier 2 with the dashboard Full upgrade, Tier 3 with M2 bring-up

## Cross-references

- `plans/multi-orchestrator-new-subscription-plan.md` — parent context
- `plans/m2-ownership-tradeoff-analysis.md` — sibling planning
- `feedback_bottleneck-restriction-chant` — cost is the constitutional bottleneck; this audit serves it
- `feedback_verified-counts` — scale baseline that money-tracking would multiply against pricing
- Dashboard upgrade work (user chose Full path) — natural integration point for Tier 2

## Sentinel artifacts when A72 lands fully

- This audit (documents the question + answers)
- `plans/anthropic-cost-tracking-implementation.md` — Tier 1/2/3 build plan (next turn or BG)
- `.claude/cache/anthropic-pricing.json` skeleton (next turn)
- `scripts/cost-tracker.py` (Tier 1 implementation; future turn)
- Dashboard widget spec (Tier 2; sequenced with Full dashboard upgrade)

## What I'm NOT proposing this turn

- No pricing fetched from Anthropic (would be guessing; user verifies directly)
- No account creation
- No script written yet (just the audit + design)
- No dashboard widget built (sequenced with the dashboard Full upgrade)
- No commitment to which subscription path — user makes that call with verified facts

