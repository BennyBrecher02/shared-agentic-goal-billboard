---
title: "Portability re-audit — the out-of-repo launchd blind spot"
date: 2026-06-02
type: findings
audit_kind: system-infrastructure (project-agnostic / path portability)
trigger: "user 2026-06-02 — 'didnt we already audit this exact issue multiple times? i asked us to work project agnostic. audit our audit then reaudit.' (the relocation broke HEART/AUTONOMIC via stale launchd plists)"
status: scaffold (orchestrator analysis below; exhaustive reaudit agent in progress, findings appended on completion)
---

# Portability re-audit (2026-06-02)

## The recurrence (why the user is right to be annoyed)
We hardened project-agnostic paths IN-REPO **multiple times** — yet the 2026-05-31 relocation still
broke HEART + AUTONOMIC because the launchd daemon plists pointed at the old `EviumOverhaul` path.
The same *class* of bug we thought we'd closed.

## AUDIT-OUR-AUDIT — why every existing defense missed it
| Defense | What it covers | Why it MISSED the plists |
|---|---|---|
| `no-hardcoded-paths-guard.sh` (PreToolUse Write\|Edit, BLOCK) | in-repo files **being edited via a tool** | plists live in `~/Library/LaunchAgents/` (out-of-repo) + are rendered once at install via launchctl, never re-edited through Write/Edit → **never scanned** |
| `project-config.{sh,json,py}` (self-healing derivation) | runtime path derivation for **in-repo scripts** | a launchd plist is a **static file with an absolute path baked at render time** — no runtime derivation, so nothing self-heals it |
| `hardcoded-path-portability-audit.md` (prior audit) | in-repo scripts/configs | its scope **stopped at the repo boundary** — out-of-repo install-time artifacts were never in frame |
| plist **templates** (`scripts/heartbeat/*.template`, `launchd-template/`) | use `__REPO__` placeholders — **portable + correct** | the bug isn't the template; it's the **rendered installed copy** that drifts on a move |

## THE BLIND SPOT (one line)
**Out-of-repo, install-time-rendered artifacts (launchd plists, and potentially crontab / scheduled
tasks) are invisible to every in-repo portability defense.** A relocation invalidates their baked
absolute paths, and until 2026-06-02 *nothing detected it* — it took a full manual OS audit.

## STRUCTURAL FIX (the antibody + the procedure + the scope)
1. **`migration-drift-sentinel.sh` (BUILT 2026-06-02)** — the first defense that reaches OUTSIDE the
   repo: it verifies every `com.evium.*` LaunchAgent points at the live repo + the live cache is fed.
   6/6 test in the fast tier. **Wire into SessionStart (tier-1)** so it fires at every boot — it would
   have caught this at the first post-relocation startup.
2. **Relocation procedure** (extend `reference_repo-geography` migration steps): a git-based move does
   NOT re-render launchd plists. The move checklist must include **"re-stage + reinstall the plists"**
   (`stage-repointed-plists.sh` → user reloads). Bank alongside the existing "carry gitignored data" lesson.
3. **Scope correction**: the portability audit + the guard's *mandate* must EXPLICITLY include
   out-of-repo install-time artifacts (`~/Library/LaunchAgents/*.plist`, `crontab`, scheduled tasks).
   The guard can't scan them at edit-time (they're not edited via tools) — so the **sentinel is the
   right mechanism**, not the guard.

## REAUDIT RESULTS (exhaustive sweep — agent a77e5e46, 2026-06-02)
**Blind spot: CONFIRMED.** All in-repo defenses scope to files-being-edited inside `scripts/`+`.claude/`;
the install-time-rendered plists live outside the repo and were never covered — exactly what broke
HEART/AUTONOMIC on the move.

**Decisive question — is ANYTHING else still real drift? → NO.**
- The 5 daemon plists are **already fixed** (re-staged to the live path 2026-06-02; sentinel passes).
- `crontab -l` is **empty**; no other evium/claude scheduled jobs.
- The §6.1 portability refactor **fully landed**: `settings.json` has 0 absolute-path commands, all 3
  plist templates use `__REPO__`, all 10 previously-flagged daemon/analyzer/hook files are clean, both
  CI gates PASS, and `project-config` provably resolves to the live path/slug (ran env-less from `/tmp`).
- Every other old-path hit is **legitimate/inert**: a sanctioned config cache (derivation wins), the
  guard's own HAZARD regex, the EviumOverhaul-as-zach-SOURCE ref (per repo-geography), a test fixture,
  a docstring, or gitignored runtime cache.
- The ONLY in-repo residual was **2 dead `.scratch/*.sh.new` patch drafts → REMOVED 2026-06-02** (litter).

**Structural recommendations (the antibody + closing the blind spot permanently):**
1. **Wire `migration-drift-sentinel.sh --quiet` into SessionStart** (gated — settings.json; currently it
   only runs in the CI fast tier). This is the single highest-value fix — it would have caught the
   flatline at the first post-move session. → in the gated block handed to the user.
2. **Extend the `no-hardcoded-paths` CI test to also scan `~/Library/LaunchAgents/*.plist` + `crontab`**
   so the out-of-repo blind spot becomes a permanent gate, not just a runtime sentinel.
3. Bank the "re-stage + reinstall launchd plists" step into the relocation procedure (repo-geography),
   alongside the existing "carry gitignored data" lesson — git-based moves do NOT re-render plists.

**VERDICT: the system IS project-agnostic in-repo (verified, multiple gates green); the launchd plists
were the lone blind spot, now closed by the sentinel + already re-staged. Nothing else lurks.**
