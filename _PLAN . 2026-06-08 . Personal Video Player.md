# Personal Video Player — Project Plan
**Last updated:** 2026-06-08 (rev 2)  
**Output file:** `video-player.html` (single standalone HTML file, no build step, open directly in browser)

---

## Context for Claude

This is a self-contained HTML5 video/audio player built iteratively by Albert across one session. The entire app lives in a **single `.html` file** — no frameworks, no bundler, no server needed. Just open in Chrome, Firefox, or Edge.

The player was built for **language learning** (Chinese + English dual subtitles) with a strong preference for:
- Local-first, no cloud dependencies
- Standalone single-file output
- Dark theme using CSS variables
- IBM Plex Sans + IBM Plex Mono fonts (loaded from Google Fonts)

When continuing work, always **edit the existing `video-player.html`** rather than creating a new file. Prefer surgical edits (`str_replace`) over full rewrites unless the change is structural. After every change, use `present_files` to deliver the updated file.

---

## Tech Stack

| Layer | Choice |
|---|---|
| Structure | Vanilla HTML5 |
| Styling | CSS (custom properties, flexbox) |
| Logic | Vanilla JavaScript (ES6+) |
| HLS streaming | `hls.js` via CDN (`cdn.jsdelivr.net/npm/hls.js@latest`) |
| Fonts | IBM Plex Sans + IBM Plex Mono (Google Fonts) |
| Storage | `localStorage` (resume positions, keyed by filename + size) |

---

## Layout Structure

```
.shell (flex column, 100vh)
  header
  .body (flex row)
    .sidebar          ← resizable, collapsible
    .sidebar-toggle   ← thin chevron strip between sidebar and main
    .main (flex column)
      .video-area     ← fills all remaining vertical space (flex: 1)
        #drop-zone          (shown when no file loaded)
        .video-container    (shown for video files, .active class)
        .audio-cover        (shown for audio files, .active class)
        #resume-bar         (floats top-center over video-area)
        #click-overlay      ← IMPORTANT: lives here at video-area level, z-index:6
      .sub-drawer     ← collapsible subtitle panel (max-height transition)
      .controls-bar   ← pinned to bottom; collapsible (C key)
        .controls-toggle  ← always visible: chevron + play button + mini seek + time
        .controls-body    ← expandable: file name row + button row (speed, subs, fs, hints)
      .hints-popup    ← absolute positioned above controls-bar
```

### Key z-index layers (video-area children)
- `#click-overlay`: **6** — must be highest inside video-area to catch all clicks
- `#resume-bar`: 10 — floats above overlay so buttons remain clickable
- `#pause-flash` / `#fs-flash`: 4 / 5 — visual feedback, below overlay
- `#subtitle-overlay`: 3

---

## Implemented Features

### Playback
- Play/pause via play button, `Space` key, or clicking anywhere on the video/audio area
- Single click on video area → play/pause (with 220ms debounce to distinguish from double-click)
- Double click on video area → fullscreen toggle
- Click on audio cover area → play/pause (no fullscreen for audio)
- Flash icon feedback on click (pause icon / play icon briefly shown)
- Flash icon on fullscreen toggle
- Auto-advance to next file in sidebar when current file ends

### Speed Control
- Preset buttons: 0.5x, 0.75x, 1x, 1.25x, 1.5x, 2x, 3x
- Fine-tune slider: 0.25x to 4x in 0.25 steps
- Keyboard: `↑` / `↓` arrows adjust ±0.25x

### Resume Position
- Saves `currentTime` to `localStorage` every 2 seconds while playing
- Also saves on pause and on page unload
- Key format: `vp_<filename>_<filesize>`
- On file load: if saved position > 5s, shows floating resume bar with "Continue from X:XX?" + "Start over" option
- Clears saved position when file ends naturally

### Dual Subtitles
- Two independent subtitle slots (EN and ZH)
- Supports `.srt` and `.vtt` formats
- Both displayed simultaneously overlaid on the video
- Sub 1: white, 15px default
- Sub 2: yellow (`#ffe066`), 18px default
- Each slot: drag & drop or click to load, visibility toggle, font size slider (11–24px / 11–28px)
- Subtitle drawer is collapsible via subtitle button in controls bar
- SRT parser handles `\r\n`, `\r`, `\n` line endings and strips HTML tags

### Audio Track Selection
- Shown inside the subtitle drawer
- Supports HLS multi-track audio via `hls.js`
- Supports native `audioTracks` API (Safari)
- Falls back gracefully to "Single audio track" label when only one track

### HLS Support
- `hls.js` is loaded from CDN
- Only activated for `.m3u8` files (NOT for `.mp4` — that caused slow loading and was fixed)
- Falls back to native `vid.src` if HLS fails fatally
- Safari uses native HLS via `vid.canPlayType("application/vnd.apple.mpegurl")`

### File Library (Sidebar)
- "Open folder" button uses `webkitdirectory` input to scan entire folder
- Filters to media extensions only (see lists below)
- Sorted alphabetically
- Search/filter box (case-insensitive substring match)
- Click a file → loads and plays immediately
- Active file highlighted with accent color
- Video files: screen icon; Audio files: music note icon
- Extension badge shown on right

**Supported video extensions:** `mp4, webm, mkv, mov, avi, m4v, ogv, flv, wmv, ts, m3u8`  
**Supported audio extensions:** `mp3, m4a, aac, ogg, flac, wav, opus, wma`

### Sidebar Resize & Collapse
- Drag the resize handle (right edge of sidebar) to resize between 160px and 520px
- Toggle button (`‹ ›`) is a separate element between sidebar and main — always visible even when sidebar is collapsed
- Keyboard: `B` toggles sidebar
- Collapse uses `width: 0 !important` + `opacity: 0` on inner content

### Controls Bar Collapse
- The controls bar has two zones: an always-visible `.controls-toggle` strip and an expandable `.controls-body`
- `.controls-toggle` contains: chevron button, play/pause button, mini seek slider, and time label — always accessible even when collapsed
- `.controls-body` contains: file name label, volume, speed row, subtitle toggle, fullscreen, and hints button
- Toggled via the chevron button or `C` key; state tracked in `controlsCollapsed` variable
- Starts collapsed by default (`controlsCollapsed = true` on load)

### Device Detection (groundwork, not yet used for logic)
- `getDeviceType()` / `isSmartphone()` / `isComputer()` helpers detect viewport width < 768px
- Currently only used for a `console.log` on load; likely groundwork for future mobile-specific behavior

### Audio Mode
- Detected by file extension (`AUDIO_EXT` array)
- Shows `.audio-cover` div (music note icon + filename) instead of video element
- `video-container` hidden, `audio-cover` shown (via `.active` CSS class)
- Audio still plays through the `<video>` element under the hood

### Layout
- Video area grows to fill all available space (`flex: 1`)
- Controls bar pinned to bottom with seek row + button row
- `min-height: 100dvh` for correct mobile viewport handling
- Responsive breakpoints:
  - `≤980px`: sidebar moves to top, sidebar toggle hidden, controls adapt
  - `≤640px`: header simplified, smaller fonts/icons

### Keyboard Shortcuts
| Key | Action |
|---|---|
| `Space` | Play / pause |
| `→` | Skip forward 10s |
| `←` | Skip back 10s |
| `↑` | Speed +0.25x |
| `↓` | Speed −0.25x |
| `M` | Toggle mute |
| `F` | Toggle fullscreen |
| `B` | Toggle sidebar |
| `C` | Toggle controls bar |

---

## Known Bugs Fixed During Session

| Bug | Fix |
|---|---|
| Sidebar toggle disappeared when collapsed | Moved toggle button outside `.sidebar` as sibling element in `.body` |
| Subtitle grid overflowed container | Added `min-width: 0` to `.sub-grid` and children; `overflow: hidden` on `.sub-card` |
| MP4 files loading slowly | Was being routed through `hls.js` unnecessarily; fixed to only use HLS for `.m3u8` |
| Click on audio cover not working | `click-overlay` was inside `video-container` (hidden in audio mode); moved to `video-area` level with `z-index: 6` |

---

## Design Tokens (CSS Variables)

```css
--bg: #0f0f11
--surface: #1a1a1e
--surface2: #222228
--border: #2e2e38
--border2: #3a3a48
--text: #e8e8f0
--text2: #8888a0
--accent: #7c6aff
--accent-dim: rgba(124,106,255,0.15)
--accent-border: rgba(124,106,255,0.4)
--yellow: #ffe066      ← subtitle 2 color
--green: #4ade80       ← loaded subtitle indicator
--radius: 10px
--radius-sm: 6px
--sidebar-w: 280px
--controls-h: 96px
```

---

## Future Steps

These were suggested at the end of the session and not yet implemented:

### High priority (language learning focused)
- **A-B Loop** — mark a start point (A) and end point (B), loop that segment indefinitely. Essential for repeating a Chinese phrase until it sticks. UI: two small markers on the seek bar.
- **Subtitle offset adjustment** — slider or +/- buttons to shift subtitle timing by ±5s in 0.1s steps. Common need when `.srt` file is out of sync with the video.

### Medium priority
- **Thumbnail preview on seek bar** — show a small video frame preview when hovering over the seek bar. Requires canvas-based frame extraction.
- **Bookmarks** — click a bookmark button to save the current timestamp with an optional label. Listed below the seek bar or in a sidebar panel. Stored in `localStorage` per file.

### Lower priority / nice to have
- **Playback history** — show recently played files even across sessions (would need to store file metadata, not the file itself, since `File` objects can't be serialized)
- **Custom subtitle colors** — let the user pick colors for sub 1 and sub 2 instead of hardcoded white/yellow
- **Subtitle search** — type a word and jump to the subtitle cue that contains it
- **Chapter markers** — if the video has chapters embedded (common in MKV), display them on the seek bar
- **Picture-in-picture** button — use the native browser PiP API (`vid.requestPictureInPicture()`)
- **Sleep timer** — auto-pause after X minutes (useful for falling asleep to audio)
