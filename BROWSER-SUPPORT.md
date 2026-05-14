# BROWSER-SUPPORT

Support matrix. Latest-only per `book/PHILOSOPHY.md` — every supported browser tracks current stable. Older versions get a polite "browser too old" page.

## Support matrix

| Browser | Floor version | Notes |
|---|---|---|
| Chrome | latest stable | WebGPU primary path enabled |
| Edge | latest stable | Chromium-based, same as Chrome |
| Firefox | latest stable | WebGL fallback (WebGPU still experimental, gated) |
| Safari macOS | latest stable | WebGPU primary (Safari 17+); WebGL fallback for earlier |
| Safari iOS | latest stable | Read-only mobile experience per `adr/mobile-read-only.md` |
| Chrome Android | latest stable | Read-only mobile experience |

Anything older: served a minimal HTML page with one line — "this app requires a current browser". No fallback engineered.

## Floor capabilities required

- ES2022 syntax (top-level await, error.cause, etc.)
- WebGL 2 (mandatory)
- WebGPU (preferred, optional)
- `BigInt` (used in some 64-bit-shaped computations)
- `structuredClone` (used in sim state copying)
- `Web Crypto API` (for blake3 in `packages/sim-engine`)
- `Compression Streams API` (for zstd alternative — fallback to wasm if absent)
- IndexedDB (for offline save queue + localStorage backup)
- Service Workers (for `adr/offline-pwa.md`)
- Web Workers (for solver computations off main thread)

## Capability detection

`packages/three-kit` exposes:

```ts
export const capabilities = {
  webgpu: () => /* runtime detection */,
  webgl2: () => /* runtime detection */,
  // ...
};
```

Per-feature consumers query capabilities, never user-agent strings. Per `book/HARD-RULES.md` "Mechanism-asserted, not call-site-asserted".

## Stress-test bounds

These define the maximum input sizes the product handles gracefully. Beyond these, fail loud with a typed error per `ERROR-CATALOG.md`.

| Resource | Limit |
|---|---|
| MIPS program lines | 10,000 |
| Encoded instructions | 40,000 (≈160KB of bytecode) |
| Data memory words | 4,096 |
| K-map variables | 6 (geometry); 12 (solver-only via Espresso) |
| K-map minterms | 2^6 = 64 (geometry); 2^12 = 4,096 (Espresso) |
| Groupings per K-map | 64 user-defined |
| Pipeline trace cycles | 1,000 (UI renders, scroll for more) |
| Pipeline trace instructions | 100 (UI renders compact) |
| Snapshot body uncompressed | 256 KB |
| Snapshot body compressed (tier-2) | 64 KB hard cap |
| Anonymous saves in localStorage | 100 (LRU eviction beyond) |

## Polyfills

- `core-js` for any missing ES2022 feature (rare, mostly Edge edge cases)
- `BigInt` polyfill not needed (all supported browsers have it)
- `structuredClone` polyfill not needed (all supported browsers have it)
- Compression Streams API: fallback to `fzstd` wasm if missing (Safari < 17)

## Caught by

- BrowserStack or Playwright cross-browser test matrix on PR + nightly
- Capability-detection unit tests
- Stress-test bounds enforced in `packages/sim-engine` + `apps/web/features/*/core/`; over-limit input throws typed error
