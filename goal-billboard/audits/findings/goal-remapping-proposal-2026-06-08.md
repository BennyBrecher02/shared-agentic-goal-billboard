---
title: "By-goal RE-MAPPING proposal — the ambiguous G2-stamped corpus (Thread B / B4)"
domain: systems / goal-billboard / data-quality
status: resolved   # 2026-06-08 nap-work: the 127 confident maps applied in the B4 wave; the 13 UNCLEAR routed (Dec-1=G21 in-wave; Dec-2/3/4 = 8 files routed in nap-work, see the UNCLEAR section)
analysis_only: false   # superseded: the UNCLEAR-section routing decisions WERE applied to frontmatter 2026-06-08 (user-authorized "pick best-fit for each"). The 127-row tables above remain a record of the already-applied B4 pass.
created: 2026-06-08T00:00Z
serves_northern_star: G2
belongs_to_goal: G18   # board goal-wiring fix is dashboard-data work (G18 owns the by-goal axis)
cluster: goal-billboard-data-quality
siblings:
  - context/markdowns/research/systems/goal-wiring-analysis.md
  - context/markdowns/goal-billboard/findings/goal-rewrite-drafts.md
---

# By-goal RE-MAPPING proposal (Thread B / B4 — the AMBIGUOUS rest)

**What this is.** The user chose to FIX the by-goal muddle, not leave it (Thread B / B4). ~320 files were
mass-stamped `serves_northern_star: G2` by `migrate-add-ns-field.py` (rule #6 fallback), which collapsed the
dashboard's group-by-goal axis to a single degenerate bucket (root cause:
`research/systems/goal-wiring-analysis.md`). The **safe-derivable** files were already backfilled with a real
`belongs_to_goal:` parent (the B4-safe pass — `belongs_to_goal` now spans G1/G2/G4/G6/G8/G9/G11/G12/G14/G15/G17/G18/G19/G20).
This doc handles **the ambiguous rest**: every file still carrying `serves_northern_star: G2` with **no
`belongs_to_goal` parent field at all**, with a PROPOSED parent goal for each, for the user to approve before
any apply.

**PROPOSE ONLY.** No frontmatter was edited. The fix is to add a `belongs_to_goal:` parent key (keep
`serves_northern_star: G2` — it is the correct *Mission-spine* field; the bug is using it as the by-goal
*parent* key, per goal-wiring-analysis §0). Each row below proposes that parent.

## Method + scope notes

- **Set definition:** files matching `serves_northern_star:[[:space:]]*G2` AND lacking any `^belongs_to_goal:`
  line, under `context/markdowns/`.
- **Backup excluded:** a `context/markdowns/goal-billboard.bak-20260608T122925Z/` snapshot (the
  backup-before-edit from today's B4-safe pass) duplicated ~98 files. It is a backup, not live corpus —
  **excluded** from every count and table below. (If the raw grep is re-run it will show ~277; the live,
  de-duplicated number is **166**.)
- **Goal taxonomy has grown since goal-wiring-analysis was written.** Two of that doc's "homeless clusters"
  now have homes: **G18** (Dev Control dashboard — the home for all dashboard/board/matrix/chamber/git-page
  work that the analysis routed to the now-*paused* G3) and **G19** (Blog automation — the home for the
  ClickUp/Cloudflare blog cluster the analysis flagged as homeless). This proposal routes to the **current**
  taxonomy (G18/G19), not the stale G3/"new-goal" targets.
- **Goal files do not self-stamp.** The 13 `G*.md` goal-definition files (active/) + 5 paused + the
  archived G5 carry `serves_northern_star: G2` but a goal does not "belong to" a parent goal — that field is
  for *child artifacts*. **Recommendation: leave the goal files' `belongs_to_goal` UNSET** (a goal is the
  parent, not a child). They are listed in their own section, not force-mapped.
- **Ideas are surface-only by design** (goal-wiring §3d: "leave ideas alone" — the ideas lane shouldn't be
  force-linked). The 5 ideas get a *topical* suggestion only, flagged do-not-auto-apply.
- Confidence: **H** = filename/title names the goal or the content is unambiguously that subsystem · **M** =
  strong topical match, worth a skim · **L** = plausible but a genuine routing judgment only the user can make.

---

## Summary line

**166 scanned · 132 confidently re-mappable (H/M) · 13 UNCLEAR (L / needs-user-judgment) · 21 set-aside
(goal-definition files + ideas — recommend NOT auto-stamping).**

Breakdown of the 132 confident: **G2** 31 · **G4** 38 · **G18** 18 · **G9** 11 · **G1** 9 · **G17** 9 ·
**G11** 5 · **G14** 4 · **G3** 3 · **G12** 2 · **G6** 1 · **G19** 1 · **G20** 8 (achieved, the repo-decouple
audits) · **G15** 1 · **G8** 3 · **G5(achieved)** 1 (the dashboard-page audit → G18-lineage). The single
largest lever is **G4** (skill/memory/hook/organic-OS/daemon substrate audits) and **G2** (the
multi-change-queue site fixes); the dashboard cluster (G18) is third.

> Net effect once applied: G2 stops being the catch-all. The 75 process/system **audits** — the bulk of the
> degenerate bucket — spread across G1/G3/G4/G6/G8/G9/G11/G14/G17/G18/G20 by subsystem; the 30
> multi-change-queue items collapse correctly onto **G2** (they ARE the Evium site fixes); the dashboard
> docs land on **G18**. The by-goal axis becomes non-degenerate.

---

## Re-mapping by PROPOSED goal

### → G2 (Evium Phase-5 site overhaul) — 31 files
The Evium site itself. The multi-change-queue is almost entirely G2 site-defect fixes (several filenames are
even suffixed `-g2`). Plus the install-photos content plan.

| File (`context/markdowns/…`) | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `multi-change-queue/verified/20260526T0924-blog-subscribe-fix-p0-02.md` | G2 | **G2** | H | blog subscribe form fix (site) |
| `multi-change-queue/verified/20260526T1230-contact-tablet-gutter-p1.md` | G2 | **G2** | H | contact page tablet layout (site) |
| `multi-change-queue/verified/20260526T1245-multifamily-quote-stats-stack.md` | G2 | **G2** | H | multifamily page stats stack (site) |
| `multi-change-queue/verified/20260526T1246-highway-econ-narrow.md` | G2 | **G2** | H | highway page narrow-viewport (site) |
| `multi-change-queue/verified/20260526T1247-services-terminal-narrow.md` | G2 | **G2** | H | services page narrow-viewport (site) |
| `multi-change-queue/verified/20260526T1300-phaseB-multishard-multifamily.md` | G2 | **G2** | H | multifamily site change (Phase-B shard) |
| `multi-change-queue/verified/20260526T1406-products-crumb-restore.md` | G2 | **G2** | H | products breadcrumb (site) |
| `multi-change-queue/verified/20260526T1410-phaseD-integration-retry.md` | G2 | **G2** | M | Phase-D site integration retry |
| `multi-change-queue/verified/20260526T1411-contact-form-card-p0-03.md` | G2 | **G2** | H | contact form card (site) |
| `multi-change-queue/verified/20260526T1416-blog-hero-equalize-p1-06.md` | G2 | **G2** | H | blog hero (site) |
| `multi-change-queue/verified/20260526T1544-eyebrow-token-p1-08.md` | G2 | **G2** | H | eyebrow style token (site) |
| `multi-change-queue/verified/20260526T1623-blog-subscribe-p0-02-requeue.md` | G2 | **G2** | H | blog subscribe requeue (site) |
| `multi-change-queue/verified/20260526T1755-blog-section-split-p2-10.md` | G2 | **G2** | H | blog section split (site) |
| `multi-change-queue/verified/20260526T1755-fleets-hero-text-shadow-firefox.md` | G2 | **G2** | H | fleets hero (site, Firefox) |
| `multi-change-queue/verified/20260526T1755-highway-hero-object-position-p1-05.md` | G2 | **G2** | H | highway hero (site) |
| `multi-change-queue/verified/20260526T1755-on-surface-variant-lift-p2-09.md` | G2 | **G2** | H | on-surface color variant (site) |
| `multi-change-queue/verified/20260526T1755-services-logo-grid-p2-11.md` | G2 | **G2** | H | services logo grid (site) |
| `multi-change-queue/verified/20260526T2057-hero-isolate-webkit-crumb-g2.md` | G2 | **G2** | H | hero/crumb WebKit — name says `-g2` |
| `multi-change-queue/verified/20260526T2057-highway-hero-iphone-small-recenter-g2.md` | G2 | **G2** | H | highway hero iPhone — name says `-g2` |
| `multi-change-queue/verified/20260527T2028-phase-b-contrast-bumps.md` | G2 | **G2** | H | Phase-B contrast bumps (site a11y) |
| `multi-change-queue/proposed/20260527T2135-p1-07a-tellus-tonal.md` | G2 | **G2** | H | Tellus tonal palette (site) |
| `multi-change-queue/proposed/20260527T2230-cb-1-hero-scrim-hardening.md` | G2 | **G2** | H | hero scrim (site) |
| `multi-change-queue/proposed/20260527T2230-cb-2-contact-card-affordance.md` | G2 | **G2** | H | contact card affordance (site) |
| `multi-change-queue/proposed/20260528T0700-eyebrow-cluster-fixes.md` | G2 | **G2** | H | eyebrow cluster (site) |
| `multi-change-queue/proposed/20260528T0700-fleets-04-caption-zorder.md` | G2 | **G2** | H | fleets caption z-order (site) |
| `multi-change-queue/proposed/20260528T0700-highway-01-mobile-hero.md` | G2 | **G2** | H | highway mobile hero (site) |
| `multi-change-queue/proposed/20260528T0700-services-05-logo-narrow.md` | G2 | **G2** | H | services logo narrow (site) |
| `multi-change-queue/wontfix/20260526T1330-phaseB-full-matrix-attempt3.md` | G2 | **G2** | M | Phase-B site matrix attempt (wontfix, still G2 work) |
| `plans/install-photos/install-photos-plan.md` | G2 | **G2** | H | real Evium install photos + `/case-studies/` route = site content |
| `plans/completed/responsive/next-steps-plan.md` | G2 | **G2** | M | responsive site follow-ups (completed) |
| `research/rabbit-holes/reference-site-original-design-patterns/README.md` | G2 | **G2** | H | The reference design = the Evium design source (goal-wiring §3a) |

> **Note — the 2 `*-g2`-suffixed wontfix matrix attempts** (`a11-6shard`, `a11-4shard`) are Phase-B *matrix
> infrastructure* attempts, not site changes — they map to **G1** (scheduler/matrix), see that section, not
> G2. The `phaseB-full-matrix-attempt3` is the borderline one (matrix run *of* G2 work) — left at G2/M; user
> may prefer G1.

### → G4 (Skill system + reactionary INFRASTRUCTURE maintenance) — 38 files
The substrate goal: skills, memory, hooks, the reactionary layer, the organic-OS biological subsystems
(Brain/Heart/Immune/Autonomic/Alignment), daemons, decision-discipline, settings/deny-list hygiene, the
portable multi-tool-OS architecture. The largest single cluster — most process-audit IDs land here.

| File | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `goal-billboard/audits/A1-reactionary-automation-audit.md` | G2 | **G4** | H | reactionary layer = G4 |
| `goal-billboard/audits/A2-critical-script-coverage-audit.md` | G2 | **G4** | M | script/hook substrate (could be G9 testing — see note) |
| `goal-billboard/audits/A3-hook-effectiveness-audit.md` | G2 | **G4** | H | hook infrastructure = G4 |
| `goal-billboard/audits/A4-skill-graph-audit.md` | G2 | **G4** | H | skill cross-link/discovery = G4 |
| `goal-billboard/audits/A5-memory-consolidation-audit.md` | G2 | **G4** | H | memory = G4 |
| `goal-billboard/audits/A8-missing-script-wrappers-audit.md` | G2 | **G4** | M | script-wrapper substrate = G4 |
| `goal-billboard/audits/A9-deny-list-audit.md` | G2 | **G4** | H | deny-list / settings hygiene = G4 substrate |
| `goal-billboard/audits/A18-plan-audit-artifact-creation-gap.md` | G2 | **G4** | H | A18 is a named G4 linked_audit (artifact-discipline) |
| `goal-billboard/audits/A19-cross-system-meta-monitoring.md` | G2 | **G4** | H | A19 is the named G4 master meta-monitoring audit |
| `goal-billboard/audits/A24-meta-loop-eval-cross-utilization.md` | G2 | **G4** | M | cross-utilization + meta-eval skill = G4 |
| `goal-billboard/audits/A25-settings-json-health-and-hook-chain-hygiene.md` | G2 | **G4** | H | settings.json + hook chain = G4 substrate |
| `goal-billboard/audits/A26-script-design-skill-utilization-and-expansion.md` | G2 | **G4** | H | agentic-script-design skill = G4 |
| `goal-billboard/audits/A28-skill-health-refresh-and-usage-metrics.md` | G2 | **G4** | H | A28 is the named G4 skill-health framework |
| `goal-billboard/audits/A29-lazy-regression-to-mean-bg-launch.md` | G2 | **G4** | M | BG-launch discipline / idle-laziness = G4/autonomic |
| `goal-billboard/audits/A32-hook-audit-testing-and-post-shutdown-automation.md` | G2 | **G4** | M | hook audit (post-shutdown half overlaps G10/G14) |
| `goal-billboard/audits/A36-true-system-time-discipline.md` | G2 | **G4** | M | time-discipline substrate (overlaps G1 scheduler) |
| `goal-billboard/audits/A37-calculation-audit-stats-and-os.md` | G2 | **G4** | M | OS-arithmetic substrate (stats half → G18) |
| `goal-billboard/audits/A38-master-skill-maker-from-user-voice.md` | G2 | **G4** | H | meta skill-maker = G4 skill system |
| `goal-billboard/audits/A39-high-potential-idea-handoff-trigger-and-slam-meter.md` | G2 | **G4** | M | idea-handoff discipline/skill = G4 |
| `goal-billboard/audits/A40-idea-gap-detector-and-recurring-sweep.md` | G2 | **G4** | M | idea-gap detector (Alignment subsystem) = G4 |
| `goal-billboard/audits/A44-critical-finding-reflex.md` | G2 | **G4** | M | Critical-Finding-Reflex discipline = G4 |
| `goal-billboard/audits/A58-organic-os-overhaul.md` | G2 | **G4** | H | Organic-OS Brain/Heart/Immune = G4 substrate |
| `goal-billboard/audits/A61-monkey-inbox-architecture-bottleneck.md` | G2 | **G4** | M | inbox-substrate architecture class = G4 |
| `goal-billboard/audits/A62-adaptive-immunity-discipline.md` | G2 | **G4** | H | adaptive-immunity (Immune subsystem) = G4 |
| `goal-billboard/audits/A63-idle-capacity-research-furthering.md` | G2 | **G4** | M | never-idle / research-furthering = G4/autonomic |
| `goal-billboard/audits/A64-stuck-state-decoupling-solid.md` | G2 | **G4** | M | SOLID/decoupling + per-letter skill refs = G4 |
| `goal-billboard/audits/A65-autonomic-system-daemon-framework.md` | G2 | **G4** | H | Autonomic daemon framework = G4 subsystem |
| `goal-billboard/audits/A66-decision-handling-discipline.md` | G2 | **G4** | H | decision-handling discipline = G4 |
| `goal-billboard/audits/A67-impact-domino-analysis-discipline.md` | G2 | **G4** | M | impact-domino discipline/skill = G4 |
| `goal-billboard/audits/A68-poll-gap-class.md` | G2 | **G4** | M | inbox-poll-gap discipline (Alignment) = G4 |
| `goal-billboard/audits/A69-cot-auto-realign-gap.md` | G2 | **G4** | M | CoT/Brain auto-realign discipline = G4 |
| `goal-billboard/audits/A74-4.8-systemwide-birdseye-audit.md` | G2 | **G4** | M | systemwide model-vs-system + skill-gaps = G4 |
| `goal-billboard/archived/audits/A15-research-folder-utilization.md` | G2 | **G4** | M | research-folder discipline = G4 |
| `goal-billboard/archived/audits/A26-compliance-pass-2026-05.md` | G2 | **G4** | H | A26 Phase-3 script-design compliance = G4 |
| `goal-billboard/archived/audits/A3-hook-effectiveness-2026-05-26.md` | G2 | **G4** | H | A3 hook-effectiveness finding = G4 |
| `goal-billboard/archived/audits/A9-deny-list-2026-05-26.md` | G2 | **G4** | H | A9 deny-list finding = G4 |
| `goal-billboard/audits/findings/A3-hook-effectiveness-2026-05-26.md` | G2 | **G4** | H | A3 hook finding = G4 |
| `goal-billboard/audits/findings/A26-compliance-pass-2026-05.md` | G2 | **G4** | H | A26 compliance finding = G4 |
| `agent-recommendations/agentic-chrome-tools.md` | G2 | **G4** | H | per-skill recommendation scratchpad = G4 skill-maintenance cadence |
| `agent-recommendations/agentic-webdesign.md` | G2 | **G4** | H | per-skill recommendation scratchpad = G4 skill-maintenance cadence |

### → G18 (Dev Control dashboard — operational command surface) — 18 files
Everything dashboard / board / matrix-tab / chamber-UX / git-page / stats-tab / inbox-UI. (goal-wiring-analysis
routed these to G3, but the user since created **G18** as their explicit home; G3 is now `paused/`.)

| File | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `goal-billboard/audits/A14-stats-time-correlation-audit.md` | G2 | **G18** | M | stats-tab correlation = dashboard stats |
| `goal-billboard/audits/A43-stats-system-major-upgrade-v2.md` | G2 | **G18** | M | stats dashboards/anomaly = dashboard |
| `goal-billboard/audits/A46-monkey-chamber-ux-de-retardize.md` | G2 | **G18** | H | Monkey-chamber UX = dashboard view |
| `goal-billboard/audits/A48-data-json-auto-sync-from-inbox-history.md` | G2 | **G18** | H | dashboard data.json/inbox = dashboard |
| `goal-billboard/audits/A49-chamber-empty-domino-and-plow-pivot.md` | G2 | **G18** | M | chamber-empty domino (chamber = dashboard) |
| `goal-billboard/audits/A50-monkey-fan-out-stuck-in-dry-run.md` | G2 | **G18** | M | monkey-click fan-out (chamber pipeline) |
| `goal-billboard/audits/A51-dashboard-textarea-survives-hmr-reload.md` | G2 | **G18** | H | dashboard textarea HMR = dashboard |
| `goal-billboard/audits/A52-bespoke-lefter-top-of-page.md` | G2 | **G18** | H | dashboard lefter nav = dashboard |
| `goal-billboard/audits/A53-matrix-birdseye-dashboard-tab.md` | G2 | **G18** | H | Matrix dashboard tab = dashboard |
| `goal-billboard/audits/A54-inbox-unsend-feature.md` | G2 | **G18** | H | inbox UNSEND = dashboard UI |
| `goal-billboard/audits/A55-dashboard-tops-scrutiny-pass-2.md` | G2 | **G18** | H | dashboard scrutiny = dashboard |
| `goal-billboard/audits/A57-dashboard-content-quality-and-uxr.md` | G2 | **G18** | H | dashboard content/UX = dashboard |
| `goal-billboard/audits/A60-goal-billboard-cot-intertwining.md` | G2 | **G18** | M | goal-billboard×CoT (board view; Brain half → G4) |
| `goal-billboard/archived/audits/A78-dashboard-scrutiny-and-plan-consolidation.md` | G2 | **G18** | H | dashboard keystone-surface scrutiny |
| `goal-billboard/archived/achieved/G5-billboard-dashboard-page.md` | G2 | **G18** | M | G5 (achieved) = the billboard dashboard page → G18 lineage (see goal-files note) |
| `goal-billboard/ideas/I-001-board-svg-strings.md` | G2 | **G18** | M | board SVG connector strings = dashboard (IDEA — do-not-auto-apply) |
| `goal-billboard/ideas/I-002-card-slide-animation.md` | G2 | **G18** | M | dashboard card animation = dashboard (IDEA — do-not-auto-apply) |
| `goal-billboard/findings/goal-rewrite-drafts.md` | G2 | **G18** | M | goal-def rewrite drafts — output of the board goal-wiring fix (G18); could be G4 |

### → G9 (Testing toolbelt — anti-bottleneck) — 11 files
Testing strategy, coverage, matrix-engine, simulator/emulator coverage, test-resource-leak prevention,
snapshot drift, baseline. (Distinct from G1, which owns the *scheduler/orchestration*.)

| File | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `goal-billboard/audits/A27-testing-toolbelt-anti-bottleneck.md` | G2 | **G9** | H | the literal G9 origin audit |
| `goal-billboard/audits/A42-idle-emulator-resource-saver.md` | G2 | **G9** | H | emulator resource-saver = testing toolbelt |
| `goal-billboard/audits/A56-idle-test-tool-resource-leak-prevention.md` | G2 | **G9** | H | test-tool leak prevention = testing |
| `goal-billboard/audits/A73-emulator-coverage-justification.md` | G2 | **G9** | H | sim/emu matrix coverage = testing |
| `goal-billboard/archived/audits/A13-visual-snapshot-baseline-drift.md` | G2 | **G9** | H | visual-snapshot baseline = testing |
| `research/external/cursor-skill-registration-2026-06-03.md` | G2 | **G4** | — | (NOT G9 — see G4-overflow note; listed under UNCLEAR) |
| `goal-billboard/audits/A2-critical-script-coverage-audit.md` (dup-listed) | G2 | **G9 or G4** | M | script *coverage* leans G9; substrate leans G4 — see note |

> **A2 + the two `a11`-shard wontfix items** straddle G9 (test coverage) and G1 (scheduler/matrix-shard
> mechanics). Placed: A2 → G4/G9 (M, user picks); `a11-6shard` + `a11-4shard` → **G1** (they are
> scheduler/shard-count experiments, see G1). To keep the count honest, A2 is counted once under G4 above.
> G9 confident total = the 5 H rows.

### → G1 (Multi-change scheduler MVP) — 9 files
The scheduler / worktree / matrix-shard orchestration engine (the thing that *runs* changes), and the
shard-count experiments.

| File | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `goal-billboard/audits/A6-worktree-hygiene-audit.md` | G2 | **G1** | H | scheduler worktree hygiene |
| `goal-billboard/audits/A10-scheduler-workaround-audit.md` | G2 | **G1** | H | scheduler deny-list workaround |
| `goal-billboard/audits/A21-parallel-capacity-underuse.md` | G2 | **G1** | M | parallel BG capacity (scheduler/dispatch; overlaps G17) |
| `goal-billboard/audits/A23-time-precision-and-three-way-integration.md` | G2 | **G1** | M | scheduler/heartbeat/time three-way (overlaps G4) |
| `goal-billboard/audits/A59-scheduler-upgrade-brainstorm.md` | G2 | **G1** | H | scheduler upgrade brainstorm |
| `goal-billboard/archived/audits/A11-phaseB-concurrency-ceiling.md` | G2 | **G1** | H | Phase-B concurrency ceiling = scheduler |
| `goal-billboard/audits/findings/A23-time-heartbeat-2026-06-07.md` | G2 | **G1** | M | A23 time/heartbeat re-audit (overlaps G4) |
| `multi-change-queue/wontfix/20260526T1420-a11-6shard.md` | G2 | **G1** | M | A11 6-shard scheduler experiment |
| `multi-change-queue/wontfix/20260526T1425-a11-4shard.md` | G2 | **G1** | M | A11 4-shard scheduler experiment |

### → G17 (Peak-throughput infra — multi-orchestrator + cluster + slam-dunk) — 9 files
Cluster, multi-Mac, multi-orchestrator, swarm-maximization, async-throughput, the portable cross-machine
torch-passing. (G17 absorbed G7+G16.)

| File | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `goal-billboard/audits/A22-cluster-and-ns-integration-gap.md` | G2 | **G17** | M | cluster-capacity gap (NS half → G8) |
| `goal-billboard/audits/A31-pre-restart-prep-master.md` | G2 | **G17** | L | 6-front master (cross-tool event-bus + bundle + research-v2) — spans G10/G11/G17/G4; see UNCLEAR |
| `goal-billboard/audits/A35-rabbit-hole-research-framework-and-agentic-survey.md` | G2 | **G15** | — | (→ G15 framework, not G17 — listed under G15) |
| `research/systems/cluster-torch-passing-2026-06-07.md` | G2 | **G17** | M | cross-NODE torch passing = cluster throughput |
| `research/rabbit-holes/RH-021-swarm-highest-wins/README.md` | G2 | **G17** | H | swarm-maximization = peak throughput |
| `research/rabbit-holes/local-ai-cluster-swarms/README.md` | G2 | **G17** | M | cluster swarms (local-AI half → G13) |
| `research/rabbit-holes/token-cheap-out-pass-2/README.md` | G2 | **G17** | M | token-cost / async throughput (could be G9) |
| `research/rabbit-holes/top-starred-agentic-github/README.md` | G2 | **G15** | — | (→ G15 framework exemplar — listed under G15/UNCLEAR) |
| `research/rabbit-holes/non-github-agentic-systems/README.md` | G2 | **G15** | — | (→ G15 survey — listed under G15/UNCLEAR) |

> Cleanly-G17 confident rows = A22, cluster-torch-passing, RH-021, local-ai-cluster-swarms,
> token-cheap-out-pass-2 (5). A31 and the G15-bound RH READMEs are pulled out to their proper sections to
> avoid double-counting; net G17 confident ≈ 5–9 depending on the L calls.

### → G11 (Session bundle + agentic-OS capture) — 5 files
Within-session capture + the analyzers + mining session JSONLs.

| File | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `goal-billboard/audits/A41-powerful-uses-of-session-jsonls.md` | G2 | **G11** | H | mining session JSONLs = capture |
| `goal-billboard/audits/A24-meta-loop-eval-cross-utilization.md` (dup) | G2 | **G4** | — | (counted under G4; the eval half brushes G11) |
| `notes/monkey-inbox-architecture-2026-05-27.md` | G2 | **G4** | — | (→ G4 inbox-substrate — listed under G4-overflow note) |

> Cleanly-G11 = A41 (1 H). G11's other natural members (`session-bundle-g11.md`, `session-data-utilization.md`)
> are already backfilled `belongs_to_goal: G11` in the B4-safe pass — they are NOT in this ambiguous set, which
> is why G11's ambiguous count is small. Counted G11 confident = 1; the "5" in the summary includes the four
> already-linked siblings for context — **for THIS doc's apply-set, G11 = A41 only.** (Corrected: G11 ambiguous
> = 1.)

### → G14 (End-of-session realignment + master audit) — 4 files
ACT-across-sessions: realignment ritual, master system audit, cross-session recurrence, post-analysis hooks.

| File | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `goal-billboard/audits/A34-end-of-session-realignment-ritual-master-audit.md` | G2 | **G14** | H | the literal G14 origin audit |
| `goal-billboard/audits/A20-hedge-holdoffs-and-unrouted-blockers.md` | G2 | **G14** | M | unrouted-blocker/realignment sweep (overlaps G4 Alignment) |
| `goal-billboard/audits/A47-bg-lifecycle-discipline-and-auto-stop.md` | G2 | **G4** | — | (→ G4 BG-lifecycle/autonomic — listed under G4? see note) |
| `goal-billboard/audits/A16-agent-idle-root-cause.md` | G2 | **G4** | — | (→ G4 idle/autonomic — see note) |

> A47 + A16 are autonomic/idle-discipline → **G4** (not G14); pulled to keep G14 to its true members. G14
> confident = A34 (H) + A20 (M) = 2 clean; A30/A31/A32/A34 form the graceful-lifecycle cluster that straddles
> G10/G14 (see UNCLEAR for A30/A31/A32). **G14 apply-set = A34, A20.**

### → G3 (Audit dashboard v2→v3 — PAUSED) — 3 files
The *evolution-methodology* dashboard goal (now paused; G18 is the active dashboard home). These three are
about the audit-dashboard's own design lineage rather than the live command surface — they arguably belong to
G3's archived lineage. **User may prefer to fold all dashboard work under the active G18** — flagged.

| File | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `goal-billboard/audits/A37-calculation-audit-stats-and-os.md` (dup) | G2 | **G4/G18** | — | (counted under G4; stats half → G18) |
| `goal-billboard/audits/A14-stats-time-correlation-audit.md` (dup) | G2 | **G18** | — | (counted under G18) |

> On reflection there is **no clean G3-only** file in the ambiguous set — every dashboard artifact here maps to
> the *active* G18, not paused-G3. **G3 apply-set = 0.** (Listed for completeness; the "3" in the first summary
> tally is withdrawn — see the corrected tally at the bottom.)

### → G12 (Local-AI single-Mac — PAUSED) — 2 files
| File | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `goal-billboard/audits/A33-local-ai-mass-wielding-research-initiative.md` | G2 | **G12** | H | the literal G12/G13 origin audit (dual-track; → G12, cross-link G13) |
| `research/rabbit-holes/RH-016-instant-inbox-ack/README.md` | G2 | **G6** | — | (→ G6 — listed under G6) |

> A33 is dual-track (single-Mac G12 + cluster G13). Propose primary **G12** with a G13 cross-link. G12 apply-set = A33.

### → G6 (Instant ack / live agent messaging) — 1 file
| File | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `research/rabbit-holes/RH-016-instant-inbox-ack/README.md` | G2 | **G6** | H | instant-inbox-ack = G6 (chat-ack latency→0); matches the B4-safe `A37-g6-phase2` precedent |

### → G8 (Northern Star deep integration) — 3 files
The NS-propagation meta-goal (the goal that *caused* the migration). Cross-goal state audits + NS-integration
gaps land here.

| File | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `research/systems/consistent-non-idle-automation-2026-06-07.md` | G2 | **G4** | — | (→ G4 autonomic last-mile — listed under G4-overflow) |
| `goal-billboard/audits/A22-cluster-and-ns-integration-gap.md` (dup) | G2 | **G17** | — | (counted under G17; NS-integration half brushes G8) |

> No clean G8-only file in the ambiguous set; the NS-integration audits also carry a stronger cluster (G17) or
> substrate (G4) signal. **G8 apply-set = 0** in this pass (G8's own goal file is in the goal-files section).
> The first-summary "G8 3" is withdrawn — corrected tally below.

### → G19 (Blog automation) — already homed
The ClickUp/Cloudflare cluster (RH-015) is already `belongs_to_goal`-linked via G19's `linked_research` and
the B4-safe pass — **no ambiguous RH-015 file remains** in this set. G19 apply-set = 0 here (noted so the user
knows it was checked, not missed).

### → G15 (Rabbit-holes framework) — 1 file + the framework-vs-content judgment
Per goal-wiring §1, G15 owns the **framework**; RH *content* attaches to its *topical* goal. The framework
audit itself → G15; the survey READMEs are a judgment call (framework-exemplar G15 vs substrate G4) — flagged.

| File | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `goal-billboard/audits/A35-rabbit-hole-research-framework-and-agentic-survey.md` | G2 | **G15** | H | the literal G15 origin audit (the framework) |
| `research/rabbit-holes/README.md` | G2 | **G15** | H | the rabbit-holes folder index = the G15 framework |
| `research/rabbit-holes/top-starred-agentic-github/README.md` | G2 | **G15 or G4** | L | agentic-system survey — see UNCLEAR |
| `research/rabbit-holes/non-github-agentic-systems/README.md` | G2 | **G15 or G4** | L | agentic-system survey — see UNCLEAR |

### → G20 (Repo-decouple — ACHIEVED) — 8 files (the portable-OS / multi-tool architecture)
The 2026-06-07/08 work on splitting the OS into a portable core + per-tool adapters (`agentic-os-orchestrators/`,
`_core/torch`, deploy-kit, Cursor adapter research). This is the *decoupling/extraction* initiative — G20's
domain (the B4-safe pass already routed the dashboard-split-readiness audits to G20). **OR** the user may want a
**new "portable multi-tool OS" goal** (G20 is marked achieved/RED-ALERT-emergency, narrower than this ongoing
architecture). Flagged — see UNCLEAR for the new-goal question.

| File | current | PROPOSED | conf | rationale |
|---|---|---|---|---|
| `research/systems/convo-torch-protocol-2026-06-08.md` | G2 | **G20** | M | `_core/torch` payload schema = decoupled-architecture |
| `research/external/agent-orchestrator-architecture-2026-06-07.md` | G2 | **G20** | M | per-env OS vs master OS (the D1 decoupling decision) |
| `research/external/config-handling-architecture-2026-06-07.md` | G2 | **G20** | M | tool-adapters config layer = decoupling |
| `research/external/leveraging-cursor-2026-06-03.md` | G2 | **G20** | M | wield-Cursor adapter = multi-tool portability |
| `research/external/cursor-skill-registration-2026-06-03.md` | G2 | **G20** | M | Cursor skills adapter = multi-tool portability |
| `plans/automation/brain-lifecycle-plan.md` | G2 | **G20** | M | unified memory-stream + handoff lineage = decoupling spine |
| `plans/port-uniqueness-initiative.md` | G2 | **G4** | — | (→ G4 — ports are settings/config substrate, NOT decouple; listed under G4-overflow) |
| `research/rabbit-holes/RH-020-cursor-integration/part-2-feature-wins.md` | G2 | **G20** | M | Cursor integration research = multi-tool |
| `research/rabbit-holes/RH-020-cursor-integration/part-3-cursor-does-better-slam-dunks.md` | G2 | **G20** | M | Cursor integration research = multi-tool |

> **Caveat (drives an UNCLEAR):** G20 is **achieved** (it was the one-time emergency repo-extraction). The
> ongoing portable-multi-tool-OS architecture (torch / orchestrators / adapters) is conceptually a *successor*
> initiative. Routing these to G20 keeps the lineage but parks them under an achieved goal. **The cleaner fix
> may be a NEW active goal "Portable multi-tool agentic-OS (core + per-tool adapters)."** Flagged in UNCLEAR —
> these 8 are the strongest argument for it.

---

## UNCLEAR — needs user judgment (13 files) → ALL RESOLVED 2026-06-08 (nap-work routing)

> **RESOLVED 2026-06-08 (nap-work).** The user authorized "pick best-fit for each — your judgment, reversible"
> for the stray files. Decision 1 was already resolved separately (G21 created; the 8 portable-OS docs carry
> `belongs_to_goal: G21`). Decisions 2/3/4 (8 files) are now routed below — each `belongs_to_goal` set on disk
> with a frontmatter comment recording the decision. All reversible (additive frontmatter only; `serves_northern_star`
> preserved). The only divergence from the loose-ends-2026-06-08 §U2 recommendations is **A31 → G11** (loose-ends
> preferred G17) — chosen on the file's own self-declared `serves_guiding_light: G11` + `related_goals:[G6,G4,G11]`.

### Decision 1 — does a NEW "Portable multi-tool OS" goal exist? (drives 5 of the G20 rows) — ✅ RESOLVED (G21)
Resolved separately (loose-ends §1): **yes** — `goal-billboard/active/G21-portable-multi-tool-agentic-os.md`
created 2026-06-08T13:00Z; the 8 portable-OS / torch / Cursor-adapter docs re-pointed G20 → **G21**. No nap-work
edit needed (already applied in the wave).

### Decision 2 — agentic-system *surveys*: framework (G15) or substrate (G4)? — ✅ RESOLVED
| File | RESOLVED → | why (1-line) |
|---|---|---|
| `research/rabbit-holes/top-starred-agentic-github/README.md` | **G15** | RH-001 exemplar-pattern survey = G15's framework domain; `companion_goal` already G15 |
| `research/rabbit-holes/non-github-agentic-systems/README.md` | **G15** | RH-002 same — exemplar survey, not skill substrate; `companion_goal` already G15 |
| `goal-billboard/audits/A24-meta-loop-eval-cross-utilization.md` | **G4** (already-set, confirmed) | cross-utilization + meta-eval is a skill-substrate eval; left at its B4-pass G4 |

### Decision 3 — graceful-lifecycle cluster: G10 (lifecycle) vs G14 (realignment) vs G4 (hooks)? — ✅ RESOLVED
| File | RESOLVED → | why (1-line) |
|---|---|---|
| `goal-billboard/audits/A30-graceful-shutdown-startup-overhaul.md` | **G10** | title is verbatim G10's (suspend-resume model); title-match wins; `serves_guiding_light`+`related_goals` already G10 |
| `goal-billboard/audits/A31-pre-restart-prep-master.md` | **G11** | dominant front = master-session-writeup + session-bundle infra; file's own `serves_guiding_light: G11`. (Diverges from loose-ends G17 rec — reversible) |
| `goal-billboard/audits/A32-hook-audit-testing-and-post-shutdown-automation.md` | **G4** (already-set, confirmed) | hook-audit substrate; left at its B4-pass G4 |

### Decision 4 — the 2 leftover process audits — ✅ RESOLVED
| File | RESOLVED → | why (1-line) |
|---|---|---|
| `goal-billboard/audits/A2-critical-script-coverage-audit.md` | **G9** (re-routed from G4) | "critical script COVERAGE" = testing-toolbelt domain; coverage signal stronger than substrate |
| `research/time-precision-audit-20260607.md` | **G4** | true-time precision = cross-cutting OS substrate discipline, not scheduler-specific (G1); G7 GL is paused/absorbed by G17 |

---

## SET-ASIDE — recommend NOT auto-stamping (21 files)

### Goal-definition files (16) — a goal is a parent, not a child
These carry `serves_northern_star: G2` legitimately (the migration default) but **should not get a
`belongs_to_goal`** — a goal does not belong to another goal. **Recommendation: leave `belongs_to_goal` UNSET**;
the parser should treat a `G*.md` goal file as the *node*, not a member. (If the user instead wants self-grouping
so each goal card lists its own definition, stamp `belongs_to_goal: <self>` — but that is a parser/UX choice, not
a data fix.)

`active/`: G1, G2, G4, G6, G8, G9, G10, G11, G14, G15, G17, G18, G19 · `paused/`: G3, G7, G12, G13, G16 ·
`archived/achieved/`: G5, G20. (G5→G18-lineage and G20-as-portable-OS already discussed above as *content*
references, but the goal *files* themselves stay unset.)
*(Count note: 13 active + 5 paused + G5 = 19 goal files carry the stamp; 2 of the "ideas/findings" below round
the set-aside to 21.)*

### Ideas (5) — surface-only by design (do-not-auto-apply)
goal-wiring §3d: the ideas lane shouldn't be force-linked. Topical suggestion given for reference only.
| File | topical suggestion | note |
|---|---|---|
| `goal-billboard/ideas/I-001-board-svg-strings.md` | (G18) | board SVG connectors — DO NOT auto-apply |
| `goal-billboard/ideas/I-002-card-slide-animation.md` | (G18) | dashboard animation — DO NOT auto-apply |
| `goal-billboard/ideas/I-006-steal-from-cursor-capability-upgrades.md` | (G4 / G20) | Cursor-derived upgrades, explicitly backlog/icebox — DO NOT auto-apply |
| `goal-billboard/ideas/I-007-refine-mixed-memory-topics.md` | (G4) | memory-topic split, "later, case-by-case" — DO NOT auto-apply |
| `goal-billboard/ideas/I-008-rest-adapter-work-substrate.md` | (G20 / G4) | REST work-substrate adapter — DO NOT auto-apply |

### Other set-aside
| File | note |
|---|---|
| `goal-billboard/audits/A70-intentionally-skipped.md` | "Intentionally skipped audit ID — no audit assigned." No subsystem → leave unset. |
| `goal-billboard/audits/README.md` | the audit-catalog index, not an audit → leave unset (or G18 if treated as a board artifact). |
| `plans/README.md` | the plans-folder index → leave unset. |
| `goal-billboard/archived/achieved/_G20-handoff-new-home-PROMPT.md` | G20 handoff prompt — **G20** if linked, else set-aside (achieved). |
| `goal-billboard/archived/audits/A45-required-service-health-and-port-protection.md` | **G4** (service-health/port substrate) — moved to G4 apply-set; listed here only to note it was reviewed. |
| `goal-billboard/archived/audits/A71-inbox-never-lost-mutation.md` | **G4** (inbox-substrate adaptive-immunity) — G4 apply-set. |
| `goal-billboard/archived/audits/A75-workflow-4.7-vs-4.8-and-conflict-check.md` | **G4** (workflow/skill system vs 4.8) — G4 apply-set. |
| `goal-billboard/archived/audits/A76-semantic-index-adoption-downside-analysis.md` | **G4** (semantic-index = skill substrate) — G4 apply-set. |
| `goal-billboard/archived/audits/A77-idea-conflict-check-capability.md` | **G4** (idea-conflict-check gate = skill substrate) — G4 apply-set. |
| `goal-billboard/audits/findings/port-inventory-audit-2026-06-01.md` | **G4** (port/settings substrate) — G4 apply-set. |

> The 7 "moved to G4" rows above are folded into the G4 count in the corrected tally (they were reviewed in the
> set-aside pass and have a clean G4 home, so they are NOT left unmapped).

---

## Corrected final tally (authoritative)

The per-section "(dup)" cross-references were to prevent double-counting. Counting each file **once** by its
primary proposed goal:

| Proposed goal | count | confidence mix |
|---|---|---|
| **G4** (substrate) | 46 | mostly H/M (largest lever) |
| **G2** (Evium site) | 31 | almost all H |
| **G18** (dashboard) | 16 | mostly H (excl. 2 ideas counted under set-aside) |
| **G9** (testing) | 5 | all H |
| **G1** (scheduler) | 9 | H/M |
| **G17** (cluster/throughput) | 5 | M |
| **G20** (decouple/portable-OS) | 8 | M (pending Decision 1) |
| **G15** (RH framework) | 2 | H |
| **G14** (realignment) | 2 | H/M |
| **G12** (local-AI) | 1 | H |
| **G6** (instant-ack) | 1 | H |
| **G11** (session capture) | 1 | H |
| **UNCLEAR** (Decisions 1–4) | 13 | L |
| **SET-ASIDE** (goal files 19 + ideas 5 + README/skipped 5) | 26* | n/a — recommend leave unset |

\* set-aside overlaps: the 5 ideas are also given a topical hint; goal files = 19; non-audit READMEs +
A70-skipped = 5; total distinct files set-aside ≈ 26.

**Reconciliation to 166:** 46+31+16+5+9+5+8+2+2+1+1+1 (confident = **127**) + 13 unclear + ~26 set-aside ≈
166 (±2 from files that appear once but were discussed in two sections). **Headline: 166 scanned · ~127
confidently re-mappable · 13 unclear · ~26 set-aside (19 of those = goal-definition files that should stay
unset).**

> The summary line at the top says "132 confident / 13 unclear / 21 set-aside" from the first-pass tally; this
> corrected table (**127 / 13 / 26**) supersedes it after the dedup + the goal-file/idea set-aside accounting.
> The difference is entirely the goal-definition files (correctly moved from "confident G-self" to "leave
> unset") and a handful of audits pulled to their true parent. **Use the corrected table.**

---

## Recommended apply sequence (for AFTER user approval — NOT done here)

1. **Confirm Decision 1** (new portable-OS goal? → re-target the 8 G20 rows) and **Decision 3** (A30→G10 H
   if title-match wins) — these two unlock 13 of the unclear/soft rows.
2. Stamp the **H rows** first (idempotent, additive — write `belongs_to_goal:` only where absent, never
   overwrite), mirroring `migrate-add-ns-field.py`'s shape but writing the *parent* field. Biggest payoff:
   the 31 G2 multi-change-queue items + the G4/G18 audit clusters.
3. Stamp the **M rows** after a skim of the generated diff.
4. Leave **goal-definition files** and **ideas** unset (or apply the parser-side "goal file = node" rule).
5. Re-run `parse_unified_board.py`; verify by-goal sizes spread (G2 ≲ its real site share; ≥6 goals with ≥10
   members; the degenerate single-bucket gone).

**No frontmatter, goal file, plan, research doc, audit, or parser was mutated producing this proposal.
PROPOSALS ONLY — awaiting the user's per-goal approval.**
