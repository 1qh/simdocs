# mobile-read-only

## Decision

Screens narrower than ~1024px get a read-only experience. Full interactivity (asm editor, K-map drag-grouping, 3D camera controls) requires precision input + screen real estate not available on small touch screens.

## What works on mobile

- All MDX learn pages fully readable, prose + diagrams fluid
- Shared permalink view (`/s/[hash]`) renders a static snapshot of the sim state with tabular summary
- Example library browseable
- Read register / memory / signal values from a saved snapshot
- Static image fallback for 3D islands (server-rendered at build time, cached at edge)
- Auth flow works (signin from mobile to claim anon saves)
- Onboarding tour readable

## What requires desktop

- Asm editor (Monaco — touch keyboard input + precision cursor required)
- K-map drag-grouping (touch is imprecise for sub-cell rectangles)
- 3D camera orbit/dolly (mouse + scroll wheel preferred; touch gestures incomplete)
- Side-by-side compare (screen real estate)

## Detection

CSS media query `@media (min-width: 1024px)` + JavaScript viewport check. Below threshold: route mounts a `<ReadOnlyExperience>` component instead of full interactive scene.

## Read-only UX

- Permalink view: snapshot rendering + tabular state + scrub bar (read-only — no step button, but pre-recorded animation can play)
- Banner at top: "interactive editing available on a larger screen" — dismissible, never-blocking
- Save button still works (snapshot of read-only state — useful if user came in via shared link)

## Static 3D fallback

Build-time generation:
- Playwright (or similar headless render) snapshots each known state into PNG
- PNG cached at Cloudflare with content-addressed cache key
- Mobile loads PNG instead of mounting R3F

For dynamic states (user lands on `/s/[hash]` with novel state):
- Server-side render via `ImageResponse` produces static snapshot at request time
- Cached forever per content-addressed identity

## Banned

- Pinch-zoom hacks that re-enable interaction below 1024px
- Mobile-specific UI layouts that bypass the read-only constraint
- Touch gesture overrides for 3D camera

## Trigger to expand

- Multi-touch K-map grouping shape proves cleanly designable AND user demand evidence
- Mobile asm editor alternative (e.g., picker-based) tested for pedagogical equivalence

Until trigger: read-only is the locked mobile experience.

## Caught by

- Responsive layout test (Playwright at 375x667, 768x1024, 1024x768, 1440x900): below 1024 mounts read-only, above mounts interactive
- Static fallback CI: PNG cache populated for known states
- Accessibility test on mobile (axe-core at small viewport)
