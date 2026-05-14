# runtime-optimization-discipline

V8 + JIT + cache-locality discipline. Pure-function compute paths in `packages/sim-engine`, `packages/boolean`, `packages/bits` follow these rules without exception.

## Typed arrays for hot state

| State | Type |
|---|---|
| Register file | `Int32Array(32)` |
| Data memory | `Int32Array(N)` (word-addressable) |
| Pipeline stage latches | SoA — separate `Int32Array` per latched field |
| K-map cell outputs | `Uint8Array(2^vars)` (0 / 1 / 2 for X) |
| K-map groupings | `Uint32Array` (bitmask per group) |
| Minterm list | `Uint16Array` (sorted) |

`Record<K, V>` banned in any hot path. Generic `Map` allowed only for compile-time-stable keys.

## Struct-of-arrays (SoA) for entity collections

Pipeline trace stored as parallel arrays:
```ts
type PipelineTrace = {
  cycle: Uint32Array;      // [cycle for each instr]
  stage: Uint8Array;       // [current stage]
  hazardKind: Uint8Array;  // [hazard category or 0]
  forwardSource: Int8Array; // [forwarding source or -1]
};
```

Not array-of-structs:
```ts
// BANNED in hot path
type Bad = Array<{ cycle: number; stage: number; ... }>;
```

Reason: V8 stores objects with shape-dependent overhead; iterating parallel typed arrays is cache-line-friendly.

## Hidden class stability

Hot-path objects locked at construction:

```ts
// good — shape locked, all fields set in constructor-shape
const machineState = Object.freeze({
  pc: 0,
  registers: new Int32Array(32),
  memory: new Int32Array(1024),
});

// banned — post-construction property addition deopts hidden class
const bad = {};
bad.pc = 0;       // 🚫
bad.registers = ...;  // 🚫
```

## Monomorphic call sites

Function call sites in sim-engine + solver must stay monomorphic — same argument shape every call.

```ts
// good — single call site, single shape
function step(state: MachineState, instr: EncodedInstr): MachineState { ... }

// banned — same call site receiving different shapes
function process(input: MachineState | KmapState | PipelineState) { ... }  // 🚫
```

Discriminated unions OK at API boundaries; hot loops dispatch monomorphic.

## Iterative solver paths

QM, Petrick, Espresso implemented iteratively:
- Explicit stack on heap (typed array)
- Explicit loop with break/continue
- No recursive descent in solver hot path

Reason: recursion has function-call overhead + stack-allocation cost. Iterative versions measurably faster for 6-var workload.

## `Object.freeze` on configuration

Frozen at module top-level:
- Control truth table per opcode
- ALU control table
- Component delay table (default)
- ISA opcode map + funct map
- Pseudo-instruction expansion table

V8 optimizes frozen-object property access slightly better; primary win is mutation-prevention against accidental in-place edit.

## Precompiled regexes

Module-level constants:
```ts
const RE_REGISTER = /^\$([a-z]+\d?|\d+)$/;
const RE_HEX = /^0x[0-9a-fA-F]+$/;
```

Never inline `new RegExp(...)` or `/.../` in a hot path.

## Mutate-in-place

In sim-engine + solver hot loops:
- `arr.length = 0` instead of `arr = []`
- `arr[i] = v` over `arr = [...arr.slice(0, i), v, ...arr.slice(i+1)]`
- Typed arrays mutated via `set()` / index assign, never `slice` + concat

## `structuredClone` over JSON roundtrip

For deep copy of `MachineState` or similar in cold paths (snapshot capture, not per-frame):

```ts
// good
const snapshot = structuredClone(machineState);

// banned
const bad = JSON.parse(JSON.stringify(machineState));  // 🚫
```

Reason: `structuredClone` handles typed arrays, BigInt, Map, Set, Date correctly + avoids JSON's lossy roundtrip + faster.

## `WeakMap` / `WeakRef` for collectable caches

Caches that should garbage-collect when keys are unreachable:
```ts
const snapshotDecodeCache = new WeakMap<SnapshotKey, MachineState>();
```

## Browser scheduling

| Mechanism | Use |
|---|---|
| `scheduler.postTask({ priority: 'user-blocking' })` | Input response, immediate feedback |
| `scheduler.postTask({ priority: 'user-visible' })` | Active UI updates, animations |
| `scheduler.postTask({ priority: 'background' })` | Telemetry batching, catalog re-index, abuse-flag scrub |
| `await scheduler.yield()` | Between work units in long-running compute (every 5ms) |
| `requestIdleCallback` | Speculative pre-compute (next frames during scrub, etc.) |

`setTimeout(fn, 0)` banned (no priority signal, deprecated for this use).

## No redirects

Every internal route resolves directly. 301 / 302 / 307 / 308 banned for internal navigation. External link redirects (oauth flows) excepted.

Reason: each redirect costs a round-trip; on slow connections this is the largest avoidable latency.

## Caught by

- `tools/lint/typed-array-hot-state.ts` — sim-engine hot state must be typed array, no `Record<>`
- `tools/lint/no-hidden-class-deopt.ts` — flags post-construction property addition on objects marked `@hot`
- `tools/lint/no-recursive-solver.ts` — solver functions must be iterative
- `tools/lint/precompile-regex.ts` — flags inline `new RegExp(...)` / `/.../` in functions
- `tools/lint/no-redirects.ts` — Next route config + Caddy config audit
- mitata micro-benchmarks per critical path with regression gate ≥10%
