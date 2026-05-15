# SCHEMAS

Convex schemas. SSOT lives in `apps/backend/convex/schema.ts`. This doc mirrors the deployed shape; CI lint diffs.

## Tables

### `users`

Auth library default tables from `@convex-dev/auth` (`authTables` spread). Fields are the standard Convex Auth shape: `email`, `name`, `image`, `phone`, `emailVerificationTime`, `phoneVerificationTime`, `isAnonymous`.

### `userProfiles`

| Field | Type | Notes |
|---|---|---|
| `userId` | `Id<'users'>` | FK to Convex `users` table id |
| `isAdmin` | `boolean` | Seeded from `BOOTSTRAP_ADMIN_EMAIL` on first matching signin |
| `createdAt` | `number` | epoch ms |
| `lastSeenAt` | `number` | epoch ms |

Indexed by `userId` (`byUserId`).

### `snapshots`

Content-addressed snapshot rows. Tier-2 only — tier-1 (URL fragment ≤1KB) lives in URLs, not the DB.

| Field | Type | Notes |
|---|---|---|
| `hash` | `string` | Canonical-JSON blake3 hex, primary access key |
| `bytes` | `number` | Compressed payload size |
| `canonicalJson` | `string` | Stored canonical JSON body |
| `kind` | `"mips" \| "kmap" \| "compare" \| "pipeline"` | Discriminator |
| `parentHash` | `string?` | Optional ancestry |
| `createdAt` | `number` | epoch ms |
| `submitterFingerprint` | `string` | Anonymous-ownership fingerprint |
| `claimedByUserId` | `Id<'users'>?` | Set when anon submitter signs in and claims |

Indexes: `byHash`, `byUser` (`claimedByUserId`), `byFingerprint` (`submitterFingerprint`).

### `rateLimitWindows`

Rolling-window counter table for `saveSnapshot` rate limiting.

| Field | Type | Notes |
|---|---|---|
| `keyHash` | `string` | SHA-256 of submitter fingerprint |
| `count` | `number` | Requests in current window |
| `windowStartMs` | `number` | Window-start epoch ms |

Indexed by `keyHash` (`byKeyHash`).

## Functions

### Mutations

| Function | Auth | Args | Returns |
|---|---|---|---|
| `saveSnapshot` | none (anon allowed) | `hash, bytes, canonicalJson, fingerprint, kind, parentHash?` | `{ created: boolean, hash }` |
| `claimAnonSnapshots` | signed-in | `fingerprint, userId` | `{ claimed: number }` |

### Queries

| Function | Auth | Args | Returns |
|---|---|---|---|
| `loadSnapshot` | none | `hash` | snapshot row or `null` |

### Actions

`@convex-dev/auth` standard handlers (Google OAuth + Anonymous providers) — see `auth.ts`.

## Rate limiting

30 requests per 60_000 ms window per fingerprint. 31st throws `rate_limit_exceeded`. Counter table `rateLimitWindows` keyed by SHA-256 of fingerprint.

## Schema evolution

Expand-contract per `book/HARD-RULES.md`. Add new field as optional, dual-write or backfill, then promote to required in a later release.

## Caught by

- Spec-of-code lint diffs this doc's table list against `apps/backend/convex/schema.ts` table names
- Live convex-client tests against local self-host stack cover saveSnapshot/loadSnapshot/rate-limit
