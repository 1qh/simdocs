# speculative-loading

## Decision

Speculation Rules API (Chrome 109+) prerenders likely-next routes on hover + idle. Other browsers fall back to `<link rel="prefetch">`. View Transitions API (Next 16) drives the navigation animation.

## Speculation Rules JSON

Emitted in document `<head>`:

```html
<script type="speculationrules">
{
  "prerender": [
    { "where": { "href_matches": "/datapath" }, "eagerness": "moderate" },
    { "where": { "href_matches": "/kmap" }, "eagerness": "moderate" },
    { "where": { "href_matches": "/pipeline" }, "eagerness": "moderate" },
    { "where": { "href_matches": "/compare" }, "eagerness": "moderate" },
    { "where": { "href_matches": "/learn/*" }, "eagerness": "conservative" },
    { "where": { "href_matches": "/s/*" }, "eagerness": "conservative" }
  ],
  "prefetch": [
    { "where": { "href_matches": "/*" }, "eagerness": "moderate" }
  ]
}
```

Eagerness levels:
- `eager` — prerender on page load (heavy, used only when navigation is near-certain)
- `moderate` — prerender on hover or 200ms after pointer enter
- `conservative` — prefetch only, no prerender

## Cross-browser fallback

Speculation Rules → Chromium-only.

Firefox / Safari fallback:
- `<link rel="prefetch">` for likely-next routes injected via `next/link` (already default)
- `<link rel="modulepreload">` for code-split bundles

## View Transitions API

Next 16's `unstable_ViewTransition` wraps cross-route transitions:

```tsx
import { unstable_ViewTransition as ViewTransition } from 'next/view-transitions';

<ViewTransition name="sim-canvas">
  {/* shared element across routes */}
</ViewTransition>
```

Shared elements between routes (camera-bookmark transitions when navigating between `/datapath` and `/compare`, etc.) animate smoothly.

Reduced-motion respected — View Transitions collapse to instant snap.

## Prerender storage

Browser caches up to ~10 prerendered documents (Chrome default). When user actually navigates, prerendered version commits instantly (perceived 0ms latency).

## Disable conditions

Per Speculation Rules spec, prerender suppressed when:
- Save-Data header is set (user is on metered connection)
- Memory pressure (browser auto-throttle)
- Power saver mode

Detection automatic; no code required.

## Bandwidth budget

Prerender consumes bandwidth proactively. Cap:
- `moderate` eagerness on 4 routes max
- `conservative` everywhere else
- Don't prerender `/s/*` permalink unless user has hovered the share link

Eagerness tuned post-deploy based on real-user navigation patterns + CDN egress trend.

## Caught by

- HTML emission smoke: every page emits Speculation Rules script
- Navigation latency test (Playwright): hover + click measured; prerendered routes < 50ms perceived
- View Transition test: shared element renders identical bounding box across pre/post navigation
- Bandwidth audit: prerender egress < 20% of total traffic
