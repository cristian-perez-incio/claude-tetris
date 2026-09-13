# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Vanilla Tetris. No build, no deps, no package.json, no tests. 3 files: `index.html`, `style.css`, `game.js`.

## Run

Open `index.html` directly, or serve statically:

```bash
python3 -m http.server 8000
```

There is no lint/build/test command — none exists in this project.

## Architecture

All game logic lives in `game.js` (single file, no modules). Global mutable state (`board, current, next, score, lines, level, paused, gameOver, dropAccum, ...`) is declared once at top and mutated throughout — not encapsulated in a class/object.

- **Board**: `ROWS x COLS` matrix, cell value `0` = empty or `1-7` = piece color index (see `COLORS`/`PIECES`).
- **Piece rotation**: `rotateCW` transposes+reverses the shape matrix; `tryRotate` applies it with wall-kick offsets `[0,-1,1,-2,2]`.
- **Collision**: `collide(shape, ox, oy)` is the single source of truth, used for movement, rotation, ghost piece, and spawn checks.
- **Game loop**: `requestAnimationFrame`-driven `loop()` accumulates `dt` and advances the piece when `dropAccum >= dropInterval`; `dropInterval` shrinks as `level` increases (`max(100, 1000 - (level-1)*90)`).
- **Line clear / scoring**: `clearLines()` splices completed rows out (from bottom, re-checking the same index after splice), scores via `LINE_SCORES` table multiplied by `level`, and recomputes `level`/`dropInterval`.
- **Rendering**: `draw()` clears+redraws grid, locked board, ghost piece (`ghostY()`, alpha 0.2), and the falling piece every frame; `drawNext()` renders the preview canvas separately.
- Input is a single `keydown` listener that ignores input when `paused || gameOver` (except `KeyP` which always toggles pause).

Tunable constants at top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `<canvas id="board">` width/height in `index.html` to match.
