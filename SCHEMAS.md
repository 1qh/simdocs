# SCHEMAS

Convex schemas. SSOT lives in `apps/backend/convex/schema.ts`. This doc carries the spec form per `adr/ssot-precedence.md`; CI lint diffs.

## Tables

### `users`

Created and managed by `@convex-dev/auth`. Fields below are the project's additions on top of the auth-library defaults.

| Field | Type | Notes |
|---|---|---|
| `email` | `string` | Canonical lowercase NFC |
| `name` | `string?` | Display name from OAuth profile, max 200 chars, trimmed |
| `image` | `string?` | OAuth profile image URL, must start with `https://`, max 2000 chars |
| `emailVerificationTime` | `number?` | Convex Auth standard |

Indexed by `email` for lookup on signin.

### `userProfiles`

| Field | Type | Notes |
|---|---|---|
| `userId` | `string` | Foreign key to `users.email` (the canonical key, not Convex id) — matches byerag pattern |
| `role` | `"user" \| "admin"` | Seeded from `BOOTSTRAP_ADMIN_EMAIL` on first matching signin |
| `updatedAt` | `number` | epoch ms |
| `updatedBy` | `string` | `"self"` on signin, `"admin"` if changed by admin |

Indexed by `userId`.

### `snapshots`

Content-addressed snapshot rows. Tier-2 only — tier-1 (URL-fragment) lives in URLs, not the DB.

| Field | Type | Notes |
|---|---|---|
| `hash` | `string` | blake3 16-hex-char prefix, primary access key |
| `bodyStorageId` | `string \| null` | Convex file-storage ID when body > inline-threshold |
| `bodyInline` | `bytes \| null` | Inline body when ≤ inline-threshold |
| `contentType` | `"datapath" \| "kmap" \| "pipeline" \| "compare"` | Discriminator |
| `version` | `number` | Schema version embedded in body, mirrored here for index/migration |
| `createdAt` | `number` | epoch ms |
| `ownerUserId` | `string?` | Nullable for anonymous snapshots |
| `abuseFlag` | `null \| { reason: string; flaggedAt: number; flaggedBy: string }` | Set by `flagAbuse` mutation; loading flagged hash returns 410 |

Indexed by `hash` (primary), `ownerUserId` (for `/me` list), `createdAt` (for moderation queue).

### `anonClaims`

Append-only audit of anonymous-claim events.

| Field | Type | Notes |
|---|---|---|
| `userId` | `string` | Who claimed |
| `hash` | `string` | Which snapshot |
| `claimedAt` | `number` | epoch ms |

Indexed by `userId`, `hash`.

## Functions

### Mutations

| Function | Auth | Args | Returns |
|---|---|---|---|
| `saveSnapshot` | none (anon allowed) | `hash, bytes, contentType, version` | `{ hash }` (idempotent) |
| `claimAnonSnapshots` | signed-in | `hashList: string[]` | `{ claimedCount }` |
| `flagAbuse` | none | `hash, reason` | `void` |

### Queries

| Function | Auth | Args | Returns |
|---|---|---|---|
| `loadSnapshot` | none | `hash` | `{ bytes, contentType, version }` (404 if missing, 410 if flagged) |
| `mySnapshots` | signed-in | `cursor?` | paginated list of owned snapshots |

### Actions

| Function | Auth | Args | Notes |
|---|---|---|---|
| Convex auth callbacks | n/a | per `@convex-dev/auth` | createOrUpdateUser, redirect — mirrors byerag patterns |

## Schema evolution

Expand-contract per `book/HARD-RULES.md`. Add new field as optional, dual-write or backfill, then promote to required in a later release. Migration tracked in `apps/backend/scripts/migrate-snapshots.ts` style scripts.

## Caught by

- Spec-of-code lint diffs this doc's tables against `apps/backend/convex/schema.ts`
- `convex-test` covers every mutation + query with round-trip assertions
- Schema migration smoke runs every committed migration script against fresh DB
