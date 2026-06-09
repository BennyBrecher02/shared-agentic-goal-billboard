---
audit_id: A34
title: "End-of-session realignment ritual — bundle-analyzer outputs become realignment moment that triggers master system audit; full analyzer + post-analysis hook ecosystem"
status: in_progress
catalogued: 2026-05-27T02:54:00Z
priority_when_run: P1
estimated_effort: very large (extends G11 Phase 2+ scope; new G14 goal; 13 analyzers + 13 post-analysis hooks + master audit framework)
trigger: 2026-05-27 03:10Z — user vision *"new hook at the end of every session after the bundle analyzer produces the Master session writeup we us it as a moment for realignment and then we would launch a master system audit, expand on all of this and all other ideas for the analyzer and post analysis hooks."* Builds on G11 Phase 1 POC (just landed) + A30 Phase 4 shutdown pipeline.
deferral_reason: NONE — directly extends G11; the pipeline pattern is established; this audit specifies its full scope.
related_goals: [G14, G11, G10]  # G14 NEW — realignment ritual + master audit; G11 = bundle analyzer; G10 = lifecycle
related_plans: [context/markdowns/plans/automation/end-of-session-realignment-ritual-plan.md]
serves_northern_star: G2
belongs_to_goal: G14
serves_guiding_light: G14
related_refs:
  - .claude/skills/agentic-quality-discipline/references/session-bundle-discipline.md (G11 Phase 1)
  - .claude/skills/reactionary/ (event-triggered counterpart)
  - context/markdowns/plans/automation/session-bundle-infrastructure-plan.md (G11 master)
  - context/markdowns/plans/automation/cross-tool-event-bus-plan.md (A31 Front 2 — substrate for analyzer hooks)
findings:
  - "Phase 2 LANDED 2026-05-27 02:55Z — 4 new sub-analyzers (#2-5 from the 13-analyzer table) + aggregator extension shipped; 5 analyzers run in parallel on the POC bundle in 0.14s wall-clock (vs the ≤30s Phase 3 budget), proving the parallelism model scales easily to 13. Per-analyzer outputs all between 2.5KB-8.5KB on a 13.23-hour session bundle with 180 BGs."
---

# A34 — End-of-session realignment + master system audit

## The vision (user framing)

> "new hook at the end of every session after the bundle analyzer produces the Master session writeup we us it as a moment for realignment and then we would launch a master system audit, expand on all of this and all other ideas for the analyzer and post analysis hooks"

**The pipeline gets a THIRD stage:**

```
Stage 1 (G10 / A30):   snapshot               (current state captured)
Stage 2 (G11 Phase 1+): bundle + analyzers    (journey captured + per-session insights)
Stage 3 (G14 / A34 NEW): master system audit + realignment   (cross-session synthesis + next-session priorities)
```

Stage 3 is where the agentic OS becomes self-improving across sessions, not just within them.

## Why this matters

G11 Phase 1 POC produced rich per-session insights (themes, audit changes, token curves). But:
- Insights stayed inside the session bundle
- No cross-session synthesis ("session N vs session N-1 vs N-2")
- No realignment digest (concrete next-session priorities)
- No master system audit (overall agentic-OS health scorecard)
- Master writeup (Front 0 today) was a one-off, not a recurring ritual

User's framing makes the writeup a **recurring realignment ritual** that triggers structural action (master system audit), not a passive retrospective.

## The 3-stage pipeline (full design)

### Stage 1 — Snapshot (existing; G10 / A30)
- `scripts/graceful-shutdown-snapshot.sh` — captures current state to JSON
- 1s wall-clock; atomic write; archive copy

### Stage 2 — Bundle + analyzers (existing G11 Phase 1; expanding to ≥13 in Phase 2+)
- `scripts/session-bundle-create.sh` — captures journey
- 13 sub-analyzers in parallel (see ANALYZER EXPANSION below)
- Aggregator merges → `insights.md`
- Bundle directory archived

### Stage 3 — Master system audit + realignment (NEW; G14 / A34)
- `scripts/master-system-audit.sh` runs AFTER analyzers complete
- Reads bundle insights + cross-session bundles (last N) + current goal billboard
- Produces `reports/master-system-audits/<session-end-ts>/` containing:
  - `realignment-digest.md` — top 5 next-session priorities
  - `health-scorecard.md` — overall agentic-OS health (0-100)
  - `drift-radar.md` — subsystems drifting (memory violated? skills aging? bugs piling?)
  - `cost-projection.md` — at current burn, when does what hit?
  - `ns-advancement.md` — how much did this session advance current NS?
  - `recurrence-across-sessions.md` — patterns repeating session-over-session
  - `recommendations.md` — concrete actions for next session
- Triggers 13 post-analysis hooks (see POST-ANALYSIS HOOKS below) — domino effects
- Output flows into next session's resume briefing

## ANALYZER EXPANSION — 13 sub-analyzers

Today's Phase 1 has 1 (themes). Phase 2 plan adds 4. A34 brings total to 13.

| # | Analyzer | Phase | Purpose |
|---|---|---|---|
| 1 | `analyzer-themes.py` | ✅ Phase 1 | Cluster topics from BG descriptions + audit titles |
| 2 | `analyzer-kpi-deltas.py` | ✅ Phase 2 | Compute KPI changes: ack-latency, parallel-capacity, pushback rate, skill health |
| 3 | `analyzer-token-efficiency.py` | ✅ Phase 2 | Tokens-per-deliverable analysis |
| 4 | `analyzer-recurrence-detection.py` | ✅ Phase 2 | Failure patterns repeated within session |
| 5 | `analyzer-domino-effectiveness.py` | ✅ Phase 2 | Per producer event, what consumers fired + outcome (depends on A31 Front 2 event-bus — JUST LANDED) |
| 6 | **`analyzer-goal-velocity.py`** | NEW Phase 3 | Per-goal: how much advanced this session? Phase changes? Stalled flagged. |
| 7 | **`analyzer-skill-usage.py`** | NEW Phase 3 | Which skills cited in BG prompts? Quadrant migration (per A28 cross-section matrix) |
| 8 | **`analyzer-hook-fire-frequency.py`** | NEW Phase 3 | Which hooks fired? Which didn't? Effectiveness re-run extending A3. |
| 9 | **`analyzer-ns-attribution.py`** | NEW Phase 3 | Token spend per NS/GL (G8 Phase 4 prep — finally data-backed) |
| 10 | **`analyzer-bug-trend.py`** | NEW Phase 3 | Bugs accumulated vs resolved this session; trend line |
| 11 | **`analyzer-asks-satisfaction.py`** | NEW Phase 3 | % of A18 trigger-phrase asks that landed artifacts |
| 12 | **`analyzer-pushback-rate.py`** | NEW Phase 3 | Frustration markers per N prompts (A29 KPI cross-session) |
| 13 | **`analyzer-cluster-readiness.py`** | NEW Phase 3 (gated on G7) | If cluster online: capacity utilization; bottleneck nodes |

All 13 run in parallel (Popen) via aggregator. Total wall-clock budget: ≤30s (parallelism wins).

## MASTER SYSTEM AUDIT COMPONENTS — 7 modules

Each module reads multiple analyzer outputs + cross-session bundles to produce a synthesized view:

### Module A — Realignment Digest (`realignment-digest.md`)
Top 5 next-session priorities, ranked by `impact × urgency / cost`. Examples of items it might surface:
- "Click SET-005 (highest-leverage unblock — wires entire hook layer)"
- "Address P1-04 monkey decision (41+ hours pending; blocking signoff)"
- "Phase 2 skill refresh: agentic-device-testing is the 3rd in queue"
- "Run vq:capture — currently 5h+ stale"
- "Consider G12 pilot launch (BG #111 research landing means Phase 5 ready)"

### Module B — Health Scorecard (`health-scorecard.md`)
Overall agentic-OS health (0-100 weighted aggregate of):
- Skill health quadrant balance (HH:HL:LH:LL)
- Hook chain health (% wired vs orphan)
- Goal staleness (% goals updated <6h ago)
- Audit closure rate (% in_progress → resolved per session)
- Memory rule violation count (Layer 3 enforcement signal)
- Bug billboard size + trend
- Asks-satisfaction rate
- Cost compliance (tokens vs budget)

### Module C — Drift Radar (`drift-radar.md`)
Where is the system drifting?
- Skills aged >30 days without refresh
- Memory rules getting violated (recurrence-detect signals)
- Plans stale (kind:active but no edits in N sessions)
- Hooks never firing (per analyzer-hook-fire-frequency)
- Goals stuck in same phase >3 sessions

### Module D — Cost Projection (`cost-projection.md`)
At current burn rates, when does what hit?
- Claude token ceiling (today: 51M / observed 30M ceiling — already past)
- Disk space (bundles + cache accumulating)
- Hook chain latency growth (each new hook adds ms)
- BG dispatch saturation (5-ceiling pressure)

### Module E — NS Advancement (`ns-advancement.md`)
Per G8 Phase 4 (token attribution): what % of this session's tokens advanced the current Northern Star? Trend across sessions.

### Module F — Recurrence-Across-Sessions (`recurrence-across-sessions.md`)
A19 within-session recurrence is good. CROSS-SESSION recurrence is the META: same pattern N sessions in a row = systemic issue requiring deeper fix.

Example: if "inbox-miss" appears in 5 consecutive session-bundle themes, it's not a fix-this-session issue — it's a STRUCTURAL gap requiring overhaul.

### Module G — Recommendations (`recommendations.md`)
Synthesized actionable list combining all above. Pre-formatted as a next-session prompt that the resume hook surfaces.

## POST-ANALYSIS HOOKS — 13 ideas (cascade after master system audit)

Each is a small script that consumes master system audit output + executes a structural action:

| # | Hook | What it does |
|---|---|---|
| 1 | `auto-update-goal-phases.sh` | Updates goal phase + updated_ts based on bundle activity per goal |
| 2 | `auto-archive-resolved-audits.sh` | Moves A* with status=resolved to `archived/audits/` |
| 3 | `auto-generate-next-session-focus.sh` | Synthesizes resume briefing pre-population from realignment-digest |
| 4 | `auto-update-northern-star-marquee.sh` | If recommendation shifts focus, update G2 phase or signal new NS |
| 5 | `auto-deprecate-stale-plans.sh` | Plans untouched for N sessions → propose moving to `paused/` |
| 6 | `auto-promote-insights-to-skill-refs.sh` | Recurring patterns from drift-radar become new skill refs |
| 7 | `auto-bug-billboard-from-findings.sh` | Recurring failures from analyzer outputs auto-create bug entries |
| 8 | `auto-asks-log-priority-tag.sh` | Outstanding asks tagged for next-session priority |
| 9 | `auto-cache-cleanup.sh` | Rotate old session-snapshot archives; clear stale dedup TTLs; cap bundles at 30 |
| 10 | `auto-research-index-refresh.sh` | Rebuild research-index.md (per A31 Front 5 + A33 new sub-folder needs) |
| 11 | `auto-memory-rule-violation-report.sh` | Surface where rules were violated this session |
| 12 | `auto-cross-session-pattern-detect.sh` | N sessions of same pattern → escalate to audit catalog (new A-entry) |
| 13 | `auto-skill-refresh-queue-update.sh` | Based on analyzer-skill-usage + A28 cross-section, update Phase 2 refresh queue |

Each hook is small (<200 lines), invoked sequentially after master audit completes. Total post-analysis wall-clock: <10s.

## Full pipeline timeline (post-shutdown)

```
T+0       User triggers graceful shutdown
T+1s      Stage 1: snapshot complete (synchronous; non-blocking)
T+2s      User can close laptop / drive home
          (background work begins)

T+5s      Stage 2 starts: bundle creator
T+6s      Bundle creator done (1s)
T+6s      Analyzers fan out (13 parallel via Popen)
T+8s      Fastest analyzers done (themes, hook-fire-frequency)
T+15s     Mid analyzers done (kpi-deltas, recurrence)
T+30s     Slowest analyzers done (token-efficiency, cluster-readiness)
T+30s     Aggregator merges all → insights.md

T+30s     Stage 3 starts: master system audit
T+35s     7 master audit modules run (mostly sequential; some parallel)
T+40s     Master audit reports landed in reports/master-system-audits/

T+40s     Post-analysis hooks fire (13 sequential; <10s)
T+50s     Pipeline complete

         (Next session opens to fully realigned state)
```

Total post-shutdown async wall-clock: ~50s. User shutdown experience: 2s (snapshot only).

Compared to current Phase 1: this adds 12 analyzers + master audit + 13 post-hooks. Compute cost: significant (local — no Claude tokens). Per A33 framing: this is exactly where local AI compute pays off massively.

## Integration with A33 (local AI mass-wielding)

Master system audit modules could be implemented partially via local AI:
- Module A (realignment digest) → general-purpose local model synthesizes from analyzer outputs
- Module B (health scorecard) → small model classifies + scores
- Module F (recurrence-across-sessions) → vision/pattern model

After A33 G12 pilot: master system audit runs WITHOUT Claude tokens. Pure local compute.

This closes a powerful loop: agentic OS introspects, recommends, acts — all locally — and surfaces only the highest-value decisions to Claude (me).

## Cost discipline

Per G9 + A33:
- Total post-shutdown pipeline: <60s wall-clock (parallel)
- Token cost: ZERO post-A33 G12 pilot landing (master audit + analyzers run local)
- Pre-A33: Claude-token-bounded; cap analyzer count to 5 concurrent until A33 G12 lands

If budget breached at any stage: pipeline backs off — full audit becomes per-week instead of per-session.

## Phasing (see plan)

See `plans/automation/end-of-session-realignment-ritual-plan.md`.

- **Phase 1 (this turn, foreground)**: A34 + plan + G14 goal + memory rule + audit catalog + steering log
- **Phase 2** ✅ LANDED 2026-05-27 02:55Z: G11 Phase 2 — 4 more analyzers (#2-5 above) shipped; parallel wall-clock 0.14s on POC bundle (vs 5×3-5s sequential)
- **Phase 3**: 8 more analyzers (#6-13) — multiple BGs
- **Phase 4**: Master system audit framework + 7 modules
- **Phase 5**: 13 post-analysis hooks
- **Phase 6**: Integration with resume hook (insights surface at session start)
- **Phase 7**: Cross-session pattern detection (Module F deep impl)
- **Phase 8**: Local AI integration (depends on G12 pilot)

## Verification (per phase)

After Phase 4:
1. Master system audit runs end-to-end on a synthetic bundle
2. 7 modules produce expected output files
3. Health scorecard returns 0-100 score
4. Realignment digest lists top 5 priorities

After Phase 6:
5. Next session's resume hook surfaces master audit insights
6. Recommendations are actionable (not vague)

After Phase 7:
7. Cross-session pattern detection fires when ≥3 consecutive bundles show same theme

After Phase 8:
8. Master audit + analyzers run with ZERO Claude tokens (pure local compute)

## Cross-references

- G11 — bundle analyzer (Stage 2 owner)
- G10 / A30 — graceful shutdown lifecycle (Stage 1 owner)
- G14 NEW — this audit's owner
- A24 (meta-loop-eval) — monthly cross-session synthesis; A34 is its per-session counterpart
- A28 (skill health) — feeds analyzer-skill-usage + auto-skill-refresh-queue
- A29 (lazy regression) — pushback rate KPI tracked by analyzer-pushback-rate
- A19 (recurrence) — within-session catch; A34 Module F is cross-session
- A33 (local AI) — Phase 8 dependency; massive cost reduction
- A31 Front 2 (event-bus) — substrate for post-analysis hooks
- Reactionary skill — sibling pattern (event-triggered; A34 is session-shaped)

## Lessons (preliminary)

- Per-session retrospection (G11) is incomplete without cross-session synthesis (G14)
- "Master writeup as realignment moment" is the user's structural insight — it's not a passive document, it's a TRIGGER
- The 3-stage pipeline (snapshot → bundle → master audit) mirrors laptop suspend → boot → user-login flow
- Local AI (A33) is THE enabler for sustainable per-session deep introspection
- Each analyzer + post-hook is small. The POWER is in the COMPOSITION.
