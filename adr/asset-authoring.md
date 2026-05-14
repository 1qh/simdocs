# asset-authoring

## Decision

Pure-procedural. Every visual is authored in code via R3F primitives + TSL shaders + drei materials. Zero `.glb`, `.fbx`, `.obj`, `.hdr` (beyond drei's bundled HDRIs), or any binary asset in the repo.

## Why

- Agent-only execution per `book/PHILOSOPHY.md` — Blender-authored assets require operator or commissioned-artist authoring; that path blocks the single-agent loop.
- Code is the SSOT for visuals — diff-able, reviewable, deterministic.
- TSL + R3F + drei reach AAA aesthetic ceiling for the scenes this product needs (machined silicon, emissive bus traces, glass enclosures, signal pulses, depth-of-field stage transitions).
- No binary churn in git.
- Reference shops (Lusion, Active Theory, much of the pmndrs example set) reach premium aesthetic primarily via code, not authored assets.

## Allowed external assets

- drei `Environment` bundled HDRIs (apartment, city, dawn, studio, sunset, warehouse) — packaged with drei, not added to repo.
- Open-source PBR textures procedurally generated at build time (normal maps from procedural noise, etc.) — generated, not imported.

## Banned

- `.glb`, `.gltf`, `.fbx`, `.obj`, `.usdz`, `.usdc` model files in repo.
- `.hdr`, `.exr`, `.ktx2` texture assets in repo (beyond drei's bundled set).
- External CDN-loaded model URLs.
- Blender → glTF pipeline tooling.

## Caught by

- Repo-asset lint scans for any of the banned extensions under any path; zero hits required.
- Substrate stays pure TS — `packages/three-kit` source is `.ts` + `.tsx` only.
