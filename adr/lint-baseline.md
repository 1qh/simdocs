# lint-baseline

Per `book/HARD-RULES.md` "Maximum-strictness lint baseline". Every linter enabled with every rule default-on. Removals require false-positive evidence.

## Active linters

| Tool | Concern |
|---|---|
| biome | Format, import-sort, lint, JSON, YAML, MD |
| oxlint | Fast Rust-based lint pass |
| eslint | TypeScript-strict + react + react-hooks + jsx-a11y + import-x |
| sherif | Workspace dep-version consistency |
| tsc --noEmit | TypeScript strict type check |
| convex-test | Convex function tests |
| fast-check | Property-based test integration |
| `tools/lint/agent-first-output.ts` | `ok` on success scan on every script + Make recipe |
| `tools/lint/zero-fallback.ts` | `?? <literal>` against config reads + getenv patterns |
| `tools/lint/check-no-infinite-wait.ts` | Unbounded `while` / `until` / polling without deadline |
| `tools/lint/check-staleness.ts` | Direct deps with no upstream release in 6 months |
| `tools/lint/stack-presence.ts` | Each STACK.md pick consumed by source |
| `tools/lint/no-school-refs.ts` | Banned vocab per `AGENT-DOCTRINE.md` |
| `tools/lint/atemporal-docs.ts` | Banned temporal phrasings in docs |
| `tools/lint/substrate-boundary.ts` | `packages/*` source contains no domain vocab |
| `tools/lint/spec-of-code.ts` | STACK + SCHEMAS + ISA + datapath ADRs diff against code artifacts |

## Tight thresholds (TypeScript)

- `strict: true`
- `noUncheckedIndexedAccess: true`
- `noImplicitOverride: true`
- `noFallthroughCasesInSwitch: true`
- `noPropertyAccessFromIndexSignature: true`
- `exactOptionalPropertyTypes: true`
- `verbatimModuleSyntax: true`

## ESLint

- `@typescript-eslint/strict-type-checked`
- `@typescript-eslint/stylistic-type-checked`
- `eslint-plugin-react-x` (`recommended-typescript`)
- `eslint-plugin-react-dom` (`recommended`)
- `eslint-plugin-jsx-a11y` (`recommended`)
- `eslint-plugin-import-x` (typed)
- `eslint-plugin-unicorn` (full)
- `eslint-plugin-functional` (no-let where pure)

## Caveat — timing-dependent thresholds

Per `book/HARD-RULES.md` "Timing-dependent lint thresholds: never -warnings-as-errors". Wall-clock-bounded warnings (TypeScript compiler perf hints, slow-expression flags) stay as warnings, not errors. Visible in build output, never block CI under parallel-subagent load.

## File-level disable hygiene

Every disable carries `## reason: <one-line>` per `book/HARD-RULES.md`. Disables placed above `'use client'` per biome + Next interaction. CI fails on undocumented disables.

## Caught by

- `bun run check` green on every push + every commit (pre-commit hook).
- CI lint manifest enumerates every spec-of-code lint; missing wiring fails CI.
- Disable-rationale lint scans for un-`reason`-ed disables.
