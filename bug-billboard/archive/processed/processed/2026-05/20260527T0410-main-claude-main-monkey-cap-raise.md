# Monkey-chamber "Other" text input cap raised 500→5000 chars (FIXED + dominoed)

- **id:** 20260527T0410-main-claude-main-monkey-cap-raise
- **discovered:** 2026-05-27T04:10:35Z
- **agent:** claude-main
- **worktree:** main
- **route:** dashboard (/audit-timelapse/2026-05-25-overnight/)
- **section:** monkey panel — "Other" free-text input
- **viewport-bucket:** all
- **severity:** medium
- **status:** done
- **defect-type:** content-issue
- **dedupe-key:** dashboard|monkey-panel|text-cap-too-small|all

## Description

500-char cap on the monkey-chamber "Other" textarea was truncating real design directions. User explicit: *"the 500 cap on text input is too short, up it accordingly and make sure that change domino's adjustments anywhere that would be effected by this increasement."*

Raised to 5000 chars (10×) — generous for design directions, still well under backend payload cap.

## 4 edits in `public/audit-timelapse/2026-05-25-overnight/index.html`

- Line ~6544: `${len}/500` → `${len}/5000` (count display)
- Line ~6587-88: comment + `slice(0, 500)` → `slice(0, 5000)` (defensive trim)
- Line ~6753: placeholder text `max 500 chars` → `max 5000 chars`
- Line ~6754: `maxlength="500"` → `maxlength="5000"`
- Line ~6758: initial count display `0/500` → `0/5000`

## Domino check (no other adjustments needed)

Verified:
- `scripts/agent-inbox-server.py` `MAX_BODY = 16 KB = 16384 bytes`. 5000 chars × ~1.5 bytes/char UTF-8 worst case ≈ 7.5KB — **well under** the backend cap.
- Other `500` references in `scripts/` are unrelated (HTTP status codes, file-read truncation limits at 500/2500 bytes for different code paths).
- No data.json schema validation on length.
- `data-decision-id` round-trip works with longer texts.

## Verification

1. Reload dashboard monkey tab
2. Click in any "Other" textbox
3. Count shows `0/5000` instead of `0/500`
4. Placeholder reads `max 5000 chars`
5. Type 500+ chars — input continues to accept up to 5000

## Cross-references

- Task #42 (Add 'Other' option + text input to Monkey Chamber) — origin
- Paired with dashboard-reset fix (same response, same urgent user prompt)
