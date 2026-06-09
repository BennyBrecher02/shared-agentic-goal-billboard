# A80 — Malformed `data.json` Culprit (Board #11)

**Status:** READ-ONLY diagnosis complete. Fix proposed, NOT applied.
**Priority:** LOW — the new regenerable dashboard (`build_render_model.py`) already tolerates this corruption (`load_json_lenient` recovers the first valid doc + emits a repair warning). Clean closure, not urgent.
**File:** `reports/timelapse/2026-05-25-overnight/data.json`
**Verdict in one line:** Not an append-mode (`>>`) write. It's a **concurrent read-modify-write race across ~7 mergers that all share one scratch file (`data.json.tmp`) with zero locking.**

---

## The culprit

There is **no single guilty `script:line`** in the sense the board #11 hypothesis assumed (a stray `>>` / `open('a')`). I verified: **no writer anywhere uses append mode or `open('a')`** on `data.json`. Every writer uses the canonical read → mutate-one-key → tmp-write → atomic-rename pattern.

The bug is **structural / emergent**: 7 independent merger scripts each do read-modify-write on the same `data.json`, **all writing through the identical temp filename `data.json.tmp`**, launched **concurrently as backgrounded `nohup … &` jobs** by two hooks, with **no mutual exclusion of any kind**.

### The shared collision point — every merger resolves to the SAME tmp path

| Merger | Block | tmp line | resolves to |
|---|---|---|---|
| `scripts/render-bg-pattern-data.py` | `bg_launch_pattern` | **:157** `tmp = data_file.with_suffix(".json.tmp")` | `data.json.tmp` |
| `scripts/render-asks-data.py` | `asks` | :381 same | `data.json.tmp` |
| `scripts/render-stats-data.py` | `stats` | :536 same | `data.json.tmp` |
| `scripts/render-skill-metrics-data.py` | `skill_metrics` | :224 same | `data.json.tmp` |
| `scripts/render-cluster-data.py` | `cluster` | :200 `DATA_JSON.with_suffix(DATA_JSON.suffix + ".tmp")` | `data.json.tmp` |
| `scripts/render-meta-monitoring-data.py` | `meta_monitoring` | :272 same as bg | `data.json.tmp` |
| `scripts/render-meta-loop-eval-data.py` | `meta_loop_eval` | :161 same as bg | `data.json.tmp` |

Both `with_suffix(".json.tmp")` and `with_suffix(suffix + ".tmp")` collapse to the exact same name `data.json.tmp` (verified empirically).

### Launched concurrently, no locking
- `scripts/hooks/dashboard-data-prime.sh` (SessionStart) fires 4 of these mergers as parallel `nohup … &` jobs (+ the bg-pattern pipeline).
- `scripts/hooks/billboard-data-refresh.sh` (PostToolUse Write|Edit) fires 6 of them as parallel `nohup … &` jobs.
- Grep for `flock | mkdir-lock | lockfile | fcntl | LOCK_EX` across all mergers + both hooks: **zero matches.** Nothing serializes them.

So when a goal-billboard / plans edit lands (or at SessionStart), 4–7 of these processes run simultaneously, each: `payload = json.loads(read data.json)` → set its one key → `write data.json.tmp` → `replace → data.json`. They stomp each other's `data.json.tmp` and `data.json`.

---

## The mechanism — why a *trailing fragment* and not just a lost-update

The corruption is **trailing extra-data**: a complete, valid JSON document, followed by ~344 orphan bytes that are the **tail of a `bg_launch_pattern` block** (last `recent_50` entry + `],"verdict":"regression flat"}}`).

Valid JSON ends at byte **563985**; file is **564329** bytes; the trailing 344 bytes are:
```
   },
      {
        "ts": "2026-05-29T01:04:42.771Z",
        "bg_count": 5,
        "pressure_score": 7.0,
        "response_words": 207,
        "prompt_excerpt": "lets go for this again: \"im not gonna be around for one of these days so we realy have like 1 day to use 85% of a full 2"
      }
    ],
    "verdict": "regression flat"
  }
}
```
That fragment is **byte-identical to the END of the valid doc's own `bg_launch_pattern` block** (same `ts`, `bg_count`, `pressure_score`). It is a leftover tail, not a fresh appended row. (Note: the `2026-05-29T01:04:42` timestamp is *content* — a prompt timestamp inside the data — not a wall-clock; the file mtime is 2026-05-28 21:10:24.)

**This is the signature of a SHORTER document being written over a LONGER one without the underlying bytes being truncated** — i.e. two writers overlapping on the shared `data.json.tmp` inode. Reproduced exactly:

```
short doc ends at its own  }}            <- new closing braces
leftover long-doc tail:    rdict":"regression flat"}}   <- survives past the close
```
Real reproduction output (two writes to one tmp path, second shorter, no re-truncate):
`{"bg_launch_pattern":{"recent_50":[],"verdict":"flat"}}rdict":"regression flat"}}`
— same class as the live corruption: `…verdict":"regression flat"}}` dangling after a valid close.

### The interleaving (one of several that produce this)
1. Writer **A** (e.g. `render-bg-pattern`, whose payload includes the *long* full `recent_50`) opens `data.json.tmp` and writes the long doc.
2. Writer **B** (a merger whose payload is *shorter* — e.g. a block that shrank, or a stale read missing A's growth) opens the **same `data.json.tmp`** and writes its shorter doc.
3. Because both share the one scratch name with no lock, B's shorter content lands over A's bytes (and/or the two `replace()` calls race the inode), leaving A's `bg_launch_pattern` tail past B's closing brace.
4. The surviving `data.json` = valid-doc-from-one-writer **+** orphaned-tail-from-another.

The reason the orphan tail is specifically a `bg_launch_pattern` tail: that block is the **largest variable-length block** (50-entry `recent_50` array with long `prompt_excerpt` strings), so it is the most likely to be the *longest* writer in the race and thus the one whose tail survives a shorter overwrite. It is the victim's fingerprint, not the perpetrator's signature — i.e. `render-bg-pattern-data.py` is the **most-implicated single writer by probability**, but the defect is the shared-tmp + no-lock pattern common to all 7.

### Why "atomic rename" did not save it
`tmp.replace(data_file)` / `os.replace` *is* atomic for a single writer. But atomicity of the **rename** does not serialize the **write to the shared tmp file** that precedes it. Two processes writing the same `data.json.tmp` before either renames defeats the guarantee. `Path.write_text` truncates on its own `open('w')`, but it is not one syscall — the open/write/close of process B can interleave with process A's, and the two `replace()` operations race over which inode becomes `data.json`.

---

## Proposed fix (DO NOT APPLY — documented only)

The board #11 ask phrased it as "rewrite-whole-doc instead of append." That framing is slightly off (there is no append), but the **fix family is the same spirit**: make each merger's write collision-free. Three layered options, smallest-first:

**Fix 1 (one-line-per-merger, minimal — recommended first):** give each merger a **unique tmp filename** so they never share scratch state. In every merger, change:
```python
tmp = data_file.with_suffix(".json.tmp")
```
to a per-process-unique name, e.g.:
```python
tmp = data_file.with_suffix(f".json.{os.getpid()}.tmp")   # +import os where missing
```
This removes the shared-`data.json.tmp` overlap. The `os.replace` is already atomic, so unique tmp names make each merger's publish atomic and non-overlapping. (7 one-line edits — `render-bg-pattern-data.py:157`, `render-asks-data.py:381`, `render-stats-data.py:536`, `render-skill-metrics-data.py:224`, `render-cluster-data.py:200`, `render-meta-monitoring-data.py:272`, `render-meta-loop-eval-data.py:161`.)

> Fix 1 alone closes the *trailing-fragment corruption*. It does NOT close lost-updates: two mergers that both `read → mutate → replace` concurrently can still clobber each other's *block* (last-writer-wins on the whole doc), but the file stays *valid JSON*. For a LOW-priority file the new dashboard doesn't read, valid-but-occasionally-stale is acceptable.

**Fix 2 (correctness — close lost-updates too):** wrap the read-modify-write in a shared advisory lock so the 7 mergers serialize. A single `flock` on a sidecar lockfile (`reports/timelapse/<run>/.data.json.lock`) around read→mutate→replace in each merger. This is the proper opsys fix (filesystem-atomic mutual exclusion per `agentic-script-design` synchronization-layer guidance) and matches existing `mkdir`-lock / `flock` patterns elsewhere in the repo.

**Fix 3 (architectural — the real direction):** stop having 7 writers mutate one shared doc at all. This is exactly what `build_render_model.py` (F0) already does — read all sources read-only, emit ONE consolidated `render-model.json`. The timelapse `data.json` merge-fan-in is legacy; once the regenerable dashboard fully supersedes it, the 7-merger race surface disappears. **This is why the finding is LOW priority** — the architecture is already migrating away from the defective pattern.

### Immediate one-liner to clean the existing corrupt file (optional, not required)
```bash
python3 - <<'PY'
import json
p="reports/timelapse/2026-05-25-overnight/data.json"
raw=open(p,encoding="utf-8").read()
obj,_=json.JSONDecoder().raw_decode(raw.lstrip())
open(p,"w",encoding="utf-8").write(json.dumps(obj,ensure_ascii=False,indent=2))
PY
```
(Truncates the 344 orphan bytes by rewriting only the first valid doc. Not applied — read-only task.)

---

## Cleared of suspicion
- `scripts/dashboard/build_render_model.py` — **read-only**, as hypothesized. Its only write is the new `render-model.json` (atomic, unique path). It is in fact the *victim's advocate*: `load_json_lenient` (lines 75–96) already detects & recovers this exact corruption and emits the "trailing extra-data … flag for repair" warning now visible in `reports/dashboard-build/dashboard.html`'s MODEL warnings. NOT the culprit.
- `scripts/run-bg-launch-pattern-analysis.py` — appends only to the **JSONL cache** (`open(CACHE_JSONL, "a")` at :426), never touches `data.json`. NOT the culprit.
- `scripts/timelapse/generate.py` — writes `data.json` once via `write_text` at :532 during a full regen (out_dir is a per-run dir); not part of the per-edit concurrent fan-out. Not implicated in this corruption.

## Root attribution (for the board)
- **Primary culprit (class):** shared-`data.json.tmp` + no-lock concurrent read-modify-write across the 7 `render-*-data.py` mergers, fanned out in parallel by `dashboard-data-prime.sh` and `billboard-data-refresh.sh`.
- **Most-implicated single writer (by probability):** `scripts/render-bg-pattern-data.py:157` — its block is the longest variable-length payload, so its tail is the one that survives a shorter concurrent overwrite (hence a `bg_launch_pattern` fragment is what's left dangling).
- **Mechanism:** non-serialized writes to one scratch file → a shorter doc overwrites a longer doc's bytes / racing `replace()` → orphaned trailing fragment past a valid close = JSON "Extra data".
- **Proposed fix:** Fix 1 (per-PID unique tmp name, 7 one-liners) closes the corruption; Fix 2 (flock) closes lost-updates; Fix 3 (already underway via F0 render-model) retires the pattern. Not applied per task scope.

BG-COMPLETE-SENTINEL
