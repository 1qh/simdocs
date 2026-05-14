# GOAL

Ship a world-class 3D interactive visualizer that contains:

- **MIPS datapath visualizer** — single-cycle topology, every encodable instruction animated step-by-step with active components lit, control signals shown, critical path overlay, side-by-side instruction comparison, pipeline stage-time diagram with hazard detection.
- **Karnaugh map tool** — 2D for ≤4 vars, 3D for ≥5 vars, truth table / Boolean expression / minterm-list / maxterm-list input, interactive grouping, automatic prime implicants + essential prime implicants + minimal SOP + minimal POS, optional solver reveal.
- **Single web app**, anonymous-first, optional login for cross-device persistence, content-addressed shareable permalinks for every sim state.

## What "world-class" means here

- Visual aesthetic at Apple/NVIDIA silicon-reveal tier — see `UX-DOCTRINE.md`
- 60 fps on mid-tier laptop hardware, WebGPU primary path with WebGL fallback
- 100% keyboard-driven (every action invocable without mouse)
- WCAG AA accessibility floor, reduced-motion respected
- Deterministic sim engine — same input produces frame-accurate same animation
- Save/share permalink for every sim state, content-addressed, edge-cacheable
- Substrate published OSS, foundation-app demonstrates every substrate primitive generically

## What it must run

- Operator zoo runs on a single MacBook through the same compose stack as production
- Production deploys to dokploy VM with Cloudflare DNS + edge cache, Convex self-host instance for persistence
- Migration laptop ↔ on-prem ↔ cloud-VM is re-point IaC + restore backups, never rewrite

## What ships in the locked floor

Every feature in `REQUIREMENTS.md` and every "more not less" ratchet on top. No phased carve-outs. Items genuinely outside scope live in `NON-GOALS.md` with explicit trigger.
