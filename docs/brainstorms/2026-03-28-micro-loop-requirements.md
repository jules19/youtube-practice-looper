---
date: 2026-03-28
topic: micro-loop
---

# Micro-Loop

## Problem Frame
During a practice session, a musician hears a single note or transition that needs isolation. The current workflow requires stopping, carefully repositioning both A and B markers, drilling the fragment, then dragging markers back. This is 4+ interactions that break flow and lose context. Micro-loop collapses it to one keypress.

## Requirements
- R1. Pressing M (or tapping a "Micro" button) creates a temporary loop of ±1.5s centered on the current playhead position
- R2. The original A/B markers and speed are preserved — micro-loop uses shadow state, not the real A/B
- R3. Pressing M again (or any manual A/B action: Set A, Set B, drag marker) exits micro mode and restores the original A/B
- R4. A clear visual indicator shows when micro mode is active (e.g. badge on timeline, different loop-range color, "MICRO" label)
- R5. The micro-loop region is clamped to valid bounds (0 to duration)
- R6. Micro mode does not affect the loop counter — reps in micro mode don't increment the main count
- R7. Micro mode does not persist to localStorage — it's always session-only

## Success Criteria
- Can hear a tricky note, hit M, and immediately loop around it without losing main A/B positions
- Exiting micro mode restores exactly where you were
- It's obvious at a glance whether micro mode is active

## Scope Boundaries
- No configurable radius in v1 (fixed at ±1.5s) — can add later
- No micro-loop within a micro-loop (pressing M while in micro just exits)
- No interaction with video history — micro state is ephemeral
- No new persistence — nothing saved to localStorage

## Key Decisions
- **Centered on playhead** — ±1.5s around current position, not a trailing window. Matches the "what was that?" mental model.
- **M key toggles** — same key enters and exits. Setting A or B manually also exits as a natural escape hatch.
- **Shadow state** — micro-loop does not touch the real A/B variables. This keeps the implementation isolated from the existing save/restore flow.
- **Don't count micro reps** — micro-loop is for isolation/diagnosis, not structured practice. Counting reps would be misleading.

## Outstanding Questions

### Deferred to Planning
- [Affects R4][Design] Exact visual treatment for micro mode indicator — color change on loop-range, badge, or both
- [Affects R1][Technical] Whether the tick loop should use separate micro variables or temporarily override A/B with a restore mechanism

## Next Steps
-> `/ce:plan` for structured implementation planning
