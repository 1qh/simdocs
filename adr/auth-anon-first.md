# auth-anon-first

## Decision

Anonymous-first. Every sim, save, share, group, step, run, reset, scrub, compare, pipeline-view, critical-path-view works without identity. Signin is one subtle accent in the nav corner, sole purpose cross-device persistence.

## Provider

`@convex-dev/auth` + `@auth/core/providers/google` — matches byerag pattern verbatim. Single OAuth provider (Google) covers the locked floor. Passkeys + magic link grind in per "only more never less" once foundation is green.

## Flow

```mermaid
sequenceDiagram
    Anon as Anonymous user
    LS as localStorage
    Web as Next app
    Convex as Convex backend

    Anon->>Web: Use any sim, save snapshot
    Web->>Convex: saveSnapshot(body) — anonUser = null
    Convex-->>Web: hash
    Web->>LS: append hash to anonHashes[]
    Web-->>Anon: /s/<hash> permalink, ready

    Anon->>Web: (one time) Subtle toast — sign in to access from anywhere
    Anon->>Web: (optional) Click signin
    Web->>Convex: Google OAuth flow via @convex-dev/auth
    Convex-->>Web: userId
    Web->>Convex: claimAnonSnapshots(anonHashes from LS)
    Convex-->>Web: ok
    Web->>LS: clear anonHashes
```

## Signin UX

- Single small "sign in" affordance in nav corner — text link, no button shape, no badge.
- After first anonymous save: one dismissible toast — "save to your account to access anywhere" — never reappears, no nag.
- No modal walls anywhere. No "sign up to continue". No paywall.
- Signed-in adds `/me` route (saved snapshots list); UX otherwise identical.

## Bootstrap admin

`BOOTSTRAP_ADMIN_EMAIL` env var seeds an `admin` role on first matching signin, mirrors byerag pattern. Admin role used only for abuse moderation surfaces (flagged-hash review).

## Email canonicalization

Reuses byerag's `validateProfileEmail` helper. Lowercase, NFC normalize, validate per RFC. No alias stripping (`+` allowed).

## Redirect validation

Reuses byerag's `validateRedirectTo` helper. Only allows redirects to configured `SITE_URL` origin(s). Off-site redirects fail loud.

## Caught by

- `tools/lint/auth-wall.ts` asserts every route under `apps/web/src/app/*` (except `/me` + `/admin`) renders full functionality with no auth cookie.
- Smoke test boots anon session, exercises every sim feature, asserts no `401` / `403` / signin redirect.
- E2E test exercises full signin → claim → load-claimed-snapshot flow.
