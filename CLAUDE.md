# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Japanese-language stretch/rest interval timer PWA (ストレッチタイマー). It is a zero-dependency static site: no package.json, no build step, no tests, no framework. The entire application — markup, CSS, and JavaScript — lives in `index.html`.

## Development

There is nothing to build or install. To run locally, serve the directory over HTTP (the service worker does not work from `file://`):

```
python3 -m http.server 8000
```

Deployment is GitHub Pages serving the repository root from `main`. All URLs in `index.html`, `sw.js`, and `manifest.webmanifest` are relative (`./...`) so the app works under the Pages subpath — keep new asset references relative.

## Files

- `index.html` — the whole app (UI, styles, timer logic) in one inline `<script>` IIFE
- `sw.js` — service worker: network-first for navigation, cache-first for other assets
- `manifest.webmanifest` — PWA manifest (standalone, portrait, Japanese)
- `icons/` — PWA icons

## Architecture of index.html

- **State machine**: a `state` variable with values `idle | ready | running | paused | done`. `ready` is a 3-second countdown before the first phase.
- **Timer loop / time source**: the audio element's `currentTime` is the clock. `sync()` maps it to the current phase and remaining time; it runs from `requestAnimationFrame` while visible and from the audio `timeupdate` event in the background (to update the lock-screen title). There is no independent JS timer, so display and sound can't drift apart.
- **Plan**: the comma-separated input parses into a `plan` array of seconds. Index parity determines the phase: even = ストレッチ (stretch), odd = 休憩 (rest). This parity convention drives colors, labels, and the per-second tick sound.
- **Audio**: on start, every beep for the run is pre-rendered into an 8-bit mono WAV Blob (`buildAudio()` / `renderWav()`) and played through an `HTMLAudioElement`, so it counts as media playback and keeps sounding while the user is in another app or the screen is locked. The intro file is the 3 s ready countdown + first cycle (+ finish sound when not looping); in loop mode a cycle-only file is then played with `audio.loop=true`. `audio.play()` must be called inside the start-button click handler (user gesture) — don't move that. Pause/resume state follows the audio `pause`/`play` events so lock-screen controls (Media Session) and interruptions stay in sync. To change a sound, edit the event lists in `cycleEvents()` / `buildAudio()`; nothing beeps at runtime.
- **Wake lock**: `navigator.wakeLock` is requested on start (and again when the page becomes visible) and released on reset/done to keep the screen on.
- **Persistence**: two localStorage keys — `stretch-timer-settings` (current inputs, auto-saved on every change) and `stretch-timer-presets` (named preset list managed via the save/delete buttons). `syncPresetSelection()` keeps the dropdown matching the current inputs; call it after anything that changes the inputs programmatically.

## Service worker caching

`CACHE_VERSION` in `sw.js` must be bumped whenever cached assets change or new ones are added (and new assets added to the `ASSETS` list), otherwise installed PWAs keep serving stale files. Navigation requests are network-first so `index.html` updates reach users quickly; everything else is cache-first.

## Conventions

- UI text, code comments, and commit messages are written in Japanese.
- Keep the app self-contained: no external dependencies, CDNs, or build tooling.
