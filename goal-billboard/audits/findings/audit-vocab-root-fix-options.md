---
title: Audit-status vocab — the root cause + how to stop the drift (your call)
status: RESOLVED 2026-06-01 — Option C ratified + implemented (audit-close.sh now supports both tiers; see DECISION below)
created: 2026-05-31
relates_to: findings/status-reconciliation-2026-05-31.md
---

# Why the audit statuses keep drifting — and the 3 ways to stop it

## ✅ DECISION (2026-06-01) — Option C (two-tier), ratified + implemented
**Chosen: C.** `completed` = done/delivered; `verified_complete` = done AND passed the A62 four-tether adaptive-immunity gate (a higher, *earned* bar). Both are valid terminal states.

**Why C, not the "recommended" A:** naive A collapses the two into one word — silently discarding the earned tier that 5 audits genuinely cleared (A71/A75/A76/A77/A78). `verified_complete` is the audit catalog's reflection of the system's core adaptive-immunity discipline; erasing it deletes the only signal of which audits actually closed that loop.

**Correction to the root-cause framing below:** `audit-close.sh` does NOT naively stamp `verified_complete` "every time" — it *refuses to close without a `verify_test`* (the A62 gate), so the earned tier was already correctly gated. The real defect under C was the opposite: **no tooling path to the *lower* tier**, so audits that shipped but aren't four-tether-verified could never close (→ ~56 stuck `in_progress`).

**What changed (implemented 2026-06-01, tested):** `audit-close.sh` gained `--completed` → closes at the plain done-tier (`status: completed` + `closed:` timestamp, no `verify_test` required). The default/`--verified` path is unchanged (still gated on a `verify_test`). `completed` is now reachable AND `verified_complete` stays earned. README + _README already encoded two-tier; this closes the script (item 3) the smart way — two-tier, not collapse.

## The root cause (plain version)
Your audits are supposed to use ONE word for "done." But three places disagree on what that word is:
1. **`audits/README.md`** says the done-word is **`completed`**.
2. **`audits/_README.md`** (the template new audits copy from) had a "verified-complete gate" that produced
   **`verified_complete`**. *(The cleanup just fixed this template line.)*
3. **`scripts/audit-close.sh`** (the script that closes an audit) still stamps **`verified_complete`** every
   time it runs.

So every time you close an audit with that script, it writes `verified_complete` — which doesn't match the
README's `completed`. That's why the cleanup we just did will **slowly undo itself**: the next few closed
audits drift right back to the non-canonical word.

**The only piece still leaking is #3 — the script.** Fixing it means editing `scripts/audit-close.sh`, which
is in your gated zone (I don't touch scripts without your OK). That's the "not safe for me to just do" part.

## Your 3 options

### Option A — make `completed` canonical everywhere  *(recommended)*
- Edit `audit-close.sh` so it stamps `status: completed` (not `verified_complete`).
- Align the `_README.md` "gate" wording to match (the template menu line is already fixed).
- **Why:** matches your README + the cleanup just applied, so nothing re-drifts. Smallest change.
- **Cost:** one ~1-line edit to a gated script. I'd prepare the exact diff for you to apply (or to OK).

### Option B — make `verified_complete` canonical instead
- Update `README.md` to declare the done-word `verified_complete`, and re-run the cleanup to use it.
- **Why you might:** if you value that a "verified" close is stricter than a plain "done."
- **Cost:** contradicts the cleanup just applied (would re-label the 21 archived ones); more churn.

### Option C — keep BOTH as a real two-tier system
- Define `completed` = done, and `verified_complete` = done **and** independently verified (a higher bar).
- Teach the README + the dashboard's audit view to show them as two distinct, meaningful states.
- **Why you might:** you actually want to see at a glance which closures were verified vs just declared done.
- **Cost:** most work; the dashboard needs to understand two done-states.

## My recommendation
**Option A.** Smallest change, matches what your README already says, and it locks in the cleanup so this
never recurs. The moment you say go, I'll prepare the exact `audit-close.sh` diff (one line) for your review
— the same "I hand you the change, you apply it" pattern we used for `settings.json`.
