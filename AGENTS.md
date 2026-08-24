# AGENTS.md

Pac-Man clone for learning spec-driven development (see `README.md`, in Spanish). Vanilla JS + Canvas — no package.json, no bundler, no tests, no lint config. Don't invent build steps that don't exist.

## Run

- Open `src/index.html` in a browser (or serve statically). That's the whole dev loop; there is no install/build/test command.

## Architecture: globals via script order

- Files share state through top-level globals, loaded by `<script>` tags in strict dependency order at the bottom of `src/index.html`: `maze.js` → `game.js` → `render.js` → `main.js`. A new file must be inserted **before** its consumers, and nothing is importable/exportable.
- `maze.js` exports the level: `MAZE` (parsed grid), `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`. `game.js` holds rules/state (`createGame`, `update`); `render.js` draws (`draw`); `main.js` owns the loop, keyboard, and overlay screens.
- The maze is authored as 31 strings of 28 chars in `MAZE_STR` with legend `#`=wall, `.`=dot, space=walkable, `-`=pen gate. Keep rows exactly 28 chars and symmetric if editing.
- `createGame()` copies `MAZE` into `game.grid`; never mutate `MAZE` itself (dots are eaten from the copy, restart relies on the original).

## Spec-driven workflow (the point of this repo)

- `/spec` (design) and `/spec-impl` (implement) skills live in `.agents/skills/` (installed via `skills-lock.json`). Specs go in `specs/NN-slug.md`, sequentially numbered, with a status field.
- Only implement a spec whose status means "Approved" (e.g. `Aprobado`); `/spec-impl` creates branch `spec-NN-slug`. Never flip a spec to Approved yourself — the user does.

## Conventions

- Spanish everywhere user-visible: README, UI strings, code comments. Reply in the user's language.
- Code style uses spaces inside parens: `foo( bar )`, `( row ) => ...`. Match it.
