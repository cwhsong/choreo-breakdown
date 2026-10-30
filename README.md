# Choreo Looper

A practice tool for learning dance choreography from video, 8-count by 8-count.

Load any choreo video from your device, set the tempo and where count 1 lands, then loop each 8-count at slow speed in a mirrored view before running the whole routine at full speed.

## Features

- Works with any video file on your device (MP4, MOV, WebM). Videos never leave your device.
- Tempo detection from the song's audio, plus manual BPM, ½ time / ×2 and tap tempo.
- Set count 1 anywhere, then nudge it by a beat or by 0.02s.
- Loop a single 8-count, or chain 8-counts together.
- Speed from 25% to 150% in 5% steps.
- Mirror view, count clicks, and a big on-screen count.
- Per-count notes for each 8-count, saved in your browser per video.

## Keyboard shortcuts

| Key | Action |
| --- | --- |
| Space | Play / pause |
| ← → | Back / forward one count (Shift for the &) |
| ↑ ↓ | Previous / next 8-count |
| [ ] | Speed −5% / +5% |
| L | Loop on/off |
| M | Mirror on/off |
| C | Count clicks on/off |
| T | Tap tempo |
| 1 | Set count 1 at the current moment |
| , . | Nudge count 1 earlier / later |

## Run it locally

Open `index.html` in a browser, or serve the folder:

```
python3 -m http.server
```

then go to http://localhost:8000.

## Deploy

The site is a single static page published with GitHub Pages from the `main` branch. Push to `main` and the live site updates in about a minute.
