# PERFORMANCE

Explicit performance contract. Every budget is a CI gate; violation fails the build. Targets are world-class ceiling, not Web Vitals "good" floor.

## User-perceived budgets

| Metric | Budget | Measured at |
|---|---|---|
| LCP (Largest Contentful Paint) | ≤ 1.0s on cable, ≤ 2.0s on slow 4G | Real-user (RUM) + Lighthouse-CI synthetic |
| INP (Interaction to Next Paint) | ≤ 16ms on desktop (single 60Hz frame), ≤ 50ms on mid-tier laptop, ≤ 100ms on mobile read-only | RUM + Playwright trace |
| CLS (Cumulative Layout Shift) | ≤ 0.01 | RUM + Lighthouse-CI |
| TBT (Total Blocking Time) | ≤ 100ms | Lighthouse-CI |
| TTI (Time to Interactive) | ≤ 2.0s on cable | Lighthouse-CI |
| TTFB (Time to First Byte) | ≤ 200ms edge-cached, ≤ 500ms origin | RUM + Caddy access log |
| Step-animation transition | ≤ 400ms total, no dropped frames | Playwright trace |
| Long task count (RUM) | 0 tasks > 50ms during steady interaction | PerformanceObserver in `apps/web/src/lib/rum.ts` |
| Active DOM nodes per route | ≤ 500 | Playwright DOM count assertion |

## Frame budget

| Display refresh | Target | Floor | p99 frame time |
|---|---|---|---|
| 60 Hz | 60 fps | 60 fps | ≤ 16.67 ms |
| 120 Hz / ProMotion | 120 fps | 60 fps | ≤ 8.33 ms when 120fps active |
| 144 Hz gaming | 144 fps | 60 fps | ≤ 6.94 ms |
| 240 Hz gaming | 240 fps | 60 fps | ≤ 4.16 ms |
| 360 Hz gaming | 360 fps | 60 fps | ≤ 2.77 ms (cap) |

`<Canvas frameloop="demand">` ensures idle frame cost = 0 ms (no render when no state changes). Idle CPU + GPU usage = 0 (verified via `r3f-perf` + Chrome perf trace).

Refresh-rate detection at canvas mount via `screen.refreshRate` (or `matchMedia('(update: fast)')` proxy); animation step duration scales inversely with refresh rate so transitions feel identical across displays. Refresh rate above 360 Hz capped at 360 fps target — diminishing returns past human perception.

## 3D scene-specific budgets

| Resource | Budget |
|---|---|
| Draw calls per frame | ≤ 50 in hero datapath scene, ≤ 100 in K-map toroidal, ≤ 200 absolute cap |
| Triangles per frame | ≤ 200,000 |
| Active materials | ≤ 30 unique |
| Active textures | 0 (pure-procedural per `adr/asset-authoring.md`) |
| Shader compile time first use | ≤ 100 ms per shader |
| First 3D paint after hydration | ≤ 500 ms |
| Scene swap (route change, mode toggle) | ≤ 200 ms, zero dropped frames during transition |
| Geometry / material allocations per frame | 0 (pooled, reused) |

## Asset budgets

| Asset class | Budget | Enforcement |
|---|---|---|
| Initial bundle (non-editor, non-3D routes) | ≤ 150 KB gzip | bundle-size CI gate |
| Editor-route bundle | ≤ 500 KB gzip (Monaco dynamic-imported) | bundle-size CI gate |
| 3D-route bundle | ≤ 400 KB gzip (three + drei + product features) | bundle-size CI gate |
| CSS shipped per route | ≤ 25 KB gzip | bundle-size CI gate |
| Per-request JSON payload | ≤ 4 KB gzip (snapshot reads) | Route Handler smoke |
| Font subset (initial) | Latin + math + Boolean symbols only, ≤ 30 KB woff2 | Font loading hook |
| Font additional weights | Lazy-loaded after first paint, ≤ 50 KB total | Font loading hook |
| Image / texture | All procedural; zero external textures | Repo-asset lint |

## Sim-engine compute budgets

| Operation | Budget |
|---|---|
| `step(state)` single-cycle MIPS | ≤ 1 ms per cycle |
| `canonicalize(state)` + `blake3(bytes)` | ≤ 10 ms for typical snapshot |
| `deserialize(bytes)` | ≤ 5 ms |
| Quine-McCluskey 4-var | ≤ 5 ms |
| Quine-McCluskey 6-var | ≤ 50 ms (Web Worker, off main thread) |
| Espresso 12-var (deferred geometry) | ≤ 500 ms (Web Worker) |
| Pipeline trace arrange 100 cycles × 20 instructions | ≤ 30 ms |
| Critical-path longest-path walk | ≤ 5 ms |

Operations above 16ms on main thread → Web Worker required.

## Runtime memory

| Resource | Ceiling |
|---|---|
| JS heap (steady state, after one full sim cycle) | ≤ 100 MB |
| GPU memory (R3F context) | ≤ 200 MB |
| Live event listeners | ≤ 500 |
| Open `requestAnimationFrame` callbacks | ≤ 5 unique sources |
| Object pool allocations per frame | 0 (pooled per `adr/compute-budgets.md`) |

Memory leak detection via Playwright `page.coverage` + heap snapshot diff between identical sim cycles.

## Cold-start vs warm-cache

| Path | Cold | Warm |
|---|---|---|
| Landing route | ≤ 1.0s LCP | ≤ 200ms LCP (CF edge cached) |
| `/s/[hash]` shared permalink | ≤ 1.5s LCP (one origin hop) | ≤ 200ms LCP (CF edge cached forever, immutable) |
| `/datapath` interactive | ≤ 2.0s TTI | ≤ 800ms TTI |
| `/kmap` interactive | ≤ 1.5s TTI | ≤ 600ms TTI |
| `/learn/*` MDX | ≤ 1.0s LCP | ≤ 200ms LCP |

## Network

| Aspect | Target |
|---|---|
| Protocol | HTTP/3 (QUIC) preferred, HTTP/2 fallback. Caddy + Cloudflare both support. |
| Compression | Brotli level 6 (balance between ratio and CPU). gzip fallback. |
| CDN cache hit ratio | ≥ 95% (content-addressed share paths → effectively 100%) |
| Resource hints | `<link rel="preconnect">` to Convex + Cloudflare; `<link rel="preload">` for critical fonts + above-fold assets |
| HTTP/3 0-RTT | Enabled where supported |

## Build + DX perf

| Operation | Budget |
|---|---|
| Turbopack incremental rebuild | ≤ 500 ms |
| Turbopack full build | ≤ 30 s |
| HMR module update | ≤ 100 ms |
| Dev server cold start | ≤ 5 s |
| Test suite (bun test, single package) | ≤ 30 s |
| Test suite (all packages) | ≤ 90 s |

## CI gates

| Gate | Tool | Threshold |
|---|---|---|
| Bundle-size budget | `bundlesize2` or equivalent | Per-route budget above |
| Bundle diff per PR | `next-bundle-analyzer` + CI comment | Regression > 5% flagged on PR |
| Lighthouse-CI synthetic | `@lhci/cli` | LCP, INP, CLS, TBT, TTFB per budgets above |
| Playwright trace | Playwright `--trace on` | INP, dropped frame count, frame p99 |
| Heap snapshot diff | Custom script | Leak detected = fail |
| `memlab` leak detection | Meta's memlab CI | Detached DOM / closure leaks |
| Long-task RUM aggregate | PerformanceObserver client → `/api/rum` | 0 tasks > 50ms during interaction |
| DOM node count | Playwright | ≤ 500 active per route |
| 3D frame timing | `r3f-perf` + Playwright | Locked frame budget over 10s sample |
| Draw-call audit | Three.js renderer.info | Per-scene draw-call budget |
| Shader compile time | Custom probe | First-use compile ≤ 100ms; AOT verified |
| Sim-engine micro-benchmark | `mitata` or equivalent | Per-operation budgets above |
| Network protocol assertion | curl + smoke | HTTP/3 negotiated when available |
| Early Hints assertion | curl smoke | 104 Early Hints emitted on slow paths |
| Cache hit ratio | CF Analytics API or origin log analysis | ≥ 95% weekly |

## Optimization patterns (locked)

- React Server Components for everything non-interactive
- Suspense streaming with skeleton geometry while 3D scene loads
- Asset preload hints from RSC (`packages/three-kit` exposes `preload()` helpers)
- `useTransition` for heavy state swaps in sim
- zustand transient subscribes for hot-loop state (skip React render)
- Instanced meshes for any geometry count > 100
- BVH raycasting (drei `Bvh`) for cell picking in K-map
- Frame-loop on demand (`<Canvas frameloop="demand">`) — only render when state changes, freezes when idle
- Geometry / material pooled and reused — no per-frame allocations
- Object pooling for hot-loop Vector3 / Quaternion / Matrix4 allocations
- AOT shader compilation at scene mount via `renderer.compile(scene, camera)` — eliminates first-use shader compile stutter
- Pre-warmed Worker pool at app boot — QM + assembler + pipeline workers idle-warm, zero spin on first heavy compute
- Texture KTX2 + zstd for any future raster (currently none per asset-authoring ADR)
- HTTP/3 + Brotli compression at Caddy (level 6 dynamic, level 11 static at build)
- 104 Early Hints via Caddy — preload critical fonts + 3D bundle while origin computes
- Web Workers for compute > 16ms (QM, Espresso, pipeline analyzer, assembler for > 100 line programs)
- Pixel ratio capped at 2 on high-DPI displays to prevent over-rendering
- `contain: strict` on 3D scene container for layout isolation
- `content-visibility: auto` on off-screen panels (sidebar register/memory/control tables out of viewport)
- Critical CSS inlined via Tailwind 4 JIT
- React Compiler enabled, all files compiled — manual `useMemo` / `useCallback` deleted
- CSS containment on every Card / Panel surface
- `'use cache'` server-side memoization for QM solver results + assembler output + critical-path computation, keyed by input hash (per `adr/server-cache-budgets.md`)
- `cache()` from React 19 for RSC fetch deduping within a request
- `fetchpriority="high"` on critical fonts + above-fold 3D bundle
- View Transitions API for cross-route navigation (Next 16 native)
- Speculation Rules API — prerender likely-next routes on hover/idle (per `adr/speculative-loading.md`)
- Stale-while-revalidate at CDN — survive flagged-then-purge edge race for permalink reads
- Preemptive scrub seeking — when user drags timeline scrubber, pre-compute frames ahead of cursor in idle gap
- Service Worker pre-caching of likely-next-routes on idle (PWA layer)
- OffscreenCanvas + R3F-in-Worker — entire 3D rendering off main thread (per `adr/offscreen-canvas-render.md`)
- Typed arrays for sim-engine state — `Int32Array(32)` register file, `Uint8Array` memory, no `Record<>` for hot state (per `adr/runtime-optimization-discipline.md`)
- Struct-of-arrays (SoA) layout for pipeline trace + K-map cell state arrays (cache locality)
- Hidden class stability — hot-path object shapes locked at construction, no post-construction property addition
- Monomorphic call sites — sim-engine + solver paths stay monomorphic (V8 inline cache)
- `Object.freeze` on hot config objects (delay table, control truth table, opcode map)
- Iterative (not recursive) QM + Petrick implementation
- Scheduler API (`scheduler.postTask` with explicit priorities) for compute dispatch
- `await scheduler.yield()` between work units in long-running hot paths
- `requestIdleCallback` for telemetry batching, abuse-flag scrubs, catalog index updates
- Mesh batching via `BufferGeometry.mergeGeometries` for static datapath substrate elements
- Programmatic LOD via drei `Detailed` — simpler geometry at distance
- `renderer.autoClear = false` with manual clear of only-dirty regions
- Texture atlas for shader-baked etched labels (single texture binding)
- GPU instancing required for any repeated mesh > 8 instances
- Eager per-page font subsetting via fonttools at build (only glyphs present on the page)
- `structuredClone` over `JSON.parse(JSON.stringify(...))` for deep copy
- `WeakMap` / `WeakRef` for collectable caches (snapshot decode cache, etc.)
- Server Action streaming responses for QM step-through reveal + assembler progress
- Zero 301/302 redirects — direct routing always
- Precompiled regexes at module top-level, reused
- Mutate-in-place over `slice` / `concat` in hot loops

## Anti-patterns banned

- Per-frame `setState` in `useFrame` (mutate refs instead)
- New object props on R3F components in render (`useMemo` or stable refs)
- Synchronous JSON parsing > 16 ms on main thread (Web Worker)
- Layout thrash (batch DOM reads/writes)
- Render-blocking third-party scripts (none allowed regardless)
- `setState` cascades (use `useReducer` or zustand)
- Synchronous `JSON.stringify` of large objects on main thread
- Reading `getBoundingClientRect` during animation frame (cache)
- `forEach` in hot loops (use `for` loop)
- New `Vector3()` / `Matrix4()` / `Quaternion()` allocations in `useFrame` (pool them)
- Mounting heavy components synchronously without Suspense boundary
- Cold Worker spin on first heavy compute (workers must be pre-warmed)
- First-use shader compile during user-visible animation (AOT compile at scene mount)
- Off-screen panel rendering when scrolled out of viewport (use `content-visibility: auto`)
- `useEffect` for read-after-render measurements that fire on every render (use `useLayoutEffect` or `useSyncExternalStore`)
- Re-allocation of large arrays during hot path (Float32Array pre-allocated and reused)
- Post-construction property addition on hot-path objects (hidden class deopt)
- Megamorphic call sites in sim-engine + solver (V8 inline-cache deopt)
- Recursive QM / Petrick / Espresso (use iterative)
- `Record<K, V>` for sim-engine hot state (typed arrays only)
- 301/302 redirects (direct routing always)
- `slice` / `concat` in hot loops (mutate-in-place)
- `JSON.parse(JSON.stringify(...))` for deep copy (use `structuredClone`)
- Inline regex literals re-created in hot path (precompile at module top-level)

## Deferred-with-trigger ratchets

| Ratchet | Trigger to activate |
|---|---|
| WebGPU compute shaders for QM / Espresso solver | Real-world measurement shows current Worker path exceeds budget under realistic load |
| WASM SIMD for solver hot path | WASM solver path lands (per `adr/compute-budgets.md`) and SIMD measurably outperforms scalar |
| Multi-region anycast origin | Operator commits to multi-region infra; current single-region origin hits latency ceiling on real users |
| Edge-rendered RSC (per-region origin) | Same as above |
| SharedArrayBuffer + COOP/COEP | Worker `postMessage` serialization > 5ms p95 on hot-path data |

## Caught by

- Bundle-size CI gate + per-PR diff comment
- Lighthouse-CI gate (LCP / INP / CLS / TBT / TTFB)
- Playwright performance trace + DOM node count
- Heap-snapshot diff smoke + `memlab` leak detection
- r3f-perf overlay in dev (locked threshold, fails dev build if breached)
- Draw-call budget assertion in dev
- Shader-compile probe smoke (AOT verified)
- Sim-engine micro-benchmark CI
- HTTP/3 negotiation smoke
- 104 Early Hints emission smoke
- CDN cache hit ratio weekly review
- Long-task RUM aggregate (zero > 50ms during interaction)
- Worker-warmup smoke (workers responsive within 5ms of first message)
