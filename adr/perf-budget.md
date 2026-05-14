# perf-budget

## Decision

Bundle-size + Lighthouse-CI + frame-budget + heap-leak gates. Per-route budgets from `PERFORMANCE.md`.

## Bundle-size CI gate

`bundlesize2` or equivalent. Config file enumerates each route's gzip ceiling:

```json
[
  { "path": ".next/static/chunks/pages/_app-*.js", "maxSize": "120kb" },
  { "path": ".next/static/chunks/datapath-*.js", "maxSize": "500kb" },
  { "path": ".next/static/chunks/kmap-*.js", "maxSize": "500kb" },
  { "path": ".next/static/chunks/pipeline-*.js", "maxSize": "400kb" },
  { "path": ".next/static/chunks/editor-*.js", "maxSize": "600kb" },
  { "path": ".next/static/css/*.css", "maxSize": "30kb" }
]
```

## Lighthouse-CI gate

`@lhci/cli` runs against deployed staging URL per push:

```json
{
  "ci": {
    "collect": { "url": ["https://staging.<domain>", "https://staging.<domain>/datapath", "https://staging.<domain>/kmap"] },
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.95 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 1500 }],
        "cumulative-layout-shift": ["error", { "maxNumericValue": 0.02 }],
        "total-blocking-time": ["error", { "maxNumericValue": 150 }],
        "speed-index": ["error", { "maxNumericValue": 2000 }]
      }
    }
  }
}
```

## Frame-budget gate

Playwright trace + `r3f-perf`:
- Record 10s of sim at 60fps target
- Assert: zero dropped frames, p99 frame time < 16.67ms
- Assert: zero long tasks > 50ms during interactive sim

## Heap-leak gate

Custom script:
1. Boot Playwright page, navigate to `/datapath`
2. Run a full sim cycle, capture heap snapshot
3. Reset, run identical cycle again, capture second snapshot
4. Diff: zero detached DOM nodes, zero growing closures, ≤ 10% heap growth

Leak detected → CI fails with snapshot diff attached.

## CI matrix

Run on every push to main + every PR. Caching aggressive (Lighthouse-CI tokens, Playwright browser cache).

## Per-route enforcement points

| Route | Enforcement |
|---|---|
| `/` landing | Lighthouse LCP < 1500ms |
| `/datapath` | Lighthouse + frame budget + bundle |
| `/kmap` | Lighthouse + frame budget (3D mode) + bundle |
| `/pipeline` | Lighthouse + bundle |
| `/compare` | Frame budget × 2 (two scenes) |
| `/s/[hash]` | LCP < 1800ms cold |
| `/learn/*` | LCP < 1200ms (mostly RSC) |

## Caught by

- bundle-size CI gate
- Lighthouse-CI gate
- Playwright frame-budget trace gate
- Heap-leak diff gate
- All wired in `tools/perf/*` and exercised on every push
