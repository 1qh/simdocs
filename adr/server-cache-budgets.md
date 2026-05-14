# server-cache-budgets

## Decision

Next 16 `'use cache'` directive used aggressively for server-side memoization of pure-function compute. `cache()` from React 19 for request-deduping inside RSC.

## What's cached

| Surface | Mechanism | Cache key | TTL |
|---|---|---|---|
| Assembler output | `'use cache'` | hash of asm source | forever (content-addressed) |
| Quine-McCluskey solver result | `'use cache'` | hash of truth table + don't-cares | forever (content-addressed) |
| Espresso solver result | `'use cache'` | hash of truth table | forever (content-addressed) |
| Critical-path computation per (instruction, delay-table) | `'use cache'` | hash of inputs | forever (content-addressed) |
| Pipeline trace arrangement | `'use cache'` | hash of program + forwarding-mode + stall-mode | forever |
| OG image generation | `Cache-Control: public, immutable` at edge + `'use cache'` at origin | content hash | forever |
| Snapshot load (`loadSnapshot`) | Convex internal cache + edge cache | hash | forever (immutable) |
| Sitemap | `'use cache'` | content version | rebuild on content change |
| `mySnapshots` query | Convex reactive cache | userId | reactive invalidation |

## What's NOT cached

- Auth-sensitive queries (`mySnapshots` only via Convex reactive cache, never server `'use cache'`)
- Health check + readiness endpoints (`no-store`)
- Per-user state of any kind
- Any computation with non-deterministic output

## RSC fetch deduping via `cache()`

```ts
import { cache } from 'react';

export const loadExampleManifest = cache(async () => {
  // FS read, expensive
  return await readMdxFrontmatterAll('content/examples');
});
```

Called from multiple RSC components in same request → single execution.

## Cache invalidation

Content-addressed entries (assembler, solver, critical-path) never invalidate — input hash changes → new key.

Non-content-addressed entries (sitemap):
- Tag-based revalidation via Next `revalidateTag` on content change
- Build-time pre-generation for static catalog

## Cache storage

Default: Next's in-memory + filesystem cache. Multi-instance deployments share via Redis adapter (deferred until multi-instance deployed).

## Caught by

- `tools/lint/use-cache-discipline.ts` asserts `'use cache'` only on pure-function modules + queries (no auth-touching code)
- Cache hit ratio per cached operation tracked via OBSERVABILITY
- Cache stampede test: 100 concurrent requests for same key → 1 origin compute, 99 cache hits
