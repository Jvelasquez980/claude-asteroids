# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A vanilla-JS clone of the arcade game Asteroids, rendered on an HTML5 canvas. No build tooling, no framework, no dependencies, no package.json — just `index.html` + `game.js`.

## Running it

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no build step, no test suite, and no linter configured. There is nothing to compile — edit `game.js` and reload the browser to see changes.

## Architecture

Everything lives in `game.js` (single file, `'use strict'`, no modules). It's organized top-to-bottom as:

- **Input** — `keys`/`justPressed` maps populated by `keydown`/`keyup` listeners; `pressed(code)` consumes a one-shot "just pressed" edge (used for shooting/restart so holding the key doesn't repeat-fire per frame).
- **Entity classes** — `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has `update(dt)` and `draw()`, and sets `this.dead = true` when it should be removed. Asteroids split into two smaller ones via `split()` (size 3 → 2 → 1 → gone); `RADII`/`SPEEDS`/`POINTS` arrays are indexed by size.
- **Module-level mutable state** — `ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state` (`'playing' | 'dead' | 'gameover'`), `deadTimer`. There's no game object/class; these are plain top-level `let`s mutated directly by `update()`/`draw()`.
- **`update(dt)`** — branches on `state` first (gameover/dead handle their own reduced logic and return early), otherwise: reads input, advances all entities, does bullet↔asteroid and ship↔asteroid collision (brute-force O(n²), fine at this scale), filters dead entities out of arrays by reassigning `bullets = bullets.filter(...)` etc., and calls `nextLevel()` when `asteroids.length === 0`.
- **`draw()`** — clears canvas, draws entities back-to-front (particles → asteroids → bullets → ship), then HUD/overlay.
- **Game loop** — `requestAnimationFrame` loop at the bottom computing `dt` in seconds (capped at 0.05s to avoid physics blowups on tab-switch lag).

Positions wrap toroidally at the canvas edges via `wrap(v, max)` — apply this to any new entity that moves, or it'll fly off-screen permanently.

Canvas is fixed at `W = 800`, `H = 600` (also set as the `<canvas>` element's `width`/`height` attributes in `index.html`, not via CSS).

UI text (HUD, game-over overlay) is in Spanish; keep new user-facing strings consistent with that.
