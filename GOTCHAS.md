# GOTCHAS

Captured at every milestone per `book/PHILOSOPHY.md` "Capture gotchas at every milestone". Each entry: what surprised, why, fix, caught-by.

## Format

```
### <topic kebab-case>

What surprised: <observation>
Why: <mechanism>
Fix: <runnable next step>
Caught by: <verifier target>
```

## Entries

### webgpu-webxr-incompatible

What surprised: `WebGPURenderer` cannot drive a WebXR session.
Why: three.js `XRManager` is WebGL2-only (issues #28968 / #30806). Browser-side WebGPU+WebXR bindings only behind Chromium canary flags.
Fix: substrate `Renderer.create({ xrRequested })` returns `WebGLRenderer` when XR session requested. Decide at mount; renderer swap mid-session is infeasible.
Caught by: foundation-demo XR smoke + renderer-detection unit test (XR-requested mock yields WebGL path).

### postprocessing-v6-webgl2-only

What surprised: `postprocessing` v6 (pmndrs/vanruesc) only supports WebGL2, not WebGPU.
Why: v6 predates TSL/WebGPU stabilization. v7 beta has partial WebGPU but API churn.
Fix: per-renderer chain — `WebGPURenderer` uses three's native TSL `PostProcessing` pipeline; `WebGLRenderer` uses `postprocessing` v6. Substrate dispatches.
Caught by: post chain test per renderer.

### opfs-shader-binary-cache-impossible

What surprised: Browsers don't expose compiled shader binaries to JS; OPFS-cached compiled shader binaries cannot be implemented.
Why: WebGL doesn't expose program binaries. WebGPU has no `getCachedPipeline` API. Persistent cache is browser-internal only.
Fix: rely on `Material.compileAsync` + browser-internal pipeline cache. Persistent cache only for TSL source.
Caught by: spec — drop OPFS-shader-binary item from PERFORMANCE.md.

### webgpu-no-multi-queue

What surprised: WebGPU `device.queue` is singular; "multi-queue" model doesn't exist in user-facing API.
Why: spec issue #1065 still open; multi-queue tracked but no shipping date.
Fix: encode compute + render passes in one `commandEncoder`; submit once. GPU does dependency tracking — compute overlaps render on capable hardware without explicit queue management.
Caught by: spec — drop "WebGPU 3-queue model" item from CONCURRENCY.md.

### drei-instances-perf-regression-at-scale

What surprised: drei `<Instances>` measurably slower than raw `InstancedMesh` above ~128 nodes.
Why: reconciler overhead per `<Instance>` JSX node accumulates.
Fix: use raw `<instancedMesh>` + `setMatrixAt` + `instanceColor` for K-map dynamic recoloring + any high-count scenario. `<Instances>` only below ~128 nodes for ergonomic declarative use.
Caught by: `tools/lint/instance-count-discipline.ts` (when written) — high-count instance JSX flagged.

### three-lod-no-hysteresis

What surprised: `THREE.LOD` thrashes at level boundaries with jittery cameras.
Why: no built-in deadband; switches at exact threshold.
Fix: hand-rolled `useFrame` with 10% deadband around each threshold per `adr/three-stack.md`.
Caught by: LOD smoke test with jittering camera should see no >1 level swap per second.

### r3f-perf-stale

What surprised: `r3f-perf` last released v7.2.3 ~1 year ago, no WebGPU timer-query support.
Why: maintenance lapsed.
Fix: use `stats-gl` (Renaud Rohlinger) — drop-in R3F component with WebGPU timer queries.
Caught by: STACK pick + version-pin.

### react-three-a11y-stale

What surprised: `@react-three/a11y` last released v3.0.0 in 2021. May not support React 19.
Why: maintenance lapsed.
Fix: hand-roll DOM proxies (<100 LOC) inside `apps/web/features/*/a11y/`. Sync state bidirectionally.
Caught by: a11y smoke test.

### forced-colors-canvas-blind

What surprised: `@media (forced-colors: active)` does not penetrate canvas — WebGL/WebGPU ignores forced colors entirely.
Why: by design; canvas is opaque to forced-colors.
Fix: render 2D SVG fallback for datapath under `forced-colors: active` using `currentColor` + system color keywords.
Caught by: `a11y.forced-colors` Playwright test with `forcedColors: 'active'` emulation.

### gpu-pixel-not-cross-machine

What surprised: WebGL/WebGPU pixel output not bit-identical across vendors (Apple/Intel/AMD/NVIDIA).
Why: hardware rasterization differs by GPU + driver + OS.
Fix: per-platform Playwright snapshot baselines (auto-suffixed). Pair pixel tests with structural assertions. Use SwiftShader for deterministic CPU-rendered trace baselines.
Caught by: visual regression per platform + structural scene-graph hash.

### bun-zstd-not-cross-version-deterministic

What surprised: `Bun.zstdCompress` output varies across Bun versions / level / window-log.
Why: Bun bumps embedded zstd library.
Fix: hash uncompressed canonical bytes, not compressed. Use `CompressionStream("deflate-raw")` for permalink transport (universal, no header variance). Reserve `Bun.zstdCompress` for at-rest storage with explicit version pinning.
Caught by: cross-machine hash stability fixture test.

### safe-stable-stringify-set-map-silent-drop

What surprised: `safe-stable-stringify` silently drops `Set` and `Map` values (stringifies as `{}`).
Why: JSON has no Set/Map equivalent.
Fix: forbid Set/Map in canonical-path Zod schemas via `.strict()`. Pre-normalize to sorted arrays / plain objects.
Caught by: codec round-trip property test + Zod boundary validation.

### bigint-lossy-canonicalize

What surprised: `safe-stable-stringify` default coerces `BigInt` to `Number` (lossy).
Why: JSON has no BigInt.
Fix: use `{ bigint: 'string' }` option or pre-encode as `{ $bigint: "123" }` envelopes. For MIPS register values use `Int32Array` typed state to avoid BigInt entirely.
Caught by: codec stability test on fixtures containing large integers.

### react-three-offscreen-html-incompatible

What surprised: `drei/Html`, `drei/View`, `drei/Hud` don't work inside OffscreenCanvas worker.
Why: they require real DOM access.
Fix: render K-map as DOM/SVG above transparent worker canvas, OR keep K-map on main-thread R3F root with only the MIPS datapath in worker. Recommend the former — simpler.
Caught by: design — K-map UI is DOM/SVG; MIPS datapath is R3F-in-worker.

### abortsignal-not-structured-cloneable

What surprised: `AbortSignal` is not structured-cloneable across workers as of 2026.
Why: WHATWG proposal (dom #948) still open.
Fix: send `{ kind: 'abort', id }` message; worker maintains `Map<id, AbortController>` and aborts locally. For sync CPU loops, check generation counter periodically.
Caught by: worker cancellation smoke (abort mid-solve returns early).

### speculation-rules-prerender-double-counts-analytics

What surprised: prerendered pages execute JS, so analytics events fire BEFORE user navigates.
Why: prerender literally pre-runs the page.
Fix: gate every analytics call on `document.prerendering === false`; listen for `prerenderingchange` to flush queued events on activation.
Caught by: Plausible event smoke under prerender mock.

### http3-0rtt-not-for-post

What surprised: HTTP/3 0-RTT cannot safely carry POST/mutation requests.
Why: Chrome 145+ surfaces 425 Too Early to JS instead of transparent retry; replay attacks possible.
Fix: 0-RTT GET/HEAD/OPTIONS only. Configure Caddy to disable 0-RTT for unsafe methods.
Caught by: server config audit + smoke that POST never gets 0-RTT.

### server-timing-needs-tao

What surprised: cross-origin `Server-Timing` and resource timing phases are invisible to JS without `Timing-Allow-Origin`.
Why: privacy gate.
Fix: emit `Timing-Allow-Origin: *` on all Convex + CDN responses.
Caught by: CORS header smoke.

### http-status-103-not-104

What surprised: Early Hints HTTP status is 103, not 104.
Why: spec.
Fix: corrected across all docs.
Caught by: curl smoke asserting 103 response.

### next-pwa-dead-use-serwist

What surprised: `next-pwa` last meaningful release was 2022.
Why: maintenance lapsed.
Fix: use `serwist` (`@serwist/next`) — Workbox-spirit, first-class App Router + Next 16 + TypeScript service workers + revision-keyed precache.
Caught by: STACK pick + version-pin + service worker smoke.

### bundlesize2-unmaintained-use-size-limit

What surprised: `bundlesize2` last release 2022.
Why: maintenance lapsed.
Fix: use `size-limit` + `andresz1/size-limit-action` for PR comments + `hashicorp/nextjs-bundle-analysis` for per-route diff.
Caught by: bundle-size CI gate.

### longtask-superseded-by-loaf

What surprised: Long Animation Frames API (LoAF) shipped Chrome 123 and supersedes longtask for RUM.
Why: LoAF gives script attribution + render time + blocking durations — strictly more useful.
Fix: `PerformanceObserver({ type: 'long-animation-frame' })` with longtask fallback for older browsers.
Caught by: RUM aggregate dashboard.

### inp-16ms-p75-unrealistic

What surprised: INP ≤ 16ms at p75 is unrealistic across compositor-blocking interactions (dropdown open with paint, etc.).
Why: a single ≥16ms paint inside a CrUX session torpedoes p75 quickly.
Fix: tiered target — 16ms median / 50ms p75 / 100ms p95.
Caught by: web-vitals v4+ RUM with INP attribution + LoAF.

### detect-gpu-db-stale

What surprised: `detect-gpu` benchmark DB is ~12 months stale as of May 2026.
Why: maintainer cadence.
Fix: augment with `hardwareConcurrency` + `deviceMemory` + user override toggle. Render preview first, upgrade async — never gate first paint.
Caught by: device-tier unit test against synthetic navigator mocks.

### framer-motion-3d-dead

What surprised: `framer-motion-3d` is abandoned. Rebrand to `motion` did not bring official three integration.
Why: motion team didn't carry it forward.
Fix: use `motion`'s imperative `animate()` on three properties (works because Vector3 is `{x,y,z}` to motion). Hand-rolled `damp3` critically-damped springs in `useFrame` for continuous tracking. `MathUtils.damp3` is the built-in helper.
Caught by: STACK pick + animation smoke.

### drei-environment-preset-cdn-fetch

What surprised: drei `Environment preset="city"` fetches HDRI from CDN.
Why: lazy-loading; docs explicitly say not for production.
Fix: use drei-bundled preset (`'studio'` for industrial) which is allowed per `adr/asset-authoring.md`, OR self-host higher-res HDRI lazy-loaded after first paint.
Caught by: network audit — no CDN HDRI requests at runtime.

### meshtransmissionmaterial-webgpu-edge-cases

What surprised: drei `MeshTransmissionMaterial` works under WebGPU but with documented edge cases (extra render targets, depth/refraction issues per forum #88608).
Why: WebGPU + transmission render-target stacks not fully validated.
Fix: have `MeshPhysicalMaterial` with `transmission: 1` as plan-B fallback. Adaptive quality switches to fallback on low-tier devices.
Caught by: visual regression of glass material + per-renderer baselines.

### intl-formatting-locale-variance

What surprised: `toLocaleString` / `Intl.NumberFormat.format` output varies by browser locale.
Why: by design.
Fix: use explicit format strings in canonical paths. Domain-language formatting only.
Caught by: cross-machine hash stability test.

### intl-collator-iteration-order

What surprised: `Intl.Collator` sort order varies by locale; broke cross-machine hash test in another project.
Why: locale-aware.
Fix: use plain `<` on UTF-16 code units in canonical sort paths.
Caught by: `tools/lint/no-determinism-leak.ts` greps for `Intl.Collator` in `packages/sim-engine`.

### crypto-randomuuid-non-deterministic-permalink

What surprised: `crypto.randomUUID()` for permalink IDs breaks content-addressing.
Why: random by definition.
Fix: derive IDs from `blake3(parent || index)` so permalinks are content-addressed end-to-end.
Caught by: permalink stability test.

### playwright-linux-fonts-vary

What surprised: Playwright 2026 on Linux runners ships different fontconfig than macOS — font rendering differs in canvas screenshots.
Why: distro fonts vary.
Fix: mask text overlays in canvas screenshots OR bake SDF text into MSDF geometry. Use cross-platform font subsets.
Caught by: per-platform visual regression baselines + font subset CI.

### react-strict-mode-r3f-double-mount

What surprised: R3F v9 + React 19 StrictMode now inherits across nested renderers — previously hidden double-mount side-effects surface.
Why: StrictMode behavior in concurrent renderer.
Fix: audit `useEffect` touching `gl.domElement` or non-disposable globals; ensure cleanup is idempotent.
Caught by: dev-mode smoke under StrictMode.

### threejs-r170-srgb-explicit

What surprised: Three.js r170+ requires explicit `texture.colorSpace = SRGBColorSpace` on custom material paths.
Why: r155 changed lighting model + r170 tightened color management.
Fix: set explicitly in `packages/three-kit` material factories.
Caught by: visual regression on PBR materials.

### tsl-color-space-finicky-on-texture-loads

What surprised: TSL color-space handling on custom texture loads is inconsistent (three.js #30467).
Why: pipeline still maturing.
Fix: pure-procedural materials per `adr/asset-authoring.md` — no external textures, so issue avoided.
Caught by: design — no texture loads.

### voiceover-application-role-broken

What surprised: `role="application"` on canvas breaks VoiceOver iOS rotor + TalkBack.
Why: mobile SR handle application role poorly per Marco Zehe / Bogdan Cerovac 2024.
Fix: never `role="application"`. DOM proxies are buttons/gridcells in `role="region"`. K-map uses `role="grid"` + `gridcell` children.
Caught by: a11y smoke per SR + axe-core (won't catch SR issues directly, only structural).

### convex-optimistic-vs-useoptimistic

What surprised: mixing `useOptimistic` (React 19) with Convex `withOptimisticUpdate` causes double-flicker.
Why: both fire optimistic state.
Fix: use Convex's `withOptimisticUpdate` for Convex mutations exclusively; reserve `useOptimistic` for non-Convex Server Actions only.
Caught by: optimistic update smoke (single state transition, no flicker).

### scheduler-posttask-safari-pending

What surprised: `scheduler.postTask` priorities not in Safari 2026.
Why: Safari 18.4 ships `yield` but not `postTask` priorities.
Fix: feature-detect + MessageChannel-microtask fallback chain.
Caught by: scheduler smoke on Safari.

### background-sync-chromium-only

What surprised: Background Sync API only in Chromium.
Why: Firefox/Safari haven't shipped.
Fix: primary offline-replay path is `online` event + boot-time replay. SW background-sync is bonus when available.
Caught by: offline replay smoke on Firefox/Safari simulators.

### avp-no-ar-no-joint-hand-skeleton

What surprised: Apple Vision Pro visionOS 2 (2026) is VR-only (no AR module); no exposed 25-joint hand skeleton.
Why: by design — privacy + Natural Input model (gaze + pinch).
Fix: design VR-only path. Gaze + pinch must be first-class input; never gate UX on AR module or pose-recognition skeleton.
Caught by: AVP target smoke (when WebXR trigger fires).

### convex-self-host-blake3-not-builtin

What surprised: Convex runtime (V8-based, not Bun) doesn't have `Bun.CryptoHasher`, and `SubtleCrypto` doesn't support blake3.
Why: runtime differences.
Fix: `@noble/hashes/blake3` is the canonical lib — works in Convex runtime, Bun runtime, and browser.
Caught by: cross-runtime hash test.

### reduced-motion-in-snapshot-schema

What surprised: replay determinism requires reduced-motion flag in snapshot — recording on non-reduced-motion device played back on reduced-motion produces different trace.
Why: keyframes branch on reduced-motion.
Fix: include `reducedMotion: boolean` in snapshot schema; replays respect recorded value.
Caught by: replay determinism test under both motion modes.
