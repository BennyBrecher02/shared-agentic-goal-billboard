---
goal_id: G22
title: Guitar-learning practice tool (personal side project + first cross-tool sandbox)
status: active
track: side-project
northern_star: false
guiding_light: false
priority: P3 (personal side project; fills idle Claude-build-wait time)
created: 2026-07-03
updated: 2026-07-12
status_tag: wave4-built-verified
gated_on: "GATE CLEARED 2026-07-03: plan approved (fuzzy-wobbling-lemur.md) + Fable 5 live + benchmark CLEAN. Remaining user-side: phone PWA-install test + the v2 workshop (alphaTab player)."
phase: "WAVE 4 SHIPPED + live-verified (2026-07-03): 4a mic-source Transcribe (Shazam-for-tabs, aa13e72) + 4b Gear Presets (amp + Rushead Max canon, 25-song KB, Meyda analyzer, My Gear panel; 333b377). CORRECTION 2026-07-12: the amp is a Fender CHAMPION 20 (20W, 8-inch, 4 voices Tweed/Black Panel/British/Metal, onboard FX: FX SELECT + FX LEVEL + TAP), user photo-confirmed — the shipped 'Frontman 10G' canon was a misidentification; woodshed gear canon being migrated by a parallel agent. Canon ROADMAP.md is the feature ledger. Matrix->Woodshed repoint LANDED (hash routes + vq spec + 14/14 proof subset; OS stub-nag retired). NEXT: user guitar-in-hand field test picks wave 5; full 3-tier matrix pass = the v1.0 quality gate."
serves_northern_star: none
linked_plans:
  - ~/Claude/Code/woodshed/ROADMAP.md (THE canon feature ledger/checklist — keep checkboxes current every wave)
linked_audits: []
linked_bugs: []
linked_changes: []
project: AgenticOS
---
# G22 — Guitar-learning practice tool (side project + cross-tool sandbox)

## Why it exists
User (2026-07-03): learning electric guitar ~a few months; wants a tool to practice/learn during idle
Claude-build-wait time instead of doomscrolling. Wanted: learn songs/solos/chords/scales, a chromatic tuner,
fretboard learning, and the KEY feature — turn AUDIO into TABS (doesn't read chords yet). Open on form factor.
Deliberately chosen as the FIRST sandbox to exercise the cross-tool ecosystem (deploy-kit init, Cursor
AGENTS.md + rules, Claude-runs-the-matrix delegation) on a safe personal repo — explicitly NOT a client project
(CAHE/BLS are off-limits for experimentation).

## Salvage from the old proposal (GuitarOS_Proposal.docx, early 2026)
KEEP (feature vision is good): theory engine (scales/chords/intervals/diatonic/CAGED/circle-of-fifths/
Nashville/capo), chromatic tuner, metronome, interactive fretboard explorer (multi-tuning, scale/chord
highlight, chord diagrams), practice tracker + streaks, lick library, gear/string-tension calc, ear trainer,
chord quiz, scale speed trainer, backing-track generator, "what chord is this?".
RECONSIDER: form factor — old doc = Python + customtkinter DESKTOP; a web/PWA or Flutter is likely better for
phone+desktop personal use (research pending).
DROP: the "7-agent, learn-multi-agent-dev" framing — that was for a Claude beginner; we now HAVE a mature
multi-agent system, so architecture should serve the app, not perform a lesson.
HARD PART: audio->tabs is research-grade — MVP = monophonic single-note/solo first, chords later.

## Status
2 research agents running (2026-07-03). Plan to be workshopped with the user, then built in Fable 5.
