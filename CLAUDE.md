# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running

No build, no dependencies, no tests, no linter. Vanilla HTML/CSS/JS loaded via `<script src="game.js">`.

- `open index.html` — run directly
- `python3 -m http.server 8000` — serve locally (recommended)

Verify changes by loading the page and playing; there is no test suite to run.

## Architecture

`game.js` is the whole game, written as top-level script state plus free functions (no classes, no modules). Key implications:

- Mutable globals declared on one `let` line: `board, current, next, score, lines, level, paused, gameOver, lastTime, dropAccum, dropInterval, animId`. `init()` resets all of them; `restart-btn` calls `init()` directly.
- `board` is a `ROWS × COLS` matrix of `0` (empty) or a piece type `1–7`, which indexes both `COLORS` and `PIECES`. The type index *is* the color — keep those two arrays aligned (index 0 is `null` padding in both).
- Piece shapes are square matrices; `rotateCW` transposes + reverses. `tryRotate` does simple wall kicks by trying x-offsets `[0,-1,1,-2,2]` — not SRS.
- `collide(shape, ox, oy)` is the single validity check used by movement, rotation, ghost projection and game-over detection. Any new movement must go through it.
- Loop is `requestAnimationFrame`-driven (`loop`): accumulates `dt` into `dropAccum`, drops a row when it exceeds `dropInterval`. Pause/game-over call `cancelAnimationFrame(animId)`; unpausing resets `lastTime` before restarting the loop, so don't restart it without that.
- Rendering redraws everything each frame: `drawGrid` → board cells → ghost (alpha 0.2) → current piece, all via `drawBlock`.
- Level/speed: level rises every 10 lines, `dropInterval = max(100, 1000 - (level-1)*90)`, set only in `clearLines`.

## Gotchas

- `COLS`/`ROWS`/`BLOCK` in `game.js` must stay in sync with the hardcoded `width`/`height` of `<canvas id="board">` in `index.html` (300×600 = 10×30 by 20×30). Same for `#next-canvas` (120×120) and the `NB = 30` constant in `drawNext`.
- `game.js` grabs DOM ids at load time (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`, `theme-toggle`). Renaming any id in `index.html` breaks the script at startup.
- Theme (light/dark) is CSS-variable driven: colors live under `:root` (dark, default) and `:root.light` in `style.css`. The canvas grid line color isn't CSS, so `game.js` reads the `--grid-line` custom property via `getComputedStyle` whenever the theme toggles — keep that in sync if new canvas-drawn colors are added. Preference persists in `localStorage` under `tetris-theme`.
- User-facing strings are Spanish (`PAUSA`, `Puntuación`, `Reiniciar`); README is Spanish too. Keep new UI text in Spanish.
