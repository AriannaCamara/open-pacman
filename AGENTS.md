# AGENTS.md

Juego de Pac-Man en Vanilla JS. **Sin build, sin dependencias, sin tests.**

## Ejecutar y verificar

- No hay `package.json` ni scripts. Abrir `src/index.html` directamente en el navegador (o servirlo con cualquier servidor estático).
- No existen lint/typecheck/test. Verificar cambios manualmente abriendo el juego y revisando la consola del navegador.

## Arquitectura (lo no obvio)

- Los 4 scripts son globals clásicos sobre `window`, **sin ES modules ni imports**. El orden en `src/index.html` importa: `maze.js` → `game.js` → `render.js` → `main.js`. Al añadir un archivo, respetar el orden de dependencias.
- `maze.js` expone `MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`; `game.js` expone `createGame`/`update`/`DIRS`; `render.js` expone `draw`. Todo se comunica vía estos globals.
- `MAZE` es **pristino e inmutable**. Cada `createGame()` copia la matriz a `game.grid` para poder comer dots y reiniciar. Nunca mutar `MAZE`.
- Rejilla 28x31, celda `(x, y)` con origen arriba-izquierda. Códigos: `0` vacío, `1` pared, `2` dot, `3` puerta. Posiciones de actores son fraccionarias mientras se mueven (velocidades en celdas/frame en `game.js`); `TUNNEL_ROW = 14` envuelve horizontalmente.
- Canvas 560x620 con `TILE = 20` (`render.js`).

## Convenciones

- Código (identificadores) en inglés; comentarios y textos de UI en español. El proyecto y sus specs se escriben en español.
- Sin comentarios superfluos: solo los que explican decisiones no obvias (ver el estilo de `maze.js`/`game.js`).

## Spec Driven Development

- Las features se diseñan con el skill `/spec` y se implementan con `/spec-impl`. Los specs van en `specs/NN-slug.md`.
- Nunca escribir código durante la fase de spec; `/spec-impl` solo procede si el spec está en estado `Approved`/`Aprobado`.
- `.agents/skills/` se instala desde `klerith/fernando-skills` (ver `skills-lock.json`). No editar a mano los skills instalados ni `skills-lock.json`.
