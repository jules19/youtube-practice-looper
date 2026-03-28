---
date: 2026-03-28
topic: video-history
---

# Video History

## Problem Frame
When practicing with multiple YouTube videos, there's no way to return to a previous video without re-pasting its URL. The app only remembers the last-loaded video. Musicians who rotate between several practice videos lose their loop points and speed settings each time they switch.

## Requirements
- R1. Automatically add a history entry each time a new video is loaded (no manual "save" step)
- R2. Each history entry stores: video ID, video title, URL, A/B markers, and playback speed
- R3. Display history as a collapsible list below the URL input area
- R4. Tapping a history entry loads that video and restores its A/B markers and speed
- R5. Keep the most recent 10 unique videos; oldest entries drop off when the limit is exceeded
- R6. If a video already in history is loaded again, move it to the top and update its stored A/B/speed
- R7. Allow deleting individual history entries (small X button or swipe)
- R8. Persist history in localStorage

## Success Criteria
- Can switch between two practice videos and return to the first with markers and speed intact
- History survives a page reload
- List stays clean and scannable (max 10 entries, most recent first)

## Scope Boundaries
- No named sections/bookmarks within a single video (that's the separate "Section memory" roadmap item)
- No import/export of history (covered by the existing roadmap item #9)
- No cloud sync — localStorage only

## Key Decisions
- **Save full practice state per entry** — so returning to a video is instant resume, not just a URL shortcut
- **Collapsible list below URL input** — visible but not cluttering the practice area
- **Cap at 10 entries** — keeps the list short and scannable
- **Auto-save on load, no manual action** — history should be effortless

## Outstanding Questions

### Deferred to Planning
- [Affects R2][Technical] How to retrieve the video title from the YouTube IFrame API (or whether to fall back to showing the video ID)
- [Affects R3][Design] Exact collapsed/expanded UI treatment — icon, toggle text, animation

## Next Steps
-> `/ce:plan` for structured implementation planning
