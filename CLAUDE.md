# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Tetris en JavaScript vanilla + HTML5 Canvas. Sin dependencias, sin build, sin tests, sin linter. Solo 3 archivos: `index.html`, `style.css`, `game.js` (README.md está en español, UI también).

## Ejecutar

Abrir `index.html` en el navegador, o servidor local, p. ej. `python -m http.server 8000` y visitar `http://localhost:8000`.

## Arquitectura (`game.js`)

Todo el juego vive en un único script (`'use strict'`, estado global a nivel de módulo, sin clases ni módulos ES).

- **Estado global** (`board, current, next, score, lines, level, paused, gameOver, lastTime, dropAccum, dropInterval, animId`): se reinicia en `init()`, que también es el handler del botón de reinicio.
- **Tablero**: matriz `ROWS x COLS` (20x10) de enteros; `0` = vacío, `1..7` = índice de pieza. Ese mismo índice indexa `PIECES` (formas) y `COLORS` (color) — agregar una pieza exige tocar ambos arrays y `randomPiece()` (usa `Math.random()*7+1`, número de piezas hardcodeado).
- **Bucle**: `loop(ts)` vía `requestAnimationFrame`; acumula `dropAccum` y baja la pieza cuando supera `dropInterval`. Pausa y game over cancelan el RAF (`cancelAnimationFrame(animId)`); reanudar llama `loop` manualmente.
- **Ciclo de pieza**: `lockPiece()` = `merge()` → `clearLines()` → `spawn()`. `spawn()` detecta game over si la nueva pieza colisiona.
- **Puntuación/nivel** en `clearLines()`: `LINE_SCORES[n] * level`; nivel = `floor(lines/10)+1`; `dropInterval = max(100, 1000-(level-1)*90)`. Soft drop +1 por celda, hard drop +2 por celda.
- **Rotación**: `rotateCW` (matriz) + `tryRotate` con wall kicks horizontales simples `[0,-1,1,-2,2]` (no SRS).
- **Render**: `draw()` repinta todo cada frame (grid, tablero, fantasma con alpha 0.2 vía `ghostY()`, pieza actual); `drawNext()` solo al hacer spawn. `drawBlock` es compartido por ambos canvas (`#board`, `#next-canvas`).
- **Input**: un único listener `keydown` usando `e.code` (flechas, `KeyX` rotar, `Space` hard drop, `KeyP` pausa).

## Acoplamientos a vigilar

- `game.js` obtiene elementos del DOM por id al cargar (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`): renombrar ids en `index.html` rompe el juego.
- Tamaño de canvas (`BLOCK=30`, 10x20) está duplicado entre las constantes de `game.js` y los atributos `width`/`height` en `index.html`; cambiar uno exige cambiar el otro.
