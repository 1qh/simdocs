# adaptive-quality

## Decision

3D scene quality adapts to runtime FPS + boot-time device tier. High-end devices get full detail; mid-tier downgrades automatically; low-tier renders read-only static. User can override.

## Device tier detection at boot

```ts
const tier = (() => {
  const cores = navigator.hardwareConcurrency ?? 4;
  const memory = (navigator as any).deviceMemory ?? 4;  // GB
  const gpuHint = detectGpuTier();  // via `detect-gpu` package, locked per `adr/oss-import-audit.md`
  if (cores >= 8 && memory >= 8 && gpuHint >= 2) return 'high';
  if (cores >= 4 && memory >= 4 && gpuHint >= 1) return 'mid';
  return 'low';
})();
```

| Tier | Default render |
|---|---|
| `high` | Full LOD, full postprocessing chain, 120fps target on capable display |
| `mid` | LOD reduced one step, postprocessing chain trimmed (SSAO off, bloom radius reduced), 60fps target |
| `low` | Read-only static rendering via `adr/mobile-read-only.md` fallback, no interactive 3D |

User-override toggle exposed in settings (force `high` / `mid` / `low`).

## Runtime adaptive quality via drei `PerformanceMonitor`

```tsx
<PerformanceMonitor
  bounds={(refreshrate) => [60, refreshrate]}
  flipflops={3}
  onDecline={() => quality.degrade()}
  onIncline={() => quality.restore()}
/>
```

When FPS sustains below floor (60 for high/mid tier, display refresh for high-refresh) for 3 frames:
1. Pixel ratio drops one step (1.5 → 1.0)
2. LOD swaps to lower detail
3. Postprocessing intensity reduced (bloom + SSAO step down)
4. Particles + emissive trace density reduced

When FPS sustains above floor for 3 frames:
1. Reverse the degradation in step order

Adaptive state held in zustand store; consumed by every scene component.

## What stays fixed regardless of tier

- Locked datapath topology (every component visible, every signal traceable)
- Locked K-map geometry (2D/3D toroidal as appropriate)
- Pedagogically essential animations (step transitions, signal pulses)
- A11y surfaces (DOM proxies, keyboard nav, focus rings)
- Text labels + diegetic readouts (etched labels stay readable)

Adaptive quality affects polish, not pedagogy.

## What can downgrade

- Pixel ratio (1x ↔ 2x)
- Material complexity (PBR with clearcoat + anisotropy → basic PBR → flat shaded)
- Postprocessing chain depth (bloom + SSAO + DoF + chromatic ab → bloom only → none)
- Particle density
- LOD tier (full → simplified → far-view)
- Shadow quality (soft → hard → off)
- Emissive trace pulse intensity (full → simplified → static)

## Boot decision UX

First visit on a low-tier device:
- Mount detection runs at app boot, before scene loads
- If `low` detected, mount read-only experience + show toast: "your device is in compact mode — change in settings"
- If `mid`, mount interactive with reduced default — no toast
- If `high`, mount full detail

## Caught by

- Device-tier detection unit test against synthetic `navigator` mocks
- `PerformanceMonitor` integration test — simulated FPS drop triggers degradation
- Recovery test — FPS restore triggers upgrade
- A11y test under each tier — keyboard nav + screen reader work identically
- Visual regression baselines captured per tier
