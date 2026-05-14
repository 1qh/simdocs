# DETERMINISM

Same input → frame-accurate same output, across machines, across sessions, across time. Required for content-addressed share permalinks to be replay-able and verifiable.

## Contract

```mermaid
flowchart LR
    Input[(asm program + initial state + interaction script)] --> Engine[sim-engine]
    Engine --> CanonicalTrace[Canonical execution trace]
    Engine --> CanonicalAnimation[Canonical animation keyframes]
    CanonicalTrace --> Hash[blake3 hash]
    CanonicalAnimation --> Hash
    Hash --> Permalink[(content-addressed permalink)]
```

The permalink IS the proof that two clients running the same input produce identical results. Hash mismatch = determinism violated.

## Required guarantees

1. **State-machine determinism** — `step(state, input) → state'` is a pure function. No randomness, no clocks, no system state read inside.
2. **Animation determinism** — keyframes are deterministic functions of state + step index, not wall-clock-derived.
3. **Codec determinism** — `serialize(state) → bytes` produces identical bytes across machines for identical state. Key ordering, number formatting, compression level fixed.
4. **Hash determinism** — blake3 of identical bytes = identical hex. (Trivially satisfied by blake3.)
5. **Cross-version replay** — old snapshots replay correctly under new code. Schema versions retained; deserializers kept.

## Banned in `sim-engine` source

| Pattern | Banned because |
|---|---|
| `Math.random()` | Non-deterministic per call. Use seeded RNG passed in `state`. |
| `Date.now()`, `performance.now()` | System clock leak. Use `state.cycle` or `state.tickCount`. |
| `Object.keys(o)` for iteration order on plain objects | Insertion-order-dependent across runtimes (historically). Use sorted keys explicitly. |
| `JSON.stringify(o)` without replacer | Key order undefined. Use canonical-stringify helper. |
| `for...in` | Inherited keys vary. Use `Object.entries`. |
| `Set` / `Map` iteration relied upon for state | Iteration order varies subtly. Sort explicitly. |
| `requestAnimationFrame` callback state | Frame timing differs per machine. Animation tied to logical step, not wall-clock. |
| Floating-point comparison (`===` on results of computation) | Bit-exact across platforms not always guaranteed. Use typed integer state where possible. |
| Locale-aware formatting (`toLocaleString`, `Intl.*`) | Locale varies. Use explicit format strings. |

## Caught by

- `tools/lint/no-determinism-leak.ts` — greps `packages/sim-engine` source for every banned pattern; zero hits required
- Cross-machine golden hash test: same input fixture produces same blake3 hash on CI runners (Linux x86, macOS arm) and developer laptops
- Replay test: old snapshot fixtures committed at each schema version; CI replays each, asserts identical resulting state

## Time abstraction

Inside `sim-engine`:

```ts
type Clock = {
  cycle: number;       // current logical cycle
  stepIndex: number;   // sub-cycle step (0..4 for IF..WB)
  rngSeed: number;     // if any sim ever needs randomness
};
```

Never `new Date()`, never `performance.now()` inside the engine.

Outside the engine (UI, animation player):
- Wall-clock is used for animation duration scheduling
- But the animation FRAMES (keyframes, easing curves) are derived from `state.stepIndex`, not from wall-clock
- Wall-clock affects WHEN a frame renders, never WHAT it renders

## Replay invariants

A snapshot includes:
- Source program (asm or pre-encoded words)
- Initial machine state (registers, memory)
- Interaction script (which UI actions were taken — steps, breakpoints, view changes)
- Schema version

Replaying a snapshot:
1. Load source + initial state
2. Replay interaction script step-by-step
3. Final state must hash-match the snapshot's recorded final-state-hash
4. Frame-N state must hash-match per-frame committed hashes (for animation replay verification)

## Anti-patterns banned in product code

- "Optimization" that introduces non-determinism (e.g., parallel reduce with race in result order)
- Caches keyed by anything other than canonical input hash
- `if (process.env.NODE_ENV === 'development')` branches that change observable state
- Conditional logic on `navigator.userAgent` that alters sim outcomes

## Caught by

- Hash-stability fixture test (per snapshot version)
- Property-based test (`fast-check`): generate random valid input, run twice on same machine, assert identical hash
- Cross-machine CI test: same fixture on Linux + macOS runners, assert identical hash
- Replay smoke test: every old-schema-version fixture replays green
