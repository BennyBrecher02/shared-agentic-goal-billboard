---
audit_id: A25
title: "settings.json health + hook chain hygiene — guard unwired, port bug, 18 orphan hooks, chain getting long"
status: in_progress
catalogued: 2026-05-26T23:50:00Z
priority_when_run: P0
estimated_effort: medium
trigger: |-
  2026-05-26 23:48Z — user audit *"hows our settings.json looking? i havent asked in a while and we've grown our system so much. look for areas we could/should improve."* Discovered: settings-edit-guard.sh is NOT wired (behavioral protection only); test:e2e dev-server check uses wrong port (4399 instead of 4321); 18 hooks on disk are orphan (pending patches not applied); hook chain grows to 48+ entries after patches.
deferral_reason: NONE — running immediately. Critical-severity items (guard unwired, port bug) need user approval to fix but the audit + plan must land now so they don't slip.
related_goals: [G4]
related_plans: [context/markdowns/plans/automation/settings-json-cleanup-plan.md]
serves_northern_star: G2
belongs_to_goal: G4
serves_guiding_light: G4
related_refs:
  - .claude/skills/agentic-quality-discipline/references/settings-edit-guard.md (if exists; check)
  - .claude/settings.json
  - scripts/hooks/settings-patches/
findings: []
---

# A25 — settings.json health + hook chain hygiene

## Why this audit matters

The settings.json is the **structural enforcement spine** of the entire agent operating system. Every memory rule + skill ref relies on hooks defined here. If the file is inconsistent or hooks are unwired, the behavioral discipline is doing all the work — that's load-bearing only as long as the agent (me) follows it perfectly.

User explicitly asked: *"look for areas we could/should improve."*

## Findings (10 items, severity-ranked)

### 🔴 Critical

#### 1. `settings-edit-guard.sh` exists on disk but is NOT wired

The guard script lives at `scripts/hooks/settings-edit-guard.sh`. There is no settings.json entry that invokes it (no PreToolUse Write|Edit matcher referencing it). The protection I've cited all session ("settings-edit-guard blocks me") has been **behavioral** (memory rule + agent discipline) not **structural** (hook enforcement).

Impact: If a future agent OR a compromised agent bypassed memory rules, settings.json could be edited unilaterally. The safety model assumes the guard exists; it doesn't.

Fix: wire `settings-edit-guard.sh` via PreToolUse Write|Edit matcher with the appropriate filter (only fires when target is `.claude/settings.json`).

#### 2. test:e2e dev-server check uses wrong port

settings.json line 34 contains an inline bash check:
```bash
curl -s --max-time 2 -o /dev/null -w "%{http_code}" http://localhost:4399/
```

But Astro dev server runs on **port 4321**. So this check ALWAYS reports "dev server not responding on :4399" regardless of whether Astro is up. False-positive warnings on every Bash tool call that contains "test:e2e."

Fix: change port to 4321 (and consider extracting to a dedicated script — see Finding 7).

### 🟡 High

#### 3. 18 hooks on disk are orphan (unwired)

```
ack-latency-measure.sh
established-workflows-check.sh
heartbeat-drain.sh
hedge-detect-drain.sh
hedge-detect.sh
lost-work-drain.sh
lost-work-token-cost.sh
parallel-capacity-check.sh
pending-parallel-work-scan.sh
recurrence-detect.sh
research-cross-link-harvest.sh
research-folder-advise.sh
scheduler-heartbeat-coordinator.sh
settings-edit-guard.sh
snapshot-baseline-staleness-check.sh
state-batch-digest.sh
user-ask-artifact-check.sh
user-ask-detect.sh
```

Patches exist under `scripts/hooks/settings-patches/`:
- ack-latency.json
- ask-detect-stop.json
- established-workflows.json
- heartbeat-drain.json
- hedge-detect.json
- lost-work.json
- parallel-launch-discipline.json
- research-folder-advise.json
- state-batch-recurrence.json

Plus pending from in-flight BGs: scheduler-heartbeat-coordinator (BG #83), ns-advance-check (BG #85).

Total 11+ hooks needing wiring. SET-001..004 monkey-chamber decisions already track 4 of them. The remaining need either: separate monkey-chamber routes, OR a single "apply all pending patches" decision.

#### 4. UserPromptSubmit chain grows to 7+ hooks after patches

Currently 1 (inbox-on-prompt.sh). After patches: 7-9 hooks per prompt at ~50-400ms each = **1.5-3s of pre-response latency** on every prompt.

Mitigations:
- Performance baseline each hook + budget enforcement
- Parallel execution? (claude-code doesn't support hook parallelism but could be designed)
- Lazy hooks: e.g. heartbeat-drain only fires if heartbeat-pending.jsonl exists
- Conditional ordering: cheapest first, expensive last

#### 5. `agent-inbox-read.sh` runs on PreToolUse `.*` matcher

This fires on EVERY tool call (Bash, Edit, Write, Read, etc.). With long responses doing many tool calls, this hook runs many times per response. Cost depends on its implementation (file mtime check + drain if updated should be cheap, but worth measuring).

Potential redundancy: `inbox-on-prompt.sh` on UserPromptSubmit drains inbox at prompt entry. The PreToolUse `.*` version drains in-between tool calls. The latter exists because messages arriving mid-response wouldn't otherwise be visible. Confirmed-needed; just expensive.

### 🟡 Medium

#### 6. `scheduler-data-refresh.sh` runs on BOTH Write|Edit AND Bash PostToolUse

Lines 60-61 (Write|Edit matcher) AND line 76 (Bash matcher) both invoke `scheduler-data-refresh.sh`. Likely intentional (Edit and Bash both can change scheduler state) but worth verifying — could simplify or document the reasoning.

#### 7. Inline bash for test:e2e check is anti-pattern

Line 34's complex inline bash is hard to maintain. Extract to `scripts/hooks/test-e2e-dev-server-check.sh` so it's testable + readable.

### 🟢 Low

#### 8. Mixed hook locations

Some hooks live under `scripts/` root (sync-memory.sh, audit-recommendations.sh, etc.) and others under `scripts/hooks/`. Reasonable historical reason (older hooks at root; newer ones in hooks/) but inconsistent.

#### 9. No hook-health check

No mechanism to surface silently-failing hooks. A hook that returns exit 1 + prints to stderr is invisible unless the user reads logs. Add `scripts/hooks-health-check.sh` at SessionStart that:
- Runs each hook with `--health-check` flag (each script supports it as a no-op probe)
- Reports hooks that fail
- Surfaces in next-prompt context

#### 10. Permissions section bloat

100+ allow rules + 70+ deny rules. A9 audit in catalog already covers this — needs running. Some likely overlap (e.g. `rm scripts/*` allowed alongside many `rm -*` denies — interaction unclear).

## Root causes

1. **Patches accumulate without landing** — we keep building hooks faster than wiring them
2. **No automated "settings-edit safe path"** — the safety route (monkey chamber yes/no per patch) hasn't been used because monkey decisions are still pending
3. **No hook chain performance baseline** — we add hooks without measuring impact
4. **Settings.json is hand-maintained** — every change is a manual JSON edit; no schema validation

## Fix (designed; cleanup plan landing same turn)

See `plans/automation/settings-json-cleanup-plan.md`. Three phases:

### Phase 1 — Critical immediate fixes (needs user nod)
- Fix port 4399 → 4321
- Wire settings-edit-guard.sh
- Apply the 4 patches via monkey-chamber SET-001..004 (already pending)

### Phase 2 — Hook chain wiring (needs user nod for each patch group)
- Add a "settings.json batch apply" monkey-chamber decision covering all 11+ pending patches
- Define ordering for UserPromptSubmit + Stop chains
- Apply in one settings.json batch edit (single user-gated session)

### Phase 3 — Hygiene + monitoring (no user nod needed)
- Add `scripts/hooks-health-check.sh` at SessionStart
- Move inline test:e2e to dedicated script
- Audit redundant `scheduler-data-refresh.sh` invocation
- Run A9 (deny-list audit) and A3 (hook effectiveness audit) — both in catalog, never run

## Verification

After Phase 1+2:
1. settings-edit-guard.sh fires on attempted Write/Edit to settings.json
2. test:e2e dev-server check correctly detects Astro on port 4321
3. All 18 orphan hooks wired (settings-patches/ directory empty)
4. UserPromptSubmit chain has 7+ hooks; total time <3s measured

After Phase 3:
5. hooks-health-check.sh fires at SessionStart + reports zero failures
6. A9 permissions audit completes; bloat reduced
7. A3 hook effectiveness audit completes; obsolete hooks pruned

## Status

IN PROGRESS. Phase 3 (hygiene) can land via BG when ceiling opens. Phase 1+2 require user-approved monkey-chamber decisions.

## Progress

- 2026-05-26T23:59:59Z — SET-005 added to monkey chamber (`reports/screenshot-monkey/data.json`). Unified batch decision: 9 on-disk patches + META-fix wiring `settings-edit-guard.sh` + Phase 1.1 port-4399->4321 fix. 4 options (yes-lift-and-apply-all / yes-select-subset / no-manual-merge / defer). User-gated; pending click. Once user picks `yes-lift-and-apply-all`, Phase 1+2 of the cleanup plan close in one atomic settings.json edit. SET-001..004 remain visible but the batch route is the recommended path for efficiency + chain-ordering coherence.

## Cross-references

- A9 — Deny-list audit (in catalog; never run)
- A3 — Hook effectiveness audit (in catalog; never run; this is the right trigger to run it)
- `settings-edit-guard.md` skill ref (if exists)
- Monkey chamber SET-001..004 — already pending decisions for 4 of the patches
- A18 + A19 + A20 + A21 + A22 + A23 + A24 — all produced patches that land via this audit's cleanup

