# compute-budgets

## Decision

Per-operation compute budgets locked in `PERFORMANCE.md`. Discipline locked here.

## Web Workers

Any compute > 16 ms on main thread runs in a Web Worker. Locked Workers:

| Worker | Owns |
|---|---|
| `qm-solver.worker.ts` | Quine-McCluskey + Petrick selection for ≥ 4-var K-maps |
| `espresso.worker.ts` | Espresso heuristic for ≥ 7-var (geometry deferred, solver still runs) |
| `pipeline-analyzer.worker.ts` | Pipeline trace arrangement + hazard detection for long programs |
| `assembler.worker.ts` | Asm parse + encode for programs > 100 lines (incremental for short) |

Worker communication via `Comlink` (typed RPC over `postMessage`).

## WASM consideration

Pure-TS QM 6-var typically completes < 50 ms — within budget without WASM. WASM trigger: real-world measurement shows 6-var QM exceeds 50ms p95 on mid-tier laptop. Implementation: AssemblyScript or Rust → wasm-pack → loaded in `qm-solver.worker.ts`. Deferred until measured.

## Object pooling

Three.js hot-loop allocations (Vector3, Quaternion, Matrix4, Color, Euler) are pooled. `packages/three-kit` exposes:

```ts
import { v3, q, m4 } from '@/three-kit/pool';

// in useFrame:
const tmp = v3();    // acquire from pool
tmp.set(x, y, z);
mesh.position.copy(tmp);
v3.release(tmp);    // return to pool
```

Banned in `useFrame`:
- `new THREE.Vector3()`
- `new THREE.Quaternion()`
- `new THREE.Matrix4()`
- Any allocation that escapes the frame

Pool size auto-grows; max bounded at 1024 per type to catch leaks.

## React-side hot path

- `useFrame` callback signature: read state from `useStore.getState()` directly (transient subscribe), never `useState`/`useEffect` inside frame loop
- Mesh refs accessed via `useRef`, never reactive props
- Mesh `.position` / `.rotation` / `.scale` mutated directly via pooled vectors, never destructured + reassigned

## SharedArrayBuffer + COOP/COEP

Deferred. Triggers:
- Real-world measurement shows worker `postMessage` serialization > 5ms p95 on hot-path data
- AND substrate gains an obvious use case (concurrent multi-tab sim state sharing, etc.)

Implementation cost: COOP + COEP headers required everywhere → breaks third-party embeds (currently n/a) + requires audit of every external resource. Not worth until measured.

## Microbenchmark harness

`tools/bench/` carries `mitata`-based micro-benchmarks per sim-engine + boolean + bits operation. CI gate fails on regression > 10% from committed baseline.

Per-operation tracked:
- `step()`, `canonicalize()`, `hash()`, `deserialize()`
- `qm()` per variable count
- `pipeline.arrange()` per cycle count
- `criticalPath.longest()`

## Compiler

React Compiler enabled when stable on Next 16. Drops manual `useMemo` / `useCallback` boilerplate; compiler infers memoization. Reduces source noise; no perf regression expected (compiler ≥ hand-tuned for most cases).

## Caught by

- Per-operation CI gate via `mitata` baseline
- Worker presence lint asserts heavy compute imports from a worker file
- Object-pool lint asserts no `new THREE.Vector3()` / `new THREE.Matrix4()` inside `useFrame` body
- Heap-snapshot diff catches pool leaks
