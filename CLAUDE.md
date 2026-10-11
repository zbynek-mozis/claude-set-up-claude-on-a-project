# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Minimal Express 4 REST API (users + health check) with an in-memory store, used as a starter project for the Claude Code course.

## Commands

- `npm run dev` — start the API with `node --watch` on `http://localhost:3000` (`PORT` env overrides)
- `npm test` — run all tests (built-in `node:test` runner, discovers `tests/*.test.js`)
- `node --test --test-name-pattern="<regex>"` — run tests whose name matches, e.g. `"health"`
- `npm run lint` — ESLint 8 (`eslint:recommended`); CI runs lint then tests on Node 22

## Conventions

- CommonJS only: `require` / `module.exports`, not `import`/`export` (ESLint `sourceType: "script"`).
- Tests use `node:test` + `node:assert` + `supertest`, not Jest/Mocha. Import `app` from `../server` and call `request(app)` — never start a real server in tests.
- Routes never touch data directly; all reads/writes go through functions exported from `db/store.js`. Add new store functions there rather than mutating arrays in route handlers.
- Error responses are JSON `{ "error": "<message>" }` with the matching status (400 validation, 404 not found); successful creates return 201.
- Each route handler gets a one-line comment `// METHOD /path — what it does`.

## Architecture

- `server.js` builds the Express app, mounts `express.json()` and one router per resource (`/users`, `/health`), and only calls `listen()` when run directly (`require.main === module`). It exports `app` for tests.
- `routes/<resource>.js` — one `express.Router()` per resource; a new resource means a new file plus an `app.use()` line in `server.js`.
- `db/store.js` — module-level in-memory arrays; data resets on restart and **is shared across tests in the same process** (e.g. a `POST /users` in one test is visible to later tests).
- Route params arrive as strings; convert (`Number(req.params.id)`) before comparing to store ids.

## Gotchas

- `.gitignore` lines 2–7 are indented, and git treats leading spaces as part of the pattern — so `.env` and `.claude/settings.local.json` are **not** ignored by this repo's `.gitignore`. Don't create or commit `.env`; check `git status` before committing.
