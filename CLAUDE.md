# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla-JS Tetris. Three files, no dependencies, no build, no tests, no package.json.

- `index.html` — DOM structure, two `<canvas>` elements (`board` 300×600, `next-canvas` 120×120).
- `style.css` — dark/retro arcade styling.
- `game.js` — all game logic (~300 lines), single file, top-level module-scope state (no classes, no build modules).

## Run

Open directly or serve statically:

```bash
open index.html                # macOS
python3 -m http.server 8000    # or any static server
npx serve .
```

No lint/build/test commands exist in this repo.

## Architecture

`game.js` uses global mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`) declared once and reset in `init()`.

- **Board**: `ROWS × COLS` matrix; each cell is `0` (empty) or a piece color index (1–7).
- **Pieces**: `PIECES` array of square matrices (index 0 unused). `randomPiece()` deep-copies a shape and centers it. Rotation via `rotateCW` (transpose + reverse rows), applied through `tryRotate()`, which attempts wall kicks `[0, -1, 1, -2, 2]` before giving up.
- **Collision**: `collide(shape, ox, oy)` checks bounds and overlap with locked board cells; used for movement, rotation, ghost-piece projection, and spawn (game-over check).
- **Game loop**: `requestAnimationFrame`-driven `loop(ts)` accumulates `dt` into `dropAccum`; once it exceeds `dropInterval`, the piece drops one row or locks (`lockPiece` → `merge` + `clearLines` + `spawn`).
- **Scoring/leveling**: `LINE_SCORES = [0,100,300,500,800]` × `level`; hard drop adds 2 pts/row, soft drop 1 pt/row. Level increments every 10 cleared lines; `dropInterval = max(100, 1000 - (level-1)*90)`.
- **Rendering**: `draw()` clears and redraws grid, locked board, ghost piece (`globalAlpha = 0.2`, computed via `ghostY()`), and current piece each frame. `drawNext()` renders the preview canvas separately.
- **Input**: single `keydown` listener switches on `e.code` (arrows, `KeyX` rotate, `Space` hard drop, `KeyP` pause); ignored while paused/game-over except unpause.

Tunable constants live at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `PIECES`, `LINE_SCORES`. If `COLS`/`ROWS`/`BLOCK` change, update the `board` canvas `width`/`height` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).
