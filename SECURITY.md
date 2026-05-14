# SECURITY

Threat model + defense layers + operator-local secrets root.

## Threat model

| Threat | Mitigation |
|---|---|
| Malicious snapshot content (XSS via shared URL) | Snapshot bodies are typed binary, deserialized via `sim-engine` codec; no `eval`, no HTML, no inline scripts. UI renders only typed fields. |
| Abuse via shared permalinks (offensive Asm comments etc.) | `flagAbuse` mutation + admin review queue; flagged hashes return 410 + cache purge at CF |
| OAuth account takeover | Standard `@auth/core` Google provider, redirect validation against allowed origins (mirrors byerag pattern), no email aliases stripped |
| CSRF | `@convex-dev/auth` provides cookie-based session; Convex mutations require auth token in body, not just cookie |
| XSS in user-authored Asm / Boolean expr | Editor sanitizes via Monaco's typed model; rendering uses React, no `dangerouslySetInnerHTML` anywhere in product |
| Rate-limit abuse (mass anonymous saves to fill storage) | Per-IP rate limit on `saveSnapshot` via Convex action limits + sliding-window counter |
| Supply-chain (compromised npm dep) | Renovate auto-bumps with minimum-release-age gate (per `book/HARD-RULES.md` "Major dep bumps reviewed within one week"), CVE gate, manifest-diff gate |
| Secrets leak | Single secrets root per `book/HARD-RULES.md` "Single secrets root", never committed, never in tracked docs |

## Defense in depth (per `book/HARD-RULES.md`)

Every safeguard has at least two independent enforcement points:

| Invariant | Layer 1 | Layer 2 |
|---|---|---|
| No raw HTML in user content | Editor's typed model rejects raw HTML at parse time | React rendering uses JSX only, no `dangerouslySetInnerHTML` |
| Auth required for `/me` | Route segment auth check via `@convex-dev/auth` middleware | Convex `mySnapshots` query asserts identity, returns 401 if anon |
| Abuse-flagged hash blocked | `loadSnapshot` returns 410 | Cache-Control header on flagged response sets max-age=0 + CF purge call |
| Owner-only mutations | Convex `mySnapshots` filters by `ownerUserId` | Route handler asserts cookie-identity matches request identity |

## Single secrets root

Per `book/HARD-RULES.md`. Single tree owned by operator. Bootstrap script reads from there, populates alternate paths via symlink. Generated / regenerable secrets (Convex deploy keys, OAuth client secrets that rotate) live outside the root, regenerate from originals.

Operator-local path lives in agent persistent memory, never in this doc.

## Required env vars (validated by Zod at boundary)

| Var | Used by |
|---|---|
| `CONVEX_SELF_HOSTED_URL` | Next + Convex client |
| `CONVEX_DEPLOYMENT_URL` | Convex deployment target |
| `CONVEX_DEPLOY_KEY` | Convex deploy CLI |
| `SITE_URL` | Auth redirect validation (allowed origins) |
| `GOOGLE_CLIENT_ID` | OAuth |
| `GOOGLE_CLIENT_SECRET` | OAuth |
| `BOOTSTRAP_ADMIN_EMAIL` | Admin role seeding |

Zod v4 schema in `apps/web/server/env.ts` + `apps/backend/convex/env.ts`. Per `book/HARD-RULES.md` "Zero fallback" — missing required env = throw at boot, never silently default.

## Logging + redaction

- No PII in logs (no emails, no user ids in plaintext logs)
- Auth tokens, OAuth secrets, Convex deploy keys never logged
- Errors quoted exact in dev; redacted in prod (full stack to internal logs only)

## TLS

- Caddy auto-provisions Let's Encrypt certs
- HSTS on at Cloudflare
- Always-HTTPS redirect

## Responsible disclosure

Substrate packages are public OSS. Vulnerability disclosures are accepted via private email to the maintainer (address provided in the substrate repo's `SECURITY.md`, never public issue).

Triage:
- Acknowledge within 72h
- Fix or patched-mitigation within 14 days for high-severity
- Public disclosure after fix lands, with credit to reporter unless they opt out
- CVE assignment for confirmed vulnerabilities affecting OSS substrate

## Caught by

- `tools/lint/no-dangerously-set-inner-html.ts` greps for `dangerouslySetInnerHTML` anywhere
- `tools/lint/no-eval.ts` greps for `eval`, `Function(` constructors
- Zod boundary validation runs on every env read
- Spec-of-code lint asserts every documented secret env appears in env schema
- Rate-limit smoke test: rapid saveSnapshot hits get 429
