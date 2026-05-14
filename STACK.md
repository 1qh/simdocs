# STACK

Locked picks per layer. Every named pick is spec-of-code per `book/HARD-RULES.md` "STACK-presence lint mandatory" — CI verifies each pick is actually consumed.

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
      framer motion 3D springs
    Editor
      Monaco editor
      Custom MIPS grammar
      Custom Boolean expression grammar
    Persistence
      Convex self host
      at convex dev slash auth
      at auth slash core providers Google
      Convex codegen typed API
    Content addressable share
      Blake3 hash
      Zstd canonicalization
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
      simple git hooks
      tsdown bundler for packages
    Test
      Bun test
      happy dom global registrator
      testing library react
      fast check property based
      convex test
      Playwright E2E
      Stryker mutation testing
      axe core a11y
      pa11y contrast
    Perf
      bundlesize2 bundle gate
      lhci Lighthouse CI
      r3f perf frame budget
      web vitals RUM
    Observability
      Plausible Analytics self host
      structured JSON logs
    PWA
      next pwa or Workbox service worker
    OG
      next ImageResponse
    CI
      GitHub Actions matched to claude2b pattern
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
| `sim-engine` | Deterministic state machine + scrub + snapshot codec (blake3 + zstd + canonicalize) | `@noble/hashes` for blake3, `fzstd` or `@zstd-js/zstd` for zstd |
| `editor` | Monaco wrapper + custom language wiring | Monaco |
| `bits` | Two's complement, sign-extend, hex / bin / dec conversions, bit slicing | None |
| `boolean` | Truth table, QM, Petrick, Espresso, prime implicants, hazard analysis | None |

OSS-import-first scan per package logged in the package's ADR — see `adr/three-stack.md`, `adr/editor-monaco.md`, `adr/boolean-package.md`, etc.

## Product app stack

`apps/web` (Next.js):
- Routes: `/` landing, `/datapath` MIPS sim, `/kmap` K-map tool, `/pipeline` pipeline view, `/compare` side-by-side, `/learn/*` MDX explainers, `/s/[hash]` shared snapshot, `/me` (auth-gated, optional)
- Server: RSC + Server Actions for save/share/assemble, Route Handlers for Convex webhooks
- Client: 3D canvas islands, Monaco editor, zustand stores, framer-motion-3d transitions
- Auth: `@convex-dev/auth` + Google OAuth (matches byerag pattern), anon-first, signin = optional cross-device persistence
- Convex client: `convex` + `@convex-dev/auth` React bindings

## Convex backend stack

`apps/backend`:
- Convex self-host instance reachable via `CONVEX_SELF_HOSTED_URL`
- Schema: snapshots, users, userProfiles (mirrors byerag's profile-with-role pattern), share-index
- Functions: `saveSnapshot`, `loadSnapshot`, `claimAnonSnapshots`, auth callbacks
- Auth providers: Google via `@auth/core/providers/google`
- File storage: Convex built-in for snapshot bodies >1KB
- Scheduled jobs: abuse-flag sweeps if/when triggered

## Operator zoo (compose locally + Helm in cluster)

Same images, same env shape, same bootstrap. Scale parameters differ between deployment topologies.

| Service | Image |
|---|---|
| `next-app` | Built from `apps/web` Dockerfile, Next standalone output |
| `convex-backend` | `ghcr.io/get-convex/convex-backend:latest` (self-host) |
| `caddy` | `caddy:latest` with cache module |

## Verifier targets

- `make verify.local` — pure self-host stack green
- `make verify.bearer` — with Cloudflare CDN + DNS in front, green
- `make verify.fresh` — bootstrap from secrets dump + clean state, green
- All three exercised periodically per `book/PHILOSOPHY.md` "Seamless machine migration"

## Caught by

- Stack-presence lint per pick (`tools/lint/stack-presence.ts`)
- Staleness gate (`tools/lint/check-staleness.ts`)
- ADR-presence lint — every pick on this page has an ADR file under `adr/`
- Spec-of-code allowlist entry — `STACK.md` is itself spec-of-code per `book/HARD-RULES.md`
