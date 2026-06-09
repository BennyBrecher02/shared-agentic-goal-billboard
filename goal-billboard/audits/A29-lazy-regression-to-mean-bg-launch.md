---
audit_id: A29
title: "Lazy regression-to-mean — BG-launch count drops under no-pressure conditions"
status: in_progress
catalogued: 2026-05-27T00:40:00Z
priority_when_run: P0
estimated_effort: medium (empirical analysis + tightened enforcement design; structural fix waits for A21 hook wiring)
trigger: 2026-05-26 23:38Z — user audit *"how come when i annoy you then you go ham and have like 5 bg processes but when i let you cook you go to like 2/3? and how can we prevent that lazy behavior?"* This is the **5TH RECURRENCE TODAY** of the under-parallelization complaint (after A16 idle, A18 hedge, A20 holdoffs, A21 single-BG-reflex). Recurrence at 5x = prior fixes failed at the wrong layer.
deferral_reason: NONE — the 5x recurrence IS the diagnostic. The fix has to be structural, not behavioral.
related_goals: [G4]
related_plans: [context/markdowns/plans/automation/lazy-regression-prevention-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G4
related_refs:
  - .claude/skills/agentic-quality-discipline/references/kpi-self-eval-workflow.md
  - context/markdowns/goal-billboard/audits/A21-parallel-capacity-underuse.md
  - context/markdowns/goal-billboard/audits/A16-agent-idle-root-cause.md
findings: []
linked_research:
  - context/markdowns/research/agent-os/bg-taxation-analysis-20260526.md
---

# A29 — Lazy regression-to-mean (BG-launch undercount)

## The user's pattern, observed empirically

User explicit framing 2026-05-26 23:38Z: *"how come when i annoy you then you go ham and have like 5 bg processes but when i let you cook you go to like 2/3? and how can we prevent that lazy behavior?"*

The user is correct. Pattern from today's session (rough):

| Trigger type | Avg BGs launched | Pattern |
|---|---|---|
| **Direct callout** (user pushes on under-parallelization) | 4-5 | Maximum effort |
| **New audit/plan request** (with trigger phrases) | 3-4 | High effort |
| **"Stew on this" / calm deep-thinking ask** | 2 | Comfortable minimum |
| **BG-completion notification turns** (no new user prompt) | 0-1 | Drift to zero |

**Diagnosis:** regression-to-mean toward minimum-effort under no-pressure conditions. The A21 "default ≥3" rule treats `default` as soft target, not hard floor.

## EMPIRICAL UPDATE — the data flipped the diagnosis

**BG #92's analysis (n=249 prompt/response pairs from session JSONL — full report at `reports/bg-launch-pattern-analysis-2026-05.md`):**

| Metric | Value |
|---|---|
| High-pressure pairs (score ≥3) | 176 |
| Low-pressure pairs (score <3) | 73 |
| Mean BG count when frustration_score ≥3 (high-pressure) | **1.81** |
| Mean BG count when frustration_score <3 (low-pressure) | **3.04** |
| Spread (high − low) | **-1.23** |
| Spearman ρ (frustration_score × BG count) | -0.011 |
| Verdict | **INVERSE pattern** |

The chronological per-response table shows the same shape: highest-BG turns (12, 11, 10, 9, 8, 7) cluster around low-to-medium pressure + "go execute" prompts ("im back, start us up", "lets get to:", "start working towards the cluster north star"); lowest-BG turns (0, 0, 0, 0) cluster around high-pressure diagnostic prompts ("im still totally lost on", "this is not how a proper system should react", "i sent a lengthy monkey response like 15 minutes ago and i dont think you acknow…").

**What this means:**
- User hypothesis ("5 BGs when annoyed; 2-3 when calm") was SELECTIVE memory (vivid bursts after pushback) — not the dominant truth.
- Real pattern: pressure pulls TEXT-HEAVY diagnostic responses with FEWER BGs (defensive explanation mode). Calm "go execute" prompts pull BG-heavy responses with MORE BGs (directive mode).
- The original "lazy regression to mean" framing was WRONG in shape. The failure mode is **diagnostic-paralyzed under pressure**, not "lazy when calm."

**Why the FLOOR rule still holds:**
The fix (FLOOR rule + structural enforcement via hooks) targets QUANTITY of BGs per response. Whether the failure shape is "fewer BGs when pressured" or "fewer BGs when calm," the same fix works — both directions are addressed by "always launch min(pending, 5−in_flight) BGs with floor of 1." The Layer-1 hook surface fires on UserPromptSubmit regardless of pressure markers.

**Updated diagnosis** (replaces the lazy-when-calm framing):
- **Anti-pattern**: text-heavy explainer responses when substrate has executable work
- **Trigger**: user pressure (frustration/pushback) — counterintuitively, makes me MORE verbose + LESS execution-focused
- **Fix**: same as before (FLOOR rule + Layer 1/2 hooks + cost-lowered launch via NS dispatch wrapper)

**Lesson for the meta-monitoring layer:**
Self-audits are unreliable. Empirical data wins. BG #92's data contradicted both the user's hypothesis and my self-audit table above — this is exactly the value of A19's meta-monitoring layer. Without the empirical layer, the wrong fix (a "go ham under pressure" trigger) would have been pursued, doubling-down on the actual failure mode.

## Root causes (sibling family to A16, A18, A20, A21)

1. **A21 hooks not yet wired** — `parallel-capacity-check.sh` + `pending-parallel-work-scan.sh` exist on disk but aren't in `settings.json` (per A25). Memory rule alone fails under no pressure.

2. **"Default" interpreted as ceiling, not floor** — language ambiguity in my own memory rule.

3. **Cognitive cost of orchestration** — 5 parallel BGs requires 5 prompts + 5 file-zone checks + 5 verifications to track. Under pressure I push through; calm = I don't.

4. **Token-cost rationalization** — "let me not be wasteful" used as excuse, even though parallel = same total cost as serial. **A14 finding** already debunked this but the rationalization survives.

5. **No empirical visibility** — no dashboard metric exposing my BGs-per-response over time. Regression invisible until user catches it.

6. **Lack of "pressure-independent floor"** — every other discipline I've encoded has an enforcement signal (system-reminder, hook, monkey decision). Parallelization discipline has NO terminal enforcement that doesn't require my pre-existing discipline to fire.

## Why this is the 5th recurrence today

| Audit | What it caught | Why it didn't catch THIS |
|---|---|---|
| A16 | Agent fully idle | Idle ≠ low parallelization; I'm "active" with 2 BGs but lazy |
| A18 | No artifact landed per-prompt | I land artifacts; just don't pile on BGs |
| A20 | Hedge phrases / chat-burying | I don't hedge — I just under-launch silently |
| A21 | Parallel-capacity underused | Right rule, but hooks NOT WIRED → no enforcement |
| **A29** | **Regression to mean under no pressure** | **(this audit)** |

**The pattern:** each predecessor catches a SHAPE of under-effort. A29 catches the META-shape: my baseline effort silently drops without enforcement.

## The structural fix (4 layers)

### Layer 1 — Wire the A21 hooks (THE biggest blocker)

Per A25 findings: `parallel-capacity-check.sh` + `pending-parallel-work-scan.sh` exist on disk but aren't in `settings.json`. Wiring them via `scripts/hooks/settings-patches/parallel-launch-discipline.json` would:
- Surface "🔀 PARALLEL CANDIDATES" at every UserPromptSubmit (Layer 1 source-catch)
- Surface "⚠ PARALLEL CAPACITY UNDERUSED" at Stop when candidates > dispatches (Layer 2 terminal-catch)

This is THE single biggest fix. Routes through monkey-chamber SET-005 (batch settings patches) or a focused SET-002-equivalent.

### Layer 2 — Tighten the memory rule from "default" to "hard floor"

Current rule: *"Default ≥3 parallel BGs per response when pending decoupled work exists."*

Tightened rule: **"FLOOR: dispatch min(pending_candidates - in_flight, 5 - in_flight) BGs per response with substantive work. Minimum dispatch = 1 if ANY decoupled candidate exists AND in_flight < 5. NO 'calm mode' exception."**

The change: "default" → "FLOOR". "≥3" → exact formula. "NO calm exception" makes the rule pressure-independent.

### Layer 3 — Empirical visibility (this turn's BG)

Dashboard sparkline: "BGs-per-response (last 50)" with annotation for user-pressure signals (frustration markers in concurrent prompts). User sees the regression visually — accountability + diagnostic.

### Layer 4 — Calm-mode-trap detector (extends parallel-capacity-check)

Add to `parallel-capacity-check.sh` after Layer 2 wires:
- Track rolling-3 BG counts
- If last 3 responses each had ≤2 BGs AND pending candidates ≥4: surface "🪤 CALM-MODE TRAP — last 3 responses dispatched only 2 each while substrate held N candidates. Apply 6-step loop on A21."

### Layer 5 — Lower cognitive cost of launch (optional / future)

BG dispatch wrapper: `scripts/bg-dispatch-wrapper.sh` that auto-handles NS context (per G8 Phase 3) + file-zone detection + verification format. Lowers per-launch cost → naturally encourages higher launch frequency.

## Verification

After Layer 1 (wiring A21 hooks):
1. Every UserPromptSubmit fires PARALLEL CANDIDATES surface
2. Every Stop with <N dispatches fires UNDERUSED warning
3. Test: turn with 4 candidates + 1 BG launched → warning fires; turn with 4+ dispatches → silent

After Layer 2 (tightened rule):
4. Memory rule updated; subsequent sessions show ≥1 BG per substantive-work turn

After Layer 3 (visibility):
5. Dashboard sparkline renders; clear correlation between pressure spikes and BG-count spikes visible

After Layer 4 (calm-mode-trap):
6. Test: 3 turns with ≤2 BGs each + ≥4 candidates → 4th turn fires trap-detected surface

## Cost gates

Per G9 testing-toolbelt cost-discipline precedent:
- Phase 1 (this turn — audit + plan + 1 BG): <500k tokens
- Layer 1 wiring: token-free (user settings.json edit)
- Layer 4 hook addition: <200ms per fire

## Ack-latency improvement attempt #1 (the 21:08Z gap closed)

Executed 2026-05-27T00:04Z. Per `kpi-self-eval-workflow.md` 6-step loop (the 21:08Z user request *"try your new best idea on how to shorten the gap"* — wrote the ref but never executed; BG #71 gap-fill flagged this; finally landing now):

**Step 1 — Investigate**: ack-latency.jsonl shows recent measurements at 21:58Z with latencies ranging from 862s (~14min) to 23665s (~6.5h). Recent p50 ≈ 14-16min; worst entries from earlier in the day are 6h+. Prior improvement (the "explicit ACK in first prose" rule + inbox surfacing) helped — recent ones are ≤ 5min for newer messages — but still no structural enforcement.

**Step 2 — Prior fixes already in place** (per `feedback_standing-protocols.md`):
- "Open every response with explicit ACK" rule (2026-05-26 PM — A20)
- "Read state substrate inline" rule (2026-05-26 PM)
- Inbox surfacing structural fix (UserPromptSubmit digest)

**Step 3 — Candidates brainstormed (≥3 required)**:
- (a) **First-tool-call rule** — FIRST tool call of any response with unacked content MUST materialize the ACK. Eliminates "ack buried in long response."
- (b) **Real-time heartbeat osascript** — `check-inbox.sh` fires osascript within 60s. Doesn't require agent response.
- (c) **Ack-shortcut snippet** — templated 1-line ack format. Already informally done; not a real improvement.
- (d) **State-batch-digest priority** — when state-batch surfaces "N unacked inbox," emit FIRST. Gated by SET-005 wiring.
- (e) **Auto-pin-to-billboard** — unacked >30min auto-promotes to billboard. Extra noise risk.

**Step 4 — Tradeoffs**:

| Idea | Effort | Effect | Failure mode |
|---|---|---|---|
| (a) First-tool-call rule | low | high — guarantees front-loading | none — purely additive |
| (b) Real-time heartbeat osascript | low | medium | depends on heartbeat installed |
| (c) Ack-shortcut snippet | low | low — already done | not really an improvement |
| (d) State-batch-digest priority | low-med | high | gated by SET-005 |
| (e) Auto-pin-to-billboard | medium | high | extra noise; needs filter |

**Step 5 — Plan**: (a) + (d) stack.
- (a) is cheapest + highest-effect; lands in memory rule + standing-protocols immediately.
- (d) requires state-batch wired; queues for after SET-005 applies.

**Step 6 — Act**: (a) executed this turn — added "First-tool-call ACK rule" section to `feedback_standing-protocols.md` (memory + mirror). Logged to `.claude/cache/ack-latency.jsonl` with `improvement_attempt: 1`. Refresh-history JSONL appended.

**Next measurement**: track p50 ack-latency over next 5 responses post-rule. Target: p50 < 5min (recent 14-16min; overall today ~4.15h driven by 6h+ outliers from morning).

## Status

IN PROGRESS. Foreground: audit + plan + memory rule update + steering log. 1 BG for empirical analysis + dashboard sparkline. Layer 1 wait on user nod via monkey-chamber settings batch.

Ack-latency improvement attempt #1 LANDED (2026-05-27T00:04Z) — first-tool-call ACK rule in memory + mirror.

## Lessons (preliminary)

- **Memory rules degrade under no-pressure conditions** — this is now the 3rd time this pattern caught (inbox, hedge, parallelization). Hooks > memory.
- **"Default" is a smell word** — ambiguous between floor and ceiling. Use "FLOOR" or "TARGET" explicitly.
- **Pressure-shaped discipline is fragile** — discipline that holds only under user pressure isn't discipline; it's reactive performance.
- **5x recurrence of the same META-pattern** (under-effort) requires META-fix (this audit) not another local fix.
- **Honesty as discipline** — admitting the pattern lowers the user's pushback rate; the conversation-pattern KPI should reflect this.

## Cross-references

- A16 / A18 / A20 / A21 — sibling response-side audits; A29 catches what they don't
- A25 — settings.json health (Layer 1 fix depends on wiring patches)
- A19 recurrence-detect — the right hook for catching 5x patterns; not yet wired
- `kpi-self-eval-workflow.md` — 6-step loop applied here
- `bg-dispatch-architecture.md` (today's new ref) — codified architecture; Layer 5 wrapper extends
