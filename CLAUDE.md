# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step, no dependencies. Open `index.html` directly in a browser or serve it with any static server:

```bash
# Python 3
python3 -m http.server 8000

# Node.js
npx serve .
```

Then open `http://localhost:8000`.

## Architecture

Three files, no framework:

- **`index.html`** — DOM structure: `<canvas id="board">` (300×600 px) for the board, `<canvas id="next-canvas">` (120×120 px) for the preview, HUD elements (`#score`, `#lines`, `#level`), and `#overlay` for pause/game-over states.
- **`style.css`** — Dark/retro arcade theme. Uses flexbox layout, CSS variables, and `backdrop-filter` on overlays.
- **`game.js`** — All game logic (~305 lines, `'use strict'`, no modules).

### game.js internals

**State** is held in module-level `let` variables: `board` (2-D array `ROWS×COLS`, cells are `0` or color index 1–7), `current`/`next` piece objects `{type, shape, x, y}`, and loop control (`animId`, `dropAccum`, `dropInterval`, `lastTime`).

**Key functions and their roles:**

| Function | Role |
|---|---|
| `collide(shape, ox, oy)` | Bounds + overlap check against the board |
| `rotateCW(shape)` | Returns a new clockwise-rotated matrix |
| `tryRotate()` | Applies rotation with wall kicks (offsets `[0, -1, 1, -2, 2]`) |
| `lockPiece()` | `merge()` → `clearLines()` → `spawn()` |
| `clearLines()` | Iterates bottom-up; splices full rows and unshifts empty ones |
| `ghostY()` | Projects current piece downward to find drop Y |
| `draw()` | Clears canvas, draws grid + board + ghost (α=0.2) + current piece |
| `loop(ts)` | `rAF` loop: accumulates dt, drops piece on interval, calls `draw()` |
| `init()` | Full reset — called on page load and on restart button click |

**Drop speed formula:** `Math.max(100, 1000 − (level − 1) × 90)` ms per row.

**Scoring:** `LINE_SCORES[cleared] * level` for line clears; +2 pts/cell for hard drop; +1 pt/row for soft drop.

### Tunable constants (top of game.js)

`COLS`, `ROWS`, `BLOCK` (px per cell), `COLORS` (array indexed 1–7), `LINE_SCORES`. If `COLS`, `ROWS`, or `BLOCK` change, update the `width`/`height` attributes on both `<canvas>` elements in `index.html` to match (`COLS×BLOCK` and `ROWS×BLOCK`).
