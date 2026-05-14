# perf-budget

## Decision

Bundle-size + Lighthouse-CI + frame-budget + heap-leak gates. Per-route budgets from `PERFORMANCE.md`.

## Bundle-size CI gate

`size-limit` + `andresz1/size-limit-action` for PR comment (bundlesize2 is unmaintained as of 2022). Config:

```js
// .size-limit.cjs
module.exports = [
  { name: "initial",     path: ".next/static/chunks/main-*.js",                limit: "150 KB", gzip: true },
  { name: "datapath",    path: ".next/static/chunks/app/datapath/page-*.js",   limit: "400 KB", gzip: true },
  { name: "kmap",        path: ".next/static/chunks/app/kmap/page-*.js",       limit: "400 KB", gzip: true },
  { name: "pipeline",    path: ".next/static/chunks/app/pipeline/page-*.js",   limit: "400 KB", gzip: true },
  { name: "editor",      path: ".next/static/chunks/app/editor/page-*.js",     limit: "500 KB", gzip: true },
  { name: "css",         path: ".next/static/css/*.css",                       limit: "25 KB",  gzip: true },
];
```

Complement: `hashicorp/nextjs-bundle-analysis` for per-PR per-route diff comments.

## Lighthouse-CI gate

`@lhci/cli` (Lighthouse 12.6+, LHCI 0.15.x) runs against deployed staging URL per push:

```js
// lighthouserc.cjs
module.exports = {
  ci: {
    collect: {
      startServerCommand: "bun run start",
      url: ["http://localhost:3000/", "http://localhost:3000/editor", "http://localhost:3000/sim"],
      numberOfRuns: 5,
      settings: { preset: "desktop", throttlingMethod: "simulate" },
    },
    assert: {
      assertMatrix: [
        { matchingUrlPattern: "/sim", assertions: {
            "largest-contentful-paint":      ["error", { maxNumericValue: 1500, aggregationMethod: "median" }],
            "total-blocking-time":           ["error", { maxNumericValue: 100 }],
            "cumulative-layout-shift":       ["error", { maxNumericValue: 0.01 }],
            "resource-summary:script:size":  ["error", { maxNumericValue: 400_000 }],
        }},
      ],
    },
    upload: { target: "temporary-public-storage" },
  },
};
```

`numberOfRuns: 5` + `aggregationMethod: "median"` for false-positive reduction. Exclude variability-heavy metrics (`speed-index`) from `error` tier.

## Frame-budget gate

Playwright trace + `stats-gl` (r3f-perf stale):
- Record 10s of sim at locked framerate target per display tier
- Assert: zero dropped frames, p99 frame time < target_ms (16.67 / 8.33 / 6.94 / 4.16 / 2.77 per 60/120/144/240/360 Hz)
- Assert: zero LoAF entries > 50ms during interactive sim (longtask fallback for older browsers)

## INP measurement via web-vitals + LoAF

```ts
await page.addInitScript(() => {
  import("https://unpkg.com/web-vitals?module").then(({ onINP }) => {
    (globalThis as any).__inp = [];
    onINP((m) => (globalThis as any).__inp.push(m), { reportAllChanges: true });
  });
});
// drive interactions, then:
const inp = await page.evaluate(() => (globalThis as any).__inp);
// assert tiered: median ≤ 16ms, p75 ≤ 50ms, p95 ≤ 100ms
```

Mid-tier emulation: Playwright `--cpu-throttling-rate=4` + `--network-conditions=Slow 4G`.

## Heap-leak gate

Two-layer: custom snapshot diff (cheap, fast) + `memlab` (deep, runs nightly):

1. Boot Playwright page, navigate to `/datapath`
2. Run a full sim cycle, capture heap snapshot
3. Reset, run identical cycle again, capture second snapshot
4. Diff: zero detached DOM nodes, zero growing closures, ≤ 10% heap growth

`memlab` scenario file detects detached `HTMLCanvasElement`, retained `WebGLRenderer`, retained `BufferGeometry` across repeat cycles. Pair with `THREE.Cache.clear()` between repeats.

Three.js leak surface specifically: undisposed geometry/material/texture, EffectComposer/RenderTarget on resize, drei `useGLTF` cache disables disposal — call `useGLTF.clear()` explicitly on unmount.

## CI matrix

Run on every push to main + every PR. Caching aggressive (Lighthouse-CI tokens, Playwright browser cache).

## Per-route enforcement points

| Route | Enforcement |
|---|---|
| `/` landing | Lighthouse LCP < 1000ms cable / < 2000ms slow 4G |
| `/datapath` | Lighthouse + frame budget + bundle |
| `/kmap` | Lighthouse + frame budget (3D mode) + bundle |
| `/pipeline` | Lighthouse + bundle |
| `/compare` | Frame budget × 2 (two scenes) |
| `/s/[hash]` | LCP < 1500ms cold (one origin hop) / < 200ms warm (CF edge immutable) |
| `/learn/*` | LCP < 1000ms (mostly RSC) |

## Caught by

- size-limit CI gate + per-PR diff comment via nextjs-bundle-analysis
- Lighthouse-CI gate (LCP / INP / CLS / TBT / TTFB)
- Playwright frame-budget trace + DOM count
- Heap-leak diff + nightly memlab
- mitata micro-benchmarks per sim-engine critical path with regression gate ≥10%
- All wired in `tools/perf/*` and exercised on every push
