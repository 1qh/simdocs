# share-content-addressed

## Decision

Two-tier content-addressed share. Tier-1 URL-fragment for tiny states. Tier-2 Convex storage for larger states. Both content-addressed via blake3 hash of canonical bytes.

## Tier-1 — URL fragment

State canonicalized + zstd-compressed + base64url-encoded → fragment.

```
state ≤ 1KB compressed → /s#<base64url-payload>
```

- Zero backend roundtrip for save or load
- Permalink works offline
- No infrastructure cost at any scale
- Free for the most viral cases

## Tier-2 — Convex storage

State exceeds tier-1 threshold → Convex mutation stores body, returns hash.

```
state > 1KB compressed → /s/<blake3-truncated-16-chars>
```

- `sim-engine` canonicalizes + hashes
- Convex `saveSnapshot(body, contentType)` is idempotent on hash (conditional insert: skip if exists)
- Convex file storage holds the body for large snapshots; small ones inline in document
- CDN-cacheable response (`Cache-Control: public, immutable, max-age=31536000, s-maxage=31536000`)

## Canonicalization

Per `packages/sim-engine` codec:
- Object keys sorted deterministically
- Numbers in shortest exact representation
- Arrays untouched (order is part of the state)
- zstd compression, fixed level
- Output is canonical bytes; same state → same bytes → same hash

## Hash

Blake3, first 16 hex chars (8 bytes, ~1.8e19 namespace, collision-safe well past CCU floor).

## Schema versioning

Snapshot body carries `{ v: <int>, ... }`. `sim-engine` keeps deserializer for every published `v`. Old hashes forever-replay-able. New `v` lands via expand-contract — add new field, keep old shape working through one version, drop old in next major.

## Read path (high CCU shape)

```
GET /s/<hash>
  → Cloudflare CDN edge (bearer cache)
    → cache hit → response, zero origin cost
    → cache miss → Caddy → Next RSC → Convex query loadSnapshot(hash) → response with immutable headers
```

Content-addressed + immutable = forever-cacheable. Viral snapshot viewed millions of times → one origin fetch per CF PoP, then edge serves.

## Write path

```
Server Action saveSnapshot(state)
  → sim-engine canonicalize(state) → bytes
  → sim-engine hash(bytes) → hash
  → if bytes.length ≤ 1024: return /s#<urlfragment>
  → else: Convex mutation saveSnapshot(hash, bytes, contentType) idempotent
        → return /s/<hash>
```

## Abuse handling

`flagAbuse(hash, reason)` mutation sets `abuseFlag` on the snapshot row. Loading a flagged hash returns 410 Gone + cache-purge header. Admin role reviews flagged queue.

## Determinism guarantee

Same `(MIPS program, K-map exercise, sim state)` → identical canonical bytes → identical hash. Substrate codec is the SSOT for this property; property-test enforces idempotence.

## Caught by

- Codec round-trip property test (`fast-check`) — `deserialize(serialize(state)) === state` for any state.
- Codec stability test — hash of fixed test inputs is committed; any change to canonicalization that shifts the hash fails CI.
- Schema-version migration test — old fixtures deserialize correctly under every published `v`.
