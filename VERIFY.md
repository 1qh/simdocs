# VERIFY

Enumeration of every verifier gate. Per `book/PHILOSOPHY.md` "Cheapest faithful repro" + `book/HARD-RULES.md` "Ledger consultation on session start".

Every gate wrapped by ledger recorder (`make ledger` reports green-at-HEAD).

## Gates

### Format + lint

| Gate | Command | Asserts |
|---|---|---|
| `format` | `bun run fix` | lintmax (biome + oxlint + eslint) all green |
| `lint.disable-reasons` | `tools/lint/disable-reasons.ts` | Every disable has `## reason:` |
| `lint.no-school-refs` | `tools/lint/no-school-refs.ts` | Banned vocab not present in any source / doc |
| `lint.atemporal-docs` | `tools/lint/atemporal-docs.ts` | No temporal phrasing in docs |
| `lint.no-fallback` | `tools/lint/zero-fallback.ts` | No `??` / `\|\|` against config reads |
| `lint.no-infinite-wait` | `tools/lint/check-no-infinite-wait.ts` | Every wait has deadline |
| `lint.no-determinism-leak` | `tools/lint/no-determinism-leak.ts` | `packages/sim-engine` source has no banned non-deterministic patterns |
| `lint.no-school-refs` | greps for school vocab | Zero hits required |
| `lint.substrate-boundary` | `tools/lint/substrate-boundary.ts` | `packages/*` has no domain vocab |
| `lint.agent-first-output` | `tools/lint/agent-first-output.ts` | Every script + Make target `ok` on success |
| `lint.cloudflare-bearer` | `tools/lint/cloudflare-bearer.ts` | No Worker / KV / D1 / Pages Functions imports |
| `lint.no-dangerous-html` | `tools/lint/no-dangerously-set-inner-html.ts` | Zero `dangerouslySetInnerHTML` |
| `lint.no-third-party-trackers` | `tools/lint/no-third-party-trackers.ts` | No tracker SDKs |

### Type check

| Gate | Command | Asserts |
|---|---|---|
| `tsc` | `tsc --noEmit` per workspace | Zero TS errors under strictest settings |

### Codegen freshness

| Gate | Command | Asserts |
|---|---|---|
| `gen.check` | `make gen.check` | Regen + diff = zero |

### Spec-of-code drift

| Gate | Command | Asserts |
|---|---|---|
| `spec.datapath` | `tools/lint/spec-of-code/datapath-diff.ts` | `MIPS-DATAPATH.md` tables match generated topology |
| `spec.isa` | `tools/lint/spec-of-code/isa-diff.ts` | `MIPS-ISA.md` tables match generated ISA |
| `spec.stack` | `tools/lint/spec-of-code/stack-presence.ts` | `STACK.md` picks consumed in source |
| `spec.schemas` | `tools/lint/spec-of-code/convex-schema-diff.ts` | `SCHEMAS.md` matches `apps/backend/convex/schema.ts` |
| `spec.lint-baseline` | `tools/lint/spec-of-code/lint-baseline-diff.ts` | `adr/lint-baseline.md` matches `lintmax.config.ts` |

### Unit tests

| Gate | Command | Asserts |
|---|---|---|
| `test.unit` | `bun test` per workspace | All unit tests pass |
| `test.property` | `bun test --property` | fast-check property tests pass |
| `test.golden` | `bun test golden-trace` | Per-instruction golden traces match committed |
| `test.codec-roundtrip` | `bun test codec` | Serialize → deserialize round-trip |
| `test.codec-hash-stability` | `bun test codec.stability` | Hash unchanged for committed inputs |
| `test.replay` | `bun test replay` | Old-schema-version fixtures replay green |
| `test.crossmachine-hash` | CI Linux + CI macOS run same fixture | Identical hash |

### Integration tests

| Gate | Command | Asserts |
|---|---|---|
| `test.convex` | `convex-test` | Mutations + queries round-trip against in-mem Convex |
| `test.auth-flow` | scripted | Anon → signin → claim flow green |
| `test.rate-limit` | scripted | Rapid saveSnapshot returns 429 |

### E2E tests

| Gate | Command | Asserts |
|---|---|---|
| `test.e2e.anon` | Playwright | Anon session exercises every public route, no auth wall |
| `test.e2e.share` | Playwright | Save → permalink → load → state hash matches |
| `test.e2e.compare` | Playwright | Compare mode renders + steps both panes |
| `test.e2e.pipeline` | Playwright | Pipeline diagram + hazard + forwarding cells correct |
| `test.e2e.kmap-2d` | Playwright | 2D K-map grouping produces expected SOP |
| `test.e2e.kmap-3d` | Playwright | 3D toroidal K-map wrap-edge grouping works |

### Performance tests

| Gate | Command | Asserts |
|---|---|---|
| `perf.bundle-size` | bundle-size CI gate | Per-route budgets per `PERFORMANCE.md` |
| `perf.lighthouse` | `@lhci/cli` | LCP, INP, CLS budgets |
| `perf.frame-budget` | Playwright trace + r3f-perf | 60 fps locked over 10s sample |
| `perf.heap-leak` | scripted heap diff | No leak between identical sim cycles |

### Accessibility tests

| Gate | Command | Asserts |
|---|---|---|
| `a11y.axe` | `axe-core` via Playwright per route | Zero AA violations |
| `a11y.contrast` | `pa11y` per theme variant | Contrast budgets per `A11Y.md` |
| `a11y.keyboard` | Playwright keyboard-only | Per-route keyboard matrix exercised |

### Visual regression

| Gate | Command | Asserts |
|---|---|---|
| `visual.datapath` | Playwright snapshot per instruction step | Matches baseline |
| `visual.kmap` | Playwright snapshot per known truth table | Matches baseline |
| `visual.pipeline` | Playwright snapshot per known program | Matches baseline |

### Mutation tests

| Gate | Command | Asserts |
|---|---|---|
| `mutate.sim-engine` | Stryker | ≥ 80% mutation score on `sim-engine` |
| `mutate.boolean` | Stryker | ≥ 80% mutation score on `boolean` |
| `mutate.bits` | Stryker | ≥ 80% mutation score on `bits` |

### Local-first verification

| Gate | Command | Asserts |
|---|---|---|
| `verify.local` | `make verify.local` | Pure compose stack green, no internet |
| `verify.bearer` | `make verify.bearer` | With CF + Convex self-host green |
| `verify.fresh` | `make verify.fresh` | Bootstrap from clean state |

### Deploy verification

| Gate | Command | Asserts |
|---|---|---|
| `smoke.deploy` | scripted | Deployed URL serves landing + sim routes |
| `smoke.share` | scripted | Share-load round-trip against deployed |

## Ledger consultation

Per session start: `make ledger` reports green-at-HEAD. `make ledger.stale` reports gates unverified at HEAD. Re-running green-at-HEAD without reason is a violation.

## Caught by

- Every gate above wired via ledger recorder
- CI runs every gate on every push
- Manifest lint asserts every gate has a wired script
