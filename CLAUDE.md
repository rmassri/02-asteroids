# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A clone of the classic arcade game **Asteroids**, built with pure HTML5 Canvas and vanilla JavaScript (ES6+). No frameworks, no bundler, no dependencies, no build step — the entire game lives in a single file, `game.js`, loaded by `index.html`.

## Running the game

There is no build/test/lint tooling — just open the file or serve it statically:

```bash
open index.html          # or double-click it
npx serve .               # then visit http://localhost:3000
```

## Architecture

Everything lives in `game.js` (a single flat file, no modules/imports), organized top-to-bottom into these sections (marked by `── ─` comment banners):

- **Input** — `keys` (held state) and `justPressed` (edge-triggered, consumed via `pressed(code)`) populated by `keydown`/`keyup` listeners.
- **Utils** — `wrap` (toroidal position wrapping), `dist`, `rand`, `randInt`.
- **Entity classes** — `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has its own `update(dt)` and `draw()`; entities mark themselves `dead = true` rather than removing themselves, and are filtered out of arrays elsewhere.
- **Game state** — module-level `let` variables (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`). `state` is one of `'playing' | 'dead' | 'gameover'`.
- **`update(dt)`** — the core game loop step, branching on `state`. Handles input-driven shooting, per-entity updates, bullet↔asteroid and ship↔asteroid collision (brute-force O(n²) distance checks against `radius`), asteroid splitting, scoring, life loss, and level progression (`nextLevel()` when `asteroids.length === 0`).
- **Draw functions** — `drawLifeIcon`, `drawHUD`, `drawOverlay`, `draw()`. Rendering is immediate-mode: clear canvas, draw particles → asteroids → bullets → ship → HUD → overlay.
- **Main loop** — `requestAnimationFrame`-driven `loop(ts)` computing `dt` (clamped to 0.05s) and calling `update(dt)` then `draw()`.

Key conventions:
- World is toroidal: all positions wrap via `wrap(v, max)` against canvas dimensions `W = 800`, `H = 600`.
- Asteroids have `size` 3 (large) → 2 (medium) → 1 (small); `RADII`, `SPEEDS`, `POINTS` arrays are indexed by size. Destroying one calls `.split()`, which spawns two of the next size down (or none at size 1).
- Ship spawns with temporary `invincible` time (visualized as blinking) after death/level start.
- Game state transitions: `initGame()` (fresh run) → `playing` → `dead` (brief respawn delay, `deadTimer`) → back to `playing`, or → `gameover` (Space to restart via `initGame()`).

When modifying gameplay, keep new logic inside the existing single-file structure and follow the established pattern of per-entity `update`/`draw` methods plus `dead` flags for removal.
