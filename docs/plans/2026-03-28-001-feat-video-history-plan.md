---
title: "feat: Add video history list"
type: feat
status: completed
date: 2026-03-28
origin: docs/brainstorms/2026-03-28-video-history-requirements.md
---

# feat: Add video history list

## Overview

Add a collapsible history list below the URL input that remembers the last 10 videos with their A/B markers and speed, so musicians can instantly resume practicing any recent video.

## Problem Statement

The app only remembers the last-loaded video. Musicians who rotate between practice videos lose their loop points and speed each time they switch. (see origin: docs/brainstorms/2026-03-28-video-history-requirements.md)

## Proposed Solution

Replace the single `yt-looper-state` localStorage key with a history array (`yt-looper-history`) that stores up to 10 entries. The most recent entry doubles as "current state" — on page load, restore from `history[0]`. This eliminates the two-sources-of-truth problem.

### Data Shape

```js
// localStorage key: 'yt-looper-history'
[
  {
    videoId: 'abc123xyz00',
    title: 'Giant Steps - John Coltrane',  // from player.getVideoData().title
    url: 'https://youtube.com/watch?v=abc123xyz00',
    A: 12.5,
    B: 28.3,
    speed: '0.75'
  },
  // ... up to 10 entries, most recent first
]
```

### Key Architectural Decisions

1. **Replace `yt-looper-state` with `yt-looper-history`** — single source of truth. On first load after migration, seed history from the old key, then delete it. (Resolves SpecFlow gap #2)

2. **Update current entry on every `saveState()` call** — piggyback on existing save triggers (A/B set, speed change, drag end). History entries always reflect the latest markers/speed, not just load-time defaults. (Resolves SpecFlow gap #1 and #3)

3. **Save before switching** — when `loadVideoFromInput()` fires, save current A/B/speed to `history[0]` before resetting for the new video. (Resolves SpecFlow gap #3)

4. **Dedup by `videoId`** — same video loaded via different URL formats is matched by extracted ID, not raw URL. (Resolves SpecFlow gap #5)

5. **Title fallback** — show video ID if `player.getVideoData().title` is not yet available. Backfill title on next `onStateChange` callback. (Resolves SpecFlow gap #4)

## Acceptance Criteria

- [ ] Loading a new video automatically adds it to history (R1)
- [ ] Each history entry stores videoId, title, URL, A, B, speed (R2)
- [ ] Collapsible list appears below URL input, collapsed by default (R3)
- [ ] Tapping an entry loads that video and restores A/B markers and speed (R4)
- [ ] History capped at 10 entries; oldest drops off (R5)
- [ ] Re-loading existing video moves it to top with updated A/B/speed (R6)
- [ ] X button deletes individual entries (R7)
- [ ] History persists across page reloads via localStorage (R8)
- [ ] Switching from Video X to Video Y saves X's current A/B/speed before switching
- [ ] Page load restores from `history[0]` (replaces old `yt-looper-state` behavior)
- [ ] Migration: existing `yt-looper-state` data seeds history on first load
- [ ] History section hidden when empty
- [ ] Current video highlighted in list
- [ ] Touch targets work on iPad (min 52px per `--tap` variable)

## Implementation Steps

All changes in `index.html`.

### Step 1: Data layer — new localStorage functions

Update `saveState()` / `loadState()` to work with a history array instead of a single object:

```js
const HISTORY_KEY = 'yt-looper-history';
const MAX_HISTORY = 10;

function loadHistory() {
  try {
    const h = JSON.parse(localStorage.getItem(HISTORY_KEY));
    if (Array.isArray(h)) return h;
  } catch (e) {}
  // Migration from old single-state key
  try {
    const old = JSON.parse(localStorage.getItem('yt-looper-state'));
    if (old && old.videoId) {
      localStorage.removeItem('yt-looper-state');
      return [old];
    }
  } catch (e) {}
  return [];
}

function saveHistory(history) {
  try {
    localStorage.setItem(HISTORY_KEY, JSON.stringify(history.slice(0, MAX_HISTORY)));
  } catch (e) {}
}
```

Update `saveState()` to write current A/B/speed into `videoHistory[0]` and call `saveHistory()`.

### Step 2: Hook into video load flow

In `loadVideoFromInput()`:
1. Before resetting A/B, call `saveState()` to persist current video's markers
2. After `extractVideoId()`, upsert the new video into history (move to front if exists, or prepend)
3. Title will be empty initially — backfill in `onStateChange` when `player.getVideoData().title` becomes available

### Step 3: HTML — collapsible history section

Add between `.url-row` and `.player-wrap`:

```html
<div id="historySection" class="history-section" style="display:none">
  <button id="historyToggle" class="history-toggle">
    ▶ Recent videos <span id="historyCount"></span>
  </button>
  <div id="historyList" class="history-list" style="display:none">
    <!-- dynamically rendered -->
  </div>
</div>
```

### Step 4: CSS — history styling

Style to match existing dark theme using CSS custom properties (`--panel-2`, `--border`, `--muted`, `--accent`, `--radius`, `--tap`). Key concerns:
- Entry rows: full-width, min-height `var(--tap)` for touch
- X delete button: `stopPropagation()` to avoid triggering load
- Current video: subtle accent border or highlight
- Collapsed/expanded: toggle `▶`/`▼` arrow on the button
- Responsive: entries should stack naturally at narrow widths

### Step 5: JS — render and interaction

**Note:** Do not use `history` as a variable name — it shadows `window.history`. Use `videoHistory` throughout.

```js
function renderHistory() {
  if (videoHistory.length === 0) { historySection.style.display = 'none'; return; }
  historySection.style.display = '';
  historyCount.textContent = `(${videoHistory.length})`;
  historyList.innerHTML = '';
  videoHistory.forEach((h, i) => {
    const entry = document.createElement('div');
    entry.className = 'history-entry' + (h.videoId === videoId ? ' active' : '');
    entry.dataset.index = i;
    const info = document.createElement('div');
    info.className = 'history-info';
    const title = document.createElement('div');
    title.className = 'history-title';
    title.textContent = h.title || h.videoId;  // safe — no innerHTML
    const meta = document.createElement('div');
    meta.className = 'history-meta';
    meta.textContent = `${fmtTime(h.A)} – ${fmtTime(h.B)} · ${h.speed}x`;
    info.append(title, meta);
    const del = document.createElement('button');
    del.className = 'history-delete';
    del.dataset.index = i;
    del.textContent = '✕';
    entry.append(info, del);
    historyList.appendChild(entry);
  });
}
```

Uses `textContent` instead of `innerHTML` to avoid XSS from YouTube video titles. DOM refs (`historySection`, `historyList`, `historyCount`) are cached at init time per Step 7.

Event delegation on `historyList`:
- Click on `.history-entry` → load video, restore A/B/speed, re-render
- Click on `.history-delete` → `stopPropagation()`, splice from array, save, re-render. If deleting the current video, keep it playing but remove the highlight
- Click on `historyToggle` → toggle list visibility, swap arrow

### Step 6: Restore on page load

Replace current `loadState()` + `validSave` block:
1. `const videoHistory = loadHistory();`
2. If `videoHistory[0]` exists and is valid, restore videoId/A/B/speed/url from it
3. Otherwise use existing defaults

### Step 7: Init order (TDZ safety)

Per the documented TDZ solution (`docs/solutions/runtime-errors/temporal-dead-zone-localstorage-crash.md`):
1. DOM const declarations first
2. `loadHistory()` and state restoration
3. Event listeners
4. YouTube API init

New history DOM elements (`historySection`, `historyToggle`, `historyList`) must be acquired in step 1 alongside existing DOM refs.

## Scope Boundaries

- No named sections/bookmarks within a video (separate roadmap item: Section memory)
- No import/export (separate roadmap item #9)
- No cloud sync — localStorage only
- No keyboard shortcuts for history navigation (can add later)

(see origin: docs/brainstorms/2026-03-28-video-history-requirements.md)

## Sources

- **Origin document:** [docs/brainstorms/2026-03-28-video-history-requirements.md](docs/brainstorms/2026-03-28-video-history-requirements.md) — key decisions: save full practice state per entry, collapsible list below URL input, cap at 10, auto-save on load
- **TDZ solution:** [docs/solutions/runtime-errors/temporal-dead-zone-localstorage-crash.md](docs/solutions/runtime-errors/temporal-dead-zone-localstorage-crash.md) — init order constraints
- **Current localStorage usage:** `index.html:395-419` — `saveState()`/`loadState()` to replace
- **Video load entry point:** `index.html:555-572` — `loadVideoFromInput()` to modify
- **DOM insertion point:** `index.html:330-338` — between `.url-row` and `.player-wrap`
- **Video title API:** `player.getVideoData().title` — available after `onStateChange` callback
