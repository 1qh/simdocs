# OG-IMAGES

Dynamic Open Graph card spec. Viral share signal — when a permalink is dropped in any chat or social surface, the preview card must communicate the sim state at a glance.

## Generation

Next 16 `ImageResponse` API. Route Handlers under `apps/web/src/app/api/og/[type]/[hash]/route.ts`.

## Card variants

### `/api/og/datapath/<hash>`

For shared datapath permalinks.

Layout:
- Background: industrial near-black with subtle PCB-substrate texture (procedural noise via ImageResponse SVG)
- Left half: stylized 2D datapath schematic (SVG, not 3D — OG must render server-side)
- Right half: instruction text + step + key registers
- Bottom: small mono `sim/datapath` watermark (no brand name — domain language only)

Dimensions: 1200 × 630 (OG standard).

### `/api/og/kmap/<hash>`

For shared K-map permalinks.

Layout:
- Background: same industrial tone
- Left half: stylized 2D K-map (4x4 or 8x8 grid depending on var count) with current groupings rendered as colored rectangles
- Right half: variable count + expression + groupings count
- Bottom: `sim/kmap` watermark

### `/api/og/pipeline/<hash>`

For shared pipeline permalinks.

Layout:
- Stage-time diagram condensed to 6 instructions × 8 cycles
- Hazards highlighted
- Bottom: `sim/pipeline` watermark

### `/api/og/compare/<hash>`

For compare-mode shares.

Layout:
- Two mini-datapath schematics side by side
- Diff highlights
- Bottom: `sim/compare` watermark

### `/api/og/learn/<slug>`

For learn-page shares.

Layout:
- Title of the learn page
- One representative diagram from the page (cached at build)
- Bottom: `sim/learn` watermark

## Caching

- `Cache-Control: public, immutable, max-age=31536000, s-maxage=31536000`
- Cloudflare edge caches forever per content-addressed identity
- Cache key = full URL including hash
- Purge on `flagAbuse` (per `RUNBOOK.md`)

## Determinism

OG image generation is deterministic per `DETERMINISM.md` — same input snapshot produces identical PNG bytes. Allows CDN to dedupe on content hash if ever needed.

## Metadata wiring

Each route's `generateMetadata` emits:

```ts
export async function generateMetadata({ params }: { params: { hash: string } }) {
  return {
    openGraph: {
      title: titleForSnapshot(params.hash),
      description: descriptionForSnapshot(params.hash),
      images: [`https://<domain>/api/og/datapath/${params.hash}`],
      type: 'website',
    },
    twitter: {
      card: 'summary_large_image',
      images: [`https://<domain>/api/og/datapath/${params.hash}`],
    },
  };
}
```

Per `API-CONVENTIONS.md` metadata rules.

## Fonts

- Mono: JetBrains Mono via `@vercel/og` font loader (embedded in build, no runtime fetch)
- Subset to Latin + common math/Boolean symbols

## Privacy

OG generator reads only the public snapshot body. No identity-bound rendering. Same hash → same image for everyone.

## Caught by

- Snapshot test: known fixtures produce committed-PNG byte-match
- Smoke: every share-route returns 200 + valid PNG with correct dimensions
- Cache test: identical hash request twice returns same `ETag`
