# SPEC 01 — IA de los 4 fantasmas

> **Estado:** Aprobado
> **Depende de:** —
> **Fecha:** 2026-10-11
> **Objetivo:** Dar a los 4 fantasmas del juego una IA propia y diferenciada, de las cuales Blinky persigue agresivamente a Pac-Man, con salida escalonada de la pen.

## Scope

**In:**

- Aumentar `GHOST_STARTS` de 2 a 4 fantasmas, dentro de la pen existente (reutiliza la geometría actual del laberinto).
- Cuatro comportamientos clásicos, uno por fantasma: Blinky (persecución directa), Pinky (emboscada), Inky (flanqueo), Clyde (tímido por distancia).
- Salida escalonada de la pen por retardo de tiempo fijo por fantasma.
- Re-aplicar la salida escalonada tras perder una vida.
- Pintar cada fantasma con el color correspondiente de `GHOST_COLORS`.

**Fuera de alcance (para futuros specs):**

- Power pellets, modo asustado y comer fantasmas.
- Fases alternantes scatter/chase del arcade original.
- Velocidades distintas por fantasma o aceleración progresiva (Cruise Elroy).
- Cambios a la geometría del laberinto o a la pen.
- Sonidos y animaciones nuevas.

## Modelo de datos

`maze.js` — `GHOST_STARTS` pasa a 4 entradas con identidad y posición:

```js
const GHOST_STARTS = [
  { x: 13, y: 14, name: 'blinky', kind: 'blinky' },
  { x: 14, y: 14, name: 'pinky',  kind: 'pinky'  },
  { x: 12, y: 14, name: 'inky',   kind: 'inky'   },
  { x: 15, y: 14, name: 'clyde',  kind: 'clyde'  },
];
```

`game.js` — estado nuevo por fantasma y contador de frames:

```js
const RELEASE_DELAYS = {        // frames (~60fps): 0s, 2s, 4s, 6s
  blinky: 0, pinky: 120, inky: 240, clyde: 360,
};

// dentro de createGame() y resetPositions()
game.frame = 0;                 // se incrementa en update()
ghost = {
  x, y, dir: 'up', speed: GHOST_SPEED,
  name: 'blinky', kind: 'blinky',
  releaseFrame: game.frame + RELEASE_DELAYS[ kind ],
};
```

Convenciones:

- Coordenadas: origen arriba-izquierda; posiciones fraccionarias mientras se mueve.
- Distancia para decidir giro: Manhattan; distancia para Clyde: euclídea (umbral 8 celdas).
- `releaseFrame` es absoluto respecto a `game.frame`; se recalcula en cada `resetPositions`.

## Plan de implementación

1. **maze.js**: ampliar `GHOST_STARTS` a 4 entradas con `name`/`kind` y posiciones dentro de la pen. Test manual: el juego carga y se dibujan 4 fantasmas con los 4 colores.
2. **game.js**: añadir `game.frame` (incrementado en `update`) y `RELEASE_DELAYS`; gatear `moveGhost` para que un fantasma no se mueva hasta `game.frame >= g.releaseFrame`. Test manual: los fantasmas salen de la pen de forma escalonada (Blinky primero).
3. **game.js**: refactorizar `decideGhost` en `ghostTarget(game, g)` (por `kind`) + un `chooseBestDir(game, g, target)` compartido; implementar `blinky` y `pinky`. Test manual: Blinky se dirige al tile de Pac-Man; Pinky apunta 4 celdas delante de su dirección.
4. **game.js**: implementar `inky` (usa la posición de Blinky: `target = 2*A - Blinky`, con `A` = 2 celdas delante de Pac-Man) y `clyde` (>8 celdas → Pac-Man; ≤8 → esquina `(1, 29)`). Test manual: Inky orbita distinto según Blinky; Clyde se acerca y se retira.
5. **game.js**: actualizar `resetPositions` para reaplicar la salida escalonada (`releaseFrame` relativo al frame actual). Test manual: al perder una vida, todo vuelve a la pen y los fantasmas vuelven a salir escalonados.

## Criterios de aceptación

- [ ] El juego carga sin errores en la consola y se dibujan 4 fantasmas con los 4 colores de `GHOST_COLORS`.
- [ ] Blinky se mueve directamente hacia el tile de Pac-Man.
- [ ] Pinky apunta a 4 celdas delante de Pac-Man según su dirección.
- [ ] El objetivo de Inky depende de la posición de Blinky (`2*A - Blinky`).
- [ ] Clyde persigue a Pac-Man a más de 8 celdas y se retira a la esquina a menos de 8.
- [ ] Los fantasmas salen de la pen de forma escalonada: Blinky al instante, luego ~2s, ~4s y ~6s.
- [ ] Un fantasma que aún no ha salido permanece quieto dentro de la pen.
- [ ] Tras perder una vida, las posiciones se reinician y la salida escalonada vuelve a empezar.
- [ ] Ningún fantasma invierte el sentido salvo en un callejón sin salida.
- [ ] Se mantienen las condiciones de victoria y derrota existentes.

## Decisiones tomadas y descartadas

- **Sí:** modelo clásico de IA (Blinky/Pinky/Inky/Clyde). Es reconocible y reutiliza el `decideGhost` greedy ya existente.
- **No:** power pellets y modo asustado. Merecen su propio spec.
- **No:** ciclo scatter/chase. La persecución pura basta para diferenciar los 4 comportamientos.
- **Sí:** misma velocidad (0.1) para los 4. El cambio es de lógica, no de tuning.
- **Sí:** salida escalonada por retardo de tiempo fijo en frames (0/120/240/360). Más simple que contar dots comidos.
- **No:** contador de dots como disparador (como el arcade). Requiere contabilidad extra sin aportar a este objetivo.
- **Sí:** reutilizar la pen actual (filas 13-15, cols 11-16). Evita tocar `MAZE`.
- **Sí:** re-escalonar tras perder una vida. Comportamiento clásico.

## Riesgos

| Riesgo                                                        | Mitigación                                                                          |
| ------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| Objetivos de Inky/Clyde fuera de la rejilla (valores negativos) | `chooseBestDir` solo usa distancias numéricas; no necesita que el objetivo sea celda válida. |
| Retardos en frames dependen del ritmo de `requestAnimationFrame` | Aceptable para el objetivo; si el navegador limita la pestaña, el timing se alarga.  |
| Fantasmas esperando dentro de la pen podrían solaparse          | No hay colisión fantasma-fantasma; se acepta el solape visual.                        |

## Qué **no** entra en este spec

- Power pellets, fantasmas asustados y puntuación por comer fantasma.
- Fases scatter/chase.
- Velocidades diferenciadas o aceleración progresiva.
- Cambios a la geometría del laberinto o de la pen.

Cada uno de esos, si llega, va en su propio spec.
