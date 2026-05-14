# PERSISTENCE

Convex self-host backend. Two-tier content-addressed share. Pure-function codec in `packages/sim-engine`.

Decision + rationale: see `adr/persistence-convex.md`, `adr/share-content-addressed.md`.

## Operator zoo wiring

| Service | Endpoint |
|---|---|
| Convex backend | `CONVEX_SELF_HOSTED_URL` (operator's existing instance, deployed per claude2b pattern) |
| Next app | reads `CONVEX_SELF_HOSTED_URL` + Convex deployment URL + admin key |
| Caddy | reverse proxies both, terminates TLS |

## Two-tier write

```
saveSnapshot(state):
  bytes = sim-engine.canonicalize(state)
  hash = blake3(bytes, 8 bytes truncated, hex)
  if bytes.length ≤ 1024:
    return /s#<base64url(bytes)>
  else:
    convex.mutation.saveSnapshot(hash, bytes, contentType)   # idempotent on hash
    return /s/<hash>
```

## Two-tier read

```
loadSnapshot(/s/<hash> or /s#<fragment>):
  if fragment: bytes = base64url-decode(fragment)
  else: bytes = convex.query.loadSnapshot(hash) ⇒ bodyStorageId ⇒ fetch
  state = sim-engine.deserialize(bytes)
  return state
```

## Codec contract

`packages/sim-engine` exposes:
- `canonicalize(state) → bytes` (deterministic, zstd-compressed)
- `hash(bytes) → string` (blake3, 16 hex chars)
- `deserialize(bytes) → state` (round-trip)
- `serializeForFragment(state) → string` (URL-fragment-safe base64url)
- Schema version embedded in body

Property-tested for round-trip + hash-stability per `adr/share-content-addressed.md`.

## Sharing tiers in practice

| Sim state | Typical canonical size | Tier |
|---|---|---|
| MIPS datapath snapshot (no asm, default state) | ~200 bytes | tier-1 |
| MIPS asm program ~10 lines + state | ~600 bytes | tier-1 |
| MIPS asm program ~50 lines + state | ~2 KB | tier-2 |
| Pipeline trace of ~20 instructions | ~3 KB | tier-2 |
| K-map 4-var with groupings | ~400 bytes | tier-1 |
| K-map 6-var with groupings | ~1.5 KB | tier-2 |

## Edge caching

`/s/<hash>` paths set `Cache-Control: public, immutable, max-age=31536000, s-maxage=31536000`. Cloudflare CDN (bearer) caches at PoP. Content-addressed + immutable = forever-cacheable.

`/s#<fragment>` paths require no network round-trip after initial app load.

## Caught by

- Codec round-trip property test
- Codec hash-stability fixture test
- Tier threshold property test (states bordering 1KB exercise both tiers)
- Edge-cache header assertion on `/s/[hash]` Route Handler
- E2E share-load test against deployed instance
