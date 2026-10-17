# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No dependencies, no build, no tests, no linter, no package.json. README and UI text are in Spanish.

## Run

Open `index.html` directly, or serve statically: `python -m http.server 8000` → http://localhost:8000.

## Architecture

Three files: `index.html` (DOM + two canvases), `style.css`, `game.js` (all logic, ~300 lines, global script, `'use strict'`).

- `game.js` uses module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, ...) all reset in `init()`. Restart button calls `init()` again.
- Board is `ROWS×COLS` matrix; cells are `0` or piece type index 1–7, which indexes both `COLORS` and `PIECES` (index 0 is `null` in both).
- Gravity runs in `loop()` (rAF, accumulates `dropAccum` vs `dropInterval`). Input is handled in a single `keydown` listener; there is no key-repeat/DAS logic of its own.
- Lifecycle: `lockPiece()` → `merge()` → `clearLines()` (updates score/level/`dropInterval`) → `spawn()`; `spawn()` calls `endGame()` if the new piece collides.
- Rotation is `rotateCW` plus simple horizontal kicks `[0,-1,1,-2,2]` in `tryRotate` (not SRS).
- `PIECES` shapes are mutated per piece via copy in `randomPiece()`; random choice is uniform (no 7-bag).

## Gotchas

- Canvas dimensions are hard-coded in `index.html` (`board` 300×600, `next-canvas` 120×120). Changing `COLS`/`ROWS`/`BLOCK` in `game.js` requires updating the canvas `width`/`height` to `COLS*BLOCK` × `ROWS*BLOCK`.
- Pause (`togglePause`) cancels rAF and restarts `loop` on resume; `init()` and `endGame()` also cancel `animId` — keep that pairing to avoid double loops.
- HUD ids (`score`, `lines`, `level`, `overlay*`, `restart-btn`) are looked up at the top of `game.js`; renaming in HTML breaks the script.
