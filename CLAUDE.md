# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A playable classic Tetris implemented in vanilla JavaScript with HTML5 Canvas and CSS. No dependencies, no build step, no package manager — just three files: `index.html`, `style.css`, `game.js`.

## Running the game

There is no build/lint/test tooling in this repo. To run it, either open `index.html` directly in a browser, or serve it with any static file server, e.g.:

```bash
python3 -m http.server 8000
npx serve .
php -S localhost:8000
```

## Architecture

All game logic lives in `game.js` (single file, no modules). The pieces:

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell holds `0` (empty) or a piece-color index (1–7).
- **Pieces**: defined as square matrices in `PIECES`. Rotation is done via `rotateCW`, which transposes and reverses rows (no lookup tables of rotation states).
- **Collision** (`collide`): checks a shape against board bounds and already-locked cells.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until a non-colliding position is found, otherwise the rotation is discarded.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time in `dropAccum` and advances the piece one row when it exceeds `dropInterval`.
- **Line clearing** (`clearLines`): scans bottom-to-top, splices out full rows and unshifts empty ones at the top.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 points/row dropped, soft drop adds 1 point/row.
- **Leveling/speed**: level increases every 10 lines; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece**: `ghostY()` projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2`.

Control flow: `init()` builds the board and starts the loop → `loop(ts)` moves the piece down each `dropInterval` and calls `draw()` every frame → on landing, `lockPiece()` merges the piece into the board, clears lines, and calls `spawn()` → `spawn()` promotes `next` to `current` and generates a new `next`; if the new piece immediately collides, `endGame()` fires and the Game Over overlay is shown.

All DOM/canvas element references are grabbed once at the top of `game.js` as module-level `const`s (`canvas`, `ctx`, `nextCanvas`, `scoreEl`, etc.) and mutated throughout — there is no framework or component structure.

Keyboard input is handled by a single `keydown` listener at the bottom of `game.js`; `P` toggles pause regardless of game state, other keys are ignored while paused or after game over.

## Tuning constants

Easy-to-tweak values at the top of `game.js`: `COLS`, `ROWS`, `BLOCK` (cell size in px), `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`, `ROWS`, or `BLOCK` change, update the `width`/`height` attributes of `<canvas id="board">` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).
