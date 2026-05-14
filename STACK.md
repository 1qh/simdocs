# STACK

Locked picks per layer. Every named pick is spec-of-code per `book/HARD-RULES.md` "STACK-presence lint mandatory" — CI verifies each pick is actually consumed. Conformance with pm4ai `ALL_BANNED` map (operator-wide banned-deps catalog at `/Users/o/z/pm4ai/packages/pm4ai/src/banned.ts`) is mandatory; banned alternatives never enter dep manifests.

## Picks per layer

```mermaid
mindmap
  root((Stack))
    Runtime
      Bun
      Node 22+ where Bun lacks coverage
    Monorepo
      Turbo
      Sherif workspace consistency
      Bun workspaces
    Lang
      TypeScript strict
      Zod v4 boundary validation
    Framework
      Next.js latest
      React latest
      Server Components
      Server Actions
      PPR partial pre rendering
      MDX via at next slash mdx
      next-themes
    Three D
      R3F latest
      drei
      drei uikit in 3D HUD
      postprocessing
      leva dev only
      zustand transient
      TSL Three Shading Language
      WebGPU renderer primary
      WebGL renderer fallback
      motion for animations
    Components
      shadcn registry
      cnsync from readonly slash ui
      lucide-react icons
      tanstack-table register memory control panels
      tanstack-virtual command palette memory
      tanstack-form schema-driven forms over Zod
      tanstack-query alongside Convex client where useful
      at dnd-kit drag for 2D K-map
    Editor
      Monaco editor
      chevrotain for MIPS grammar
      chevrotain for Boolean expression grammar
    Persistence
      Convex self host
      at convex-dev slash auth
      at auth slash core providers Google
      Convex codegen typed API
    Content addressable share
      Blake3 hash via at noble slash hashes
      Zstd via Bun.zstdCompress Bun.zstdDecompress server side
      Native CompressionStream DecompressionStream client side
      URL fragment tier
      Convex file storage tier
    Edge
      Caddy reverse proxy
      Caddy cache module
      Cloudflare DNS bearer
      Cloudflare CDN bearer cache only
    Deploy
      Dokploy VM existing
      Single bootstrap script
      Helm parity to compose
    Tooling
      pm4ai monorepo orchestrator
      lintmax biome oxlint eslint
      sherif workspace lint
      simple-git-hooks
      tsdown bundler for packages
    Test
      Bun test
      happy dom global registrator
      testing library react
      fast check property based
      convex-test
      Playwright E2E
      Stryker mutation testing
      axe-core a11y
      pa11y contrast
    Perf
      size-limit bundle gate (bundlesize2 unmaintained)
      lhci Lighthouse CI
      stats-gl frame budget (r3f-perf stale)
      web-vitals RUM
      memlab leak detection
      mitata micro benchmarks
    Observability
      Plausible Analytics self host
      consola structured logging
      Error boundary client side
      Platform managed error reporting
    PWA
      Serwist via at serwist slash next (next-pwa dead)
    OG
      Next ImageResponse
    Utility
      es-toolkit
      structuredClone native
      Intl native datetime number formatting
      crypto.randomUUID native
    OSS substrate libs
      safe-stable-stringify canonicalize
      microdiff state diff
      chevrotain parser
      mini-search command palette fuzzy
      detect-gpu device tier
      culori color science
      toposort dependency graph
      shiki MDX code blocks
      remark-frontmatter MDX frontmatter
    CI
      GitHub Actions matched to operator reference deploy project pattern in memory
      Renovate auto bumps
      ledger jsonl gate outcomes
```

## Concrete versions

Per `book/PHILOSOPHY.md` Latest-only rule — every direct dep tracks upstream latest. Lockfile is build-artifact, not source-of-truth. The version catalog source-of-truth lives in `apps/web/package.json` + `packages/*/package.json` direct deps. CI staleness gate (`tools/lint/check-staleness.ts`) fires on any dep with no upstream release in 6 months.

## Substrate package stack

| Package | Concern | Owned externalities |
|---|---|---|
| `three-kit` | R3F + drei + drei-uikit + postprocessing + TSL helpers + material library + camera grammar | R3F ecosystem |
| `hud` | In-3D UI chrome backed by drei-uikit | drei-uikit |
| `design-tokens` | Palette, typography, spacing, motion easing | None (TS only) |
| `sim-engine` | Deterministic state machine + scrub + snapshot codec (blake3 + zstd + canonicalize) | `@noble/hashes` for blake3, `Bun.zstdCompress`/`Bun.zstdDecompress` server-side + native `CompressionStream`/`DecompressionStream` client-side |
| `editor` | Monaco wrapper + chevrotain grammar wiring | Monaco, chevrotain |
| `bits` | Two's complement, sign-extend, hex / bin / dec conversions, bit slicing | None |
| `boolean` | Truth table, QM, Petrick, Espresso, prime implicants, hazard analysis | None (or OSS solver wrapped per audit) |

OSS-import-first scan per package logged in the package's ADR — see `adr/three-stack.md`, `adr/editor-monaco.md`, `adr/boolean-package.md`, etc.

## Product app stack

`apps/web` (Next.js):
- Routes: `/` landing, `/datapath` MIPS sim, `/kmap` K-map tool, `/pipeline` pipeline view, `/compare` side-by-side, `/learn/*` MDX explainers, `/s/[hash]` shared snapshot, `/me` (auth-gated, optional)
- Server: RSC + Server Actions for save/share/assemble, Route Handlers for Convex webhooks
- Client: 3D canvas islands, Monaco editor, zustand stores, motion transitions
- Components: shadcn registry primitives + cnsync (operator's `readonly/ui` workspace) — Button, Dialog, Drawer, Tabs, Tooltip, Popover, Sheet, Toast, etc.
- Forms: `@tanstack/react-form` + Zod resolver
- Tables: `@tanstack/react-table` for register/memory/control panels
- Virtualization: `@tanstack/react-virtual` for command palette + memory address list
- Drag: `@dnd-kit` for 2D K-map cell drag-grouping (3D toroidal uses in-canvas R3F drag)
- Icons: `lucide-react`
- Themes: `next-themes` provider
- Auth: `@convex-dev/auth` + `@auth/core/providers/google` (matches operator's reference Convex+auth project pattern in memory), anon-first, signin = optional cross-device persistence
- Convex client: `convex` + `@convex-dev/auth` React bindings

## Convex backend stack

`apps/backend`:
- Convex self-host instance reachable via `CONVEX_SELF_HOSTED_URL`
- Schema: snapshots, users, userProfiles (mirrors operator's reference profile-with-role pattern in memory), share-index
- Functions: `saveSnapshot`, `loadSnapshot`, `claimAnonSnapshots`, auth callbacks
- Auth providers: Google via `@auth/core/providers/google`
- File storage: Convex built-in for snapshot bodies >1KB
- Scheduled jobs: abuse-flag sweeps if/when triggered

## Worker concurrency (no banned deps)

Per `CONCURRENCY.md` + pm4ai `worker` ban on `comlink` / `workerpool` / `threads` / `tinypool` / `piscina` / `worker-threads-pool`:

- Workers spawned via native `new Worker(url)` / `new SharedWorker(url)` — Bun + browsers handle natively
- Worker RPC: thin typed wrapper around `postMessage` + `MessageChannel` (≤ 30 LOC per worker class, hand-roll justified per `adr/oss-import-audit.md`)
- Worker pool: thin scheduler over native Workers with work-stealing extension (~ 50 LOC, hand-roll justified)
- Transferable handoff via `Transferable` array on `postMessage`

## Operator zoo (compose locally + Helm in cluster)

Same images, same env shape, same bootstrap. Scale parameters differ between deployment topologies.

| Service | Image |
|---|---|
| `next-app` | Built from `apps/web` Dockerfile, Next standalone output |
| `convex-backend` | `ghcr.io/get-convex/convex-backend:latest` (self-host) |
| `caddy` | `caddy:latest` with cache module |
| `plausible` | Self-host Plausible Analytics container |

## Verifier targets

- `make verify.local` — pure self-host stack green
- `make verify.bearer` — with Cloudflare CDN + DNS in front, green
- `make verify.fresh` — bootstrap from secrets dump + clean state, green
- All three exercised periodically per `book/PHILOSOPHY.md` "Seamless machine migration"

## Caught by

- Stack-presence lint per pick (`tools/lint/stack-presence.ts`)
- Staleness gate (`tools/lint/check-staleness.ts`)
- OSS-import-first lint (`tools/lint/oss-import-first.ts`) per `adr/oss-import-audit.md` — hand-roll without rejection-rationale entry = violation
- **pm4ai banned-deps lint** (operator-wide via lintmax + per-project check) — any dep matching `/Users/o/z/pm4ai/packages/pm4ai/src/banned.ts` `ALL_BANNED` entries fails CI
- ADR-presence lint — every pick on this page has an ADR file under `adr/`
- Spec-of-code allowlist entry — `STACK.md` is itself spec-of-code per `book/HARD-RULES.md`
