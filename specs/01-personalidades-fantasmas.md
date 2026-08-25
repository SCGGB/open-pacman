# SPEC 01 — Personalidades para los 4 fantasmas

> **Estado:** Aprobado
> **Depende de:** —
> **Fecha:** 2026-08-24
> **Objetivo:** Cuatro fantasmas con comportamientos propios —caza directa agresiva, emboscada, flanqueo y timidez— con salida escalonada de la pen.

## Scope

**In:**

- Ampliar `GHOST_STARTS` en `src/js/maze.js` a 4 fantasmas con `kind` y `releaseAt`.
- Caza directa (agresivo), emboscada a 4 casillas, flanqueo usando al cazador y timidez con umbral de 8 casillas.
- Salida escalonada de la pen (~0 s, ~4 s, ~8 s, ~12 s).
- Toda la IA dentro de `src/js/game.js`, extendiendo `decideGhost`.

**Out of scope (para futuras specs):**

- Power pellets / modo asustado / fantasmas comestibles.
- Modos globales scatter/chase alternados por temporizador.
- Niveles progresivos o subida de dificultad.

## Data model

```js
// maze.js — 4 salidas; hunter empieza fuera, sobre la puerta
const GHOST_STARTS = [
  { x: 13, y: 11, kind: 'hunter',   releaseAt: 0 },
  { x: 13, y: 14, kind: 'ambusher', releaseAt: 240 }, // ~4s a 60fps
  { x: 14, y: 14, kind: 'flanker',  releaseAt: 480 }, // ~8s
  { x: 15, y: 14, kind: 'shy',      releaseAt: 720 }, // ~12s
];

// game.js — nuevo campo en el estado de partida
game.frameCount = 0; // avanza en update(); compara contra releaseAt
```

Los colores ya existen en `render.js` (`GHOST_COLORS`: rojo, cyan, rosa y naranja por índice). Sin cambios en `render.js`, `main.js` ni `index.html`.

## Implementation plan

1. `maze.js`: 4 entradas en `GHOST_STARTS` con `kind` y `releaseAt`. Prueba manual: se ven los 4 colores; los `kind` nuevos caen al fallback aleatorio y el juego sigue funcionando.
2. `game.js`: `frameCount` en `createGame()`, incremento en `update()` y bloqueo del movimiento mientras `frameCount < releaseAt`. Prueba manual: salidas a los ~0/4/8/12 s; tras morir, todos se mueven sin esperar.
3. `game.js`: extraer selector greedy común (`greedyStep(choices, tx, ty)`) e implementar `ambusher`: objetivo = celda de Pacman + 4×su dirección. Prueba: el rosa corta caminos por delante.
4. `game.js`: `flanker`: objetivo = 2×(Pacman + 2 adelante) − posición del hunter (buscarlo por `kind`, no por índice). Prueba: el cyan entra por rutas laterales.
5. `game.js`: `shy`: si la distancia Manhattan es > 8 persigue como hunter; si ≤ 8 maximiza la distancia (huye). Eliminar la rama `random` explícita (queda solo como fallback ante `kind` desconocido).

## Acceptance criteria

- [ ] Se ven exactamente 4 fantasmas: rojo, rosa, cyan y naranja.
- [ ] El rojo puede moverse desde el inicio; rosa, cyan y naranja salen ~4 s, ~8 s y ~12 s (±1 s) tras empezar la partida.
- [ ] El rojo elige en cada cruce la dirección que reduce la distancia Manhattan hasta Pacman.
- [ ] El rosa se dirige hacia la casilla situada 4 posiciones delante de Pacman.
- [ ] El cyan calcula su objetivo combinando la posición del rojo con la de Pacman.
- [ ] A más de 8 casillas, el naranja se acerca; a 8 o menos, se aleja de Pacman.
- [ ] Ningún fantasma invierte su dirección salvo en callejón sin salida.
- [ ] Tras perder una vida, los 4 reaparecen y se mueven sin volver a esperar la escalonada.
- [ ] Al reiniciar la partida, los tiempos de salida parten de cero.
- [ ] Una partida de 60 s no produce errores en consola.

## Decisions

- **Sí:** las 4 personalidades clásicas del arcade adaptadas — reconocibles y con dificultad equilibrada de fábrica.
- **No:** BFS/pathfinding para el agresivo — el greedy actual ya existe y basta; mayor inteligencia iría en otra spec.
- **Sí:** IA en `game.js` extendiendo `decideGhost` — evita exponer `DIRS`/`canMove`/`isWall` como globals nuevos.
- **Sí:** `releaseAt` en frames dentro de `GHOST_STARTS` — declarativo y junto a las posiciones.
- **Sí:** tras morir, todos libres (el contador global ya supera los umbrales) — cero estado extra.
- **No:** `random` como personalidad — desaparece; queda solo como fallback defensivo.
- **Definición rápida sin más rondas de aclaración:** fase de preguntas cerrada en un solo bloque con todas las decisiones confirmadas por el usuario.

## Risks

| Riesgo | Mitigación |
| --- | --- |
| Objetivo del ambusher/flanker cae fuera del laberinto o en un muro | El objetivo solo alimenta la elección greedy de dirección; nunca se navega hacia él ni se valida. |
| El flanker depende del hunter | Buscarlo por `kind === 'hunter'`, no por índice fijo; documentar el requisito en `GHOST_STARTS`. |
| 4 cazadores efectivos = injugable | Solo el rojo persigue directo; el tímido se retira cerca y los otros dos tienen error inherente por objetivo indirecto. |

## What is **not** in this spec

- Power pellets y modo asustado.
- Scatter/chase global temporizado.
- Niveles y dificultad progresiva.

Cada uno, si aterriza, va en su propia spec.
