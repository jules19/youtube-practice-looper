---
date: 2026-03-28
topic: feature-priorities
focus: What features to build next after video history
---

# Ideation: Feature Priorities Post-History

## Codebase Context
- Single index.html (~916 lines), vanilla JS, no framework, localStorage persistence, Vercel deploy
- Just shipped: video history (10 recent videos with A/B markers and speed)
- Known constraint: TDZ/localStorage ordering bugs when adding new persisted state
- YouTube IFrame API available (getVideoData, getDuration, setPlaybackRate, etc.)

## Ranked Ideas

### 1. Shareable URL (hash-encoded state)
**Description:** Encode videoId, A, B, speed into URL hash. Anyone opening the link lands directly in that loop. One "Copy link" button.
**Rationale:** Lowest effort (~20 lines). Highest leverage — makes the app useful beyond a single device. Force multiplier for every future feature.
**Downsides:** Hash changes on every state save could pollute browser history if not handled carefully (use replaceState).
**Confidence:** 95%
**Complexity:** Low
**Status:** Unexplored

### 2. Smoother Loop Transition
**Description:** Change `t >= B` to `t >= B - 0.15` in the tick loop to seek slightly before B, eliminating the audible glitch.
**Rationale:** 3-token change with highest polish-per-line ratio. Makes the tool feel professional.
**Downsides:** Fixed offset may clip the last note on very tight loops.
**Confidence:** 90%
**Complexity:** Low
**Status:** Unexplored

### 3. Speed Ramp Trainer
**Description:** Auto-increment playback speed by a configurable step every N loops. Status pill shows progress.
**Rationale:** Canonical deliberate practice technique. Plumbing is all there (loopCount + setPlaybackRate).
**Downsides:** Needs a small config UI. More code than #1 or #2.
**Confidence:** 85%
**Complexity:** Medium
**Status:** Unexplored

### 4. Section Memory (Named Loop Slots)
**Description:** Save multiple named A/B+speed snapshots per video. Tap a chip to instantly load that loop.
**Rationale:** Natural extension of history. Turns the app into a practice notebook over time.
**Downsides:** UI design for slot management needs thought. Most complex of the survivors.
**Confidence:** 85%
**Complexity:** Medium
**Status:** Unexplored

### 5. Micro-Loop
**Description:** One button/key creates a temporary tight loop around current position without overwriting saved A/B. Second press restores.
**Rationale:** Solves the "wait, what was that note?" moment without breaking the session.
**Downsides:** Needs clear visual indicator for micro mode active.
**Confidence:** 80%
**Complexity:** Medium-Low
**Status:** Explored

### 6. Screen Wake Lock for iPad
**Description:** Use Wake Lock API to prevent screen dimming during practice.
**Rationale:** Solves physical pain of reaching for screen while holding instrument. Two lines of JS + toggle.
**Downsides:** Wake Lock API may not keep YouTube audio alive in background. Battery drain.
**Confidence:** 70%
**Complexity:** Low
**Status:** Unexplored

## Rejection Summary

| # | Idea | Reason Rejected |
|---|------|-----------------|
| 1 | Count-In (pre-roll) | Too small standalone — ship alongside speed ramp as "practice mode" |
| 2 | Speed Fine-Tuning (+/-0.05x) | Subsumed by speed ramp trainer |
| 3 | Practice Session Log | Depends on section memory first |
| 4 | Loop Difficulty Heatmap | Two layers away from buildable |
| 5 | Tempo Ladder (progression log) | Adds complexity for marginal gain over basic ramp |
| 6 | Import/Export JSON | Premature — not enough persisted data yet |
| 7 | Clipboard sniff on focus | Permissions prompt, privacy concern |
| 8 | Search-inside-the-app | Requires API key, breaks single-file constraint |
| 9 | Smart B default on load | Minor convenience |
| 10 | Tap-to-Set-A/B context menu | Niche — existing workflow sufficient |
| 11 | Loop-count goal with auto-pause | Subsumed by speed ramp trainer |
| 12 | Keyboard Help Overlay | Too minor |
| 13 | Tap-to-bracket capture | Current A/B workflow acceptable |
| 14 | Loop-first library restructure | Section memory achieves same goal without rewrite |
| 15 | Audio-only / background mode | YouTube iframe stops audio in background |
| 16 | Difficulty-anchored replay | Complex, needs foundation features first |

## Session Log
- 2026-03-28: Initial ideation — ~40 raw ideas from 5 agents, 6 survivors. User selected Micro-Loop for brainstorming.
