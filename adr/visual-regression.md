# visual-regression

## Decision

Playwright snapshot baselines per known fixture, **auto-suffixed by platform** (`name-darwin.png`, `name-linux.png`, `name-win.png`). GPU bit-identity is unattainable across vendors — pixel hash is per-renderer.

## Coverage

| Surface | Baselines |
|---|---|
| Datapath scene per locked instruction × per step | 10 instructions × 5 steps = 50 baselines × 3 platforms |
| K-map 2D per representative truth table | ~20 baselines (covers example exercises) × 3 platforms |
| K-map 3D toroidal per representative 5/6-var function | ~10 baselines × 3 platforms |
| Pipeline diagram per representative program | ~5 baselines × 3 platforms |
| Compare mode per documented instruction pair | ~5 baselines × 3 platforms |
| Learn page hero render | per published learn page × 3 platforms |
| OG image per fixture snapshot | per OG variant (1 platform — server-rendered deterministic via `ImageResponse`) |
| Foundation demo render per substrate package | 7 baselines × 3 platforms |

## Strategy

Multi-layer:
1. **Per-platform pixel baselines** for prod-shipping routes. Playwright auto-suffixes. Commit darwin / linux / win versions.
2. **SwiftShader-only single baseline** for trace tests (deterministic CPU-rendered output). Runs in Docker with `--use-gl=swiftshader` on Linux CI. Fastest signal but doesn't catch GPU-only bugs.
3. **Structural assertions paired** with pixel: scene-graph hash, renderer.info draw-call count, mesh-position hash. A green pixel test is never the only signal.

## Determinism prerequisites

Snapshots only stable when rendering is deterministic:
- Wall-clock-free animation (per `DETERMINISM.md`)
- Frame-stepped capture via `frameloop="never"` in test mode + manual `gl.render(scene, camera)` after each `simStore.goto(stepIndex)`
- Camera frozen (no OrbitControls)
- `dpr={1}` fixed (no devicePixelRatio variance)
- `antialias: false` on screenshot path
- Fonts: 2026 Playwright on Linux runners ships different fontconfig vs macOS — mask all text overlays in canvas screenshots OR bake SDF text into MSDF geometry
- `WebGLRenderer.outputColorSpace = THREE.SRGBColorSpace` explicitly

## Test driver pattern

```ts
// app code, exposed only under window.__test
window.__sim = {
  step(n=1) { for (let i=0;i<n;i++) simStore.getState().stepTick(); invalidate(); },
  goto(stepIndex) { simStore.getState().replayTo(stepIndex); invalidate(); },
};

// playwright
await page.goto('/sim?permalink=' + hash + '&paused=1');
await page.evaluate(() => window.__sim.goto(42));
await page.waitForFunction(() => window.__sim.renderedStep === 42);
await expect(page.locator('canvas')).toHaveScreenshot('step-42.png');
```

Screenshot of step N is purely a function of (program, schema, GPU).

## Playwright config

```ts
expect: {
  toHaveScreenshot: {
    threshold: 0.01,          // per-pixel color delta
    maxDiffPixelRatio: 0.005, // 0.5% pixels allowed to differ
    animations: 'disabled',
    caret: 'hide',
    scale: 'css',
  },
},
use: { deviceScaleFactor: 1, viewport: { width: 1280, height: 720 } },
```

3D scene fixtures override with `maxDiffPixelRatio: 0.02-0.03` for AA-edge tolerance.

## Update policy

Visual diff failure → developer reviews diff image → if intentional, runs `make visual.update` to refresh baselines + commits.

No silent baseline updates. `--update-snapshots` flag requires `UPDATE_SNAPSHOTS=1` env var so it's never accidental in PRs. Every refresh has a commit message explaining the visual change.

## CI matrix

Three platforms × per-fixture. ubuntu-24.04-x86 (Linux + SwiftShader trace baseline), macos-15-arm64 (darwin pixel baselines), windows-2022 (win pixel baselines).

| Gate | Threshold |
|---|---|
| `visual.datapath` | per-platform 0.5% pixel-diff tolerance |
| `visual.kmap` | per-platform same |
| `visual.pipeline` | per-platform same |
| `visual.compare` | per-platform same |
| `visual.learn` | per-platform same |
| `visual.og` | single baseline (server-rendered deterministic) |
| `visual.foundation` | per-platform same |
| `visual.trace` | SwiftShader single baseline (CPU-deterministic) |
| `structural.scene-graph` | scene-graph hash equality cross-platform |
| `structural.draw-calls` | `renderer.info.render.calls` matches expected per fixture |

## Caught by

- Playwright `toHaveScreenshot()` per fixture per platform
- SwiftShader trace runs in Docker on Linux CI for cross-platform deterministic baseline
- Structural assertion suite per fixture (scene-graph + draw-call count)
- CI fails on any unmatched baseline
- `make visual.update` regenerates baselines under operator control with UPDATE_SNAPSHOTS=1 guard
