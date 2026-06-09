---
title: Full-history dropped/partial request audit — synthesis
created: 2026-06-07
scope: 33 prior sessions; extractors flagged 26 DROPPED + 122 PARTIAL candidates
method: DEDUP recurring asks → CROSS-REF the live system (asks-log, goal-billboard active/paused/pins/ideas/proposals/audits, src/, plans/) → DROP anything since built/closed/superseded → AGE + classify the genuine remainder
critical-constraint: the user HATES phantom/false gaps — every "still-open" below was confirmed NOT satisfied in the live system; everything satisfied was demoted to "already-done / superseded"
sources:
  - context/markdowns/goal-billboard/asks-log.md
  - context/markdowns/goal-billboard/{active,paused,pins,ideas,proposals,audits}/
  - context/markdowns/goal-billboard/stale-goals-triage-2026-06-07.md
  - .claude/cache/goal-billboard-status.digest
  - src/pages/ (live site = Back Lawrence Shul, post-pivot)
  - context/markdowns/plans/ (TODO, scheduler-dashboard, ci-failure-emails, bls-mobile-build, cursor-composer-eval)
---

# Full-history request audit — 2026-06-07

The headline: **almost everything flagged has already landed or been structurally superseded.** Of the 148
raw candidates, exactly **3 distinct asks are genuinely still open**, and even those are *partials of
in-flight goals*, not forgotten requests. The flag volume is inflated by two artifacts: (1) **12 of the 26
DROPPED are the same inbox-ack complaint** hitting the May-31 weekly-limit wall over and over, and (2) the
oldest site asks belong to **Evium**, a project the work has since **pivoted away from** (the live site is
now Back Lawrence Shul).

## Histogram

| Bucket | Count | Notes |
|---|---:|---|
| **Total candidates seen** | **148** | 26 DROPPED + 122 PARTIAL |
| Collapsed by dedup | 12 → 1 | the recurring inbox-ack complaint (12 instances) = 1 theme |
| **Addressed / built** | ~118 | landed in a goal, audit, plan, or the live repo |
| **Superseded** (pivot or later-session pivot) | ~24 | Evium site asks (project pivoted to BLS); overnight "one-thing" asks (next session re-scoped) |
| **Already-done** (specific artifact on disk) | 5 | TODO.md, A66 decision-handling, 4.8-vs-4.7 benchmark, reset-API verdict, scheduler before/after |
| **Genuinely still-open** (deduped) | **3** | all partials of active goals — see list |

> **No new phantom gaps surfaced.** Every "drop" the extractors flagged is either (a) work that shipped,
> (b) a request the user themselves re-scoped or abandoned in a later session, or (c) one of the 3 live
> partials below. The dramatic-sounding rebukes ("are you batshit crazy", "don't be retarded") all map to
> disciplines that were subsequently built (A66 decision-handling, parallel-capacity).

---

## The 3 genuinely-still-open asks (sorted by age)

### 1. Instant inbox-acknowledgment — kill the ack-latency gap  ·  classify: **partial-never-finished**
- **recurrence_count: 13** (12 raw inbox-ack complaints + 1 explicit "instant inbox acknowledgment is the
  most important throughput improvement")  ·  **first_asked: 2026-05-27**  ·  **last_asked: 2026-05-28**  ·
  **age: ~11 days / spanned ~8 sessions**
- **User words:** *"Instant inbox acknowledgment — kill the ~3-minute ack gap; this is the most important
  throughput improvement"* and, repeatedly when limit-walled, *"I sent an inbox ages ago — why haven't you
  acknowledged it yet??"*
- **Live-system state:** substantially built but **not finished**. `G6` (live-agent-messaging) Phase 3
  fan-out is live; `A71` inbox-never-lost VERIFY 5/5 PASS closed the *vanish* bug; `A68` (poll-gap class)
  deployed **3 hook shapes** (Notification / Stop / Heart-tier-low daemon) — but `A68.status: in_progress`
  because **all 3 await a user `settings.json` line** to actually fire. So the instant-ack symptom the user
  hammered is still reachable: the machinery exists, the wiring is user-gated and unwired.
- **Why not a phantom:** the 12 wall-hit complaints were each cut off by the weekly limit (not agent
  neglect), BUT the underlying capability is provably still `in_progress`, not shipped.
- **Suggested closure path:** surface the A68 settings.json gate to the user as an explicit yes/no (the hooks
  are deployed + dry-run-tested; one settings line arms instant-ack). Then flip `A68` → resolved and update
  `G6` phase. This is a *user-gated finish*, not new build.

### 2. The full SECOND-PASS 100%-coverage matrix run on the CURRENT site (the gate before wielding the matrix for new changes)  ·  classify: **partial-never-finished**
- **recurrence_count: 5** (3867fc3b, 44bafa35, 3d595f56, d7a8cac8, + "true full matrix per-page/section/
  device" in 23f93bfd)  ·  **first_asked: 2026-05-27**  ·  **last_asked: 2026-05-28**  ·  **age: ~11 days /
  ~5 sessions**
- **User words:** *"the full second-pass end-to-end 100%-coverage run must happen on our CURRENT site before
  we wield the matrix for any requested change (that was always the plan)."*
- **Live-system state:** **partially overtaken, never explicitly closed.** `G2` did advance a **9-device
  real-hardware sweep** (5 iOS + 4 Android) → `G2-CLOSE-READY=YES` on disk; plans
  `g2-second-pass-audit-operational-plan.md` + `g2-full-coverage-plow-plan.md` exist. But (a) those runs
  kept hitting the auto-mode-classifier outage + Bash-block + weekly wall and never completed the *declared*
  full per-page/per-section/per-device pass in one go, and (b) **the project pivoted to BLS**, so "the
  current site" the gate referred to (Evium) is no longer the active site. The *intent* (a 100%-coverage
  gate before wielding the matrix) was never formally satisfied or formally retired.
- **Suggested closure path:** make an explicit call with the user — either (a) declare the gate **satisfied**
  by the G2 9-device sweep + close it, or (b) **re-point it at BLS** (the live site) as a one-time
  full-coverage pass. Right now it's in limbo. Recommend (a): the 9-device sweep + CLOSE-READY verdict
  effectively met the bar for Evium, and the gate's premise (Evium) is stale.

### 3. "Why were we just sitting idle with no research?" — never-idle proof / prove-it demand  ·  classify: **partial-never-finished**
- **recurrence_count: 3** (23f93bfd "why were we just sitting idle", d7a8cac8 idle-no-research, + the
  "prove it — make it impossible to get stuck again" demand)  ·  **first_asked: 2026-05-27**  ·
  **last_asked: 2026-05-28**  ·  **age: ~11 days / ~3 sessions**
- **User words:** *"Did you apply the mutation/org-OS logic to the backlog failure? Prove it — make it
  impossible to get stuck in that state again… Also why were we just sitting idle with no research?"*
- **Live-system state:** the *mechanisms* shipped — `A62` adaptive-immunity, `A64` stuck-state-decoupling,
  `A63` idle-capacity-research-furthering, `A80` never-idle-protocol, `A81` never-idle-structural-enforcement
  (archived = built), plus the `research-furthering` skill. So the engineering answer to "make it impossible
  to get stuck" largely exists. **What was never delivered is the explicit "prove it"** — a demonstration /
  verification artifact shown back to the user closing the loop on *this specific* demand. The session ended
  on the user's interrupt before any reply.
- **Suggested closure path:** produce a short "prove-it" verification note: cite A62/A64/A80/A81 + the
  `never-idle-surface-log.jsonl` / `parallel-capacity-log.jsonl` as evidence the stuck-state + idle-no-research
  classes are now structurally guarded, and surface it. Low effort; closes a loop left dangling by an interrupt.

---

## Why the other 145 are NOT open (the dedup + supersession ledger)

**Inbox-ack complaints (12 raw → folded into open-item #1).** Every *"I sent an inbox ages ago…"* /
*"…12 minutes ago, why haven't you acknowledged??"* (sessions 21da9584, 23492bcf, 2fefd8f6, 3867fc3b,
3d595f56, 44bafa35, 77560b63, a42614e9, af9201e0, b92834fe, plus the 23:26 one) hit the **weekly-limit wall**
and so got no in-session reply — but they are one recurring theme, now tracked as open-item #1, not 12 gaps.

**Evium site asks — superseded by the BLS project pivot (the live `src/pages/` is now Back Lawrence Shul):**
- *"Turn the project into an Astro + Tailwind project"* (cef5954a, 2026-05-21) → **DONE**: repo is Astro v6 +
  Tailwind v4 (`astro.config.mjs`, `package.json`).
- *"Explore the reference real-estate site / fullpage flow"* + *"decoupled root copy of the reference design template"*
  (cef5954a) → **superseded**: these bootstrapped the rebuild; deep design research now lives under
  `research/rabbit-holes/reference-site-original-design-patterns/`; the template copy exists under
  `context/codebases/reference-site-template/`.
- *"Master prompt for 18 state SEO landing pages"* → **superseded by pivot**: an Evium SEO play; the live
  client is BLS (no state pages, and none wanted for a shul).
- *"hero pics for fleets/highway; multifamily-v2 decoupled copy"*, *"fullPage.js on mobile / fix iOS rail"*,
  *"iPhone white-corner = the rail"*, *"fix the 1 high-severity npm vuln"* → all **Evium Phase-5 defect
  items folded into `G2`**, which reached `G2-CLOSE-READY=YES`; not tracked as open manifesto items.
  multifamily-v2 is represented on the git page (MANIFESTO-CHECKLIST line 32).

**Already-done (specific artifact on disk):**
- *"Make a TODO overarching goal-plan"* → `plans/TODO-after-weekly-limit-resets.md` **exists**.
- *"batshit-crazy / flesh out decision-handling / never make dumb lazy nearsighted decisions"* → **DONE**:
  `A66` + `feedback_decision-handling-discipline.md` (scope-of-directive = scope-of-permission +
  always-gated list).
- *"Start a 4.8-vs-4.7 benchmark"* → **DONE**: `audits/findings/model-tier-benchmark-2026-06-02.md`.
- *"Make a reset API to re-initiate a plow after a mid-sleep limit hit"* → **answered**: `A82` verdict =
  **not safely possible** (no reset API exists; warm-resume-on-return adopted instead, ~80% built). The
  request got a thorough verdict + an adopted alternative — not a drop.
- *"Resume/finish dashboard; is it a related-task; build a related-tasks system"* (c354fe13) → dashboard is
  now **`G18` (active)**; scheduler before/after shipped; related-tasks concept lives in
  `scheduler-pr-mode-plan.md`. The *specific* "related-tasks system" wasn't built as named, but the question
  was structurally answered by G18 + the scheduler integration.

**Superseded by a later-session re-scope (overnight-guidance asks):**
- *"Continue from where you left off"* + *"Last night this failed while I slept, check"* (0d739f65) → next
  session resumed; transient.
- *"What's the best move / if I leave you on ONE thing overnight, what?"* (b92834fe, d7a8cac8, asked ~5×) →
  the user re-scoped direction in subsequent sessions; the never-idle + full-plow playbooks
  (`full-plow-v2-*.md`, `cost-right-full-plow-playbook.md`) codify the standing answer.
- *"Don't tell me to close the laptop / aim for 30 min"* (d7a8cac8) → folded into the never-idle / full-plow
  direction; the 30-min-scope framing was abandoned.
- *"Use the Chrome plugin, pretend to be a user, plow until the inbox round-trips"* (9799d4b1, 3d595f56) →
  the inbox round-trip / vanish bug was closed by `A71` (VERIFY 5/5); the *manual Chrome-plugin drill* was a
  one-off verification method, not a standing deliverable.
- *"Make me run commands / what's the point if I run up my wallet"* (ec81012e) → addressed by the
  **2026-06-01 git-op-authority decision** (Claude now runs local commits) + the cost-discipline playbooks;
  the human-gated-command complaint was structurally answered.

**The remaining PARTIAL candidates** map cleanly onto existing tracked work and need no resurfacing:
monkey-skills / monkey-hygiene / banana-backlog-clog → `A46`/`A61`/`A64`/`P-019`; parallel-agents-constantly /
7-8-never-below-7 → `A63`/`A80`/`A81`/`A84`; dashboard before/after + scratchpad → `G3`(paused)/`G18`/`A91`;
dashboard jumpy/textarea-HMR → `A51`; Cursor-with-agents / Team-2-seat / 2nd-sub → `P-018`/`P-021`/`A72`;
Pi 3-way cluster + distributed-systems skills → `research/pi-cluster/` + `G7`(paused)/`G13`(paused);
monkey-chamber overhaul (animated/draggable/glass) → `P-019`/`A46`; git-failure emails →
`ci-failure-emails-fix-plan.md`; hook-overhead research → `hook-timing-reevaluation-plan.md` +
`research/rabbit-holes/`; slam-dunks-priority / screenshots-archived-to-timelapse → `G3`/dashboard-archive
plans; lint-skill-staleness backtick-tune → `lint-skill-staleness.sh` exists; root-clutter relocate /
domain:external frontmatter / no-commit subdirective / "option A fire second-pass" → all one-shot
in-session items resolved or rolled into the above.

---

## Meta-finding

Two structural noise sources keep manufacturing apparent "drops," both worth a guard:
1. **Weekly-limit-wall complaints look like drops but aren't agent neglect** — the session simply terminated.
   The real fix is the (still-open) instant-ack wiring (#1), which would have *acknowledged* before the wall.
2. **A project pivot (Evium → BLS) orphaned a whole class of site asks** that an extractor still reads as
   "unfinished" because they were never formally retired. When the Northern Star changes, the *old* NS's
   open asks should be explicitly marked superseded so they stop re-flagging.
