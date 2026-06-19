# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Game

No build step required. Open `index.html` directly in a browser, or serve it locally:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

There is no test suite — testing is manual, in-browser.

## Architecture

This is a zero-dependency, single-page vanilla JavaScript Tetris game. All logic lives in three files:

- **`index.html`** — two `<canvas>` elements (`#board` 300×600, `#next-canvas` 120×120) plus a HUD panel and a `#overlay` div (toggled via `.hidden`) for pause/game-over states.
- **`style.css`** — dark retro theme; no functional CSS classes except `.hidden`.
- **`game.js`** — entire game in one `'use strict'` script (~305 lines).

### `game.js` structure

**Constants at top:** `COLS`/`ROWS` (10×20), `BLOCK` (30px), `COLORS` (7 entries), `PIECES` (7 shape matrices), `LINE_SCORES`.

**State** is a flat set of `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `lastTime`, `dropAccum`, `dropInterval`, `animId`). `init()` resets all of them.

**Game loop:** `init()` calls `spawn()` then schedules `loop(ts)` via `requestAnimationFrame`. Each frame: accumulate `dt` → auto-drop when `dropAccum >= dropInterval` → call `draw()` → schedule next frame.

**Key functions to know when modifying game behavior:**
- `collide(shape, ox, oy)` — collision detection used everywhere
- `tryRotate()` — SRS-style wall kicks with offsets `[0, -1, 1, -2, 2]`
- `clearLines()` — updates score/level and recalculates `dropInterval`
- `lockPiece()` → `merge()` → `clearLines()` → `spawn()` — the lock chain
- `ghostY()` — projects current piece straight down; used by `draw()` and `hardDrop()`

**Speed formula:** `dropInterval = Math.max(100, 1000 - (level - 1) * 90)` ms; level increments every 10 lines.
