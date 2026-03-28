---
title: "feat: Add micro-loop for quick phrase isolation"
type: feat
status: completed
date: 2026-03-28
origin: docs/brainstorms/2026-03-28-micro-loop-requirements.md
---

# feat: Add micro-loop for quick phrase isolation

## Overview

Add a temporary ±1.5s loop centered on the current playhead, toggled by pressing M or tapping a button. The original A/B markers are preserved and restored when micro mode exits. (see origin: docs/brainstorms/2026-03-28-micro-loop-requirements.md)

## Proposed Solution

Use shadow state variables (`microA`, `microB`, `isMicro`) that the tick loop checks before the real A/B. When micro mode is active, the tick loop uses the micro boundaries for seeking and skips loopCount increments. `updateUI()` positions the loop-range element using micro bounds. Exiting micro mode simply clears the flag and calls `updateUI()` to restore the real A/B visual.

This approach keeps micro-loop completely isolated from the persistence layer — `saveState()` always writes real A/B, and micro state is never saved to localStorage. (see origin: key decision "Shadow state")

## Acceptance Criteria

- [ ] Pressing M creates a ±1.5s loop centered on the current playhead (R1)
- [ ] Original A/B markers and speed are preserved during micro mode (R2)
- [ ] Pressing M again exits micro mode and restores original A/B display (R3)
- [ ] Setting A, Setting B, or dragging a marker exits micro mode (R3)
- [ ] Visual indicator distinguishes micro mode from normal looping (R4) — warm color on loop-range
- [ ] Micro-loop region is clamped to 0..duration (R5)
- [ ] Loop counter does not increment during micro mode (R6)
- [ ] Micro state is not persisted to localStorage (R7)
- [ ] "Micro" button visible in controls for iPad/touch users
- [ ] Keyboard shortcut listed in footer

## Implementation Steps

All changes in `index.html`.

### Step 1: State variables

Add after existing globals (~line 550):

```js
let isMicro = false;
let microA = 0;
let microB = 0;
```

### Step 2: Helper functions

```js
const MICRO_RADIUS = 1.5;

function enterMicro() {
  if (!player || isMicro) { exitMicro(); return; }
  const t = player.getCurrentTime() || 0;
  const d = safeDuration();
  microA = clamp(t - MICRO_RADIUS, 0, d);
  microB = clamp(t + MICRO_RADIUS, 0, d);
  isMicro = true;
  updateUI();
}

function exitMicro() {
  if (!isMicro) return;
  isMicro = false;
  updateUI();
}
```

### Step 3: Modify tick loop

At line ~841, change the boundary check to use effective A/B:

```js
if (isLooping && t >= (isMicro ? microB : B)) {
  if (!isMicro) {
    loopCount++;
    loopCountLabel.textContent = loopCount;
  }
  player.seekTo(isMicro ? microA : A, true);
}
```

Also update the `isLooping` check to account for micro bounds:

```js
const effectiveA = isMicro ? microA : A;
const effectiveB = isMicro ? microB : B;
const isLooping = player.getPlayerState() === YT.PlayerState.PLAYING && effectiveB > effectiveA;
```

### Step 4: Modify updateUI()

Update marker and loop-range positioning to use effective bounds when in micro mode:

```js
function updateUI() {
  const effectiveA = isMicro ? microA : A;
  const effectiveB = isMicro ? microB : B;
  const ax = timeToPx(effectiveA);
  const bx = timeToPx(effectiveB);
  // Markers always show real A/B positions
  markerA.style.left = `${timeToPx(A)}px`;
  markerB.style.left = `${timeToPx(B)}px`;
  // Loop range shows effective (micro or real) bounds
  loopRange.style.left = `${Math.min(ax, bx)}px`;
  loopRange.style.width = `${Math.max(6, Math.abs(bx - ax))}px`;
  loopRange.classList.toggle('micro', isMicro);
  aLabel.textContent = fmtTime(A);
  bLabel.textContent = fmtTime(B);
  lenLabel.textContent = fmtTime(Math.max(0, B - A));
  durationLabel.textContent = `Duration: ${fmtTime(duration)}`;
}
```

Key: the A/B markers stay at their real positions (so you can see where you'll return to), but the loop-range highlight moves to the micro region.

### Step 5: CSS for micro mode

Add a `.loop-range.micro` class with a warm color to contrast the blue normal loop:

```css
.loop-range.micro {
  background: linear-gradient(90deg, rgba(255,180,100,0.4), rgba(255,140,60,0.7));
}

.loop-range.micro.active {
  box-shadow: 0 0 12px rgba(255,160,80,0.5), 0 0 4px rgba(255,140,60,0.3);
  animation: microPulse 1s ease-in-out infinite;
}

@keyframes microPulse {
  0%, 100% { box-shadow: 0 0 12px rgba(255,160,80,0.5), 0 0 4px rgba(255,140,60,0.3); }
  50% { box-shadow: 0 0 20px rgba(255,160,80,0.7), 0 0 8px rgba(255,140,60,0.4); }
}
```

### Step 6: Exit hooks

Add `exitMicro()` call at the top of:
- `setAHere()` — after the player guard (line ~644)
- `setBHere()` — after the player guard (line ~651)
- `installDrag` `pointerdown` handler (line ~768)

### Step 7: Keyboard shortcut and button

Add `case 'm': enterMicro(); break;` to the keydown switch (line ~890).

Add a "Micro" button to the control row in HTML:

```html
<button id="microBtn">Micro</button>
```

Add in the compact control row alongside nudge buttons.

Add event listener: `document.getElementById('microBtn').addEventListener('click', enterMicro);`

Update the footer shortcuts text to include M for micro.

### Step 8: Version bump

Bump footer version to v9.

## Scope Boundaries

- No configurable radius in v1 (fixed at ±1.5s)
- No micro-within-micro (M while micro just exits)
- No interaction with video history — micro state is ephemeral
- No persistence to localStorage

(see origin: docs/brainstorms/2026-03-28-micro-loop-requirements.md)

## Sources

- **Origin document:** [docs/brainstorms/2026-03-28-micro-loop-requirements.md](docs/brainstorms/2026-03-28-micro-loop-requirements.md) — key decisions: centered on playhead, M key toggles, shadow state, don't count micro reps
- **TDZ learnings:** [docs/solutions/runtime-errors/temporal-dead-zone-localstorage-crash.md](docs/solutions/runtime-errors/temporal-dead-zone-localstorage-crash.md) — tick loop should not clobber user/mode state; don't persist transient modes
- **Tick loop:** `index.html:832-849` — boundary check and loopCount increment
- **A/B read sites:** A read at 11 sites, B at 12 sites (see research)
- **Loop-range CSS:** `index.html:162-180` — current blue gradient + active glow
- **Keyboard handler:** `index.html:888-898` — existing shortcut switch
- **setAHere/setBHere:** `index.html:643-656` — exit micro hook points
- **installDrag:** `index.html:767-793` — pointerdown is the exit hook point
- **updateUI:** `index.html:609-619` — loop-range positioning to modify
