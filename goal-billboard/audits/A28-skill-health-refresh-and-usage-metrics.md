---
audit_id: A28
title: "Skill health framework — usage metrics, freshness scanning, refresh cadence, first-pass upgrade"
status: in_progress
catalogued: 2026-05-27T00:30:00Z
priority_when_run: P0
estimated_effort: large (metrics infra + freshness scanner + per-skill refresh BGs over multiple turns; multi-day work)
trigger: 2026-05-26 23:30Z — user audit *"we've been building such a powerful system from expanding our skills and the longer we develop the better our skills have gotten — but what about the skills we started with? if our skills progressively get better and better that also must mean that our older ones are getting worse and worse comparatively."* Plus sub-request for skill usage count tracking → cross-section analysis ("rich software cross section point"). User framed as "major prompt with big implications, stew on this."
deferral_reason: NONE — the asymmetry is real (today's `agentic-testing-discipline` lands with a 14-type taxonomy; `agentic-webdesign` from 4 days ago reflects an earlier mental model). The longer this drifts, the harder the catch-up.
related_goals: [G4]
related_plans: [context/markdowns/plans/automation/skill-health-refresh-and-metrics-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G4
related_refs:
  - .claude/skills/  (all 9 skills as of today)
  - .claude/skills/reactionary/  (the event-triggered counterpart to scheduled refresh)
  - context/markdowns/agent-recommendations/  (scratchpad layer)
findings: []
---

# A28 — Skill health framework

## The user's question, broken into parts

The prompt contains 5 sub-questions:

1. **How do we CONFIRM skills are getting better?** → metrics framework
2. **The asymmetry problem**: newer skills outpace older → first-pass upgrade
3. **Refresh cadence**: when does a skill exist long enough to need an update? → opsys-scheduling
4. **Global vs local refresh**: how many at once, how often? → tier system + concurrency rules
5. **Usage count tracking** → the cross-section matrix

This audit + companion plan answer all five.

## Part 1 — Confirmation framework: 6 metrics per skill

A skill is "better" when these metrics improve over time. They're trackable; we just haven't been tracking them.

| Metric | Definition | How to compute | Why it matters |
|---|---|---|---|
| **Ref count** | Number of `.md` files under `references/` | `ls .claude/skills/<X>/references/*.md \| wc -l` | Coverage breadth |
| **Total byte count** | Sum of skill body sizes (SKILL.md + refs) | `wc -c` | Depth (proxy; not perfect) |
| **Cross-link in-degree** | How many artifacts reference this skill | `grep -rl ".claude/skills/<X>" context/markdowns/ \| wc -l` | Load-bearing-ness across project |
| **Use count (BG)** | BG dispatch prompts mentioning skill | Walk session JSONLs; grep prompt content | Real-world usage |
| **Staleness days** | Days since last meaningful update | `max(mtime)` across files | Drift indicator |
| **Scratchpad accumulation** | `agent-recommendations/<X>.md` entry count | Count `^#` or `^-` lines | Unconsolidated learnings |

Composite "health score" (0-100):
- Coverage breadth (refs ≥3 = 20pts; ≥6 = 30pts)
- Cross-link in-degree (10+ = 20pts)
- Recent use (24h ≥1 = 20pts; 7d ≥3 = 30pts)
- Staleness < 30d (15pts)
- Scratchpad pending (≤5 entries = 15pts; >30 = 0pts forces consolidation)

(Exact weights tunable from first real data.)

## Part 2 — The asymmetry problem (today's reality)

| Skill | Refs | Last touched | Days since | Likely state |
|---|---|---|---|---|
| `agentic-script-design` | 9 (today: 5 new) | 2026-05-27 | 0 | ✅ fresh (today's expansion) |
| `agentic-testing-discipline` | 2 | 2026-05-27 | 0 | ✅ fresh (today's create) |
| `agentic-quality-discipline` | ~15+ | 2026-05-26 | 0 | ✅ fresh (today's many additions) |
| `agentic-page-scrutiny` | ~8 | 2026-05-25 | 1-2 | 🟡 likely needs A19 workflow-bypass section + A25 dashboard-scrutiny lessons |
| `agentic-device-testing` | ~6 | 2026-05-25 | 1-2 | 🟡 likely needs cluster-distribution preview (G7 Phase 5) |
| `agentic-chrome-tools` | 0 refs (10 modes in SKILL.md) | 2026-05-25 | 1-2 | 🟡 modes 8-9-10 may need refining post real use |
| `agentic-webdesign` | ~6 | 2026-05-24 | 2-3 | 🔴 4-day-old; doesn't reflect today's growth (cluster, hooks, state substrate, BG dispatch architecture, time precision) |
| `agentic-design-handoff` | ~1-2 | 2026-05-25 | 1-2 | 🟡 Text v2 incident lessons (agent fabrication detection) belong here |
| `reactionary` | ~6 | 2026-05-25 | 1-2 | 🟡 today's meta-loop-eval is its scheduled-time counterpart; ref cross-link missing |

Verdict: 5 of 9 skills are likely stale relative to today's growth. None catastrophic; all addressable via Phase 2 per-skill refresh BGs.

## Part 3 — Refresh cadence design (opsys-style scheduling)

### Triggers

1. **Cadence (clock-based)**:
   - P0 skills: monthly review
   - P1 skills: quarterly
   - P2 skills: semi-annually
2. **Event (architectural change)**:
   - ≥5 audits in the skill's domain landed since last update → refresh
   - Major project pivot (new goal Northern Star) → refresh affected skills
3. **Use-based**:
   - High-use skill (top-25%) not updated in 30 days → priority bump
4. **Scratchpad-saturation**:
   - `agent-recommendations/<skill>.md` accumulates ≥30 entries → consolidation overdue
5. **Manual**:
   - `scripts/run-skill-refresh.sh <skill>` any time
6. **Drift detection (advanced; future)**:
   - Semantic comparison: do recent audits + BG outputs mention concepts not yet in the skill's refs? → suggest refresh

### Scheduling rules

- **Concurrency**: 1 skill at a time (avoid context overwhelm)
- **Fairness**: FIFO+aging within tier (oldest-update-first); aging boost prevents starvation
- **Quiet-hours**: avoid major refresh during high-velocity sessions (user signal: rapid prompts in last 30min → defer); piggyback on stop-hook quiet moments
- **Tier-aware**: P0 always before P1 before P2 unless aging boost flips order
- **Cap**: max 1 refresh BG in flight at a time (separate from A21 5-BG general ceiling — refresh is its own slot)

### Synchronization primitives (per agentic-script-design opsys-scheduling-patterns)

- **Refresh lock**: `.claude/cache/skill-refresh.lock` — filesystem atomic lock; only 1 refresh BG at a time
- **Last-refresh timestamps**: `.claude/cache/skill-refresh-history.jsonl` — append-only audit trail
- **Heartbeat scan**: tier-low (15min) checks for skills meeting cadence triggers; surfaces in heartbeat-pending

## Part 4 — Global vs local refresh

**Global** (full first-pass): now is the right moment. 5 skills likely stale; multi-day project growth = catch-up opportunity. Phase 2 of plan handles this with 1-at-a-time BGs over 5 turns.

**Local** (ongoing): post-first-pass, per-trigger refresh. Heartbeat tier-low scans; state-batch-digest surfaces top-3 stale; user can manually request.

**Concurrency**: NEVER more than 1 refresh at a time. Refresh writes to SKILL.md + multiple refs; coordination overhead is high; better serialize.

## Part 5 — Usage count tracking + cross-section matrix

Sub-request explicit: track skill usage count.

### Counting sources (composite)

1. **BG dispatch prompts** — walk session JSONL for assistant messages dispatching Agent tool; grep prompt content for skill names; weight 1.0 per mention
2. **Agent responses** — same grep, lighter weight (0.3 per mention) since references can be incidental
3. **Cross-link in artifacts** — count `references/` mentions across `context/markdowns/`; weight 0.5 per
4. **Skill tool invocations** (when wired) — direct invocation = 2.0 per (high signal)

Aggregate per skill per window (24h / 7d / 30d).

### The cross-section matrix

```
              HIGH-use (top quartile)    LOW-use (bottom quartile)
              ┌─────────────────────────┬─────────────────────────┐
HIGH-refs     │ STATUS: working great    │ STATUS: over-built /    │
(≥6 refs)     │ ACTION: maintain;        │ wrong scope             │
              │ targeted enhancements    │ ACTION: audit "still    │
              │                          │ needed?"; consider      │
              │                          │ deprecation / merge     │
              ├─────────────────────────┼─────────────────────────┤
LOW-refs      │ STATUS: under-documented │ STATUS: obsolete        │
(<6 refs)     │ ACTION: priority for ref │ candidate               │
              │ expansion (today's       │ ACTION: audit; if true  │
              │ work pattern)            │ obsolete: archive       │
              └─────────────────────────┴─────────────────────────┘
```

The matrix surfaces priorities:
- Top-right quadrant (over-built) → audit candidates
- Bottom-left (under-documented) → expansion priorities
- Bottom-right (obsolete) → deprecation candidates
- Top-left (great) → preserve + maintain

This is the user's "rich software cross section point" made operational.

## Part 6 — Dashboard surface

New panel: **"Skill Health"** in Stats Correlation tab (or Billboard tab — to be decided based on visual fit).

Renders:
- Per-skill table: name · ref count · use count (24h/7d/30d) · staleness · health score · quadrant
- 2×2 cross-section quadrant visualization
- Top-3 refresh candidates (highest aging × lowest health score)
- Sparkline of average health score over time (proxy: aggregate ref count growth + use count rolling)

Lefter mini-block: "N skills need refresh" + click navigation.

## Implementation plan (4 phases — see plan doc)

- **Phase 1 (this turn)**: A28 + plan + memory rule + 2 BGs (usage metrics + freshness scanner)
- **Phase 2 (next 5 turns)**: Per-skill refresh BGs, 1-at-a-time, P0 first
- **Phase 3 (weeks)**: Automation — heartbeat tier-low staleness scan + state-batch-digest surfacing + Stop hook freshness alerts
- **Phase 4 (monthly cadence)**: Synthesis via meta-loop-eval — skill-health digest report

## Verification

After Phase 1:
1. `scripts/render-skill-metrics-data.py` populates `data.json` with `skill_metrics` block
2. `scripts/run-skill-staleness-scan.sh` produces freshness scores for all 9 skills
3. Dashboard "Skill Health" panel renders
4. Lefter mini-block shows N stale count
5. Research doc `research/agent-os/skill-staleness-audit-2026-05.md` identifies top 3 refresh targets

After Phase 2 (cumulative):
6. All flagged stale skills refreshed
7. Cross-section matrix has shifted (fewer in bottom-left + bottom-right quadrants)

## Cost gates (per the G9 testing precedent)

The user's 25× stress on testing applies here too — refresh infrastructure can't itself become a bottleneck.

- Phase 1: <1.5M tokens cumulative
- Phase 2: <500k per per-skill refresh BG; total <3M for full pass
- Phase 3: hooks <500ms per fire (heartbeat scan budget)

If any breached: pause + reassess.

## Status

IN PROGRESS — Phase 1 complete (freshness scanner + skill staleness audit landed); Phase 2 in progress (2 of 3 P0 refreshes complete).

### Phase 2 progress

- **2026-05-26T23:52:37Z** — `agentic-webdesign` refresh complete (composite score 100.0 → addressed). Per the BG template in this doc's Section E:
  - Acquired refresh lock cleanly; released on exit.
  - 3 new refs added under `references/workflows/` (each >9K bytes):
    - `parallel-bg-for-web-design.md` — 3 BG dispatch cycles applied to web-design; file-zone rules; NS-context prefix; fabrication detection (9944 bytes)
    - `cluster-distributed-visual-work.md` — G7 cluster fanout for visual regression; capability-routed + capacity-weighted dispatch; time precision across nodes (9810 bytes)
    - `agent-orchestration-for-design-iterations.md` — 5-phase iteration lifecycle (research→scaffold→implement→verify→refine); hook chains; state-substrate touchpoints; anti-bottleneck application; NS-context flow (12362 bytes)
  - SKILL.md description updated to mention current project maturity (cluster, BG dispatch, hook chains, state substrate, time precision, anti-bottleneck testing, NS-context propagation); positioned as BREADTH with sibling skills as DEPTH.
  - SKILL.md sections updated: §1 (BG dispatch cycles + cluster mention), §6 (memory-mirror + goal billboard + standing protocols), §8 (anti-bottleneck verification gates with tier system).
  - Scratchpad seeded at `context/markdowns/agent-recommendations/agentic-webdesign.md`.
  - Refs README updated to index the 3 new workflow refs.
  - Refresh-history JSONL entry written at `.claude/cache/skill-refresh-history.jsonl`.
  - Cross-links established to: agentic-script-design (5 refs), agentic-testing-discipline (3 refs), agentic-quality-discipline (5 refs), goal G7, audits A21/A22/A25/A26/A28/A29.

- **2026-05-27T00:15:00Z** — `agentic-chrome-tools` refresh complete (composite score 100.0 → addressed). Per the BG template:
  - Acquired refresh lock cleanly (lock at `.claude/cache/skill-refresh.lock/agentic-chrome-tools/`); released on exit via trap.
  - 13 new refs added (10 per-mode + 3 cross-cutting growth refs):
    - `mode-01-default-stateful.md` — single-viewport hover/click/console (6210 bytes)
    - `mode-02-viewport-debug.md` — triggered viewport-specific repro of matrix failure (6246 bytes)
    - `mode-03-animation-filmstrip.md` — timed-screenshot animation timing (6660 bytes)
    - `mode-04-computed-style-measurement.md` — `getComputedStyle` queries (8755 bytes) — explicitly cross-links to BG #74 dashboard scrutiny (A19 Layer 3)
    - `mode-05-cross-page-state.md` — header/anchor/focus persistence across nav (7774 bytes)
    - `mode-06-network-timing.md` — `read_network_requests` for waterfall/lazy/font/404 (7513 bytes)
    - `mode-07-viewport-sweep.md` — multi-viewport interactive sweep (slow; reserved) (7738 bytes)
    - `mode-08-live-css-experimentation.md` — skip edit/save/rebuild cycle via JS inject (8373 bytes)
    - `mode-09-computed-contrast.md` — WCAG contrast math (8812 bytes) — explicitly cross-links to BG #74 dashboard scrutiny (A19 Layer 3)
    - `mode-10-focus-order-tab-loop.md` — keyboard nav verification (8268 bytes)
    - `chrome-plugin-for-bg-dispatch.md` — Chrome inside BG agents; cluster Phase 2 prep (6823 bytes)
    - `chrome-plugin-mode-decision-matrix.md` — what mode for what bug class / question / lifecycle phase (6671 bytes)
    - `chrome-plugin-cost-discipline.md` — per G9 anti-bottleneck applied to Chrome modes (7167 bytes)
  - 2 legacy/overview refs preserved: `chrome-plugin-discipline.md` (original monolith — index/overview) and `chrome-vs-preview-decision.md` (cross-tool decision).
  - SKILL.md description updated to position skill as "live-browser cross-cutting skill" with 13 refs mention; body restructured into mode-table pointing to per-mode refs (no duplication).
  - Scratchpad seeded at `context/markdowns/agent-recommendations/agentic-chrome-tools.md` with `last_consolidated: 2026-05-26` and pointer to predecessor scratchpad `chrome-plugin-discipline.md`.
  - Refresh-history JSONL entry written at `.claude/cache/skill-refresh-history.jsonl`.
  - Cross-links established to: agentic-script-design (`bg-dispatch-architecture.md`), agentic-webdesign (`workflows/agent-orchestration-for-design-iterations.md`, `workflows/parallel-bg-for-web-design.md`), agentic-testing-discipline (G9 cost framework), goal G9, audits A19/A21/A22/A28.

- **Remaining Phase 2 work**: agentic-device-testing refresh (per Section C order).

## Cross-references

- G4 — receives this scope expansion
- A24 (meta-loop-eval) — Phase 4 synthesis extends meta-loop pattern
- A26 (script-design expansion) — proved the "first-pass upgrade" model works
- A27 (testing toolbelt) — cost-gate precedent
- `reactionary/` skill — event-triggered counterpart; A28 is scheduled-time-triggered
- `agentic-script-design/references/opsys-scheduling-patterns.md` — tier system + FIFO+aging applied here
- `agent-recommendations/{skill}.md` scratchpad layer — the consolidation source
