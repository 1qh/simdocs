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
- Object keys sorted deterministically via `safe-stable-stringify`
- Numbers in shortest exact representation (integer state preferred; banned: `NaN`, `Infinity`, `-0`)
- Arrays untouched (order is part of the state)
- `Set` / `Map` / `BigInt` banned in canonical-path Zod schemas (safe-stable-stringify silently drops Sets/Maps; BigInt lossily coerces)
- Schema-version byte prefix (cross-version hash collision-free)
- Output is **uncompressed** canonical bytes; **hash uncompressed bytes** (compression artifacts vary across runtime versions)
- Compression layer separate, transport-only

## Compression for transport

Hash uncompressed canonical bytes. Compress only for transport:

- **Client + universal**: `CompressionStream("deflate-raw")` — no header / no checksum / no mtime → byte-identical across Chrome / Safari / Firefox / Node 21+ / Bun. Universal browser support 2024+.
- **Server at-rest storage**: `Bun.zstdCompress` level 19 + `windowLog: 21` allowed — but pin zstd-version-byte in URL schema OR re-canonicalize before re-hashing on decode (Bun bumps zstd, output varies per version+level+window-log).

URL fragment encodes compressed bytes as base64url:
```
canonical bytes → CompressionStream("deflate-raw") → base64url → /s#<payload>
```

`fzstd` / `pako` / `fflate` / `lz-string` all banned per pm4ai compression policy.

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
