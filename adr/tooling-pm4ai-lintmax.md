# tooling-pm4ai-lintmax

## Decision

- `pm4ai` orchestrates monorepo: status (git + dotfile drift + dep audit + CI status + Vercel status if applicable), fix (clean git + sync dotfiles + regen `CLAUDE.md` + run `up.sh`), init (scaffold), setup (operator menubar).
- `lintmax` runs biome + oxlint + eslint in single pass — `bun run fix` is the only entry point.
- `sherif` enforces workspace consistency (dep versions across workspaces).
- `simple-git-hooks` wires `pre-commit: sh up.sh && git add -u`.
- `tsdown` builds packages.
- `turbo` runs scripts across workspaces with caching.

## Invocation discipline

- Run `pm4ai status` first turn of every session against this repo to catch drift.
- Run `bun run fix` (= `lintmax fix`) before every commit.
- Never pipe `lintmax` through `head` / `tail` — masks exit code per memory feedback.
- Never use `void` operator anywhere — `biome --unsafe` strips code with it per memory feedback. Prefer `.catch(noop)` or `async/await`.
- `--no-verify` is safe ONLY after `bun run fix` already passed in the same turn — saves the pre-commit cycle on final commit.

## Lint config shape

`lintmax.config.ts` at repo root:
- biome + oxlint + eslint enabled, all rules default-on
- Removals require false-positive evidence + `## reason:` inline comment per `book/HARD-RULES.md`
- Global ops (comment deletion, compaction) run on ALL files including JSON, YAML, Markdown
- `file-level disables` placed ABOVE `'use client'` + above code per biome + Next interaction quirk
- `biome check --unsafe-parameter-object-binding-patterns=ignore` handles REST parameters per memory feedback

## CLAUDE.md regeneration

`pm4ai fix` regenerates `sim/CLAUDE.md` (the short pointer) from rules database. The repo's `CLAUDE.md` is a 1-line pointer to `simdocs/CLAUDE.md` per `book/HARD-RULES.md` "Prose docs live in book only; code repos carry machine-readable configs only".

## Caught by

- `bun run check` green = lintmax pass on all workspaces.
- `pm4ai status` reports no drift.
- Pre-commit hook fires on every commit.
- CI parity: same `bun run check` runs in GitHub Actions.
