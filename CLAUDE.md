# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Clon del clásico arcade Asteroids implementado en HTML5 Canvas puro (ES6+), sin frameworks, bundler ni dependencias. Todo el código del juego vive en un único archivo: `game.js`.

## Running

No build step. Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no test suite, linter, or package.json in this repo.

## Architecture

Everything is in `game.js`, structured top-to-bottom as:

- **Input** — `keys`/`justPressed` maps populated by `keydown`/`keyup` listeners; `pressed(code)` consumes a one-shot press (used for shooting and restart).
- **Entity classes** — `Bullet`, `Asteroid`, `Ship`, `Particle`. Each has `update(dt)` and `draw()`. All movement wraps toroidally across the canvas via the `wrap()` util (space wraps at edges, W=800 H=600).
- **Asteroid sizing** — size is an integer 3→2→1 (large/medium/small) indexing into parallel arrays `RADII`, `SPEEDS`, `POINTS`. `Asteroid.split()` produces two smaller asteroids on destruction; size 1 asteroids don't split.
- **Global mutable game state** — `ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state` (`'playing' | 'dead' | 'gameover'`), `deadTimer` are module-level `let` bindings reassigned by `initGame()` / `nextLevel()`, not encapsulated in a class.
- **Game loop** — `requestAnimationFrame(loop)` computes `dt` (clamped to 0.05s) and calls `update(dt)` then `draw()` each frame. `update()` branches on `state`: `gameover` waits for Space to restart, `dead` runs a respawn timer, `playing` runs full simulation (movement, bullet↔asteroid and ship↔asteroid collision via circle-distance checks, level completion when `asteroids.length === 0`).
- Arrays (`bullets`, `asteroids`, `particles`) are updated in place then rebuilt via `.filter(x => !x.dead)` each frame rather than mutated in place — follow this pattern when adding new entity types.
