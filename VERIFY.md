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

## Checklist

Every box flips `- [ ]` → `- [x]` only with literal evidence captured in the matching `ledger.jsonl` row's `notes` field. Lazy-tick is a violation.

### Format + lint

- [x] `format` — `bun run fix` exits silent
- [x] `lint.disable-reasons` — `tools/lint/disable-reasons.ts`
- [x] `lint.no-school-refs` — `tools/lint/no-school-refs.ts`
- [x] `lint.atemporal-docs` — `tools/lint/atemporal-docs.ts`
- [x] `lint.no-fallback` — `tools/lint/zero-fallback.ts`
- [x] `lint.no-infinite-wait` — `tools/lint/check-no-infinite-wait.ts`
- [x] `lint.no-determinism-leak` — `tools/lint/no-determinism-leak.ts`
- [x] `lint.substrate-boundary` — `tools/lint/substrate-boundary.ts`
- [x] `lint.agent-first-output` — `tools/lint/agent-first-output.ts`
- [x] `lint.cloudflare-bearer` — `tools/lint/cloudflare-bearer.ts`
- [x] `lint.no-dangerous-html` — `tools/lint/no-dangerously-set-inner-html.ts`
- [x] `lint.no-third-party-trackers` — `tools/lint/no-third-party-trackers.ts`

### Type check

- [x] `tsc.bits`
- [x] `tsc.boolean`
- [x] `tsc.design-tokens`
- [x] `tsc.editor`
- [x] `tsc.hud`
- [x] `tsc.sim-engine`
- [x] `tsc.three-kit`
- [x] `tsc.apps-web`
- [x] `tsc.apps-backend`

### Codegen freshness

- [x] `gen.check`

### Spec-of-code drift

- [x] `spec.datapath`
- [x] `spec.isa`
- [x] `spec.stack`
- [x] `spec.schemas`
- [x] `spec.lint-baseline`

### Unit tests

- [x] `test.unit.bits`
- [x] `test.unit.boolean`
- [x] `test.unit.design-tokens`
- [ ] `test.unit.editor`
- [ ] `test.unit.hud`
- [x] `test.unit.sim-engine`
- [ ] `test.unit.three-kit`
- [x] `test.unit.apps-web`
- [x] `test.property.bits`
- [x] `test.property.boolean`
- [x] `test.property.sim-engine`
- [x] `test.codec-roundtrip`
- [x] `test.codec-hash-stability`
- [x] `test.crossmachine-hash`

### Replay codec

- [x] `test.replay.v1`
- [x] `test.replay.v2`
- [x] `test.replay.v3`

### Golden traces — datapath-animated instructions

- [x] `test.golden.add`
- [x] `test.golden.addi`
- [x] `test.golden.and`
- [x] `test.golden.beq`
- [x] `test.golden.bne`
- [x] `test.golden.lw`
- [x] `test.golden.or`
- [x] `test.golden.slt`
- [x] `test.golden.sub`
- [x] `test.golden.sw`

### Golden traces — encodable extension

- [x] `test.golden.andi`
- [x] `test.golden.j`
- [x] `test.golden.lui`
- [x] `test.golden.nor`
- [x] `test.golden.ori`
- [x] `test.golden.sll`
- [x] `test.golden.srl`

### Integration

- [x] `test.convex`
- [x] `test.auth-flow`
- [x] `test.rate-limit`

### E2E — anonymous routes

- [x] `test.e2e.anon.home`
- [x] `test.e2e.anon.mips`
- [x] `test.e2e.anon.kmap`
- [x] `test.e2e.anon.compare`
- [x] `test.e2e.anon.pipeline`
- [x] `test.e2e.anon.learn`
- [x] `test.e2e.anon.foundation`
- [x] `test.e2e.anon.share`
- [x] `test.e2e.share`
- [x] `test.e2e.compare`

### E2E — pipeline hazards

- [x] `test.e2e.pipeline.raw`
- [x] `test.e2e.pipeline.waw`
- [x] `test.e2e.pipeline.war`
- [x] `test.e2e.pipeline.control`
- [x] `test.e2e.pipeline.forwarding`
- [x] `test.e2e.pipeline.stall`

### E2E — K-map 2D

- [x] `test.e2e.kmap-2d.v2-basic`
- [x] `test.e2e.kmap-2d.v3-basic`
- [x] `test.e2e.kmap-2d.v4-basic`
- [x] `test.e2e.kmap-2d.v4-dontcare`
- [x] `test.e2e.kmap-2d.v4-essential-pi`
- [x] `test.e2e.kmap-2d.v4-petrick`
- [x] `test.e2e.kmap-2d.v4-pos`
- [ ] `test.e2e.kmap-2d.v4-multi-output`
- [ ] `test.e2e.kmap-2d.v4-hazard`
- [ ] `test.e2e.kmap-2d.v4-wrap`

### E2E — K-map 3D

- [x] `test.e2e.kmap-3d.v5-basic`
- [ ] `test.e2e.kmap-3d.v5-wrap`
- [ ] `test.e2e.kmap-3d.v5-petrick`
- [ ] `test.e2e.kmap-3d.v6-basic`
- [ ] `test.e2e.kmap-3d.v6-wrap`
- [ ] `test.e2e.kmap-3d.v6-multi-output`

### Performance — bundle size

- [x] `perf.bundle-size.home`
- [x] `perf.bundle-size.mips`
- [x] `perf.bundle-size.kmap`
- [x] `perf.bundle-size.compare`
- [x] `perf.bundle-size.pipeline`
- [x] `perf.bundle-size.learn`
- [x] `perf.bundle-size.foundation`
- [x] `perf.bundle-size.share`

### Performance — Lighthouse

- [ ] `perf.lighthouse.home`
- [ ] `perf.lighthouse.mips`
- [ ] `perf.lighthouse.kmap`
- [ ] `perf.lighthouse.compare`
- [ ] `perf.lighthouse.pipeline`
- [ ] `perf.lighthouse.learn`
- [ ] `perf.lighthouse.foundation`
- [ ] `perf.lighthouse.share`

### Performance — frame budget + heap

- [ ] `perf.frame-budget.datapath`
- [ ] `perf.frame-budget.kmap-2d`
- [ ] `perf.frame-budget.kmap-3d`
- [ ] `perf.frame-budget.compare`
- [ ] `perf.frame-budget.pipeline`
- [ ] `perf.frame-budget.foundation`
- [ ] `perf.heap-leak.datapath-cycle`
- [ ] `perf.heap-leak.kmap-cycle`
- [ ] `perf.heap-leak.share-cycle`

### Accessibility — axe per route × variant

- [x] `a11y.axe.home`
- [x] `a11y.axe.mips`
- [x] `a11y.axe.kmap`
- [x] `a11y.axe.compare`
- [x] `a11y.axe.pipeline`
- [x] `a11y.axe.learn`
- [x] `a11y.axe.foundation`
- [x] `a11y.axe.share`
- [x] `a11y.axe.reduced-motion`
- [x] `a11y.axe.high-contrast`
- [x] `a11y.axe.color-blind`

### Accessibility — contrast + keyboard

- [x] `a11y.contrast.dark`
- [x] `a11y.contrast.light`
- [x] `a11y.contrast.color-blind`
- [x] `a11y.keyboard.home`
- [x] `a11y.keyboard.mips`
- [x] `a11y.keyboard.kmap`
- [x] `a11y.keyboard.compare`
- [x] `a11y.keyboard.pipeline`
- [x] `a11y.keyboard.learn`
- [x] `a11y.keyboard.foundation`
- [x] `a11y.keyboard.share`

### Visual regression — datapath per instruction

- [ ] `visual.datapath.add`
- [ ] `visual.datapath.addi`
- [ ] `visual.datapath.and`
- [ ] `visual.datapath.beq`
- [ ] `visual.datapath.bne`
- [ ] `visual.datapath.lw`
- [ ] `visual.datapath.or`
- [ ] `visual.datapath.slt`
- [ ] `visual.datapath.sub`
- [ ] `visual.datapath.sw`
- [ ] `visual.datapath.andi`
- [ ] `visual.datapath.j`
- [ ] `visual.datapath.lui`
- [ ] `visual.datapath.nor`
- [ ] `visual.datapath.ori`
- [ ] `visual.datapath.sll`
- [ ] `visual.datapath.srl`

### Visual regression — K-map + pipeline

- [ ] `visual.kmap.v2`
- [ ] `visual.kmap.v3`
- [ ] `visual.kmap.v4`
- [ ] `visual.kmap.v4-pos`
- [ ] `visual.kmap.v5`
- [ ] `visual.kmap.v5-wrap`
- [ ] `visual.kmap.v6`
- [ ] `visual.kmap.v6-wrap`
- [ ] `visual.pipeline.raw`
- [ ] `visual.pipeline.waw`
- [ ] `visual.pipeline.war`
- [ ] `visual.pipeline.control`
- [ ] `visual.pipeline.forwarding`
- [ ] `visual.pipeline.stall`

### Mutation testing

- [x] `mutate.sim-engine`
- [x] `mutate.boolean`
- [x] `mutate.bits`

### Local-first verification

- [x] `verify.local`
- [ ] `verify.bearer`
- [ ] `verify.fresh`

### Deploy verification

- [ ] `smoke.deploy.home`
- [ ] `smoke.deploy.mips`
- [ ] `smoke.deploy.kmap`
- [ ] `smoke.deploy.compare`
- [ ] `smoke.deploy.pipeline`
- [ ] `smoke.deploy.learn`
- [ ] `smoke.deploy.share`
- [ ] `smoke.share.dokploy`
- [ ] `smoke.share.cloudflare-tunnel`

### Infra

- [x] `infra.convex.local`
- [ ] `infra.convex.dokploy`
- [x] `infra.cloudflare.dns`
- [x] `infra.cloudflare.tunnel`
- [ ] `infra.ci.actions-enabled`
- [ ] `infra.ci.green-on-main`
- [x] `infra.repos.sim-pushed`
- [x] `infra.repos.simdocs-pushed`
- [ ] `infra.ledger.stale-empty`
