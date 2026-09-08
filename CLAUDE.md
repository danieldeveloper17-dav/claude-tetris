# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A classic Tetris implementation in vanilla JavaScript (ES6+), HTML5 Canvas, and CSS. No dependencies, no build step, no package.json.

## Running the game

Open `index.html` directly in a browser, or serve it with any static server:

```bash
python3 -m http.server 8000
npx serve .
php -S localhost:8000
```

There is no build, lint, or test tooling in this repo — changes are verified by opening `index.html` in a browser and playing.

## Architecture

Three files, no modules/bundler — `game.js` is loaded as a single classic script and relies on global scope:

- `index.html` — DOM structure: main `<canvas id="board">` (300×600, i.e. `COLS×BLOCK` by `ROWS×BLOCK`), a `<canvas id="next-canvas">` for the next-piece preview, HUD elements (`#score`, `#lines`, `#level`), and a shared `#overlay` used for both Pause and Game Over.
- `style.css` — dark/retro arcade visual theme.
- `game.js` — all game logic, organized around a small set of global mutable state variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) rather than a class/state object.

### Key mechanics

- **Board model**: `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece type locked there.
- **Pieces**: defined as square matrices in `PIECES` (index 0 unused/null, 1–7 are I/O/T/S/Z/J/L). `COLORS` maps the same indices to hex colors.
- **Rotation** (`rotateCW`): transpose + reverse rows. **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until a non-colliding position is found, else the rotation is discarded.
- **Collision** (`collide`): checks board bounds and existing locked cells for a shape at a given offset. Used for movement, rotation, and ghost-piece projection.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time (`dropAccum`) and advances the piece one row once `dropInterval` is exceeded, otherwise calls `lockPiece()`.
- **Locking** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the current piece into `board`, clears completed rows (scanned bottom-up, with the scan index incremented after a splice so the shifted row is rechecked), then spawns the next piece.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level` for line clears; hard drop adds 2 points per cell dropped, soft drop adds 1 point per row.
- **Leveling/speed**: level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down until collision, drawn at `globalAlpha = 0.2`.
- **Spawn/game over**: `spawn()` promotes `next` to `current` and generates a new `next`; if the new `current` immediately collides, `endGame()` fires and the shared overlay shows "GAME OVER".

### Tunable constants (in `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell pixel size), `COLORS`, `LINE_SCORES`, `dropInterval` (initial). If `COLS`/`ROWS`/`BLOCK` change, update the `width`/`height` of `<canvas id="board">` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).

## Controls

`←`/`→` move, `↑` or `X` rotate (clockwise), `↓` soft drop, `Space` hard drop, `P` pause/resume.
