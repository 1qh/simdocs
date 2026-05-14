# API-CONVENTIONS

Conventions for every server surface — Route Handlers, Server Actions, `ImageResponse`, Convex client calls.

## Server Actions

| Aspect | Rule |
|---|---|
| Naming | Verb-first, kebab-case file: `save-snapshot.ts`, `claim-anon-snapshots.ts` |
| Validation | Zod v4 schema on every input at function boundary. Invalid input throws typed `BadInputError`. Per `book/HARD-RULES.md` "Zero fallback" |
| Error shape | `{ ok: false, code: string, message: string }` returned to caller (never raw exceptions to client) |
| Success shape | `{ ok: true, data: <typed> }` |
| Auth | Wrapped in `withAuth(action, { allowAnon: boolean })`; explicit per action |
| Rate limit | Wrapped in `withRateLimit(action, { perIp: number, perWindow: number })` |
| Logging | Auto-wrapped via OBSERVABILITY logger — op name, duration, outcome |
| Idempotency | Required for write actions on resources with `Idempotency-Key` semantics (e.g. `saveSnapshot` is content-addressed, naturally idempotent) |

## Route Handlers

| Aspect | Rule |
|---|---|
| Naming | RESTful, kebab-case: `/api/snapshot/[hash]`, `/api/rum` |
| Methods | Explicit HTTP method handlers; never accept other methods |
| Status | Use HTTP status correctly: 200, 201, 204, 400, 401, 403, 404, 410 (flagged), 429, 500 |
| Content-Type | Always `application/json` unless serving a specific binary type |
| Cache-Control | Explicit per route, never default. Public + immutable for content-addressed; `no-store` for auth-sensitive |
| Validation | Zod v4 schema on every input (query params, body, headers) |
| Error shape | Same as Server Actions |

## ImageResponse (OG cards)

Per `OG-IMAGES.md`. Dynamic OG image generation via Next 16 `ImageResponse`.

Cache-Control: `public, immutable, max-age=31536000` on hash-addressed cards; `public, max-age=3600` on dynamic-content cards.

## Convex calls

| Aspect | Rule |
|---|---|
| Mutation naming | Verb-first: `saveSnapshot`, `flagAbuse`, `claimAnonSnapshots` |
| Query naming | Noun-first: `loadSnapshot`, `mySnapshots`, `examplesList` |
| Args validation | Convex validator on every function arg (`v.string()`, `v.number()`, etc.) — fail fast on type mismatch |
| Auth check | Inside every mutation that requires identity: `const userId = await getAuthUserId(ctx); if (!userId) throw ConvexError("AUTH_REQUIRED")` |
| Errors | Throw `ConvexError(...)` with typed code; client maps to user-facing message |
| Pagination | Cursor-based via Convex `paginate()`; never offset-based |
| Indexes | Every query that filters or sorts uses an index; full-table scans banned |

## Error code vocabulary

Typed enum used across server surface:

| Code | Meaning |
|---|---|
| `BAD_INPUT` | Validation failure |
| `AUTH_REQUIRED` | Caller is anonymous, endpoint requires identity |
| `FORBIDDEN` | Caller authenticated but lacks permission |
| `NOT_FOUND` | Resource absent |
| `GONE` | Resource flagged / purged |
| `RATE_LIMITED` | Rate limit hit |
| `CONFLICT` | Optimistic concurrency conflict |
| `INTERNAL` | Server error (always logged, never PII in response) |

Per `book/HARD-RULES.md` "Errors quoted exact" — dev mode returns full message + stack; prod returns code + generic message.

## Headers

| Header | Default |
|---|---|
| `Content-Security-Policy` | `default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self' <convex-url>; font-src 'self' data:` — tightened per route |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains; preload` |
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `DENY` (except `/embed/*` if ever shipped) |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | `interest-cohort=()` (no FLoC), camera/microphone/geolocation all off |

## Idempotency

Content-addressed writes (snapshots) are naturally idempotent. Same input → same hash → idempotent insert. Convex `saveSnapshot` short-circuits if hash already exists.

Other writes use explicit `Idempotency-Key` request header when applicable; server tracks recent keys in Convex with TTL.

## Caught by

- `tools/lint/api-conventions.ts` greps for Server Action / Route Handler / Convex function declarations; asserts naming + validation wrapping + error shape
- Smoke test per endpoint: valid input → success shape; invalid input → error code; missing auth → AUTH_REQUIRED
- CSP test: production response includes required headers
