# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla JS. No dependencies, no bundler, no package.json, no tests, no linter. The README and in-game text are in Spanish; keep UI strings and comments in Spanish for consistency.

## Running

Open `index.html` directly in a browser, or serve statically with `npx serve .` (http://localhost:3000). There is no build step; edits to `game.js` take effect on reload.

## Architecture

All game logic lives in `game.js` (loaded by `index.html` via a plain `<script>`, strict mode, globals). The canvas is a fixed 800×600 (`W`/`H` constants must match the `<canvas>` attributes in `index.html`).

- **Loop**: `loop(ts)` → `update(dt)` → `draw()` via `requestAnimationFrame`. `dt` is in seconds and clamped to 0.05 to avoid tunneling after tab switches. All motion is `px/s` scaled by `dt` (except ship drag, which is a per-frame multiplier `0.987`).
- **State machine**: module-level `state` is `'playing' | 'dead' | 'gameover'`. `dead` is a 2s respawn pause (asteroids keep moving, ship doesn't update); `gameover` waits for Space to call `initGame()`. Game state (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`) are top-level `let` variables reset in `initGame()`.
- **Entities**: classes `Ship`, `Bullet`, `Asteroid`, `Particle`, each with `update(dt)`/`draw()` and a `dead` flag. Dead entities are removed by reassigning arrays through `.filter(...)` each frame, not spliced in place.
- **World is toroidal**: positions wrap with `wrap(v, max)`; collisions use simple circle distance (`dist` vs `radius`) and do not account for wrapped-edge overlap. Ship-vs-asteroid uses `a.radius * 0.82` as a forgiving hitbox.
- **Asteroid sizes** are indexed 1–3 into the parallel arrays `RADII`, `SPEEDS`, `POINTS` (index 0 unused). `split()` spawns two of `size - 1`; size 1 yields nothing. Note the lookup tables are indexed by size, so smaller asteroids score more.
- **Input**: `keys[code]` tracks held keys; `justPressed` + `pressed(code)` gives a single-fire edge-triggered read (consumed on read). Shooting uses `pressed('Space')` (no auto-fire); rotation/thrust use held `keys`.
- **Levels**: clearing all asteroids calls `nextLevel()`, which spawns `3 + level` large asteroids outside a safe radius around the center, clears bullets/particles, and resets the ship (resets position and grants 3s invincibility).

## Notes

- The one power-up is **Disparo Triple**: `PowerUp` drops (12% chance per destroyed asteroid, at most once per round/level via `powerUpSpawned`, reset in `initGame()` and `nextLevel()`) and, when the ship touches it, `tripleShotTimer` runs 5s making `Ship.tryShoot(true)` fire 3 bullets in a fan. Lost on death.
- The README still mentions other power-ups and a "shooting star" asteroid; these were removed from the code (commit `13e713f`) and don't exist.
- The README demo link points to a GitHub Pages deployment of the upstream repo.
