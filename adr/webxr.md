# webxr

## Decision

WebXR (VR) mode for 3D scenes — datapath + K-map toroidal — is scoped, with explicit trigger to land.

Default state: NOT in floor. Floor ships without VR.

Trigger to flip into floor:
- `@react-three/xr` v6+ stable on locked R3F v9.5 version
- User-visible demand signal (multiple feedback requests, target audience overlap)
- Operator decides to ratchet

When trigger fires: VR mode lands as a toggle in `/datapath` and `/kmap` 3D modes, using `@react-three/xr` v6 with `createXRStore` + `<XR store={store}>` pattern.

## Critical substrate constraint: WebGPU + WebXR NOT supported in 2026

three.js issues #28968, #30806 + Mugen87 forum confirmation: `XRManager` is WebGL2-only. WebGPURenderer cannot drive an XR session. Browser-side WebXR + WebGPU bindings only behind Chromium canary flags.

**Implication**: renderer swap mid-session is infeasible. Substrate `Renderer.create({ xrRequested })` returns `WebGLRenderer` when XR session is requested. App must decide at mount.

```ts
// substrate
const isXRRequested = await navigator.xr?.isSessionSupported('immersive-vr');
const gl = isXRRequested
  ? { renderer: new WebGLRenderer(props) }
  : { renderer: await initWebGPURenderer(props) };
```

This is the single biggest substrate constraint to encode now.

## What lands when trigger fires

- WebXR session entry button in 3D-route nav, hidden when WebGPU active (force re-mount on entry)
- `<XR store={store}>` wrapper around `<Canvas>` from `@react-three/xr` v6
- `createXRStore({ hand: { teleportPointer: true }, controller: { teleportPointer: true } })`
- Controller raycast → cell selection
- Hand tracking → grouping interaction (Quest 25-joint, AVP gaze+pinch only)
- Camera bookmarks become positional teleport targets (scale-to-grasp affordance for K-map torus)
- Subtle floor / ceiling materials so the user has spatial reference
- Per-session-mode postprocessing: bloom OK in VR (caveats), disable in AR (pmndrs/xr #78 — bloom in AR alpha-channel mixing broken)

## Target device matrix at trigger

| Device | VR | AR | Hands | Notes |
|---|---|---|---|---|
| Quest Browser (Horizon OS v39+) | ✓ | ✓ passthrough | ✓ 25-joint | Primary target; 90 Hz standalone (not 120 — that's PCVR Link) |
| Apple Vision Pro Safari (visionOS 2+) | ✓ default | ✗ no AR module | gaze+pinch (no joint skeleton publicly) | Design gaze-targeting as first-class input, not pose detection |
| Samsung Galaxy XR (Android XR, Chrome) | ✓ | ✓ | ✓ | 2026 entrant |
| Desktop Chrome + USB headset | ✓ | ✓ | per device | OpenXR path |
| Firefox | ✗ in practice | ✗ | — | Effectively dropped |

## Postprocessing in XR

- `pmndrs/postprocessing` supports stereo for most effects post-three.js #18846
- **Safe in XR**: Bloom (selective), tone-mapping, vignette (sparingly — can induce nausea)
- **Break or expensive in XR**: SSAO (per-eye depth cost), DoF (gaze conflict), chromatic ab (already optical)
- **Disable in AR**: bloom alpha mixing with passthrough broken per `pmndrs/xr` #78

Substrate rule: wrap `<EffectComposer>` in `<XRConditional>` and pass a reduced effect set when `session.mode !== undefined`.

## What stays out even when trigger fires

- Multi-user VR collaboration (massive scope creep)
- Custom controller gestures beyond standard `select` / `squeeze`
- AR mode (different design considerations + AVP doesn't support it)
- 120 Hz standalone budget (Quest standalone is 90 Hz; 120 Hz only on PCVR Link)
- Joint-pose-recognition-only UX (AVP lacks hand skeleton — gaze+pinch must be first-class)

## VR pedagogy patterns (research 2025-2026)

- **Seated + teleport** beats smooth locomotion for educational viz (cognitive fatigue after ~45 min)
- **Camera bookmarks → teleport targets** is the recommended pattern
- For K-map torus: inside-the-torus viewpoint is high-value novelty — combine with scale-to-grasp affordance (shrink to grasp, expand to walk inside)

## Caught by

- Substrate compatibility lint: `packages/three-kit` source `Renderer.create()` accepts `xrRequested` flag + correct renderer selection
- Foundation demo (three-kit) renders in mocked XR session via WebGL2; smoke test asserts no WebGPU code paths active when XR
- Trigger-monitoring: `@react-three/xr` v6+ version + R3F compat tracked in dep audit
- Per-session-mode effect chain test: VR mode renders with reduced postprocessing, AR mode disables bloom
