# offline-pwa

## Decision

Service worker registered on first visit. Caches app shell + sim assets + visited example bundles + visited permalink snapshots. Sims continue working offline after first load. Saves are queued + replayed on reconnect.

## Why

- Learning tool — users sometimes work offline (plane, transit, low-connectivity environment)
- Sim is fundamentally client-side (3D + state machine), no inherent reason it can't run offline
- Asymmetric UX win for low cost
- Aligns with "world-class endpoint" — premium web apps work offline

## Service worker shape

- Tool: Workbox via `next-pwa` (locked per `adr/oss-import-audit.md`)
- Caching strategy:
  - **App shell** (HTML, JS, CSS): `StaleWhileRevalidate`
  - **3D assets** (none today, but if any HDRI / fonts): `CacheFirst`, immutable
  - **Permalink snapshots** (`/s/[hash]` API responses): `CacheFirst`, immutable, indefinite TTL
  - **Examples** (`/api/examples/[slug]`): `StaleWhileRevalidate`
  - **Fonts** (Latin subset): `CacheFirst`, immutable
  - **API mutations** (`saveSnapshot`): `NetworkOnly` with background-sync queue fallback
- Versioning: build-time hash for cache keys; new build invalidates old
- Update flow: subtle "new version available, reload" toast (single, dismissible)

## Offline save queue

Anonymous saves queued in IndexedDB when offline. On reconnect, queue replays:
1. For each queued save, recompute canonical body + hash
2. POST to `saveSnapshot` mutation
3. If success: remove from queue, update local hash cache
4. If failure: keep in queue, retry on next online event

Tier-1 URL-fragment shares work offline natively (no backend roundtrip ever).

## What requires online

- Signin (OAuth flow)
- Claim anonymous snapshots
- Load `/me` page (requires fresh Convex query)
- Initial load of an unseen permalink not in cache

Everything else continues functioning offline.

## PWA manifest

`manifest.json` with:
- Name + description (domain language, no school refs)
- Icons (procedural SVG → PNG variants at build time)
- Theme color matching design tokens
- Display: `standalone`
- Start URL: `/`
- Scope: `/`

Installable to home screen on supported browsers.

## Banned

- Workbox precaching that exceeds 50 MB total (cache budget)
- Aggressive prefetch of unvisited routes
- Notifications API (no push notifications in floor)

## Caught by

- Service-worker smoke: page works offline after first online visit
- Cache-size budget test
- Background-sync test: save queued offline → reconnect → save persisted
- Update-flow smoke: deploy new version → toast appears → reload → new version active
