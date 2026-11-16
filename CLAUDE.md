# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

An Asteroids-style arcade game written in plain HTML5 Canvas and ES6+ JavaScript. It has no dependencies, no bundler, no build step, no linter and no test suite. The README, the UI text (`NIVEL`, `PUNTAJE`, `ESPACIO PARA REINICIAR`) and the code comments are all in Spanish. Keep new user-facing text and comments in Spanish to match.

## Running

Open `index.html` directly in a browser, or serve the folder:

```bash
npx serve .   # then visit http://localhost:3000
```

Verify changes by playing the game in the browser. There are no automated checks.

## Architecture

All game logic lives in `game.js`, a single classic script (not an ES module) loaded by `index.html`. Its top-level names are globals.

- **Fixed canvas size:** `W = 800` and `H = 600` are hard-coded in `game.js` and must match the `<canvas>` attributes in `index.html`.
- **Toroidal space:** moving objects wrap at the edges with `wrap()`. `Particle` is the exception: particles do not wrap.
- **Entity pattern:** `Bullet`, `Asteroid`, `Ship` and `Particle` each have `update(dt)` and `draw()`. Entities mark themselves `dead = true` and are removed later by `.filter(x => !x.dead)` in `update()`. Collision code relies on this deferred removal, so don't splice arrays mid-loop.
- **Global mutable state:** `ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state` and `deadTimer` are module-level `let`s, reset by `initGame()` and partly reset by `nextLevel()`.
- **State machine:** `state` is `'playing'`, `'dead'` (respawn countdown driven by `deadTimer`) or `'gameover'` (press Space to restart). `update()` branches on `state` at the top.
- **Input:** `keys[code]` holds which keys are currently held, used for rotation and thrust. `pressed(code)` is a one-shot edge trigger that consumes `justPressed`, used for shooting and restarting. Space and the arrow keys have their default browser behavior suppressed.
- **Asteroid sizes:** asteroid size is `3` (large), `2` (medium) or `1` (small). `RADII`, `SPEEDS` and `POINTS` are indexed directly by size, and index 0 is unused padding. `split()` returns two asteroids of `size - 1`.
- **Loop:** `requestAnimationFrame` drives `loop()`, with `dt` in seconds and capped at 0.05. All motion constants are per-second values (px/s, rad/s) scaled by `dt`. The exception is `DRAG`, which is a per-frame multiplier, so drag strength depends on frame rate.
- **Rendering:** everything is vector-drawn in white strokes. There are no image assets apart from `favicon.svg`.

## README vs. code

The README mentions "power-ups" and special asteroid types such as a shooting star ("estrella fugaz"). Neither exists in `game.js` yet. Treat them as planned features, not existing code.
