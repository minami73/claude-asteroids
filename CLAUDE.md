# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Asteroids arcade clone in a single vanilla JS file, HTML5 canvas, no build tooling, no dependencies, no package.json.

## Run

```bash
npx serve .
```

Or open `index.html` directly in browser. There are no build/lint/test commands — none exist in this repo.

## Architecture

Everything lives in `game.js` (single file, ~420 lines), loaded directly by `index.html` via `<script src="game.js">`. No modules, no bundler.

Structure top to bottom:
- **Input**: raw `keys`/`justPressed` maps populated by keydown/keyup listeners; `pressed(code)` consumes one-shot presses (used for shooting/restart).
- **Entity classes**: `Bullet`, `Asteroid`, `Ship`, `Particle` — each owns its own `update(dt)` and `draw()`. No shared base class or ECS; plain OOP.
- **Global mutable game state**: `ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state` (`'playing' | 'dead' | 'gameover'`), `deadTimer` — declared at module scope, reassigned by `initGame()`/`nextLevel()`.
- **Game loop**: `requestAnimationFrame(loop)` drives `update(dt)` then `draw()` each frame; `dt` is clamped to 0.05s max to avoid physics blowups on tab-switch lag.
- **Collision**: brute-force O(n*m) distance checks each frame (bullets×asteroids, ship×asteroids) — no spatial partitioning, fine at this entity count.
- **World is toroidal**: all entities wrap position via `wrap(v, max)` at canvas edges (800×600, constants `W`/`H`).
- **Asteroid splitting**: size 3→2→1, each split spawns two children of `size - 1` at the same position (`Asteroid.split()`); size 1 destruction yields nothing.
- **State machine**: `update()` branches on `state` first — `gameover` waits for Space to call `initGame()`; `dead` runs a respawn timer then resets ship; `playing` is normal gameplay. Game restart and level transitions (`nextLevel()`) both reset transient arrays but preserve/increment persistent state (`score`, `level`).

When making balance/behavior changes (speeds, radii, points, spawn counts), the relevant tunables sit in named constants near each class (`RADII`, `SPEEDS`, `POINTS`, `ROT`, `THRUST`, `DRAG`, etc.) — prefer editing those over scattering magic numbers.
