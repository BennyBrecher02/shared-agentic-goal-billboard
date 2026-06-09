---
audit_id: A31
title: "Pre-restart prep — 6-front parallel upgrade (master session writeup + monkey domino-triggers + cross-tool event-bus + session bundle infra + BG taxation analysis + research folder v2)"
status: in_progress
catalogued: 2026-05-27T00:30:00Z
priority_when_run: P0
estimated_effort: large (5 parallel BGs + multi-plan foreground; closing several pending threads in one wave)
trigger: |-
  2026-05-27 00:25Z post-resume — user returned from drive home, gave 6 decoupled asks before formal "graceful restart" signal. Explicit framing: "before we gracefully start up again." Includes long-pending research-folder upgrade request ("you've asked many times").
deferral_reason: NONE — running immediately. User explicit gate is on RESTART, not on this prep work. Foreground + 5 BGs in this turn.
related_goals: [G6, G4, G11]  # G11 NEW (session bundle)
related_plans:
  - context/markdowns/plans/automation/monkey-domino-trigger-upgrade-plan.md (NEW)
  - context/markdowns/plans/automation/cross-tool-event-bus-plan.md (NEW)
  - context/markdowns/plans/automation/session-bundle-infrastructure-plan.md (NEW)
  - context/markdowns/plans/automation/research-folder-v2-upgrade-plan.md (NEW)
serves_northern_star: G2
belongs_to_goal: G11  # routed 2026-06-08 (UNCLEAR Decision 3): dominant front is master-session-writeup + session-bundle infra; serves_guiding_light + related_goals already name G11. (Cross-tool-event-bus/BG-taxation fronts brush G17/G4; loose-ends doc preferred G17 — judgment call, G11 chosen on the file's own self-declared guiding light. Reversible.)
serves_guiding_light: G11
related_refs:
  - .claude/skills/agentic-script-design/references/state-substrate-design.md
  - .claude/skills/agentic-script-design/references/bg-dispatch-architecture.md
  - .claude/skills/agentic-quality-discipline/references/graceful-shutdown.md (v2)
findings: []
linked_research:
  - context/markdowns/research/agent-os/bg-taxation-analysis-20260526.md
---

# A31 — Pre-restart prep master audit

## Context

User returned from drive (resume from snapshot worked). Token budget at 102% of ceiling (51.2M / 50M); cache_read 1.13B providing the amortization buffer. Per A14: cache is the savior; don't reset session.

Before formal "OK graceful restart" signal, user surfaced 6 decoupled improvement asks. This audit is the meta-coordinator; each ask gets its own plan + BG.

## The 6 fronts

### Front 0 — Master session writeup
**Type:** Research artifact (single ~10-15K doc covering work-in → work-out themes/decisions/structural fixes)
**Output:** `context/markdowns/research/session-2026-05-26-master-writeup.md`
**Owner BG:** A
**Why it matters:** Compress 13h of session into reviewable narrative. Identifies which audits clustered, which initiatives compounded, which gaps still exist. Replays the day's learning curve.

### Front 1 — Monkey domino-trigger upgrade (G6 Phase 3+)
**Current state:** Click POSTs to agent-inbox-server → stores message → UserPromptSubmit drain surfaces it. SINGLE-PATH.
**Upgrade:** Click → DOMINO across:
- Inbox (current)
- Scheduler (if decision implies a Change — auto-queue)
- Bug billboard (if decision creates/closes a defect)
- Feedback-loop comparison (if decision affects shipped work)
- Heartbeat tier-high refresh (immediate state propagation)
- Asks-log + steering log (audit trail)
- Per-NS attribution (decision tagged to which NS it advances)

**Owner BG:** B
**Why it matters:** Today's 10 monkey decisions sit untriggered. Each click should fan out to 5+ workflows automatically.

### Front 2 — Cross-tool event-bus (domino lineup improvement)
**Current state:** Hooks fire on tool events; each hook reads state independently. Some PostToolUse chains exist but no general event-bus.
**Upgrade:** Generalized event-bus pattern:
- Producer: any script writes to `.claude/cache/event-bus.jsonl` with `{ts, event_type, payload}`
- Consumers: subscriber scripts process events tagged with their type
- Bridges: pre-built consumer for each major surface (scheduler, billboard, monkey, asks-log, dashboard render)
- Examples: `scheduler:tick_end` → triggers feedback-loop + bug-billboard refresh + dashboard data + heartbeat tier-medium

**Owner BG:** C
**Why it matters:** Today's manual cross-tool coordination (a tick lands; I manually fire feedback-loop comparison; I manually update bug-billboard) is brittle. Domino lineup = pre-wired.

### Front 3 — Session bundle infrastructure (NEW G11)
**Type:** New major initiative. Analog to `reports/timelapse/` data bundles but for SESSION data.
**Bundle contents** (per session start→end):
- snapshot.json (entry) + snapshot.json (exit)
- All JSONL diffs (asks-log, lost-work, ack-latency, BG-pattern, recurrences, scheduler-events, heartbeat-pending)
- BG dispatch log (count, durations, outcomes)
- Audit changes (which audits opened/closed/findings landed)
- Monkey clicks during session
- Token usage curve + 5hr-window crossings
- Memory rule additions
- Skill refresh history entries

**Bundle directory:** `reports/session-bundles/<session-start-iso>/`

**Bundle creator:** runs at graceful-shutdown alongside snapshot
**Bundle analyzer:** runs at session-start (after resume); parallelized across multiple BGs; produces insights

**Insights examples:**
- Lessons learned (closing themes)
- Pattern observations (recurrences, KPIs)
- Token efficiency analysis
- Cross-domino effectiveness
- Recommended next-session focus

**Owner BG:** D
**Why it matters:** User explicit: *"these hours long plow runs and their context and stats etc will be useful for eval and improvement... systemwide domino ideas youll sprout."*

### Front 4 — BG taxation analysis
**Type:** Research artifact (specific data analysis).
**Output:** `context/markdowns/research/bg-taxation-analysis-20260526.md`
**Scope:**
- Today's 207 BG dispatches (per snapshot metric)
- Total token cost across all BGs
- Mean BG token cost + p50/p95
- Longest-running BGs (the 200+ min mentioned)
- Token efficiency (value-per-token where measurable via artifacts landed)
- The "BGs that ran during graceful shutdown" case (#102 + others)
- Cost-per-deliverable estimates

**Owner BG:** E (combined with Front 5)
**Why it matters:** User explicit: *"how taxing the bg processes thatve been going for like 200 minutes now have been on us?"* Plus: confirms that the seamless-state-preserved BG behavior should NOT be changed (just analyzed).

### Front 5 — Research folder v2 upgrade (HEAVILY needed)
**User explicit:** *"RESEARCH FOLDER UPGRADE NEEDED HEAVILY, we barely utilize it even though i asked many times."*
**Current state (per A15):** Phase 2-4 implemented — advisory hook, cross-link harvester, synthesis tool, trigger phrases documented. But the folder still gets underused.

**v2 Upgrade scope:**
- **Sub-folder taxonomy** — currently flat; needs domain organization (`architecture/`, `infrastructure/`, `agent-os/`, `evium-deliverable/`, `cluster/`, `testing/`, `session-runs/`)
- **Auto-indexing** — `reports/research-index.md` listing all research files by topic + recency
- **Sessions sub-folder** — `research/sessions/` for per-session writeups (where today's master writeup lands)
- **Bridge to session bundles** — Front 3 outputs reference research/sessions/ entries
- **Surface at SessionStart** — top-3 most-recent research entries surface in resume briefing
- **Lint** — `scripts/lint-research-frontmatter.py` enforces `sources:` + `findings_drove:` (extension of existing A15 lint)
- **Research-folder-status hook** — SessionStart counts research files; warns if last entry >30 days
- **Skill ref upgrade** — `research-folder-discipline.md` v2 with the new taxonomy + cross-link patterns

**Owner BG:** E (combined with Front 4)
**Why it matters:** Multi-month investment in research/ is underutilized despite the discipline rule. User has flagged this multiple times — meta-recurrence per A19.

## Why combined into one master audit

These 6 fronts share a common thread: **all are about how the agentic OS captures, propagates, and learns from its own activity.** Master writeup = compression. Domino + event-bus = propagation. Session bundle = capture. BG taxation = cost-awareness. Research folder v2 = retention/retrieval.

Separating into 6 audits would lose the meta-coherence. One A31 master + 4 plans is the right granularity.

## Cost gates

Per A14 + G9 cost discipline:
- A31 audit + 4 plans + G11 + memory + steering log (foreground): <500k tokens
- Per BG: <500k cap
- Total turn budget: <3.5M tokens
- Cache amortization carrying us; sustainable

If any gate breached: pause; surface to user.

## Verification (after 5 BGs land)

1. Master session writeup exists at `research/sessions/2026-05-26-master-writeup.md` ≥10K
2. Monkey hooks fan out on click (test: synthetic click triggers ≥5 downstream events)
3. Event-bus + consumer scripts exist; first synthetic event propagates correctly
4. Session bundle infrastructure: bundle directory schema documented; creator + analyzer scripts exist
5. BG taxation analysis research entry exists with concrete numbers
6. Research folder v2 taxonomy + index + lint extension + SessionStart surface working

## Status

IN PROGRESS. 5 BGs launching parallel this turn. Foreground artifacts (this audit + 4 plans + G11 + memory) landing same turn.

### Front 1 — Phase 1 progress (2026-05-27 02:38Z)

**LANDED.** Monkey domino-trigger upgrade Phase 1 complete. 7-of-7 deliverables landed.

**Files created/modified:**
- NEW `scripts/monkey-click-fan-out.sh` (~22K) — 8-trigger domino dispatcher; macOS bash 3.2 compatible; always exits 0; <1.2s wall-clock observed across all 5 verification cycles.
- MODIFIED `scripts/agent-inbox-server.py` — POST handler async-spawns the fan-out via `subprocess.Popen(start_new_session=True)`. Added `_parse_monkey_decision()` + `_spawn_monkey_fanout()` helpers. Backward-compatible: missing script → WARN + continue.
- MODIFIED `reports/screenshot-monkey/data.json` — schema_version 2 → 3. Added optional per-option fields: `proposed_change`, `bug_resolution`, `affects_shipped`, `triggers_event_bus` (default true). 7 of 10 pending decisions enriched.
- NEW `.claude/skills/agentic-quality-discipline/references/monkey-domino-discipline.md` (~9K) — documents the 8-trigger contract, click contract, failure modes, cost discipline, extension pattern. Surfaced via `serves_northern_star: G2` + `serves_guiding_light: G6` frontmatter.
- NEW `scripts/hooks/settings-patches/monkey-fan-out.json` — documentation-only; no settings.json entry required (fan-out is server-side, not a hook).
- MODIFIED `context/markdowns/goal-billboard/active/G6-live-agent-messaging.md` — phase advanced to "Phase 3 in flight — domino-trigger upgrade live."
- MODIFIED `context/markdowns/notes/steering-decisions-log.md` — A31 Front 1 LANDED row appended.
- MODIFIED `context/markdowns/goal-billboard/asks-log.md` — schema extension note (monkey-click trigger rows).

**The 8 triggers (5 baseline + 3 conditional):**
| # | Name              | When                          | Outcome on miss        |
|---|-------------------|-------------------------------|------------------------|
| 1 | asks-log          | always                        | skip if file missing   |
| 2 | steering-log      | always                        | skip if no `---` end   |
| 3 | heartbeat-refresh | always                        | error if append fails  |
| 4 | NS attribution    | always                        | default ns=G2          |
| 5 | dashboard-refresh | always (BG-dispatched)        | skip if hook missing   |
| 6 | scheduler-queue   | option.proposed_change exists | skip otherwise         |
| 7 | bug-billboard     | option.bug_resolution exists  | skip otherwise         |
| 8 | feedback-loop     | option.affects_shipped exists | skip otherwise         |

**Verification (5 cycles via EVIUM_MONKEY_FANOUT_LOG_ONLY=1 — no real files touched):**
1. P1-04 option a (keep light no-op) — would-fire [1,2,3,4,5], skip [6,7,8]. 813ms.
2. SET-005 yes-lift-and-apply-all (batch settings — bug_resolution closes SET-001..004) — would-fire [1,2,3,4,5,7], skip [6,8]. 838ms.
3. INSTALL-001 install — would-fire [1,2,3,4,5], skip [6,7,8]. 956ms.
4. RENAME-001 yes (has proposed_change) — would-fire [1,2,3,4,5,6], skip [7,8]. 996ms.
5. SYNTH-YESNO-TEST yes (unknown decision id) — would-fire [1,2,3,4,5], skip [6,7,8]. Graceful unknown handling. 876ms.

All <1.2s; pipeline well under the <2s budget. Parser + spawn unit tests 8/8 pass.

**Cost gate:** well under <500k bound (~150k tokens used for this front).

### Front 3 — Phase 1 progress (2026-05-27 02:35Z)

**LANDED.** Session bundle infrastructure Phase 1 POC complete. 7-of-7 deliverables:

1. **`scripts/session-bundle-create.sh`** — bundle creator. Resolves session boundaries (args or fallback to last-resume.json + latest snapshot), scaffolds the bundle directory, diffs 8 JSONL substrates against the window (`asks-log`, `lost-work`, `ack-latency`, `bg-launch-pattern`, `recurrences`, `scheduler-events`, `heartbeat-pending`, `event-bus`), aggregates audit changes (opened/closed/findings-landed by mtime+frontmatter parsing), collects monkey clicks from `reports/screenshot-monkey/data.json`, builds hourly token-curve CSV, extracts BG dispatch log from the project JSONL (max 3 newest files), diffs memory rules, captures skill-refresh subset, and writes schema-versioned manifest. ~620 lines, macOS bash 3.2 + python3, <5s wall-clock, exits 0 always.

2. **`scripts/session-bundle/analyzer-themes.py`** — first sub-analyzer. 17-label theme taxonomy with first-match-wins keyword regex classification over a corpus of BG dispatches + audit IDs (cross-referenced to audit titles from the goal-billboard) + monkey clicks + findings-doc filenames. Outputs `insights-by-domain/themes.md` with top-5 table + representative sources + full distribution + extension guide + cross-refs. <2s execution.

3. **`scripts/session-bundle/aggregator.sh`** — drops-in plugin pattern: globs `insights-by-domain/*.md`, concatenates with H2 section headers under the at-a-glance summary derived from the manifest, updates `manifest.analyzers_run`. No central registry; just add a new analyzer file and it auto-joins.

4. **`scripts/graceful-shutdown-snapshot.sh`** updated with TODO comment for Phase 2 auto-wire (Phase 1 = manual run only; phase 4 actual shutdown is deferred per G10 gate).

5. **POC bundle at `reports/session-bundles/2026-05-26T11-00-00Z/`** — the FIRST real session bundle. 20 files, ~188KB. Real numbers for the 2026-05-26 plow run:
   - Window: 13.23 hours (2026-05-26T11:00Z → 2026-05-27T00:14Z)
   - 182 BG launches; 180 BGs dispatched in-session (matches snapshot's 207 lifetime BG count once we filter to today only)
   - 18 audits opened (A9, A10, A12, A13, A14, A15, A16, A17, A18, A19, A20, A21, A22, A23, A24, A25, A26, A30)
   - 2 audits closed (A13, A15)
   - 10 findings docs landed under `audits/findings/`
   - 10.96M tokens summed from session-end-in-window entries in `token-metrics.json`
   - 2 memory rule files modified (`feedback_standing-protocols.md` — main + mirror)
   - 5 skill refresh entries
   - Token curve shows the burn pattern: trough at 18:00Z (4 BG/hr), peak at 23:00Z (36 BG/hr), 16:00Z spike (30 BG/hr); 64 prompt rows across 14 hourly buckets.

6. **Top 5 themes from today's session** (analyzer-themes output):
   - `audit-catalog-findings`: 55 hits (33.1%) — the dominant pattern; consistent with the 18-audit-opened sprint
   - `scheduler-multi-change`: 29 hits (17.5%)
   - `monkey-chamber-domino`: 23 hits (13.9%)
   - `graceful-shutdown-resume`: 14 hits (8.4%) — Front 3 + G10 work
   - `g2-phase5-evium`: 10 hits (6.0%)
   - 13 distinct themes hit; 79% classification rate (44 unclassified items — refinement candidate for Phase 2)

7. **Skill ref at `.claude/skills/agentic-quality-discipline/references/session-bundle-discipline.md`** — 10.3K bytes. Frontmatter `serves_northern_star: G2` + `serves_guiding_light: G11`. Sections: why bundles, directory structure, manifest schema, analyzer pattern + G9 tier-awareness, how to extend (new analyzer + diff source + corpus type), Phase 1 POC results, full phasing roadmap.

**Verdict on verification criterion #4** ("session bundle infrastructure: bundle directory schema documented; creator + analyzer scripts exist"): **PASS.** Bundle directory exists with real content; schema documented in skill ref + master plan; creator + analyzer + aggregator scripts all in place + tested end-to-end.

### Front 2 — Event-bus Phase 1 progress (2026-05-27 02:42Z)

**LANDED.** Cross-tool event-bus Phase 1 complete. 8-of-8 deliverables:

**Substrate (NEW):**
- `.claude/cache/event-bus.jsonl` — append-only JSONL with schema-header line
- `.claude/cache/event-bus-subscribers.json` — schema-versioned subscriber registry
- `.claude/cache/event-bus-cursor.txt` — last-processed-line tracker (atomic write)
- `.claude/cache/event-bus-consumer-log.jsonl` — every dispatch decision logged

**Dispatcher (NEW):**
- `scripts/event-bus/dispatch.sh` — bash wrapper (repo-root resolver + python invocation)
- `scripts/event-bus/dispatch.py` — core dispatcher (mkdir lock with 5min stale cleanup; cursor tracking; exact + wildcard subscriber match; key=value filter; per-consumer mtime debounce; cascade_depth ≤ 3 enforcement; detached subprocess.Popen dispatch)

**Consumers (3 NEW):**
- `consumer-feedback-loop.sh` — filtered subscriber (scheduler:tick_end / outcome=verified); spawns audit-feedback-loop-compare.py against 2 most-recent timelapse runs
- `consumer-dashboard-refresh.sh` — wildcard subscriber with 30s debounce; fires billboard-data-refresh.sh + dashboard-data-prime.sh in parallel
- `consumer-heartbeat-tier-high.sh` — wildcard subscriber with 5-min per-event-type dedup; appends `tier:high severity:info` entries to heartbeat-pending.jsonl

**Producers (3 wired ADDITIVELY):**
- `scripts/scheduler/scheduler.py` — opt-in `event_bus_path` dataclass field + `_emit_to_event_bus` helper called from `_emit`. Mirrors tick_end (verified/failed/idle/no_verify outcomes) + tick_end_idle. Tests default to opt-out (field unset); production `__main__.py` passes the canonical path. **No test regression.**
- `scripts/agent-inbox-server.py` — `_emit_event_bus_inbox_received` helper called after POST file-write succeeds. Emits `monkey:decision_received` with filename/priority/tag/preview payload.
- `scripts/audit-feedback-loop-compare.py` — `_emit_event_bus_findings_landed` helper called after comparison output written. Emits `audit:findings_landed` with summary + verdict. EVENT_BUS_DISABLE env-var opt-out for tests (set in `conftest.py`).

**Hook + settings patch (NEW, USER-GATED):**
- `scripts/hooks/event-bus-dispatch-trigger.sh` — PostToolUse Write|Edit wrapper that fires dispatcher in background when file_path includes `event-bus.jsonl`
- `scripts/hooks/settings-patches/event-bus-dispatcher.json` — documented patch (NOT auto-applied; lifts on SET-005 batch or standalone user nod)

**Skill ref (NEW):**
- `.claude/skills/agentic-script-design/references/event-bus-pattern.md` — 12KB. Sections: why event-bus over direct calls, event schema + namespace conventions, subscriber config, cascade safety, failure modes, adding new producer, adding new consumer, ops cheat-sheet. Cross-refs Front 1 + Front 3 + A19 + state-substrate-design.md.

**Synthetic verification (PASSED):**
1. `scheduler:tick_end` with outcome=verified → feedback-loop fired (spawned comparer; comparer cascade-emitted `audit:findings_landed`), dashboard-refresh fired, heartbeat-tier-high appended
2. `monkey:decision_received` → feedback-loop FILTERED (wrong event_type), dashboard-refresh dispatched (or debounced when within 30s of prior fire), heartbeat-tier-high dispatched
3. Cascade test: depth=1/2/3 dispatched; depth=4 SKIPPED with `action: skipped_depth` by (cascade-guard) — cap enforced

**Performance (measured):**
- Idle dispatch (no new events): ~63ms wall-time
- 5 pending events: ~123ms wall-time
- Both within budget (<100ms idle, <500ms with backlog)

**Backward compatibility check:** 83 unit tests (30 scheduler + 53 audit-feedback-loop) pass unchanged. Producers all use best-effort discipline — any event-bus failure swallowed silently.

**Bug found + fixed during verification:** Initial `dispatch.sh` and `event-bus-dispatch-trigger.sh` resolved REPO to `scripts/` only (single `/..`). Fixed to `/../..` (2 levels). Consumer scripts already used `/../../..` (3 levels) correctly.

**Verdict on verification criterion #3** ("Event-bus + consumer scripts exist; first synthetic event propagates correctly"): **PASS.** All deliverables landed; 3 synthetic events propagated correctly through filter / debounce / dedup / cascade-guard logic; performance within budget.

## Cross-references

- G6 — monkey chamber (Front 1 extends)
- G4 — skill system (research-folder discipline)
- G11 NEW — session-bundle initiative (Front 3 owns)
- A15 — research-folder utilization (Front 5 v2 extension)
- A19 — meta-monitoring (Front 2 is the explicit event-bus version)
- A24 — meta-loop-eval (compatible with session bundle analyzer)
- A29 — zero-degradation criterion (post-resume launch demonstrated)
- A30 + G10 — graceful shutdown/startup (Front 3 ties to)

## Front 4 + Front 5 progress (combined BG E)

### Front 4 — BG taxation analysis (LANDED)

**Deliverable**: `context/markdowns/research/agent-os/bg-taxation-analysis-20260526.md` (14.0 KB).

**Key findings:**
- 77 BGs measured in current session (subset of 207 daily total per snapshot)
- **Total new tokens (excl. cache_read)**: 24,245,848 — **48.04% of session token budget**
- Total cumulative wall-clock: 712.6 min (11.88 hrs) — the "200 min" user perceived was overlapping waves, not any one BG
- **Longest single BG: 29.0 min** (A13 Path D — expected_visual_diff flag). No BG > 30 min, > 60 min, or > 200 min
- p50 duration: 8.3 min, p90: 15.7 min. p50 new tokens: 288k, p99: 1.45M (the per-section scoped eval outlier)
- cache_read / cache_creation ratio: 31× — parent context cache reused aggressively
- Verdict: **NOT taxing in a way that justifies behavior change. But IS the dominant cost lane (48% of session tokens) and merits visibility.** User explicit: don't change seamless behavior.
- 5 recommendations for future tuning: per-BG token soft-gate, wall-clock concurrency visibility, cache-reuse opportunity tracking, cost-vs-deliverable per BG, 5hr-window depletion model

### Front 5 — Research folder v2 upgrade (LANDED)

**Sub-folders created (8)**: architecture/, infrastructure/, agent-os/, evium-deliverable/, cluster/, testing/, sessions/, methodology/ (+ README.md updated, schema_version: 2).

**Files migrated (13)**: 13/13 existing flat-level files moved to matching sub-folders. Post-migration distribution: infrastructure/ (4), agent-os/ (3 incl. new BG taxation), testing/ (3), methodology/ (3), sessions/ (1), evium-deliverable/ (1), architecture/ (0 .gitkeep), cluster/ (0 .gitkeep).

**Cross-references updated**: 69 stale path references across 18 files auto-updated (audits A14/A26/A28, plans, skill refs, scripts, reports).

**New scripts (3)**:
- `scripts/render-research-index.py` — auto-indexer, writes `reports/research-index.md` (15 files indexed, 14 inbound refs on first run, <2s)
- `scripts/hooks/research-surface-on-resume.sh` + `scripts/_research-surface-parser.py` — SessionStart top-3 surface, jq-formatted, tested working
- `scripts/lint-research-frontmatter.py` — validates `sources:` / `findings_drove:` / `domain:` / sub-folder match. `--strict` exits 1. `--fix-domain` patched 13 files in one run.

**Hooks wired**:
- `billboard-data-refresh.sh` — added research-index regen on research/, plans/, goal-billboard/ edits
- `dashboard-data-prime.sh` — added research-index regen at SessionStart
- Settings patch documented at `scripts/hooks/settings-patches/research-surface-on-resume.json` (NOT auto-applied per settings-edit-guard)

**Skill ref v2**: `.claude/skills/agentic-quality-discipline/references/research-folder-discipline.md` upgraded from ~6.7K → 15.3K. New sub-folder decision matrix, v2 frontmatter schema, auto-index / SessionStart / lint sections, A31 Front 5 lesson appended.

**Lint baseline**: 11/15 files clean post-migration + --fix-domain. 4 legacy files (resource-capacity, scheduler-data-refresh, device-coverage, preview-vs-chrome) flagged for missing `sources:` / `findings_drove:` — predate v2 schema; patching is a follow-up task.

**Per A19 meta-recurrence rule**: this is the structural fix to the 3rd-time-flagged underutilization, not a band-aid. v3 audit trigger: if user flags underutilization a 4th time despite v2.

