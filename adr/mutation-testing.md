# mutation-testing

## Decision

Stryker mutator runs against substrate packages with stable surface:
- `packages/sim-engine`
- `packages/boolean`
- `packages/bits`

Threshold: ≥ 80% mutation score per package. Below → CI red.

## Why these packages

- Pure functions, deterministic outputs → mutation testing produces reliable signal
- Critical correctness (encoding/decoding, Boolean minimization, bit manipulation) where a silent bug compounds
- Heavy fast-check property coverage already in place; mutation testing measures whether those properties actually constrain behavior

## Why not all packages

- `packages/three-kit`, `hud`, `editor`, `design-tokens` — primarily visual / integration surface; mutation score not informative
- `apps/web` features — covered by golden-trace + E2E; mutation testing too slow

## Configuration

`stryker.config.json` per measured package:
- TypeScript checker
- Bun runner
- HTML + JSON reporters
- Threshold: high=85, low=80, break=80

## CI integration

| Gate | Tool | Threshold |
|---|---|---|
| `mutate.sim-engine` | Stryker | ≥ 80% |
| `mutate.boolean` | Stryker | ≥ 80% |
| `mutate.bits` | Stryker | ≥ 80% |

Run on PRs against affected packages (changed-file detection). Full-matrix nightly.

## Caught by

- Stryker exit code in CI
- Mutation score trend in observability dashboard
- Per `book/HARD-RULES.md` "Maximum-strictness lint baseline" mutation row
