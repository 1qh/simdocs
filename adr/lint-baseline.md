# lint-baseline

Per `book/HARD-RULES.md` "Maximum-strictness lint baseline". Every linter enabled with every rule default-on. Removals require false-positive evidence.

## Code linting

**`lintmax` is the single entry point.** Orchestrates biome + oxlint + eslint internally with operator-curated config. `bun run fix` runs all three in one pass across TS / TSX / JS / JSX / JSON / YAML / Markdown.

Banned:
- Adding eslint plugins, rules, or configs outside lintmax — lintmax owns eslint surface
- Adding stylelint, dependency-cruiser, knip, depcheck, size-limit, eslint-plugin-X, or any code-lint tool that overlaps lintmax
- Bypassing lintmax with raw `eslint` / `biome` / `oxlint` invocations except inside lintmax itself

If a code-lint concern is not covered by lintmax: file upstream against lintmax (`/Users/o/z/lintmax/`), not in this project.

## Type check

- `tsc --noEmit` per workspace
- TypeScript strict settings:
  - `strict: true`
  - `noUncheckedIndexedAccess: true`
  - `noImplicitOverride: true`
  - `noFallthroughCasesInSwitch: true`
  - `noPropertyAccessFromIndexSignature: true`
  - `exactOptionalPropertyTypes: true`
  - `verbatimModuleSyntax: true`

## Workspace consistency

- `sherif` — dep-version consistency across workspaces

## Test / property / mutation / a11y / perf gates

| Tool | Concern |
|---|---|
| Bun test | Unit + integration |
| `fast-check` | Property-based |
| `convex-test` | Convex functions |
| Playwright | E2E + visual regression + a11y axe-core |
| Stryker | Mutation score ≥ 80% on sim-engine, boolean, bits per `adr/mutation-testing.md` |
| `axe-core` (via Playwright) | Zero WCAG AA violations per `A11Y.md` |
| `pa11y` | Contrast budgets per `A11Y.md` |
| `@lhci/cli` | Lighthouse-CI gates per `adr/perf-budget.md` |
| `bundlesize2` | Bundle-size gates per `adr/perf-budget.md` |
| `memlab` | Heap-leak detection per `adr/perf-budget.md` |
| `r3f-perf` | Frame budget per `PERFORMANCE.md` |
| `mitata` | Sim-engine micro-benchmarks per `adr/compute-budgets.md` |

## Project-specific non-code lints (hand-rolled `tools/lint/*.ts`)

These are NOT code lints — they enforce docs prose, banned vocab, domain policy, spec-vs-code drift. No OSS replacement fits.

| Script | Concern |
|---|---|
| `agent-first-output.ts` | `ok` on success scan on every script + Make recipe |
| `no-school-refs.ts` | Banned vocab per `AGENT-DOCTRINE.md` |
| `atemporal-docs.ts` | Banned temporal phrasings in docs prose |
| `substrate-boundary.ts` | `packages/*` source contains no domain vocab from `GLOSSARY.md` |
| `glossary-coverage.ts` | Every domain term used in product code appears in `GLOSSARY.md` |
| `spec-of-code/*.ts` | Per-allowlist spec-vs-code diff (datapath, ISA, stack, schemas) |
| `no-determinism-leak.ts` | Banned non-deterministic patterns in `packages/sim-engine` per `DETERMINISM.md` |
| `typed-array-hot-state.ts` | Sim-engine hot state typed array, no `Record<>` |
| `no-hidden-class-deopt.ts` | Post-construction property addition on `@hot`-annotated objects |
| `no-recursive-solver.ts` | Solver functions must be iterative |
| `precompile-regex.ts` | Inline regex literals in functions |
| `no-redirects.ts` | Next route config + Caddy config redirect audit |
| `velocity-mode-marker.ts` | `.github/workflows/*.yml` carry disable marker while velocity-mode active |
| `error-catalog.ts` | Every error code literal in source appears in `ERROR-CATALOG.md` |
| `telemetry-events.ts` | Every event-emit site matches `TELEMETRY-EVENTS.md` catalog |
| `use-cache-discipline.ts` | `'use cache'` only on pure-function modules + non-auth queries |
| `oss-import-first.ts` | Hand-rolls match exception entries in `adr/oss-import-audit.md` |
| `auth-wall.ts` | Banned wall phrases in product copy |
| `brand-presence.ts` | Brand name appears only after locked + no other invented names |
| `state-layer.ts` | Per-layer state ownership patterns from `STATE-MANAGEMENT.md` |
| `stack-presence.ts` | Each `STACK.md` pick consumed by source |
| `zero-fallback.ts` | `?? <literal>` against config reads + getenv patterns |
| `check-no-infinite-wait.ts` | Unbounded wait loops without deadline |
| `check-staleness.ts` | Direct deps with no upstream release in 6 months (complements Renovate) |
| `api-conventions.ts` | Server Action / Route Handler / Convex naming + validation shape per `API-CONVENTIONS.md` |
| `no-third-party-trackers.ts` | Banned analytics SDK imports per `OBSERVABILITY.md` |
| `cloudflare-bearer.ts` | No Cloudflare Worker / KV / D1 / Pages Functions imports |
| `worker-pool-sizing.ts` | Workers spawned with correct count formula per `CONCURRENCY.md` |
| `abort-signal-coverage.ts` | Every async exported fn declares `signal?: AbortSignal` |
| `promise-all-discipline.ts` | Sequential `await` on independent operations |
| `forms-progressive.ts` | `<form>` elements wired to Server Action via `action` prop |
| `no-determinism-leak.ts` | Determinism patterns per `DETERMINISM.md` |

Some of these patterns may move into lintmax upstream if operator decides they generalize — until then they live here as project-specific guards.

## Caveat — timing-dependent thresholds

Per `book/HARD-RULES.md` "Timing-dependent lint thresholds: never -warnings-as-errors". Wall-clock-bounded warnings stay as warnings, not errors. Visible in build output, never block CI under parallel-subagent load.

## File-level disable hygiene

Every lintmax disable carries `## reason: <one-line>` per `book/HARD-RULES.md`. Disables placed above `'use client'` per biome + Next interaction. CI fails on undocumented disables.

## Caught by

- `bun run check` green on every push + every commit (pre-commit hook, post velocity-mode exit)
- CI lint manifest enumerates every project-specific lint; missing wiring fails CI
- Disable-rationale lint scans for un-`reason`-ed disables
