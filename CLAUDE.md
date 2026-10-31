# Choreo Breakdown App — notes for Claude

## What this is
A practice tool for dance. The user loads a choreography video from their own device and the app provides tools to help learn choreo, such as mirroring, changing speed, looping, and breaking into 8-counts. 

It is a single static page hosted on GitHub Pages. There is no server and no build step; videos never leave the user's device.

## Layout
Everything lives in `index.html`: CSS in `<style>`, markup, then one IIFE in `<script>`. The script is organized by `// ----------` comment headers:

- **preset**: `DATA` holds the hand-written breakdown for the original "Love Is a Drug" class video (8 eight-counts + an ending). `isPreset(file)` matches it by file name or size; `PRESET` holds its tempo and count-1 time.
- **state**: `st = {bpm, t0, notes}` is the per-video timing and notes. `PER` = seconds per count (60/bpm), `SEC` = seconds per 8-count. `nSec()` = number of 8-counts in the video. Always call `recalc()` (via `rebuild()`) after changing `st.bpm` or `st.t0`.
- **timing math**: `secStart(i)`, `secEnd(i)`, `secIndexAt(t)`, and `countInfo(t)` (count label, &, and count-in before `t0`).
- **loop**: the loop is stored as 8-count indices `loopI..loopJ`, not times, so it stays correct when tempo or count 1 changes. `loopTimes()` converts to seconds (with a small pre-roll).
- **timing controls**: BPM input, ½ time / ×2, tap tempo (measured in video time so it works at any playback speed), set count 1, nudges.
- **tempo detection**: `detect(file)` decodes the audio with Web Audio, builds an onset envelope (low-pass + full-band log energy, rectified difference), and grid-searches 85–170 BPM with phase. It returns the beat grid; count 1 still has to be set by the user.
- **render loop**: `render()` updates the playhead, the big count, the current 8-count and the highlighted count row; `tick()` enforces the loop each animation frame.
- **loading a video**: `useFile(file, remember)` sets the per-video key, restores or initializes `st`, and rebuilds when metadata loads.

## Storage
- `localStorage['clp-prefs']`: mirror, speed, loop, clicks, theater view (global).
- `localStorage['clp:<file name>|<file size>']`: `{bpm, t0, notes}` per video. `notes[i] = {title, c: [8 strings]}`.
- IndexedDB database `clp`, store `v`, key `video`: the last loaded video file, so it reloads automatically.

Keep these keys backward compatible, or migrate old data, so users don't lose their notes.

## Conventions
- Keep it dependency-free and buildless unless the user decides otherwise. External scripts, if ever needed, should come from a CDN.
- Left/right in notes are always the dancer's own.
- Colors are CSS variables on `:root` with a dark-mode override; use the tokens instead of new hex values.
- The layout must work at phone width (the user practices with it on a phone).
- Wrap every `localStorage`/IndexedDB access in try/catch.
- Test by opening `index.html` in a browser (or `python3 -m http.server`) and loading a real dance video.

## Ideas on the roadmap
- Export/import notes as a JSON file (to move notes between devices).
- Skeleton overlay and webcam follow-along scoring with MediaPipe Pose (JS).
- A counting toggle between the fast pulse and half-time feel.
- Split into `index.html`, `style.css`, `app.js` once the file gets too large.
