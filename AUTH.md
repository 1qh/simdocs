# AUTH

Identity is optional. All features work anonymously. Signin sole purpose: cross-device persistence.

Mechanism + flow + UX rules: see `adr/auth-anon-first.md`.

## Routes

| Route | Auth requirement |
|---|---|
| `/`, `/datapath`, `/kmap`, `/pipeline`, `/compare`, `/learn/*`, `/s/[hash]` | None |
| `/me` | Signed-in only — saved snapshots list |
| `/admin` | Signed-in + `userProfiles.role === 'admin'` — abuse moderation surface |

## Provider

Google OAuth via `@auth/core/providers/google` + `@convex-dev/auth/server`. Single provider in floor.

Ratchet path: passkeys via `@auth/core/providers/passkey` lands when foundation is green per "only more never less".

## Bootstrap admin

`BOOTSTRAP_ADMIN_EMAIL` env var (semicolon-separated list of allowed admin emails). First signin matching seeds `role: 'admin'` on `userProfiles`. Subsequent signins from same email keep the role.

## Anonymous-claim

Anonymous saves stored in `localStorage` under key `anon-saved-hashes`. On signin, `claimAnonSnapshots(hashList)` server mutation links every hash to the new user; localStorage cleared.

## Signin friction

Banned anywhere in product:
- Modal walls
- "Sign up to continue"
- "Create account" buttons before any value delivered
- Email-required prompts before save
- Login redirects from public routes

Allowed:
- Single text link "sign in" in nav corner
- One dismissible toast after first anonymous save, never reappears

## Caught by

- `tools/lint/auth-wall.ts` scans for banned phrases in product copy
- E2E smoke: anon session exercises every public route, asserts zero auth redirects
- Claim-flow E2E: anon save → signin → assert `/me` lists the claimed snapshot
