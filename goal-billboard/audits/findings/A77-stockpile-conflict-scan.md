---
finding_id: A77-stockpile-conflict-scan
parent_audit: A77
title: First-pass adversarial conflict scan of the week's accumulated idea stockpile
schema_version: 1
created: 2026-05-29T00:00Z
methodology: A76-style adversarial hunt (try to BREAK the pile — find ideas that already conflict/duplicate/contradict)
scope: read-only scan of goal-billboard/active (15 goals) + pins (5 files, index claims 16) + plans/** (95 plans) + recent audits A6x/A7x + RH-014/017/018/019/020
file_zone: this file ONLY (A77.md + workflow + skill-ref owned by sibling BGs)
worry_addressed: user 2026-05-28 — "make sure we arent adding conflicting ideas as we have been stockpiling ideas all week"
---

# A77 — First-pass stockpile conflict scan

This is the **first run** of the A77 conflict-check capability, applied retroactively to the week's stockpile (the thing A77 is being built to prevent going forward). I read the goals, pins, plans, recent audits, and recent rabbit holes adversarially — trying to break the pile by finding ideas that already fight each other.

**Headline: the pile is mostly coherent, but it is NOT clean. There are 4 real collisions of HIGH-or-above severity, all in the same blast radius: the multi-orchestrator / cluster / local-AI infrastructure wave that exploded this week (G7/G12/G13/G16/G17 + RH-017/018/019/020 + the daemon plans).** That's expected — it's the newest, fastest-growing, least-settled zone. The older substrate (scheduler, hooks, dashboard, audit pipeline) is fragmented in places but not contradictory. The single most dangerous finding is a live infrastructure drift (the BG cap), not a planning one.

---

## Ranked collision table

Severity scale: **CRITICAL** (live system says two different things right now) · **HIGH** (two designed-but-unbuilt ideas will collide on contact / falsified justification) · **MED** (overlap/fragmentation that wastes effort or confuses) · **LOW** (naming adjacency / historical cruft).

| # | Idea A | Idea B | Collision type | Severity | Recommended resolution |
|---|---|---|---|---|---|
| **1** | `bg-dispatch-architecture.md` + `dispatch-bg.sh` say **BG cap = 5** ("5-in-flight ceiling", lines 16 / 248-250) | commit `7d29384` **"raise BG cap 5→8"** + memory-mirror (`feedback_autonomic-system`, `feedback_standing-protocols`, matrix-discipline) say **8** | **Stale/superseded — LIVE CONTRADICTION** | **CRITICAL** | **Pick-one + propagate.** User explicitly raised 5→8 (repeated request, per commit 7d29384). The skill ref + the dispatch wrapper's own gate-comment + A21 audit all still teach 5. Reconcile to 8 everywhere (or document 5=default / 8=current-explicit). This is the one collision where the *running system* contradicts itself today. |
| **2** | **RH-020 #1 "adopt claude-context-local semantic index now"** (★★★★ slam-dunk) | **G12/G13 + RH-017** "Qdrant RAG over `context/` on the Pis owns embeddings" + **A76 DEFER verdict** | **Conflict — duplicate RAG stack, already partly caught** | **HIGH** | **Sequence (already decided — make it stick).** A76 DEFERRED RH-020's adopt-now precisely because G12 will own embeddings. But A76 said "G12 *will* own embeddings" as if unspecified — RH-017 *already concretely specifies Qdrant RAG over the context/ forest*. So it's not "future G12 might"; it's "RH-017 already wrote the competing design." Resolution: fold RH-020 #1 into the RH-017/G12 RAG decision; mark RH-020 #1 as **superseded-by-G12-RAG**, not a live candidate. A76 caught the conflict; the stockpile hasn't recorded the supersession yet. |
| **3** | **Daemons running on M4 NOW** — `com.evium.daemon10.plist` is **loaded** (`launchctl list` confirms); daemon2-5 plists created (dry-run) | **RH-019 S1 + G17-P3** "Pis host the 8-daemon catalog 24/7"; **RH-018 #4** "M2 is the cost-tracker daemon host" | **Contradiction in approach — 3 proposed homes for the same daemons** | **HIGH** | **Pick-one home per daemon + document the migration.** Three docs place the autonomic daemons in three different places (M4 LaunchAgents today / Pi #1 24/7 / M2 cost-tracker). RH-019's own framing ("Pis are the always-on tier; Macs can sleep") is the most defensible end-state, but daemon10 is *already* on M4. Need an explicit "daemons start on M4, migrate to Pi #1 when Pi comes online" sequence, or they'll get built twice. Currently no doc reconciles "loaded on M4" with "will live on Pi." |
| **4** | **G7** (cluster + true parallelization, P1, guiding_light, active) + **G16** (M2 slam-dunk wave, P0, active) | **G17** (umbrella, P0, `absorbs_goals:[G7,G16]`) | **Duplicate / unresolved absorption** | **HIGH** | **Merge (resolve the absorption).** G17 declares it absorbs G7+G16, but both remain `status: active` with frontmatter notes saying "pause/merge deferred to user." Three active goals where one claims to be the umbrella of the other two = the exact A74-D2 finding, still open. 15 active goals is 188% of the healthy band largely *because* of this triple-count. Resolution: user decides — fold G7+G16 to `phase: under-G17` (not abandoned), leaving G17 as the single active infra umbrella. Until then the billboard triple-counts one initiative. |
| **5** | **m2-mac-test-farm.md** — "M2 = Playwright compute-farm node / shard target" (status: DEFERRED, Framings 2+4 "remain plausible") | **RH-018 premise shift** — "M2 = independent Claude Code orchestrator with its own subscription" (explicitly: "original M2-as-compute-farm framing now **superseded**") | **Stale/superseded — mislabeled** | **MED** | **Dedup label.** RH-018 says the compute-farm framing is *superseded*; the plan itself says only *DEFERRED* and still advertises Framings 2+4 as "plausible." A reader hitting m2-mac-test-farm.md cold would not know it's been reframed. Resolution: stamp m2-mac-test-farm.md as **superseded-by-RH-018/G16/G17**, keep the shard-split *content* (it's reused by G16 slam-dunk #1) but kill the "M2 is a compute node" framing. |
| **6** | **cluster-readiness.md** line 23: "M2 (next, SSH) — Playwright **+ iOS sim** — 3 shards" | **m2-mac-test-farm.md** line 16 + **RH-018** + **RH-019**: "M2 8GB **too tight for iOS Sim**; ~2-3 shards, non-emulator only" | **Stale capability claim** | **MED** | **Dedup/correct.** cluster-readiness still lists iOS-sim-on-M2 as an M2 capability; three later docs invalidate it (8GB RAM). Also "3 shards" vs RH-019's "2-3 shards." Resolution: correct cluster-readiness M2 row to "non-emulator Playwright only, 2-3 shards, NO iOS sim" to match the settled reality. Low blast radius (it's a capacity table) but it's a wrong fact that feeds matrix-shard-partition planning. |
| **7** | **RH-018 14-idea M2 slam-dunk catalog** (matrix shard / dashboard builder / research farm / cost-tracker / audit-eval / heartbeat secondary / CoT consolidator / bug-groomer / snapshot worker / lighthouse / …) | **RH-019 8-idea Pi slam-dunk catalog** (S1 daemons / S2 chromium shard farm / S3 heartbeat / S4 lighthouse / S5 snapshot-drift / S6 bug-groomer / …) | **Duplicate — same jobs assigned to two different nodes** | **MED** | **Pick-one owner per job.** At least 5 jobs appear in BOTH catalogs with different owners: heartbeat-secondary (RH-018 #6 M2 vs RH-019 S3 Pi #2), bug-billboard-groomer (RH-018 #11 M2 vs RH-019 S6 Pi #1), snapshot-drift (RH-018 #9 M2 vs RH-019 S5 Pi #2), lighthouse (RH-018 #10 M2 vs RH-019 S4 Pi #1), CoT-consolidator/session-analyzer (RH-018 #7/#12 M2 vs the Pi daemon tier). G17 is *supposed* to be the reconciler but its phase table doesn't assign these jobs to a single node. Resolution: one authoritative node-assignment table in G17 (or a new ref) — each job has exactly one home. Right now the two RHs would have you build heartbeat-secondary on both M2 and a Pi. |
| **8** | **5 dashboard plans**: dashboard-overhaul / dashboard-archive-full-upgrade / dashboard-content-quality-and-uxr / dashboard-layout-neaten / dashboard-scrutiny / operational-dashboard-expansion | each other | **Fragmentation (not contradiction)** | **LOW-MED** | **Leave + index.** These are mostly sequential phases (overhaul → scrutiny → layout-neaten → content-quality → archive-upgrade) authored over different days, not contradictory designs. But there's no single "dashboard roadmap" stitching them, so a reader can't tell which supersedes which. Resolution: a 5-line ordering header in dashboard-overhaul-plan (the earliest) pointing to the sequence. Low priority — no design conflict, just discoverability. |
| **9** | **4 multi-change-scheduler plans** (plan / implementation / phase-A3-A5 / phase-D) + **G1** | each other | **Fragmentation (historical, mostly executed)** | **LOW** | **Leave.** G1 says Phase D + A11 + G2-launch all done; the 4 plans are the build-phase breadcrumbs from 2026-05-25/26. Historical fragmentation, not an active contradiction. No action needed beyond noting they're a completed family. Candidate for `plans/completed/` move (needs the standing re-read + user yes per plans-completed convention). |
| **10** | **heart-decouple-from-scheduler-plan** ("built, dry-run, A74-D1 SPOF fix") | **heart-tier-medium-consumer-design** ("drafted, design-only") | **Adjacency (complementary, not dup)** | **LOW** | **Leave.** Both touch the Heart but at different layers — one decouples the tick from the scheduler (independence), the other designs a consumer for tier-medium events. Complementary. Flagging only to confirm it's NOT a duplicate after inspection. |
| **11** | **Pins index** header: "**18 pins**" (per A77 task) / body: "**16 items**" | **pins/ directory**: only **5 files** exist (P-001, P-002, P-010, P-017, P-018) + _index | **Stale/phantom count** | **LOW-MED** | **Dedup the count.** The index narrates 16 pins (P-001..P-016) in a table but only 5 have actual files; P-003..P-009, P-011..P-016 are table-rows-without-files. Either they were never spun out or the index is aspirational. Resolution: reconcile the index to the files on disk (or create the missing pin files if they're real). A conflict-check gate would flag "claimed N, found M." |

---

## The top 3 collisions to resolve (if the user does nothing else)

1. **#1 — BG cap 5-vs-8 (CRITICAL).** This is the only one where the *running system* disagrees with itself *today*. The dispatch wrapper's gate comment and the canonical skill ref still enforce/teach 5; the user explicitly raised it to 8 and the memory-mirror reflects 8. A subagent reading the skill ref will cap at 5; one reading memory will go to 8. Fix the skill ref + `dispatch-bg.sh` comment to 8. ~10 min.

2. **#4 — G7/G16/G17 triple-count (HIGH).** Three active goals for one initiative. This is *why* the goal count is at 188% of healthy (A74-D2 already flagged it; it's still open). One user decision ("fold G7+G16 under G17") collapses three active goals to one and de-bloats the billboard. The frontmatter is already staged for it ("pause/merge deferred to user").

3. **#2 + #3 + #7 together — the daemon/RAG/node-assignment tangle (HIGH).** These three are really one knot: *where does each piece of the infra wave live, and who owns embeddings?* RH-020-index vs G12-Qdrant (embeddings), M4-vs-Pi-vs-M2 (daemons), RH-018-M2 vs RH-019-Pi (the shared slam-dunk jobs). A76 untangled one thread (embeddings → defer to G12). The remaining fix is **one authoritative node-assignment table** under G17 that says, per job: this lives on M4 now / migrates to Pi-1 / stays M2 / is owned by G12's RAG. Without it, the slam-dunk wave will build the same daemon on two machines.

---

## The stockpile's health: coherent core, turbulent frontier

**Verdict: NOT accumulating contradictions wholesale — but the infra frontier is outrunning its own reconciliation.**

- **The older substrate is coherent.** Hooks, the audit pipeline, the bug billboard, the scheduler family, the dashboard plans, the post-shutdown pipeline (G10/G11/G14) — these are *fragmented* (many small plans) but I found **zero hard contradictions** among them. G10/G11/G14 are a clean Stage-1/2/3 pipeline, not duplicates. The fragmentation there is discoverability cost, not conflict.

- **The infra wave (this week's growth) is where every real collision lives.** G7/G12/G13/G16/G17 + RH-017/018/019/020 + the 6 daemon-design plans were all created 2026-05-26→28, fast, in response to "M2 is coming / cluster / local-AI / cursor." That burst created: 1 live contradiction (BG cap), 1 duplicate-goal triple (G7/G16/G17), 1 duplicate RAG stack (RH-020 vs G12/RH-017), 1 daemon-home contradiction (M4/Pi/M2), 2 superseded-but-unlabeled framings (m2-test-farm, cluster-readiness iOS-sim), and 1 duplicate slam-dunk catalog (RH-018 vs RH-019). **6 of the 7 HIGH/MED collisions are in this one zone.** That's the signature of a research burst that out-ran its consolidation pass — exactly the pattern A77 exists to catch.

- **A76 already proves the methodology works.** A76 ran this adversarial hunt on ONE idea (RH-020 semantic index) and correctly: rejected the cloud server, falsified RH-020's "retires snapshot-baseline" claim, and surfaced the G12-embeddings-ownership conflict + the privacy landmine. That's collision #2 + a falsified-justification, caught by hand. A77 generalizes that — and this scan shows the *demand* is real: 4 more HIGH-severity collisions of the same shape were sitting unaudited.

---

## What a built-in A77 gate would have caught earlier (the value case)

Walking each collision back to the moment it *entered* the pile, here's what an automated conflict-check at idea-graduation would have flagged in real time:

| Collision | When it entered | What the gate would have said at that moment |
|---|---|---|
| #1 BG cap 5→8 | commit 7d29384 (cap raised) | "You changed the cap to 8 but `bg-dispatch-architecture.md` + `dispatch-bg.sh` + A21 still say 5 — propagate or document the divergence." (A *ripple-awareness* check — RH-020 slam-dunk #3, not yet built.) |
| #2 RH-020 index vs G12 RAG | RH-020 part-3 authored (2026-05-28) | "This proposes adopting an embeddings index, but RH-017/G12 already specify Qdrant RAG over context/. Two RAG stacks — sequence or pick one." (A76 caught this *manually* the next day — the gate would have caught it *at authoring*.) |
| #3 daemon home M4/Pi/M2 | RH-019 created (2026-05-28) | "RH-019 places the 8-daemon catalog on Pis, but daemon10 is already a loaded LaunchAgent on M4 and RH-018 puts cost-tracker on M2. Reconcile the home." |
| #4 G7/G16/G17 | G17 created (2026-05-28) | "G17 declares `absorbs_goals:[G7,G16]` but both are still `status:active`. Adding G17 triple-counts the initiative — pause/merge the absorbed goals or mark them under-umbrella." (A74-D2 caught this *after the fact*; the gate fires *on goal creation*.) |
| #5/#6 superseded framings | RH-018 created (2026-05-28) | "RH-018 says it supersedes m2-mac-test-farm's compute-farm framing and invalidates iOS-sim-on-M2 — but those source docs aren't marked superseded. Stamp them." |
| #7 dup slam-dunk catalogs | RH-019 created (2026-05-28) | "5 jobs in this Pi catalog (heartbeat-secondary, bug-groomer, snapshot-drift, lighthouse, CoT-consolidator) already appear in RH-018's M2 catalog with a different owner. Assign each job one node." |

**The pattern is unambiguous: every HIGH collision entered during the 2026-05-26→28 infra burst, and all but one (#1, the cap) were *fan-out research artifacts that nobody cross-checked against the existing pile before filing.*** A77's value is precisely a cheap read-only "does this new RH/goal/plan collide with what's already here?" pass at filing time. The 3 things worth wiring into that gate, in priority order:

1. **Goal-creation cross-check** — when a new `G<N>` lands, diff its `absorbs_goals` / scope against existing active goals (would have caught #4 at creation, the single biggest billboard-bloat source).
2. **Node/owner-assignment dedup** — when a new infra RH/plan assigns a job to a node, check whether another doc already assigned that job elsewhere (would have caught #3 + #7).
3. **Supersession-stamp prompt** — when a doc says "this supersedes/invalidates X," verify X is actually marked superseded (would have caught #2 + #5 + #6).

These three are exactly the conflict-classes that the impact-domino layer (leverage scoring) and the goal-billboard proposal gate (ask-before-adding) do **not** cover — confirming A77 composes with, rather than duplicates, the existing triage discipline.

---

## Scope notes / honesty

- I did NOT re-litigate the workflow keyword-trigger conflict — A75 (verified_complete) already mapped that exhaustively and found it LOW/latent. Out of A77's scope (that's "our system vs a 4.8 feature," not "stockpiled idea vs stockpiled idea").
- I did NOT open every one of the 95 plans — I sampled the clusters most likely to collide (infra/cluster/local-AI/daemon/dashboard/scheduler) plus all 15 goals, all pins, and the 5 recent RHs in full. A deeper sweep of the older `plans/responsive`, `plans/mobile`, `plans/install-photos` dirs was skipped as low-collision-risk (they're page-build plans, not system-design ideas) — flagging the gap honestly rather than claiming blanket coverage.
- Severity is my judgment, anchored to "does the live system contradict itself" (CRITICAL) vs "will two designs collide on contact" (HIGH) vs "wasted effort / confusion" (MED) vs "cosmetic" (LOW). The user/main may re-rank.
- All 11 are recoverable by **documentation/decision**, not code rewrites — the pile is reconcilable, not broken. None of these block G2.

BG-COMPLETE-SENTINEL
