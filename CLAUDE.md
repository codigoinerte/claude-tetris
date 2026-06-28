# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step. Open directly or use any static server:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
npx serve .
xdg-open index.html           # Linux quick open
```

## Architecture

Single-file logic (`game.js`, ~305 lines). No framework, no bundler, no dependencies.

**State** — all mutable game state lives in module-level `let` vars: `board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`.

**Board** — `ROWS × COLS` (20×10) 2D array. `0` = empty; `1–7` = piece color index.

**Game loop** — `requestAnimationFrame`-based `loop(ts)`. Accumulates delta time in `dropAccum`; when it exceeds `dropInterval` the piece drops one row or locks.

**Key function chain on lock**: `lockPiece()` → `merge()` (stamp piece onto board) → `clearLines()` (splice full rows, update score/level/speed) → `spawn()` (promote `next` to `current`, generate new `next`; if `collide` immediately → `endGame()`).

**Rotation** — `rotateCW()` transposes + reverses. `tryRotate()` tries wall kicks `[0, -1, 1, -2, 2]` columns before giving up.

**Ghost piece** — `ghostY()` projects `current` straight down; drawn at `globalAlpha = 0.2`.

**Rendering** — two canvases: `#board` (300×600) for the play field, `#next-canvas` (120×120) for the preview. `drawBlock()` is the single primitive used everywhere.

## Key tunable constants (`game.js` top)

| Constant | Default | Effect |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Board size — also update canvas `width`/`height` in `index.html` |
| `BLOCK` | 30 | Pixel size per cell |
| `COLORS` | 7 colors | Piece color palette |
| `LINE_SCORES` | `[0,100,300,500,800]` | Points per 1–4 lines cleared |

Drop speed formula: `max(100, 1000 − (level − 1) × 90)` ms, recalculated in `clearLines()` every time the level changes.
