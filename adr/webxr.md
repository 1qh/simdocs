# webxr

## Decision

WebXR (VR) mode for 3D scenes — datapath + K-map toroidal — is scoped, with explicit trigger to land.

Default state: NOT in floor. Floor ships without VR.

Trigger to flip into floor:
- Drei `xr` package proves stable on the locked R3F version
- A user-visible demand signal (multiple feedback requests, target audience overlap)
- Operator decides to ratchet

When trigger fires: VR mode lands as a toggle in `/datapath` and `/kmap` 3D modes, using `@react-three/xr`.

## Why surface this now

- The 3D substrate (`packages/three-kit`) is built compatible with `xr` package from day one — no rework needed when trigger fires
- K-map toroidal geometry is genuinely VR-pedagogical (walk around the torus, look down its axis from inside)
- Adding it after substrate already locked = retrofit + redesign

## What lands when trigger fires

- WebXR session entry button in 3D-route nav
- `<XR>` wrapper around `<Canvas>` from `@react-three/xr`
- Controller raycast → cell selection
- Hand tracking → grouping interaction (K-map)
- Camera bookmarks become positional teleport targets
- Subtle floor / ceiling materials so the user has spatial reference

## What stays out even when trigger fires

- Multi-user VR collaboration (massive scope creep)
- Custom controller gestures beyond standard `select` / `squeeze`
- AR mode (different design considerations)

## Constraint on substrate

`packages/three-kit` must not introduce patterns incompatible with WebXR session (e.g. screen-space-only postprocessing without `xr.enabled` guards). This is a substrate-design constraint, locked from foundation.

## Caught by

- Substrate compatibility lint: `packages/three-kit` source includes XR-compat helpers; postprocessing chain is `xr.enabled`-aware
- Foundation demo (three-kit) optionally renders in XR session — smoke test asserts no rendering errors
- Trigger-monitoring: drei `xr` version + R3F compat tracked in dep audit
