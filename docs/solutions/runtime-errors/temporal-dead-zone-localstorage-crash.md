---
title: YouTube Practice Looper localStorage State Not Restoring Due to Temporal Dead Zone (TDZ) and Unreliable Duration Clamping
date: 2026-03-13
category: runtime-errors
tags:
  - temporal-dead-zone
  - const-hoisting
  - localStorage
  - youtube-api
  - state-persistence
  - safari-ipad
component: Single-file HTML/JS YouTube Practice Looper App
severity: Critical
symptoms:
  - A/B markers reset to 0 on page reload
  - B marker stuck at left edge of timeline
  - A marker not visible
  - Silent ReferenceError causing fallback to defaults
  - Issue reproducible on iPad Safari home screen web app
root_cause_summary: >
  JavaScript Temporal Dead Zone crash caused by referencing videoUrlInput and
  speedSelect variables before their const declarations. When saved state existed,
  the app threw a silent ReferenceError. Secondary issue: onStateChange callback
  clamped A/B markers using unreliable getDuration() values during init.
---

## Investigation Steps

The bug was that localStorage state (video ID, A/B loop markers, playback speed) failed to restore silently on iPad Safari. Several hypotheses were tested before the root cause was found.

**Attempt 1: getDuration() returning bad values during init**

Suspected that `onStateChange` was clamping A/B markers using a bad `getDuration()` return value before the player was ready. Fix: only update duration/clamp when `getDuration()` returns a positive value. Result: did not help.

**Attempt 2: saveState() firing during player initialization**

Suspected that `saveState()` being called inside `updateUI()` was overwriting good saved state with blank/default values during player init. Fix: moved `saveState()` out of `updateUI()` so it only fires on explicit user actions. Result: still broken.

**Attempt 3: Validation guard and playerReady flag**

Added a `validSave` check to reject corrupted saved state (e.g. B <= 0 or B <= A), and a `playerReady` flag to guard clamping logic in `onStateChange`. Result: still broken.

**Attempt 4: Remove A/B clamping from onStateChange entirely**

Reasoned from first principles that init code should never modify user-set values. Removed the clamping from `onStateChange`. Simpler, but still broken.

**Attempt 5: Code review — found the real bug**

A careful read of the restore block revealed a JavaScript Temporal Dead Zone violation. The variables `videoUrlInput` and `speedSelect` were declared with `const` on lines 438-439, but were referenced inside the `if (validSave)` block on lines 424-425 — before their declarations in source order.

## Root Cause

JavaScript **Temporal Dead Zone (TDZ)**. `const` (and `let`) variables are hoisted to the top of their block scope, but they are not initialized until the engine reaches their declaration line. Any attempt to read or write the variable before that line throws a `ReferenceError`.

Because the error was thrown inside the `if (validSave)` branch — which only runs when there *is* saved state — the crash was silent and only happened when restoration was actually attempted. On a fresh load with no saved state the branch was skipped and everything appeared to work fine.

```js
// BROKEN — references appear before declarations

if (validSave) {
    // Lines ~424-425
    videoUrlInput.value = saved.videoId;   // ReferenceError: Cannot access
    speedSelect.value   = saved.speed;     //   'videoUrlInput' before init
}

// Lines ~438-439  ← declarations come AFTER the usage above
const videoUrlInput = document.getElementById('videoUrl');
const speedSelect   = document.getElementById('speedSelect');
```

## Solution

Move the `getElementById` declarations above the `if (validSave)` block so they are initialized before any code references them.

```js
// FIXED — declarations come first

const videoUrlInput = document.getElementById('videoUrl');
const speedSelect   = document.getElementById('speedSelect');

if (validSave) {
    videoUrlInput.value = saved.videoId;  // fine — already initialized
    speedSelect.value   = saved.speed;    // fine — already initialized
}
```

No logic changes were required — only reordering the declarations.

A secondary fix removed A/B clamping from the `onStateChange` callback entirely, since user-set values should never be modified by initialization code.

## Prevention Strategies

1. **Hoist DOM declarations to the top of the script** — acquire all DOM references before any code that uses them. Establish a clear init order: DOM acquisition, state restoration, event listeners, API initialization.

2. **Don't let API callbacks modify user state during init** — the `onStateChange` handler was clamping A/B markers using unreliable `getDuration()` values. Init callbacks should only update internal bookkeeping (like `duration`), never user-facing state.

3. **Save state only on explicit user actions** — calling `saveState()` from `updateUI()` meant every internal update could overwrite good state with bad values. Persistence should be triggered by user intent, not rendering.

4. **Validate saved state on load** — reject obviously invalid saved data (e.g. B <= 0, B <= A) and fall back to defaults rather than silently using corrupt values.

## Key Takeaway

TDZ bugs are hard to spot because the variable name *exists* in scope (it is hoisted), so there is no "undefined variable" lint warning — only a runtime error at the moment of access. They are especially insidious when gated behind a conditional that only evaluates in certain states (like "has saved data"), making the crash intermittent and hard to reproduce during development.
