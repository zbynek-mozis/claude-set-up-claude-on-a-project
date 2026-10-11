# NOTES

## CLAUDE.md — what's in, what's out

**In:** a one-line description, the three npm scripts plus how to run a single test by name, and the conventions Claude would otherwise get wrong or have to ask about: CommonJS only, `node:test` + `supertest` against the exported `app` (no real server), all data access through `db/store.js`, the `{ "error": ... }` response shape with status codes, and the one-line route comment style. The Architecture section covers only what's not obvious from a quick look: `listen()` runs only when `server.js` is the entry point, the store is shared across tests in one process, and route params arrive as strings.

**Out:** a file-by-file listing (Claude can `ls`), generic advice like "write clean code", the endpoint list (it's in the code and would go stale), setup steps from the README, and anything secret. Every line had to pass this test: would Claude make a mistake or ask a question without it?

**Gotcha I kept:** `.gitignore` lines 2–7 start with spaces, and git treats those spaces as part of the pattern. That means `.env` and `.claude/settings.local.json` are *not* ignored, even though the README says they are. This isn't visible from the code and it affects safety, so it gets its own line. I didn't fix `.gitignore` because the brief says not to change app files.

## Permission rules

- **allow** `npm test`, `node --test`, `npm run lint`: read-only, run all the time, safe to skip the prompt.
- **ask** `git push`: it leaves the machine, so I want to confirm each one.
- **deny** `Read(./.env)`: real secrets would live there. Without this rule Claude could read the file into the conversation, and the contents would go to the model provider and could end up in logs or a commit. Because of the `.gitignore` gotcha above, `.env` isn't even protected from `git add`. So this deny rule is the only guard.
- **deny** `git push --force` / `git push -f`: a force-push can overwrite other people's work on the remote with no easy undo.

**Known limits:** Bash rules match on the command prefix, so a reordered command like `git push origin main --force` doesn't match the deny rule. It still falls under the `ask` rule for `git push`, so it can't go out without my confirmation. `Read(./.env)` doesn't stop `cat .env` through Bash. That case is covered only because unlisted Bash commands still prompt.

## Verification

- `/memory` lists `./CLAUDE.md` as loaded project memory.
- `/permissions` shows the allow / ask / deny rules above under project settings.
- In a fresh session, "How do I run the tests here?" got the answer `npm test` / `node --test --test-name-pattern=...` straight from CLAUDE.md.
