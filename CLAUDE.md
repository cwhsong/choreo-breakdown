# Choreo Breakdown App — notes for Claude

## What this is
A practice tool for dance. The user pastes a YouTube link or loads a choreography video from their own device, and the app provides tools to help learn choreo, such as mirroring, changing speed, looping, and breaking into 8-counts. 

It is a single static page hosted on GitHub Pages. There is no server and no build step; local video files never leave the user's device.

## Layout
Everything lives in `index.html`: CSS in `<style>`, markup, then one IIFE in `<script>`. The script is organized by `// ----------` comment headers:

- **preset**: `DATA` holds the hand-written breakdown for the original "Love Is a Drug" class video (8 eight-counts + an ending). `isPreset(file)` matches it by file name or size; `PRESET` holds its tempo and count-1 time.
- **state**: `kind` is the current source (`'file'`, `'yt'` or `null`). `st = {bpm, t0, notes, loops, pos}` is the per-video record (YouTube videos also store `title`, `dur`, `added`, `last`). Build it with `normSt()` so older records without `loops`/`pos` still work. `PER` = seconds per count (60/bpm), `SEC` = seconds per 8-count. `nSec()` = number of 8-counts in the video. Always call `recalc()` (via `rebuild()`) after changing `st.bpm` or `st.t0`.
- **playback**: one interface over both sources. Use `T()` (current time), `isPaused()`, `hasSrc()`, `play()`, `pause()`, `seek()`/`rawSeek()` and `setRate()` instead of touching `vid` or `yt` directly. `stepRate()` handles [ and ]: YouTube only offers fixed speeds, so `setRate` snaps to them.
- **timing math**: `secStart(i)`, `secEnd(i)`, `secIndexAt(t)`, and `countInfo(t)` (count label, &, and count-in before `t0`).
- **loop**: two kinds, one active at a time. An 8-count loop is stored as indices `loopI..loopJ`, not times, so it stays correct when tempo or count 1 changes. An A–B loop is `ab = {a, b}` in seconds. `loopTimes()` returns whichever is active (8-count loops get a small pre-roll). `setLoop()` switches to an 8-count loop and clears `ab`; `followLoop()` does that when the user seeks outside the current loop.
- **A–B + saved loops**: `abA`/`abB` are the A and B marks (shown in the inputs even when an 8-count loop is active). `markAB(which, t, exact)` sets one; button/key presses snap to the nearest count or & when `snapOn`, typed times are exact. `st.loops[i] = {name, a, b, rate}`; `curLoop` is the active saved loop (or -1). While a saved loop is active, A/B edits and speed changes update it. `playSaved(i)` starts one. `savePos()` stores `st.pos` (on pause, every 5s while playing, on switch and on page hide); `gotDuration()` resumes there.
- **timing controls**: BPM input, ½ time / ×2, tap tempo (measured in video time so it works at any playback speed), set count 1, nudges.
- **tempo detection** (files only; the page can't read YouTube's audio, so the Detect button is hidden for YouTube): `detect(file)` decodes the audio with Web Audio, builds an onset envelope (low-pass + full-band log energy, rectified difference), and grid-searches 85–170 BPM with phase. It returns the beat grid; count 1 still has to be set by the user.
- **render loop**: `render()` updates the playhead, the big count, the current 8-count and the highlighted count row; `tick()` enforces the loop each animation frame.
- **YouTube source**: loads the IFrame Player API on demand (`loadYTApi`) into `#ytHost` (controls hidden; our UI drives it). The player only reports its time a few times a second, so `ytTime()` interpolates between reports and ignores stale reports for a moment after a seek. Seeking a cued video makes YouTube start playing, so `ytSeek` sets `ytHold` and the state handler pauses it again. `parseYT()` accepts watch, youtu.be, shorts, live, embed links or a bare 11-char ID. Errors 101/150 mean the owner blocks embedding; the user falls back to a file.
- **library**: YouTube only. It's just the `clp:yt:<id>` records, listed by `libEntries()` sorted by `last`; there's no separate index to keep in sync. A YouTube video joins the library once it loads successfully (so embed-blocked videos don't). `renderLib()`/`openLib()`/`closeLib()`/`openPiece(id)`; thumbnails come from `i.ytimg.com`. Deletes use `confirmTap()` (two taps).
- **loading a video**: `useFile(file, remember)` and `useYouTube(id)` both go through `startLoad(kind)` (stops the other source, bumps `loadTok` so stale async loads are ignored) and `gotDuration(d)`. `failLoad(msg)` resets to no source and reopens the picker with an error. The picker (`openPicker`/`closePicker`) is the drop panel; it takes a YouTube link, a file, or "Reopen last file". `?v=<id or link>` in the page URL opens a YouTube video directly, then is removed from the address bar so a reload reopens whatever was used last. `startLoad()` saves the position of the video being left and flushes pending saves (`flushSave()`).

## Storage
- `localStorage['clp-prefs']`: mirror, speed, loop, clicks, theater view, snap (global).
- `localStorage['clp:<file name>|<file size>']`: `{bpm, t0, notes, loops, pos}` per local video. `notes[i] = {title, c: [8 strings]}`; `loops[i] = {name, a, b, rate}` (seconds).
- `localStorage['clp:yt:<video id>']`: same shape for a YouTube video, plus `title`, `dur`, `added`, `last` (ms timestamps). These records are the library.
- `localStorage['clp-last']`: `{type:'yt', id}` or `{type:'file'}`, the source to reopen on the next visit (missing = old behavior: reopen the cached file).
- IndexedDB database `clp`, store `v`, key `video`: the last loaded video file, so it reloads automatically.

Keep these keys backward compatible, or migrate old data, so users don't lose their notes.

## Conventions
- Keep it dependency-free and buildless unless the user decides otherwise. External scripts, if ever needed, should come from a CDN.
- Left/right in notes are always the dancer's own.
- Colors are CSS variables on `:root` with a dark-mode override; use the tokens instead of new hex values.
- The layout must work at phone width (the user practices with it on a phone).
- Wrap every `localStorage`/IndexedDB access in try/catch.
- Test with `python3 -m http.server` (YouTube embeds don't work from file://) using both a YouTube link and a real dance video file.

## Ideas on the roadmap
- Per-8-count "solid/shaky" marks plus a drill-shaky mode; a speed ramp for saved loops (start slow, +5% every N passes).
- Library extras: trim in/out per piece, rename, playlist import (would need a YouTube Data API key).
- Export/import notes as a JSON file (to move notes between devices). YouTube routines are fully portable this way.
- Skeleton overlay and webcam follow-along scoring with MediaPipe Pose (JS).
- A counting toggle between the fast pulse and half-time feel.
- Split into `index.html`, `style.css`, `app.js` once the file gets too large.
