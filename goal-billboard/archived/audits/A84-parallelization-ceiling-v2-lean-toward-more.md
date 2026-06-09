---
audit_id: A84
title: Parallelization ceiling v2 — the cloud-light correction, biased toward MORE
status: completed
catalogued: 2026-05-29
synthesized_utc: 2026-05-29T05:15:00Z
trigger: >
  2026-05-29 — user: SECOND-PASS parallelization-ceiling audit that LEANS TOWARD INCREASING
  parallelization, not being stingy. Context: we just corrected a wrong "~8-10 RAM ceiling" that
  had been throttling CLOUD-LIGHT research/verify swarms. That ceiling only applies to LOCAL-HEAVY
  work (Playwright / iOS sims / Android emulators). Cloud-light agents (read-only research / verify /
  light script-build) are SERVER-SIDE — locally just file I/O. Empirical with 3 concurrent swarms
  (~24 agents intended) live: load 2.51/10, 80% mem free. We have been far too stingy.
method: >
  READ-ONLY. Live sysctl / vm_stat / swap / ps sampling of the machine UNDER cloud-light load,
  plus source-of-truth re-read of the two ceiling docs (true-concurrency-ceiling.md,
  swarm-maximization-v2.md), the corrected plan section (full-plow-v2-plan.md §3.3), the K-lane
  architecture (full-plow-v2-architecture.md §1.2), the RH-014 6-signal escalation framework, and
  the 5hr/weekly/paid boundary doc. No code deployed, no settings edited, no scheduling object
  created, no git mutation, no agent restarted.
serves: >
  capacity policy (the cloud-light concurrency posture) · A21 cap (cloud-light vs local-heavy split) ·
  full-plow-v2 (the K-lane ceiling correction) · A80/A81 never-idle band · the gift period (G17 cluster
  is for local-heavy, NOT cloud-light) · user-safety (the 5hr/weekly window is the real cloud-light wall)
sources:
  - context/markdowns/research/systems/true-concurrency-ceiling.md            # the "RAM binds at 8-10" verdict being CORRECTED for cloud-light
  - context/markdowns/research/systems/swarm-maximization-v2.md               # the multi-workflow "K×8 thrashes" claim — true for local-heavy, wrong for cloud-light
  - context/markdowns/plans/full-plow-v2-plan.md                              # §3.3 the corrected ceiling (user-caught) — A84 formalizes it
  - context/markdowns/plans/full-plow-v2-architecture.md                      # §1.2 the K-lane "Σ agents ≤ 8" math, drawn from local-heavy samples
  - context/markdowns/research/rabbit-holes/RH-014-async-throughput-and-token-saves/findings/parallelization-safe-escalation-criteria.md  # the 6-signal escalation governance
  - context/markdowns/research/systems/fivehr-weekly-paid-boundary.md         # the API window — the REAL cloud-light wall
  - scripts/dispatch-bg.sh                                                    # the A21 live cap (8 in-flight, floor 7)
  - scripts/sample-resources.sh                                               # the resource-sampling substrate (Layer A)
related_audits: [A21, A47, A58, A80, A81, A82]
related_memory: [feedback_bottleneck-restriction-chant, feedback_model-routing, feedback_verified-counts, feedback_idle-capacity-furthering, feedback_autonomic-system]
supersedes_for_cloud_light: >
  A82 §1 + true-concurrency-ceiling.md verdict + full-plow-v2-architecture.md §1.2 — those conclude
  "8 total, RAM-bound, cluster is the only lever past it." That is CORRECT for LOCAL-HEAVY work and
  WRONG for cloud-light. A84 splits the verdict by work class. The RAM ceiling docs are not deleted —
  they are re-scoped to local-heavy, exactly as full-plow-v2-plan.md §3.3 already began.
---

# A84 — Parallelization ceiling v2: the cloud-light correction, biased toward MORE

> **The whole thing in six sentences.** (1) The prior "**~8-10 agents, RAM binds first**" ceiling (A82 / true-concurrency-ceiling / architecture §1.2) was measured during **LOCAL-HEAVY moments** — the resource-samples log literally shows the "tight" reading was taken with **8 Playwright procs + 39 Chrome procs in flight** (a matrix run), not a research swarm. (2) **Cloud-light agents** — read-only research, verify, light script-build — run their inference on **Anthropic's servers**; locally each is **just file I/O + a 108-404 MB context process**, NOT a browser/sim/emulator. (3) **Live right now, cloud-light only:** 6 claude runtime procs, **load 2.55/10**, **11.2 GB reclaimable RAM**, 1.6 GB compressor, ~71% CPU idle — the machine is **nowhere near any local wall**. (4) For cloud-light work, **RAM is NOT the binding constraint** — the binding constraint as K grows is the **API/5hr-rolling-window + weekly-Opus cap** (the user has hit it before), with **token-burn-rate** as the leading indicator and **load > cores** as the only local backstop. (5) The new posture: **default to running MANY concurrent cloud-light workflows on this ONE M4** — not K=1 — escalating K until an API 429 / >80% window-burn / load > cores, with **only LOCAL-HEAVY work respecting the ~8-10 RAM ceiling** (and the cluster being for *local-heavy* distribution, not cloud-light). (6) Concrete: **cloud-light TOTAL-agent ceiling on this M4 = ~40-50 (start there), escalate by the protocol; per-workflow stays 8; so K_cloud-light ≈ 5-6 concurrent workflows** is safe to *start* and push from — versus local-heavy which stays at **Σ ≤ 8-10 total**.

---

## 0. The one-screen verdict table

| Question | Cloud-light answer (the correction) | Local-heavy answer (unchanged) |
|---|---|---|
| **What binds first as K grows?** | **API / 5hr-rolling-window + weekly-Opus cap** — then load > cores as a distant local backstop. RAM does NOT bind (each agent is server-side inference + a small local I/O process). | **RAM** — real browser/sim/emulator processes on this Mac's CPU/GPU/RAM. ~8-10 total before swap-thrash. |
| **Per-workflow concurrency cap** | **8** (`min(16, cores−2)` — the harness auto-cap; unchanged) | **8**, but you rarely reach it — 8 browsers already strains RAM |
| **Concurrent workflows K** | **5-6 to START, escalate higher** — bounded by API/tokens, not RAM | **K = 1** (one wide lane saturates RAM) |
| **TOTAL concurrent agents** | **~40-50 to start** (K×8), push until a real wall trips | **~8-10 total** (the historical ceiling — correct HERE) |
| **The real wall** | the **token window** (5hr + weekly-Opus) — free but finite; the gift | **local RAM** → swap → jetsam/thrash |
| **Is the cluster the lever?** | **NO** — one M4 fans cloud-light out fine; the cluster doesn't help (the bottleneck is the shared Anthropic-side window, which the cluster *also* draws from) | **YES** — distribute browsers/sims across nodes (G17/RH-018/RH-019) |
| **Default posture** | **bias to MORE**: run many lanes, escalate K, back off only on 429 / >80% burn / load > cores | **bias to careful**: Σ ≤ 8-10, one wide lane, cluster for more |

**The one-sentence correction:** the system was calibrated to a **single RAM number (8)** that is **correct for local-heavy and 5-6× too stingy for cloud-light** — because the "tight RAM" samples that produced it were taken while a Playwright matrix (8 browsers + 39 Chrome renderers) was running, not while research agents were.

---

## 1. The evidence that the prior ceiling was a LOCAL-HEAVY measurement

This is the load-bearing finding. The prior docs assert "RAM binds at ~8-10, swap 86% used, only 234 MB free." That reading is **real but mis-attributed**. The proof is in our own `reports/stats/resource-samples.jsonl` — the last sample before this audit:

```json
{"ts_iso":"2026-05-29T02:22:00Z","load1":3.11,"mem_pct_used":74.0,"swap_used_bytes":9437462528,
 "by_class":{"node-astro":{"n":2},"playwright":{"n":8,...},"chrome":{"n":39,"pmem":17.0},...},
 "work_in_flight":["playwright"],...}
```

**That "tight" sample had 8 Playwright worker procs + 39 Chrome renderer procs live** (`work_in_flight: ["playwright"]` — a matrix run). Chrome alone was eating 17% of memory across 39 procs. THAT is what consumed the RAM and drove swap to 9.4 GB — **the local-heavy test matrix, not a research swarm.** The true-concurrency-ceiling doc's own sample ("swap 86% used, 6 runtime procs") was taken on the same kind of loaded desktop (Claude app + VS Code + Chrome all open, per its own caveat in §2).

### Live sample taken FOR this audit (cloud-light load only, no matrix running)

`sysctl` / `vm_stat` / `swap` / `ps`, 2026-05-29 ~05:05-05:15 UTC, while this research swarm + sibling cloud-light work was the only agent load:

```
Chip / cores / RAM:   Apple M4 (Mac16,13) · 10 cores (4P+6E) · 24 GB
Load avg:             2.55 / 2.72 / 2.77   (against 10 cores → ~74% of a single core's worth of run-queue; CPU 71% IDLE)
PhysMem (top -l1):    19G used · 3028M wired · 1611M compressor · 4080M unused
Reclaimable RAM:      truly-free 2199 MB + inactive 8014 MB + speculative 854 MB + purgeable 151 MB
                      = 11,218 MB genuinely available
Compressor:           1611 MB (vs 6700 MB in the local-heavy sample) — 4× lower; almost no memory pressure
Real claude runtime procs (filtered): 6
```

**Cloud-light agent RSS profile** (the actual per-agent local cost — the number the ceiling math should use):

| Proc | RSS | Role |
|---|---|---|
| 5 subagent/idle sessions | **108, 109, 218, 239, 392, 404 MB** | cloud-light agents — median ~240 MB |
| 1 main 1M-context loop | **942 MB** | the orchestrator (the only heavyweight; counts as ~2-3 agents' worth) |
| **Sum of ALL runtime procs** | **2,412 MB** | the entire agent footprint |

**The contrast is decisive.** The prior docs used **790 MB/agent** as the per-agent RAM cost — but that is **the 1M-context MAIN loop**, not a cloud-light agent. A cloud-light research/verify agent is **108-404 MB** (median ~240 MB). The ceiling math was off by **~3× on the per-agent cost AND attributed a matrix run's RAM to research swarms** — compounding into a 5-6× under-estimate of how many cloud-light agents fit.

### Why cloud-light is server-side (the mechanism)

A cloud-light agent does its LLM inference on **Anthropic's servers**. Locally it is:
- a context-holding process (108-404 MB), and
- file I/O (reads source files, writes one findings artifact).

It spawns **no browser, no simulator, no emulator, no qemu** — none of the real local-compute processes that the `sample-resources.sh` cohort regexes track as `playwright` / `simulator` / `android-emu`. Those local-heavy classes are what historically caused "burn" (the 753% emulator leak, the 39-Chrome matrix). **Cloud-light work has none of that local footprint** — which is exactly why load is 2.55 and 11 GB is reclaimable while 6 agents run.

---

## 2. What ACTUALLY binds cloud-light as K grows (the four candidate walls, ranked)

The brief asks precisely this: as K (concurrent workflows) grows, which of the four limits binds first? Here is the ranking, **for cloud-light work on this M4**:

### Wall 1 (BINDS FIRST) — API / 5hr-rolling-window + weekly-Opus cap

This is the real ceiling for cloud-light. From `fivehr-weekly-paid-boundary.md` (and A82's synthesis):
- **5-hour rolling window** — short-term throttle; resets ~5h after first use in a session.
- **Weekly cap ×2** — an all-models cap **and a tighter Opus-only cap**. A heavy Opus swarm exhausts the **Opus-only** weekly cap first. This is "the gift" (raised +50% through 2026-07-13).
- **Hitting it blocks you until the weekly reset** — waiting 5h does NOT help.

**The user has hit 5hr-window rate limits before** (stated in the brief; corroborated by the window substrate existing at all). When K cloud-light workflows each run 8 agents, each agent burns ~200K-token context windows; **total burn = K × 8 × per-agent-rate**, which scales linearly with K and hits the window long before any local resource does. A82 measured full-send central burn at **~58M effective-tok/hr (29-135M/hr band)** — one pinned full-send day ≈ a normal week. **This is the wall.**

> Current state: the 5hr-window file is **stale** (reset 11h39m in the past — same staleness A82 and true-concurrency-ceiling flagged). So *right now* there is no active burn pressure — but that is a blind instrument, not headroom. **Open item carried forward (see §6): set the window from `/usage` so the real wall is visible during escalation.**

### Wall 2 — Token budget (the gift; the same window, viewed as a quota)

Not a separate wall from Wall 1 so much as its accounting face: the weekly Opus pool is finite-but-generous. The `feedback_bottleneck-restriction-chant` hard gate — **skip dispatch at >80% window-burn even in gift mode** — is the governor here. Cloud-light's "freedom to fan wide" is bounded by *spend the gift, don't exhaust it before the week's real work*.

### Wall 3 (distant) — Local CPU (load > cores)

Cloud-light agents are I/O- and network-bound; they barely touch CPU (load 2.55 with 6 agents). CPU would only bind if **K × 8 agents each did heavy local post-processing** (large file parses, JSON crunching) simultaneously — possible but rare. The backstop signal: **load1 > ncpu (10)** sustained. We are at 2.55; there is room for **~4× more agents** before load even approaches cores.

### Wall 4 (effectively never, for cloud-light) — Local RAM

The historical "ceiling." For cloud-light: 11.2 GB reclaimable ÷ ~240 MB/agent = **~46 more agents** fit *on top of the current 6* before RAM is even half-consumed — and that ignores that idle agents compress to ~108 MB. RAM binds cloud-light only if you ran **~50+ context-heavy agents simultaneously**, which the API window (Wall 1) stops you from reaching first. **For cloud-light, RAM is not the wall — the window is.**

### The ranked answer

```
Cloud-light, as K grows:   API-window (1)  →  token-budget (2, same window)  →  load>cores (3, distant)  →  RAM (4, never reached first)
Local-heavy, as agents grow: RAM (1)  →  CPU (2, far)  →  API-window (3, only on sustained burn)
```

**The binding constraint FLIPS by work class.** That is the entire correction. The prior docs collapsed both classes onto RAM because their samples were contaminated by local-heavy load.

---

## 3. The empirical escalation-test protocol (how to safely push K up on GENUINE work)

The brief asks for a protocol to push K from 3 → 4 → 6 → 8 concurrent cloud-light workflows on real work, sampling at each level. Here it is. **It rides the existing substrate** (`sample-resources.sh` already emits load/mem/swap/by-class; `claude-5hr-window.py` tracks burn; RH-014 defines the signals).

### Pre-flight (once, before escalating)
1. **Set the window from truth:** `python3 scripts/claude-5hr-window.py set <next-reset-iso>` (read from `/usage`). Without this, Wall 1 is blind — do NOT escalate against a stale window.
2. **Confirm work is GENUINE and cloud-light:** each lane = N disjoint-input read-only units (the make-work guard — never pad K to a target). If a lane spawns browsers/sims, it is LOCAL-HEAVY and does NOT get this protocol — it respects the RAM ceiling.
3. **Baseline sample:** capture load1/5/15, mem_free_pct, swap_used_bytes, compressor MB, runtime-proc count, tokens/min (from window burn-rate). Record it.

### The escalation ladder (K = 3 → 4 → 6 → 8 concurrent workflows)

At each K, hold for a **sampling dwell** (≥ 2-3 minutes of steady-state with all lanes' agents actually running, not just launched), then sample:

| Metric | Source | GREEN (keep climbing) | YELLOW (hold, observe) | RED (back off one step) |
|---|---|---|---|---|
| **API 429s / window-blocks** | dispatch errors + `claude-5hr-window.py status` | 0 | — | **any 429 / "limit reached"** → immediate back-off (Wall 1) |
| **Window burn %** | `claude-5hr-window.py` (5hr + weekly-Opus) | <60% | 60-80% | **>80%** → STOP climbing (chant gate) |
| **tokens/min** | window burn-rate delta | rising linearly with K (healthy — work is flowing) | plateau despite +K (lanes starving / queue dry) | n/a (rate isn't a danger signal; the *window %* is) |
| **load1** | `sysctl vm.loadavg` / sample | < cores−2 (≤8) | cores−2 .. cores (8-10) | **> cores (>10)** sustained → back off (Wall 3) |
| **mem_free_pct** | `sample-resources.sh` | ≥25% | 10-25% | **<10%** (would mean cloud-light unexpectedly heavy — investigate, likely a local-heavy lane snuck in) |
| **swap delta** | `swap_used_bytes` across samples | flat / falling | slow rise | **rising every sample** (thrash — again, suspect a local-heavy lane) |
| **agent wall-clock latency** | dispatch→complete time per agent | flat as K rises | creeping up | **climbing sharply** (agents waiting on the window = Wall 1 in disguise) |

### The TRUE degradation signal (the one that marks the real wall)

> **The true degradation point for cloud-light is the FIRST of: (a) an API 429 / window-block, (b) window-burn crossing 80%, or (c) agent wall-clock latency climbing sharply while load/RAM stay flat** — because (c) means agents are queuing on the *server-side window*, not on any local resource. **Load and swap are NOT the cloud-light degradation signal** (they stay flat — that is the whole point of the correction); they are only the backstop that catches a misclassified local-heavy lane. When you see local resources stay calm but latency/burn climb, **you have found the API wall — that is the ceiling, stop there.**

### Sampling cadence
- Sample at **each K level after the dwell**, plus **once mid-dwell** to catch a fast spike.
- The `sample-resources.sh` self-gate only fires when local-heavy work is in flight — for a pure cloud-light escalation it will report "idle" (no playwright/sim/emu). Use **`--force`** during the test to get a row regardless, OR read `sysctl`/`vm_stat`/window directly (the script's gate is designed for local-heavy detection, not cloud-light — a noted limitation, see §6).

---

## 4. The new default concurrency posture (biased toward MORE)

The brief asks for a new default that leans toward more, with a concrete K for this M4. Here it is.

### The posture, stated as a rule

> **Default to running MANY concurrent cloud-light workflows on this one M4. Escalate K until the API rate-limit trips OR window-burn > 80% OR load > cores. ONLY local-heavy work (Playwright / sims / emulators) respects the ~8-10 RAM ceiling — and the cluster is for distributing THAT, not cloud-light.**

This replaces the implicit prior default ("K=1 wide lane; Σ agents ≤ 8; cluster for more") **for cloud-light work only**. Local-heavy keeps the old, correct, careful posture.

### The concrete numbers for THIS M4 (10-core / 24 GB, normal apps open)

| Knob | Cloud-light value | Basis |
|---|---|---|
| **Per-workflow cap** | **8** (`min(16, cores−2)`) | harness auto-cap — unchanged, can't be exceeded by one lane anyway |
| **TOTAL cloud-light agents — START** | **~40-50** | 11.2 GB reclaimable ÷ ~240 MB = ~46 agents fit before RAM is half-used; load has 4× headroom; START conservative-of-the-generous at 40-50 and climb |
| **K (concurrent cloud-light workflows) — START** | **5-6** | 40-50 ÷ 8 per lane ≈ 5-6 lanes. This is the *starting* posture, not the ceiling — escalate per §3 |
| **TOTAL cloud-light agents — push to** | **as high as the window allows** | the real cap is Wall 1 (API/burn), discovered empirically — could be 60-80+ before the window binds, if the work is genuine and burst-shaped |
| **Local-heavy TOTAL** | **Σ ≤ 8-10** (unchanged) | RAM-bound; the A82 / true-concurrency-ceiling verdict is CORRECT here |
| **A21 BG cap (mixed/default)** | **keep 8 floor-7 for the BG-dispatch path**; cloud-light *workflows* are a separate axis | A21 governs hand-dispatched BGs (heterogeneous, often touch files); the cloud-light K-posture governs `parallel()`/`pipeline()` workflow fan-outs (read-only, disjoint). Different mechanisms, different ceilings — do not conflate |

### Why "start at 5-6, not jump to the max"

The lean-toward-more bias does **not** mean "launch 80 agents blind." It means: **stop defaulting to 1 wide lane** (the stingy error). Start at a posture that is already **5-6× the old default** (K=5-6, ~40-50 agents), confirm the window holds via §3, and climb from there toward whatever the API wall actually is. The generosity is in the *starting point* and the *willingness to climb*; the discipline is in *measuring the window* so the climb stops at the true wall, not before it (stingy) and not after it (a blocked weekly cap).

### The make-work guard still binds (both directions)

Bias-to-more is **not** bias-to-pad. K = count of lanes that have **genuine disjoint work queued**. If only 2 genuine cloud-light jobs exist, K = 2 — launching 6 empty lanes is the make-work anti-pattern the guard forbids. The posture is: **when genuine cloud-light work exists, fan it across as many lanes as the window allows; when it doesn't, idle is correct.** (Per `feedback_bottleneck-restriction-chant` + the full-plow-v2 three-layer guard.)

---

## 5. The safe-escalation + kill protocol (backing off when a real wall is hit)

### Back-off ladder (mirror of the escalation ladder, RH-014 auto-revert generalized)

| Signal tripped | Immediate action | Then |
|---|---|---|
| **API 429 / "limit reached"** | **STOP launching new lanes immediately.** Let in-flight agents finish (do NOT kill mid-flight — they've already spent their context tokens; killing wastes the spend). | Drop K by one level (6→4→3). Re-read `/usage`. Do not re-climb until the window shows headroom. |
| **Window-burn > 80%** | **Halt all new dispatch** (the chant hard gate — applies even in gift mode). | Hold at current in-flight; let burn drain below 70% before resuming. If near a weekly-Opus cap, STOP for the day — waiting 5h won't help a weekly block. |
| **Agent wall-clock latency climbing + local calm** | This IS the API wall in disguise. Treat as a soft 429: **stop climbing K**, hold current level. | Sample the window; if burn is high, back off; if burn is fine, it may be transient server load — hold one dwell, re-sample. |
| **load1 > cores (>10) sustained** | Back off one K level (likely a local-heavy lane misclassified). | **Audit the lanes** — find the one spawning browsers/sims; it does NOT belong in the cloud-light pool. Move it to the local-heavy ceiling (Σ ≤ 8-10). |
| **mem_free < 10% / swap rising every sample** | Same as above — **a local-heavy lane snuck in.** Back off, find it. | Pure cloud-light cannot cause this; its presence means misclassification. Re-scope the lane. |

### The kill switch (when back-off isn't enough)

1. **Stop the metronome / scout** so no NEW lanes launch (the tick's genuine-work predicate going false = natural stop; manually: don't dispatch more).
2. **Let in-flight cloud-light agents drain** — they finish in their own wall-clock; their tokens are already spent. Killing them wastes spend and produces no artifact (the inverse of the goal).
3. **For a true emergency** (runaway / wrong-classification thrash): the existing `idle-test-tool-killer.sh` is the LOCAL-heavy remediation arm (kills zombie sims/emulators) — but it does **not** apply to cloud-light (there's no local process to kill beyond the agent itself). For cloud-light the only "kill" is *stop dispatching*; the agents are server-side.
4. **The chant 6-step on any bottleneck:** DETECT → PAUSE → DIAGNOSE (CoT BOTTLENECK + steering entry) → FIX at the substrate (re-scope lanes / reset window / drop K) → VERIFY (re-sample) → RESUME. Never plow through a window-block.

### What stays GATED (A66 — unchanged)
- Enabling **extra-usage / unlimited spend** to push past the free window — **always-gated** (A82's standing money red line). The cloud-light bias is "use the gift fully," NEVER "spend money to fan wider."
- Installing the launchd plow-tick / editing settings.json — gated; this audit PROPOSES the posture, the user authorizes any daemonized escalation.

---

## 6. Open items surfaced (read-only — not acted on)

1. **`sample-resources.sh` is local-heavy-biased.** Its self-gate (`work_in_flight`) only fires on playwright/simulator/android-emu/matrix-lock — so during a **pure cloud-light** escalation it reports "idle" and writes nothing unless `--force`d. It also has **no cloud-light cohort class** (no `claude`-runtime-proc counter) and **no API-window / tokens-min field**. For the §3 protocol to be self-instrumenting, the sampler needs: (a) a `claude-runtime` cohort class counting filtered claude procs, (b) a `window_burn_pct` + `tokens_per_min` field pulled from `claude-5hr-window.py`, (c) a `--cloud-light` mode whose gate is "≥1 cloud-light agent in flight." **Proposal, not a build** (A66 — sampler is infra; user authorizes).
2. **The 5hr-window file is stale** (reset 11h39m in the past — same as A82 flagged). The cloud-light wall (Wall 1) is **blind** until someone runs `python3 scripts/claude-5hr-window.py set <next-reset>` from `/usage`. Escalating K against a blind window is unsafe — **this is the single highest-priority pre-req for adopting the lean-toward-more posture.**
3. **The K-lane architecture (full-plow-v2-architecture.md §1.2) hard-codes `Σ ≤ 8` as THE ceiling** drawn from the local-heavy-contaminated sample. It already has a per-work-class hook (the table at §1.2 lines 102-105 distinguishes "all lanes read-only research" → K=1). **That row is the one to correct:** for cloud-light, "all lanes read-only research" should read **K = 5-6 start, escalate to API wall** — NOT K=1. The plan's §3.3 correction (user-caught) already states this in prose; the architecture's §1.2 table and pseudocode (`TOTAL_AGENT_CEILING = 8`) should gain a **cloud-light branch** (`TOTAL_CLOUD_LIGHT_CEILING ≈ 40-50, escalate`; `TOTAL_LOCAL_HEAVY_CEILING = 8-10`). Flagged for the full-plow-v2 build, not edited here.
4. **A82 §1 and true-concurrency-ceiling.md verdict should carry a cloud-light caveat banner** (as full-plow-v2-plan.md §3.3 already does). Their "8 total, RAM binds, cluster is the lever" verdict is correct for local-heavy and should not be read as the cloud-light ceiling. **Doc-hygiene flag** — the conclusions aren't wrong, they're unscoped.
5. **The cluster (G17) is NOT the cloud-light lever.** Both A82 and the architecture say "to scale past the ceiling, add cluster nodes." That is true for **local-heavy** (distribute browsers/sims). For **cloud-light it is FALSE** — every node draws from the *same shared Anthropic-side window* (Wall 1), so adding nodes doesn't raise the cloud-light ceiling; it just hits the shared window faster from more places. **Cloud-light scales on ONE M4 (fan wide) until the window; the cluster is for local-heavy parallelism only.** Worth stating explicitly in G17 scoping so we don't build cluster capacity expecting cloud-light throughput it can't deliver.

---

## 7. TL;DR — what to adopt

1. **Split the ceiling by work class.** Cloud-light (read-only research / verify / light script-build) is **server-side** — RAM is NOT its wall. Local-heavy (Playwright / sims / emulators) is the only class the ~8-10 RAM ceiling governs.
2. **The prior ceiling was a contaminated measurement** — the "tight RAM / 86% swap" samples were taken with a Playwright matrix (8 browsers + 39 Chrome) running, and used the 790 MB **1M-main-loop** RSS as the per-agent cost. Cloud-light agents are **108-404 MB** (median ~240). Live cloud-light-only sample: load 2.55/10, **11.2 GB reclaimable**, 71% idle.
3. **For cloud-light, the binding wall is the API/5hr-window + weekly-Opus cap** (the user has hit it), with token-burn the leading indicator and load>cores a distant backstop. RAM is reached *last*, if ever.
4. **New default posture (bias to MORE):** run **K = 5-6 concurrent cloud-light workflows to START (~40-50 total agents), escalate until 429 / >80% burn / load > cores.** Per-workflow stays 8. Local-heavy stays **Σ ≤ 8-10**.
5. **The TRUE degradation signal** is **API 429 / burn >80% / agent-latency-climbing-while-local-calm** — NOT load or swap (those stay flat for cloud-light; they only catch a misclassified local-heavy lane).
6. **Escalate on the existing substrate** (`sample-resources.sh --force` + `claude-5hr-window.py` + RH-014 signals); back off by dropping K and draining (never kill mid-flight — tokens already spent). The cluster does **not** help cloud-light (shared window).
7. **Pre-req before adopting:** **set the 5hr-window from `/usage`** — it is stale, so the real wall is currently blind. Then the lean-toward-more climb is safe.
8. **Still gated (A66):** extra-usage/unlimited-spend stays the money red line; the bias is "use the gift fully," never "pay to fan wider."

---

## Cross-references

- `context/markdowns/research/systems/true-concurrency-ceiling.md` — the "8, RAM-binds" verdict A84 RE-SCOPES to local-heavy. Its §2 caveat ("workload/app-stack dependent; on a leaner desktop the ceiling rises") already hinted at this; A84 names the mechanism (work class) and the contaminating sample.
- `context/markdowns/research/systems/swarm-maximization-v2.md` Part 2 — "K×8 thrashes swap, multi-workflow is cluster-only." TRUE for local-heavy; A84 corrects it for cloud-light (multi-workflow on one host is the *right* cloud-light shape).
- `context/markdowns/plans/full-plow-v2-plan.md` §3.3 — the user-caught correction A84 formalizes into a posture + protocol + numbers.
- `context/markdowns/plans/full-plow-v2-architecture.md` §1.2 — the `TOTAL_AGENT_CEILING = 8` pseudocode that needs a cloud-light branch (Open item 3).
- `context/markdowns/research/rabbit-holes/RH-014-async-throughput-and-token-saves/findings/parallelization-safe-escalation-criteria.md` — the 6-signal escalation governance A84's §3 protocol rides (Signal 3 burn-headroom = Wall 1; Signal 4 ghost-rate; the auto-revert ladder = §5 back-off).
- `context/markdowns/research/systems/fivehr-weekly-paid-boundary.md` — the API window (Wall 1) structure: 5hr rolling + weekly all-models + weekly-Opus + the paid latch (A66-gated).
- `context/markdowns/goal-billboard/audits/A82-true-plow-ceiling-stats-reset-recovery-audit.md` — the conservative synthesis A84 splits: A82 §1's "8 total, RAM-bound" is the local-heavy half; A84 is the cloud-light half.
- `scripts/sample-resources.sh` — the Layer-A substrate; Open item 1 proposes the cloud-light cohort + window fields it needs.
- `scripts/claude-5hr-window.py` — the Wall-1 instrument; currently stale (Open item 2 — set it before escalating).
- `scripts/dispatch-bg.sh:247` — the A21 cap (8, floor 7); a *separate axis* from the cloud-light K-posture (BG-dispatch vs workflow fan-out).
- `feedback_bottleneck-restriction-chant` (memory) — the >80%-burn hard gate that governs the climb; "never let a substrate become THE bottleneck."
- `feedback_idle-capacity-furthering` + `feedback_autonomic-system` (memory) — the never-idle band the lean-toward-more posture feeds (A80/A81); more cloud-light lanes = more genuine slots kept full.
- `feedback_model-routing` (memory) — OMIT the model param so every swarm agent inherits the session model; per-stage Fast-Mode downgrade is the per-token lever that makes wide cloud-light fan-out cheaper (swarm-maximization-v2 §1.2).
- G17 / RH-018 / RH-019 — the cluster: re-scope it as a **local-heavy** lever (Open item 5), NOT a cloud-light one (shared window).

*Read-only research + design audit. Nothing built, scheduled, deployed, or edited. The posture + protocol + the architecture/sampler corrections are PROPOSALS pending user go-ahead and (for any daemonized escalation or spend) A66 authorization. This audit file is the only artifact written.*

BG-COMPLETE-SENTINEL
