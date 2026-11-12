# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No `package.json`, build, linter, or tests. The README and UI text are in Spanish; keep user-facing strings in Spanish.

## Running

Open `index.html` directly, or serve statically (e.g. `python -m http.server 8000`) and visit `http://localhost:8000`.

## Architecture

Everything lives in `game.js` as one global-scope script (`'use strict'`), loaded by `index.html`. It binds to DOM ids (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`), so renaming ids in the HTML requires matching changes there.

- **State** is a set of module-level `let` variables (`board`, `current`, `next`, `score`, `paused`, `gameOver`, ...), all reset in `init()`, which is also the restart handler.
- **Board/pieces** use integers 1–7 as both piece type and index into `COLORS`; `0` is empty. `PIECES[type]` are square matrices; rotation is `rotateCW` (no per-piece rotation states), with simple horizontal kicks `[0,-1,1,-2,2]` in `tryRotate`.
- **Lock flow**: `hardDrop`/`softDrop`/`loop` → `lockPiece()` → `merge()` → `clearLines()` (updates score, level, `dropInterval`, HUD) → `spawn()`. `spawn()` calls `endGame()` if the new piece collides.
- **Loop**: `requestAnimationFrame` in `loop()` accumulates `dropAccum` against `dropInterval`; `animId` is cancelled on pause/game over/restart. Unpausing calls `loop()` directly after resetting `lastTime`.
- **Rendering**: `draw()` redraws the full board each frame (grid, board, ghost via `ghostY()` at alpha 0.2, current piece); `drawNext()` only runs on `spawn()`.
- Changing `COLS`/`ROWS`/`BLOCK` requires updating the `<canvas id="board">` width/height in `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`).
