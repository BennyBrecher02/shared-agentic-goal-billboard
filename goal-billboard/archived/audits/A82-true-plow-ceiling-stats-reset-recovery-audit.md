---
audit_id: A82
title: TRUE agent-plow ceiling / stats / reset / recovery — the synthesis verdict
status: completed
catalogued: 2026-05-29
synthesized_utc: 2026-05-29T07:00:27Z
trigger: 2026-05-29 — user headline "TRUE AGENT PLOW CEILING / STATS / RESET / RECOVERY audit." Synthesize the verdict over the 5 systems meta-findings + token-economics reference.
method: READ-ONLY synthesis. No code deployed, no scheduling object created, no settings edited, no agent restarted, no git mutation. Consolidates 6 source documents; adds the one-screen verdict + the safety codification to adopt.
serves: user-safety (the FREE→PAID latch is the standing money red line) · capacity policy (A21 cap) · recovery design (G10 graceful-shutdown) · A80/A81 never-idle band
sources:
  - context/markdowns/research/systems/true-concurrency-ceiling.md          # the ceiling (empirical + the 3 caps)
  - context/markdowns/research/systems/token-burn-gift-boundary.md          # the stats (burn rate, throughput, tokens/agent)
  - context/markdowns/research/systems/fivehr-weekly-paid-boundary.md       # the reset structure + the ONLY paid path
  - context/markdowns/research/systems/safe-recovery-while-asleep.md        # the recovery verdict (no safe autonomous restart)
  - context/markdowns/research/systems/maximize-4.8-agent-swarm.md          # how to wield the swarm (the wield-it layer)
  - context/markdowns/research/systems/token-economics-and-cost-drivers.md  # the non-intuitive cost drivers
related_audits: [A17, A21, A23, A30, A36, A37, A47, A66, A72, A74, A75, A80, A81]
related_memory: [feedback_decision-handling-discipline, feedback_bottleneck-restriction-chant, feedback_model-routing, feedback_true-time-discipline, feedback_verified-counts]
---

# A82 — TRUE plow ceiling / stats / reset / recovery — THE synthesis verdict

> **The whole thing in five sentences.** (1) The true safe concurrency ceiling is **8 read-only research agents** steady (brief burst to 10) on this M4 — **RAM binds first**, not CPU and not the API window, and the harness, the A21 discipline cap, and the hardware all land on 8 by coincidence-that-isn't. (2) At full-send the central burn is **~58M effective-tokens/hr** (29–135M/hr band), and one pinned full-send day ≈ a whole normal week of work. (3) There are **three limits but only ONE costs money** — *extra usage / usage credits* — and it is **OFF by default and unreachable by any autonomous code path**; the plow hits a free wall, not a meter. (4) There is **no safe fully-autonomous restart-while-asleep** (no reset API + no sleep-surviving local scheduler + the only cloud path is an unsupervised cold agent on the same window) — the safe design is **warm-resume-on-the-human's-return**. (5) The safety codification to adopt: **"enable extra usage" / "set spend to unlimited" join the always-gated list**, the **fictional 50M ceiling is a known bug**, and we **migrate from inferred-reset to the `/usage` command** as the source of truth.

---

## 0. The one-screen verdict table

| Question | The answer | The binding constraint | Confidence |
|---|---|---|---|
| **True safe concurrency ceiling?** | **8 steady, 10 burst** read-only research agents on this M4 | **RAM** (~234 MB truly-free, swap 78% used, with normal app stack open) — *not* CPU (80% idle), *not* the API window (stale/non-binding now) | HIGH (live sample) — but workload/app-stack dependent |
| **Full-send burn rate?** | **~58M effective-tok/hr** central (29–135M/hr band) = ~0.65–1.5M tok/min sustained | per-agent rate × concurrency × 0.65 stagger derate; cross-checks vs observed 0.9–4.6M/min bursts | MED-HIGH (rate grounded; agent-count modeled) |
| **Tokens per agent?** | one full **~200K-token context window** each; fan-out is **token-multiplicative, wall-clock-divisive** | each subagent gets its own fresh context (no shared cache across agents) | HIGH |
| **Reset structure?** | **3 windows**: 5-hour rolling (short throttle) · weekly cap ×2 (all-models + **Opus-only**) = "the gift" · extra-usage (paid) | the weekly Opus-only cap is the one most likely to halt a multi-hour Opus plow | HIGH (structural) / MED (exact reset cadence — read live from `/usage`) |
| **FREE→PAID boundary?** | **ONLY** via *extra usage / usage credits* — opt-in, **OFF by default**, needs an explicit human click; "All transitions to API credit usage require explicit user consent" | the **account-settings latch**, upstream of all our code — not a per-token burn problem | HIGH |
| **Safe autonomous restart while asleep?** | **NO — not safely possible.** Warm-resume-on-return is the safe design (~80% already built) | no reset API + no sleep-surviving local scheduler + cloud routine = unsupervised cold agent on the same window | HIGH |
| **Safety to codify?** | extra-usage/unlimited-spend → always-gated list · 50M ceiling = bug · migrate to `/usage` | — | adopt now |

---

## 1. THE CEILING — true safe agent-concurrency (and what actually binds)

**Verdict: 8 concurrent read-only research agents steady-state, brief burst to 10.** The three caps line up almost exactly and the alignment is correct, not a problem to fight:

| Layer | The number | Where defined | Binds first? |
|---|---|---|---|
| **Workflow runtime cap** | **8** on this machine (`min(16, cores−2)` = min(16, 8); harness absolute = 16) | Claude Code dynamic-workflows `parallel()` auto-cap | the *engine* ceiling — caps any single fan-out |
| **A21 BG discipline cap** | **8** in-flight (floor 7; raised 5→8 on 2026-05-28, reaffirmed 8 on 2026-05-29) | `scripts/dispatch-bg.sh:248–251`; safety basis = file-zone disjointness | the *discipline* ceiling |
| **Machine RAM (empirical)** | **~8–10 steady / ~12–14 hard** before swap exhaustion | live `vm_stat`/`top` sample | **RAM binds first** |
| **CPU** | does **not** bind — 80% idle at full working set | `top -l 1` | never (agents are I/O-bound) |
| **API / 5hr window** | **not binding now** (stale window, reset already passed) | `claude-5hr-window.py status` | only under sustained heavy burn |

**The binding constraint is RAM.** On this Apple M4 (10 cores, 24 GB) with the normal app stack open (Claude desktop + VS Code + Chrome own most of the 23 GB used), only **~234 MB is truly free** and **swap is 78% used** — the machine is already leaning hard on compression + swap. The marginal-agent math: ~1.5–2 GB safe-to-consume budget ÷ ~300–400 MB per research agent = **~4–6 more agents fit on top of the current 4**, landing the total at ~8–10. Past ~12–14 context-heavy agents the box thrashes/jetsams. **The harness's absolute 16-wide ceiling is a *software* number this *hardware* cannot cash** with apps open.

**Two myths the source corrected:**
- The "~25 agents running now" framing was **off by ~6×** — the real runtime-agent count was **4** (one 790 MB active 1M-context main loop + idle/forked subagents). `ps | grep claude` counts the Electron desktop app + VS Code helpers, which are *not* agents. Clean filter: `ps -axo command | grep 'MacOS/claude --output-format' | grep -v Helpers/disclaimer`.
- **CPU is a non-issue** — load < cores, 80% idle. Agent work is network/IO-bound (waiting on the API), exactly as the dispatch-bg floor-note already asserts.

**To scale past 8–10 the lever is NOT this machine — it's the cluster** (G12/G13/RH-018/RH-019: M2 + Pi nodes, each carrying its own single-machine RAM math). The 16-wide harness ceiling is reachable *across nodes*, not on one M4. Closing the apps raises this box toward 12–14, but the Workflow cap still holds any single fan-out at 8.

---

## 2. THE STATS — burn rate, throughput, tokens-per-agent

**Burn-rate basis: `effective_tokens = input + output + cache_creation`** (excludes the ~10%-priced `cache_read` — the "free read"). All rates below are grounded in real `token-metrics.json` data (de-duplicated for a 7-session idle-resume artifact cluster → 20 distinct sessions).

**Empirical burn signatures:**
- **Sustained single-stream** (one orchestrator working continuously, n=8): median **73.4k tok/min**, mean 59.6k tok/min.
- **Parallel-fanout bursts** (dur <30m, ≥300 msgs, n=9): **0.9–4.6M tok/min** in tight wall-clock windows — the real-world full-send fingerprint.
- **Per-message** (sessions ≥100 msgs): median 16.8k, p90 45.6k, max 53.3k effective-tokens.

**Full-send (~25 concurrent agents) modeled** = per-agent sustained rate × 25 × 0.65 concurrency-derate (staggered launch, IO waits, slot refill):

| Per-agent | Raw 25× | Derated ×0.65 |
|---|---|---|
| LOW 30k/min | 45M/hr | **29M/hr** |
| **MED 60k/min** | 90M/hr | **58M/hr ← central** |
| HIGH 90k/min | 135M/hr | **88M/hr** |

**Central sustained estimate: ~58M effective-tokens/hr.** Sanity: 0.97M/min sustained sits correctly *below* the instantaneous bursts (0.9–4.6M/min) and *above* a single orchestrator.

**Tokens per agent:** each subagent/workflow-agent carries its **own ~200K-token context window** — own system prompt, re-reads its own files, **no shared cache across agents**. So:
- **Fan-out MULTIPLIES tokens** — 26 agents reading the same big files ≈ 26× the read cost.
- **…but fan-out is wall-clock-DIVISIVE** — those 26 finish in ~1× wall-clock. **Parallelism trades (free-gift) tokens for time + breadth.** That is the *entire* economic case for spending the gift on swarms.

**The non-intuitive cost amplifiers** (from token-economics): (1) **slow pacing → cache misses** — the prompt-cache TTL is **5 minutes**; a turn fired >5 min after the last re-creates the *entire* conversation cache at full input price (idling between turns is sneakily expensive, the "worst-of-both"); (2) **output costs ~4–5× input** — generating/writing is the expensive part, not reading; (3) **huge files re-read every turn**, **long conversations compound**, **runaway loops**. Cheap levers: tight pacing (<5 min/turn), read-heavy/write-light, bounded fan-out on *genuine* work (not N× re-runs of the same queue — a real waste example: an args bug re-ran one queue 3× = 3× cost for 1× value).

> **Honesty caveat carried forward:** the weekly *gift envelope* is **inferred, not metered** (no weekly tracker built). The burn *rate* is solidly grounded; the gift *size* is a modeled 0.40–1.09B-effective-token range (central ~0.72B), anchored on "our 7-day burn ≈ the 20% already spent." At central, **18h of continuous full-send EXHAUSTS the remaining gift in 5 of 9 scenario cells (7 of 9 ≥72%)** — full-send is gift-scale, not session-scale. The realistic crossover: **keep cumulative full-send under ~10 hours per ~18h window** to stay inside the gift with margin.

---

## 3. THE RESET STRUCTURE + the FREE→PAID boundary (the money red line)

**THREE distinct limits — only ONE can cost money, and it is OFF by default:**

| # | Limit | What it is | Reset | Hitting it | Cost |
|---|---|---|---|---|---|
| 1 | **5-hour rolling window** | short throttle; starts on your *first* message, rolling per-session so the reset time drifts | ~5h after first message | throttled for the rest of the window | **FREE** (a pause) |
| 2 | **Weekly cap ×2** = "the gift" | (a) all-models cap + (b) a separate, tighter **Opus-only** cap; introduced 2025-08-28; +50% through 2026-07-13 | weekly cadence (exact timing account-specific — read from `/usage`, don't hard-code a weekday) | locked out of the affected model(s) until the weekly reset; waiting 5h does **not** help | **FREE** (a wall) |
| 3 | **Extra usage / usage credits** | opt-in pay-as-you-go overflow at **standard API rates**, billed on top of the subscription — **the "$70 overage"** | n/a | continues billing overflow | **PAID — the ONLY paid path** |

**The exact gate (where FREE crosses to PAID):**

```
plan limit reached (5h OR weekly)
        │
        ▼  Is "extra usage" enabled in Settings?
   NO ──┴── YES
   │          │  funds + under monthly spend cap?
   ▼     NO ──┴── YES
 BLOCKED.  BLOCKED.  continues, billing @ API rate  ← THE $ CROSSING
 $0 spent. $0 spent. (separate charge)
 (the wall)
```

**Two independent latches** stand between a plow and a dollar: (1) extra-usage enabled? (off by default, needs an explicit click) (2) monthly spend cap (unless "set to unlimited"). A plow crosses into paid **only if BOTH** (1) is on AND (2) is unlimited-or-not-yet-hit. **If (1) is off, the plow blocks at $0 regardless of anything our code does.** Anthropic confirms twice: *"To enable usage credits… Click 'Enable'"* and *"All transitions to API credit usage require explicit user consent."*

**Therefore the structural guarantee is a config invariant, not a runtime-burn problem.** The risk is *not* "the plow secretly spends money" — it's **"a human (or me, on a misread instruction) enables extra usage once and forgets it's on,"** after which every plow silently bills after the plan runs dry. The safety work is about **that latch**, not per-token burn.

**Adjacent gotcha (not our boundary, but a separate always-metered path):** since 2026-04-04, using Claude *through third-party tools* or the **raw API (a stray `ANTHROPIC_API_KEY`/BYOK)** bills at API rates with **no plan-limit shield at all** — the documented "$1,800 trap" and the most likely "almost instantly burned credits" incident. Our work runs *inside Claude Code* so it stays flat-rate; but if any part of the system ever shells out to the raw API, that's a separate money leak. **Never BYOK during the gift.**

---

## 4. THE RECOVERY VERDICT — no safe autonomous restart-while-asleep

**Honest verdict: a fully unattended, no-human-in-the-loop auto-restart of this session after a 5hr reset, while the user sleeps, is NOT safely possible in this environment — and should not be attempted.** Three independent blockers, any one fatal:

1. **No reset API exists.** Anthropic does not expose the reset time; we only *infer* it (`detect-5hr-reset.py`), and the inference drifts — the live `5hr-window-reset.txt` was **9.4h stale** at research time. *You cannot build a safe autonomous trigger on a clock you cannot read.* A timestamp is never permission.
2. **No local scheduler survives sleep.** `CronCreate` (in-session) and `mcp__scheduled-tasks` both require the app open + machine awake — scheduled-tasks' own docs say it "runs on next launch," i.e. when the human reopens. `ScheduleWakeup` **does not exist in this environment** (absent from the tool list, un-loadable). The closest real analog, `launchd` + `pmset wake`, can wake the Mac but only runs a *script*/cold headless agent on the same window, unsupervised — and is **double-gated by A66**.
3. **The one cloud path (`RemoteTrigger`/claude.ai routines) is a *different* agent, not a resume.** It runs server-side (the only "fires while asleep" = yes), but it's a **cold, memoryless agent** with no access to this repo, no Chrome/Simulator/Android/Playwright matrix (the entire local verification stack for a web-design project), runs **unsupervised**, and **almost certainly shares the same account-level 5hr window** — so it either re-hits the wall or does unreviewed work on the freshly-reset bucket the user wakes up to. Creating one is itself A66-gated.

**What IS safe (and ~80% already built): "Warm Resume, Human-Gated."**
`detect ceiling → snapshot (`graceful-shutdown-snapshot.sh`, <2s, no LLM, round-trip-tested) → ONE PushNotification → [sleep; NOTHING fires; window resets untouched] → user reopens app (the only trigger — structurally guarantees "user present") → SessionStart fires `graceful-startup-resume.sh` → agent's first response resumes WARM, quoting NS phase + topic + in-flight count + pending decisions.` **The agent re-initiates the *work context* the instant a human returns; it never re-initiates the *session itself* unattended.** This mirrors the user's own graceful-shutdown gate ("NO automatic shutdown. User triggers.").

**Eight mandatory kill-conditions** gate the resume (any one → abort to "wait for explicit human go"): K1 user-present (structurally guaranteed for the autonomous case since autonomous paths aren't used) · K2 never-paid/entitlement (first-call 401/403 → abort, no retry-loop) · K3 bounded to N=1 autonomous resume per wall-event · K4 hard-stop at gift-end / window-not-actually-reset (never resume on the stale txt alone) · K5 snapshot stale >threshold · K6 resume-already-fired (idempotency) · K7 A66 high-risk action queued in the resume → surface, don't auto-execute · K8 bottleneck/cost signal → surface-only mode. **Meta-rule binding all eight: a timestamp is never permission; a human's presence (a real SessionStart) or an explicit "continue" is.**

**The two small, safe, PROPOSE-grade additions** (user-gated to install — settings.json is edit-guarded): (1) `recovery-state.json` + a kill-condition guard inside `graceful-startup-resume.sh` (~40 lines, no LLM); (2) a pre-wall snapshot trigger (extend the token-budget hook so crossing ~95% fires the snapshot + one push). Everything else exists and is round-trip-tested. **The single highest-leverage step:** wiring `graceful-startup-resume.sh` into SessionStart (a settings.json hook edit, user-gated) — without it even the human-gated warm resume doesn't auto-emit its briefing on reopen.

---

## 5. THE SAFETY CODIFICATION TO ADOPT

Three changes the synthesis recommends (all surfaced for user authorization — none auto-applied; the third is itself a memory-rule + settings concern, both gated):

### 5a. **Extend the always-gated list — the money latch** (HIGHEST leverage; 99% of the financial safety)
Add to `feedback_decision-handling-discipline`'s **High-risk-always-gated list** (currently 9 items: src/ edits · settings.json · remote push · `--update-snapshots` · LaunchAgent installs · mass deletes · chamber bypass · settings hooks deploy · memory-rule deletion):

> - **Enabling extra usage** (via `/extra-usage` or Settings > Usage > Enable)
> - **Setting the monthly spend cap to "unlimited"**
> - **Setting `ANTHROPIC_API_KEY` / any BYOK or raw-API path** during a gift period

**Rationale:** these are the *only* actions that can cross FREE→PAID. The agent must **never** enable extra usage, select unlimited, or set an API key on inferred permission — exactly the A66 class ("scope of directive ≠ scope of permission"). **Disabling extra usage is always allowed** (safe direction). Pair with a daily/pre-plow re-confirmation surface before any unattended multi-hour run: *"Confirm extra-usage is still OFF at claude.ai/settings/usage."* — since there is **no documented CLI to read the toggle state programmatically**, the honest mechanism is a human re-confirm prompt, not an automated read.

### 5b. **The fictional 50M ceiling is a KNOWN BUG** (bug-billboard candidate)
`token-metrics.json` carries `ceiling_estimate: 50000000`, `source: "default"` — a **made-up round number, 2–3 orders of magnitude off** the real per-window allowance, and `source` literally says `default` = never calibrated. **Every heartbeat "headroom / ceiling in N min" number derived from it is theater.** It does NOT open a money leak (the real guard is the account latch upstream), but it makes our predictions useless and it implicitly conflates a *planning aid* with a *financial guard* by naming things "budget"/"ceiling." Fix: either ground it in the real per-window allowance OR delete the ceiling concept and track **percentage-of-window from `/usage`** instead (cleaner — the weekly cap is partly measured in *hours*, which tokens can't represent). **Not yet on the bug billboard** (confirmed by grep). **Naming-honesty pass:** rename token "budget/ceiling" → "token-volume-planning" so neither human nor agent mistakes it for the money guard.

### 5c. **Migrate to the `/usage` command** as the source of truth (plan candidate)
Claude Code now exposes `/usage` (shipped ~late-April 2026) showing **both windows' usage %, remaining messages, and exact reset timestamps** for the 5h *and* both weekly caps (incl. Opus-only). Our entire local apparatus *infers* what the tool now just *tells us*:
- `detect-5hr-reset.py` (token-discontinuity inference) → demote to **fallback**; primary reset-time source becomes the `/usage` timestamps (read once, cached).
- Stop maintaining a guessed reset in `5hr-window-reset.txt` when `/usage` reports it directly.
- Re-architect `check-token-budget.sh` + `claude-5hr-window.py` around `/usage` output, including the **Opus-only weekly cap** so the plow can do **model-aware degradation** — when the Opus weekly cap nears exhaustion, downgrade *commodity* work to Sonnet (the deliberate documented cost-downgrade path already sanctioned in `feedback_model-routing`) rather than halt. **Wire the plow to wind down gracefully at the free wall** (finish in-flight unit, emit sentinel, stop dispatching new BGs at ≥85% of any window) instead of thrashing.

**Additional structural facts to remember (no action, just don't re-litigate):**
- Our local token apparatus is a **planning convenience, NOT a financial safety system.** The financial safety lives entirely in one account-settings toggle. Don't conflate them.
- The weekly **Opus-only** cap is the limit most likely to halt a multi-hour Opus plow — track it specifically.
- Plan mechanics change often (Anthropic shipped major changes Aug 2025, Apr 2026, May 2026). Treat dollar figures + hour ranges as as-of-date; re-verify before relying on a specific number. The **structural facts** (opt-in overage, two-tier limits, what blocks) are stable.

---

## 6. Cross-references

- **Source meta-findings** — `context/markdowns/research/systems/{true-concurrency-ceiling, token-burn-gift-boundary, fivehr-weekly-paid-boundary, safe-recovery-while-asleep, maximize-4.8-agent-swarm, token-economics-and-cost-drivers}.md`
- **`scripts/dispatch-bg.sh:248–251`** — the live A21 ceiling (8) + floor (7) + zone-disjointness safety basis
- **`.claude/workflows/research-furthering.js`** — the `parallel()` auto-refill keeper the ceiling governs (A80/A81 never-idle band)
- **`scripts/{claude-5hr-window.py, detect-5hr-reset.py}`** + `scripts/hooks/token-budget-check.sh` + `scripts/heartbeat/check-token-budget.sh` — the local token apparatus to migrate to `/usage`
- **`scripts/{graceful-shutdown-snapshot.sh, graceful-startup-resume.sh, test-shutdown-resume-roundtrip.sh}`** — the ~80%-built warm-resume machinery
- **Memory:** `feedback_decision-handling-discipline` (the always-gated list — §5a extends it) · `feedback_bottleneck-restriction-chant` (the governing cost ceiling — full-send must never silently become the gift bottleneck) · `feedback_model-routing` (Opus→Sonnet deliberate downgrade) · `feedback_true-time-discipline` (the 9.4h-stale file is the canonical "timestamp ≠ truth" case) · `feedback_verified-counts`
- **Audits:** A17 (5hr-reset alerting) · A21 (parallel capacity) · A23 (time-precision/reset inference) · A30 (graceful shutdown/startup) · A36/A37 (true-time) · A47 (BG lifecycle/auto-stop) · A66 (decision-handling — the always-gated frontier) · A72 (cost comparison/money tracking) · A74 (4.8 birdseye) · A75 (workflow 4.7-vs-4.8 mechanics) · A80/A81 (never-idle)

## 7. Spawn-worthy follow-ups (surfaced, NOT auto-created — all user-gated)

1. **STANDING-PROTOCOL addition** (§5a) — extend the always-gated list with enable-extra-usage / unlimited-spend / BYOK; add the daily pre-plow "confirm extra-usage OFF" surface. *Closes the only real money-leak latch.* (Memory-rule edit + the surface are both gated.)
2. **BUG (billboard)** (§5b) — `ceiling_estimate: 50000000 / source:default` is fictional; every heartbeat headroom number derived from it is theater. Calibrate to the real per-window allowance or switch to `/usage`-percentage tracking. (Not yet on the billboard.)
3. **PLAN** (§5c) — re-architect `check-token-budget.sh` + `claude-5hr-window.py` around `/usage` (real % + real reset timestamps + both weekly caps incl. Opus-only); retire inferred-reset to fallback; wire graceful wind-down at the free wall + Opus→Sonnet model-aware degradation.
4. **WIRING** (recovery) — install the SessionStart hook for `graceful-startup-resume.sh` + the 2 PROPOSE-grade additions (`recovery-state.json` guard + pre-wall snapshot trigger). Highest-leverage step to make warm-resume real. (settings.json edit — gated.)
5. **CALIBRATION** (one-time) — the units mismatch is unresolved: Anthropic's published window figures (~44k–220k) are not in our `effective_tokens` units, so neither the per-session ceiling nor the weekly gift is anchored to a real plan limit. Note `effective_tokens` at the moment a real session/weekly limit actually fires to anchor everything.
6. **STALE FILE** (immediate, low-effort) — `5hr-window-reset.txt` was 9.4h in the past at research time; whoever owns burn-tracking should run `python3 scripts/claude-5hr-window.py set <next-reset-iso>` (or better, read `/usage`) so the burn-headroom signal is live again.

---

*READ-ONLY synthesis. No code deployed, no scheduling object created, no settings edited, no agent restarted, no git mutation. This document is the only file written. The three codification changes (§5) and all follow-ups (§7) are surfaced for user authorization — none applied; §5a (memory-rule) and §5c/§7.4 (settings.json) are themselves on the gated list.*

BG-COMPLETE-SENTINEL
