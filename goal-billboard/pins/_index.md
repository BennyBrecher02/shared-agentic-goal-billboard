---
name: Pins — narrowed-but-not-shipped lane
description: New goal-billboard lane for work we narrowed on correctly but didn't ship. Closes the drift gap between "we identified this" and "we did this." Each pin has explicit resume criteria.
metadata:
  type: pin-index
  created: 2026-05-28
  origin_session: marathon plow 2026-05-27 → 2026-05-28
  lane_status: bootstrapping
---

# Pins — narrowed-but-not-shipped lane

When work converges to "we should do X" but X doesn't ship in the same session (rate limit / dependency / cost / pivot), it gets pinned here instead of evaporating. Pins persist across sessions. Each pin has explicit RESUME CRITERIA so we know when to unpin.

## Lifecycle

```
pinned → resumable (criteria met) → claimed (BG dispatched or work started) → landed (achieved)
                                  ↓ (after 30d no progress)
                                  abandoned
```

## Auto-pin triggers (when implemented as a hook)

- BG ghost detected → auto-pin original work
- Rate-limit hit mid-plow → auto-pin all in-flight work
- User says "park that" / "put a pin in" → auto-pin
- Dispatch list of options where we plow one → others auto-pin

## Active pins (2026-05-28 bootstrap — 16 items from marathon session)

Ranked by `leverage_g2 = (impact × downstream_unblock) / (effort + risk)`.

| Pin | Title | Leverage | Cost | NS path | Resume criteria |
|---|---|---|---|---|---|
| [P-001](P-001-wave3-audit-completion.md) | Wave 3 audit completion (5 missing devices) | **16.7** | ~2.5M tok | DIRECT G2 close | Budget refreshed AND PIL pre-shrink reliable per device |
| [P-002](P-002-consolidator-g2-close-verdict.md) | Consolidator BG → formal G2-CLOSE verdict | **14.0** | ~500k tok | DIRECT G2 close | After P-001 OR run on existing 6 evaluators |
| P-003 | Mobile Phase 6 real-iPhone test execution | 8.0 | 50min user wall | DIRECT G2 close (client signoff gate) | User has physical iPhone available |
| P-004 | iOS Simulator sweep via orchestrator | 8.0 | low tok | Layer-3 e2e | Boot iOS Sim → fire orchestrator (I can autonomously now) |
| P-005 | Android Emulator sweep via orchestrator | 8.0 | low tok | Layer-4 e2e | Boot Android Em → fire orchestrator (I can autonomously now) |
| P-006 | Hero refactor patch application | 1.0 | ~5min apply | Tangential to G2 (perf, not visual) | Next perf cycle |
| P-007 | 4 substrate audit BG ghosts (A12/A23/A43/hook-overhead) | 0.5-1.0 | ~640k tok | Substrate-divergent | Budget refresh (low priority) |
| P-008 | A69 Phase 3 — 19 remaining hooks wave-wire | 7.0 | ~80k tok | Substrate-furthering | Test Phase 1 stable 7 days |
| P-009 | Daemons #6/#7/#8 implementation | 3.0 | ~150k tok | Substrate | 7d of D2-D5 production data first |
| [P-010](P-010-local-ai-week-1-install.md) | Local AI Week 1 install (Ollama + CCR + LiteLLM) | **10.0** | $0 + ~3.5hr user setup | Cost-substrate; ends token leak | User has weekend + brew install ollama (your gesture) |
| P-011 | Cluster M2 node registration | 6.0 | ~50k tok script + user SSH | Substrate; doubles BG capacity for text-only | M2 SSH enabled (your gesture) |
| P-012 | 2 stale memory findings (MEMORY.md phantom + verified-counts refresh) | 1.5 | ~10k tok | Substrate cleanup | Anytime |
| P-013 | Research room interactive chamber | **12.0** | ~200-250k tok | Substrate; ENDS recurring brief-cost | Pick when budget can absorb |
| P-014 | Cardiovascular BG dispatch pump | 6.0 | ~150k tok | Substrate; structural A63 fix | After P-008 stable |
| P-015 | BG ghost auto-detection (dual-signal) | 5.0 | ~50k tok script | Substrate; prevents recurrence | Anytime |
| P-016 | Matrix-discipline hard 5-layer rule (`feedback_device-matrix-discipline.md`) | 4.0 | ~10k tok edit | Anti-drift discipline | Anytime |

## Resume order recommendation (when budget refreshes)

1. **P-016** (10k) — tiny, anti-drift fix. Do first.
2. **P-002** (500k) — run consolidator on existing 6 evaluators → get formal G2 verdict on what we have. NO new BG dispatch.
3. **P-004 + P-005** (low tok) — fire iOS Sim + Android Em sweeps to close layers 3-4 of true e2e. Now agent-runnable per settings lift.
4. **P-001** (2.5M) — restart wave 3 audit BGs once budget supports
5. **P-013** (200k) — research room implementation. Highest substrate compounding leverage after G2 closes.
6. **P-010** (free + your gestures) — local AI Week 1 parallel track
7. **P-008** (80k) — A69 Phase 3 hook wave-wire
8. Everything else by leverage as budget allows

## 2026-06-01 / 06-07 additions (the reconciled real-file lane)

These rows are backed by ACTUAL pin files on disk (the bootstrap table above is the original
marathon snapshot — most of its P-003…P-016 rows are fileless inline entries that were never
promoted to files; they remain as a historical record, not as live file pointers).

| Pin | Title | State | Resume criteria |
|---|---|---|---|
| [P-017](P-017-pixel-fold-capture-corruption.md) | Pixel_Fold capture corruption (emulator screencap → invalid PNG) | deferred (user-dropped from matrix) | Deferred-until-asked; do NOT pick up autonomously |
| [P-018](P-018-cursor-with-agents-integration-research.md) | Cursor-WITH-agents integration research | pinned | Benchmark shows Composer worth deeper integration, OR user unpins |
| [P-019](P-019-monkey-svg-animated-chamber.md) | Animated-monkey SVG chamber (living monkeys + banana-feeding + glass) | pinned | User wants to build it, OR unpins (under G18) |
| [P-020](P-020-zoomed-out-board-with-threads.md) | Zoomed-out board mode (birds-eye) + red-thread constellation | pinned | User wants the zoomed-out/constellation view, OR unpins (under G18) |
| [P-021](P-021-undecided-points-awaiting-user-call.md) | Undecided points awaiting user call (7, swap-v1) | pinned | User reviews + decides each |

> ✅ **Index drift RECONCILED 2026-06-07** (was flagged by the 2026-06-01 pin/todo-integrity audit).
> The dir holds **P-001 / 002 / 010 / 013 / 017 / 018 / 019 / 020 / 021** — all nine now have index
> rows (P-001/002/010/013 in the bootstrap table above; P-017–P-021 in this section). P-017–P-020
> were the previously-unindexed files; they are now linked. The bootstrap table's P-003/004/005/006/
> 007/008/009/011/012/014/015/016 rows are **fileless** (marathon-snapshot inline entries, never
> promoted to `P-*.md` files) and are retained as a historical record only. See
> `audits/findings/pin-todo-system-integrity-audit-2026-06-01.md`.

## Cross-references

- `context/markdowns/plans/TODO-after-weekly-limit-resets.md` — master TODO doc (this pin lane is the operational expression of that)
- `feedback_goal-billboard.md` — sibling pattern (active goals lane)
- `feedback_never-miss-an-idea.md` — A40 user-asks closure tracking; pins extend that to agent-narrowed work
- `feedback_decision-handling-discipline.md` — pins are the structural answer to "user explicitly narrowed → never shipped → drift"

## What this lane is NOT — the routing rule

> **CANONICAL SOURCE → [`../ARTIFACT-ROUTING.md`](../ARTIFACT-ROUTING.md).** The full 13-type
> dictionary, the "what IS this thing → where it lives" decision router, the journal↔goal-billboard
> split, and the promotion gates now live there as the **single source of truth**, machine-enforced by
> `scripts/routing-linter.sh`. The table below is a **local quick-reference** for the pins↔ideas
> near-twin collision only (they share a cloned parser — flagged in
> `audits/findings/pin-todo-system-integrity-audit-2026-06-01.md`); if it ever disagrees with
> `ARTIFACT-ROUTING.md`, **the routing doc wins.** (This pointer is what keeps the rule from drifting:
> one definition, many pointers.)

**The one-question router** — ask: *what IS this thing?*

| It is… | → home | why not a pin |
|---|---|---|
| agent-converged work, **paused** mid-flight, resumable | **`pins/`** (here, `P-*`) | — it IS a pin |
| a **user-facing idea** awaiting greenlight (surface-only) | **`ideas/`** (`I-*`) | no converged work yet |
| a **multi-session initiative** with an exit path | **`active/`** goals (`G-*`) | bigger than a paused task |
| a **bug** / defect | **`bug-billboard/`** | problem-grained, not paused-work |
| a **reflective check** ("did we do X right?") | **`audits/`** (`A-*`) | a question, not work |
| a **net-new proposal** (parked) | **`proposals/`** | not yet converged |
| a **this-session ephemeral task** | the **Task tool** (`TaskCreate`) | not persisted across sessions |

**Pins specifically = converged-but-paused WORK** (we decided X, didn't ship it). No converged work yet → it's an idea, not a pin. *(For everything beyond pins-vs-ideas — plan vs goal, finding vs idea, the promotion gates — see [`../ARTIFACT-ROUTING.md`](../ARTIFACT-ROUTING.md).)*

## Next mechanism work (when budget refreshes)

- Auto-pin Stop hook (writes to this lane when ghost/rate-limit detected)
- Dashboard render of pin lane (alongside goals + audits)
- "Resume this pin" action that dispatches the original BG prompt
- 30-day staleness sweep → abandoned/

---

**Bootstrap complete: 16 pins captured from marathon session 2026-05-27/28. Lane is now load-bearing.**
