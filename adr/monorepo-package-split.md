# monorepo-package-split

## Decision

Seven substrate packages, two app workspaces.

| Path | Role |
|---|---|
| `packages/three-kit` | R3F + drei + TSL helpers, material library, signal-pulse shader, camera grammar, postprocessing presets |
| `packages/hud` | In-3D UI chrome via drei-uikit |
| `packages/design-tokens` | Palette, typography, spacing, motion easing — pure TS, no deps |
| `packages/sim-engine` | Deterministic state machine, trace, scrub, snapshot codec (canonicalize + blake3 + zstd) |
| `packages/editor` | Monaco wrapper, language definitions, error markers |
| `packages/bits` | Two's complement, sign-extend, base conversions, bit slicing |
| `packages/boolean` | Truth table, QM, Petrick, Espresso, prime implicants, hazard analysis |
| `apps/web` | Next.js product app — features (datapath, kmap, pipeline, critical-path, compare, learn) |
| `apps/backend` | Convex schema + functions |

## Why this split, not finer or coarser

- Each package answers a single substrate concern and has a foundation-app demo proving it generically.
- Splitting `three-kit` and `hud` separately so domain-3D vs UI-3D are independently consumable.
- `design-tokens` separate from `three-kit` so non-3D surfaces (DOM HUDs, MDX learn pages) share tokens without 3D dep.
- `sim-engine` is engine-only, never imports renderers — pure state machine + codec.
- `editor` separate from product so a future sim can reuse Monaco wiring.
- `bits` and `boolean` are pure-function libraries; both consumed by `apps/web` features.
- Coarser split (e.g. merging `bits` into `sim-engine`) couples unrelated concerns; finer split (e.g. separate `qm`, `petrick`, `espresso`) creates premature surface area.

## Foundation demos required per package

| Package | Foundation demo |
|---|---|
| `three-kit` | Generic procedural circuit-board scene with emissive trace pulses + camera bookmark dolly |
| `hud` | Floating telemetry panel attached to a moving 3D object |
| `design-tokens` | Token preview page — every color, type, spacing, easing exhibited |
| `sim-engine` | Generic 4-state FSM with trace + scrub + snapshot round-trip |
| `editor` | Generic language with custom keywords + error squiggles |
| `bits` | Property-based test suite covering all conversions |
| `boolean` | Generic 4-input function minimization with PI + EPI + SOP + POS output |

Foundation demos live under `apps/web/learn/foundation/*` and are part of the substrate showcase, not the product. Per `book/SUBSTRATE.md` substrate-growth-loop.

## Caught by

- Package-presence lint asserts each `packages/*` path exists with `package.json`, `tsconfig.json`, `src/`, foundation-demo entry.
- Substrate-product boundary lint asserts `packages/*` source greps for no MIPS / Kmap / domain vocab.
- Foundation-demo lint asserts each package has its demo route registered.
