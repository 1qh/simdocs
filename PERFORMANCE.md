# PERFORMANCE

Explicit performance contract for the product. Every budget is a CI gate; violation fails the build.

## User-perceived budgets

| Metric | Budget | Measured at |
|---|---|---|
| LCP (Largest Contentful Paint) | ≤ 1.5s on cable, ≤ 2.5s on slow 4G | Real-user (RUM) + Lighthouse-CI synthetic |
| INP (Interaction to Next Paint) | ≤ 100ms on desktop, ≤ 200ms on mid-tier laptop | RUM + Playwright trace |
| CLS (Cumulative Layout Shift) | ≤ 0.02 | RUM + Lighthouse-CI |
| TTI (Time to Interactive) | ≤ 2.5s on cable | Lighthouse-CI |
| 3D scene frame budget | ≤ 16.67ms (60 fps locked) | Stats overlay + Playwright trace |
| Step-animation transition | ≤ 400ms total, no dropped frames | Playwright trace |

## Asset budgets

| Asset class | Budget | Enforcement |
|---|---|---|
| Initial bundle (non-editor routes) | ≤ 200 KB gzip | bundle-size CI gate |
| Editor-route bundle | ≤ 600 KB gzip (Monaco dynamic-imported) | bundle-size CI gate |
| 3D-route bundle | ≤ 500 KB gzip (three + drei + product features) | bundle-size CI gate |
| Per-request JSON payload | ≤ 4 KB gzip (snapshot reads) | Route Handler smoke |
| Font subset | Latin-only by default, full Unicode lazy | Font loading hook |
| Image / texture | All procedural; zero external textures per `adr/asset-authoring.md` | Repo-asset lint |

## Runtime memory

| Resource | Ceiling |
|---|---|
| JS heap (steady state, after one full sim cycle) | ≤ 150 MB |
| GPU memory (R3F context) | ≤ 250 MB |
| Live event listeners | ≤ 500 |
| Open `requestAnimationFrame` callbacks | ≤ 5 unique sources |

Memory leak detection via Playwright `page.coverage` + heap snapshot diff between identical sim cycles.

## Cold-start vs warm-cache

| Path | Cold | Warm |
|---|---|---|
| Landing route | ≤ 1.5s LCP | ≤ 300ms LCP (CF edge cached) |
| `/s/[hash]` shared permalink | ≤ 1.8s LCP (one origin hop) | ≤ 250ms LCP (CF edge cached) |
| `/datapath` interactive | ≤ 2.5s TTI | ≤ 1s TTI |
| `/kmap` interactive | ≤ 2.0s TTI | ≤ 800ms TTI |

## CI gates

| Gate | Tool | Threshold |
|---|---|---|
| Bundle-size budget | `bundlesize2` or equivalent | Per-route budget above |
| Lighthouse-CI synthetic | `@lhci/cli` | LCP, CLS, TBT per Vercel-best-practices |
| Playwright trace | Playwright `--trace on` | INP, dropped frame count |
| Heap snapshot diff | Custom script | Leak detected = fail |
| 3D frame timing | `r3f-perf` + Playwright | 60 fps locked over 10s sample |

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
- Texture KTX2 + zstd for any future raster (currently none per asset-authoring ADR)
- HTTP/2 + Brotli compression at Caddy

## Anti-patterns banned

- Per-frame `setState` in `useFrame` (banned — mutate refs)
- New object props on R3F components in render (banned — `useMemo` or stable refs)
- Synchronous JSON parsing > 100ms on main thread (banned — Web Worker)
- Layout thrash (banned — batch DOM reads/writes)
- Render-blocking third-party scripts (banned — none allowed)

## Caught by

- Bundle-size CI gate
- Lighthouse-CI gate
- Playwright performance trace
- Heap-snapshot diff smoke
- r3f-perf overlay in dev (locked threshold, fails dev build if breached)
