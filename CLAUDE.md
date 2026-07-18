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
- **Timer loop**: `requestAnimationFrame` with delta-time (`performance.now()` diffs), not `setInterval`, so it stays accurate. A `visibilitychange` handler resets `lastTick` to correct drift after backgrounding.
- **Plan**: the comma-separated input parses into a `plan` array of seconds. Index parity determines the phase: even = ストレッチ (stretch), odd = 休憩 (rest). This parity convention drives colors, labels, and the per-second tick sound.
- **Audio**: Web Audio API oscillator beeps. The `AudioContext` is created/resumed inside the start-button click handler because browsers require a user gesture — don't move that.
- **Wake lock**: `navigator.wakeLock` is requested on start and released on reset/done to keep the screen on.
- **Persistence**: two localStorage keys — `stretch-timer-settings` (current inputs, auto-saved on every change) and `stretch-timer-presets` (named preset list managed via the save/delete buttons). `syncPresetSelection()` keeps the dropdown matching the current inputs; call it after anything that changes the inputs programmatically.

## Service worker caching

`CACHE_VERSION` in `sw.js` must be bumped whenever cached assets change or new ones are added (and new assets added to the `ASSETS` list), otherwise installed PWAs keep serving stale files. Navigation requests are network-first so `index.html` updates reach users quickly; everything else is cache-first.

## Conventions

- UI text, code comments, and commit messages are written in Japanese.
- Keep the app self-contained: no external dependencies, CDNs, or build tooling.
