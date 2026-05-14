# persistence-convex

## Decision

Convex self-host instance. `CONVEX_SELF_HOSTED_URL` env var points the client at the operator-managed Convex backend. Pattern derived verbatim from the operator's existing Convex+auth + deploy reference projects (paths in agent memory).

## Why

- Convex self-host satisfies `book/PHILOSOPHY.md` Local-first hostability invariant — `ghcr.io/get-convex/convex-backend` runs in compose locally with the same image used in cluster.
- Single primitive collapses DB + functions + file storage + real-time subscriptions + scheduled jobs + auth integration. Avoids speculative generality of building separate substrate packages for each concern per `book/SUBSTRATE.md` "Premature publication" anti-pattern.
- SSOT — Convex schema in TS is the single source of truth, codegen produces typed client + server APIs.
- Operator precedent — the reference Convex+auth project (in memory) uses this exact pattern; reusing pattern reduces ops surface across operator's portfolio.
- Migration freedom — Convex export/import is the documented swap path if persistence ever needs to move off Convex.

## Schema home

`apps/backend/convex/schema.ts` is the SSOT. Convex codegen produces `_generated/api.ts`, `_generated/dataModel.ts`, `_generated/server.ts`. Both `apps/web` and `apps/backend` consume the typed API. Per `adr/ssot-precedence.md` Convex schema row.

## Tables

| Table | Purpose |
|---|---|
| `users` | Identity row, populated by `@convex-dev/auth` Google provider |
| `userProfiles` | Role + display fields, mirrors operator's reference Convex+auth project pattern (in memory) |
| `snapshots` | Content-addressed snapshot rows — `hash`, `bodyStorageId`, `createdAt`, `ownerUserId` (nullable for anon), `abuseFlag` |
| `anonClaims` | Hash → userId mapping, populated on signin to claim localStorage-tracked anon saves |

Full schema in `SCHEMAS.md`.

## Functions

| Function | Type |
|---|---|
| `saveSnapshot(canonicalBody, contentType)` | mutation, returns hash |
| `loadSnapshot(hash)` | query, returns body + metadata |
| `claimAnonSnapshots(hashList)` | mutation, signin-only, claims listed hashes for current user |
| `mySnapshots()` | query, signin-only, returns user's owned snapshots |
| `flagAbuse(hash, reason)` | mutation, sets abuseFlag, idempotent |

## Auth integration

`@convex-dev/auth` + `@auth/core/providers/google`. Per `adr/auth-anon-first.md`.

## Local-first hostability

`compose.yaml` runs `convex-backend` self-host container alongside `next-app`. Operator zoo runs on a single MacBook serving real traffic. Cluster topology uses same image scaled.

## Migration

- Self-host → other self-host: `convex export` + `convex import`, same image either side.
- Convex → another persistence layer (Postgres + S3): triggered only on documented evidence (Convex limits hit, second product needs different shape). Extraction owns sim-engine codec (pure-function, already extract-ready) + new persistence wrapper.

## Caught by

- Convex schema → typed-API codegen idempotency check.
- Smoke test boots Convex self-host in compose, runs round-trip save → load.
- `make verify.fresh` exercises full bootstrap.
