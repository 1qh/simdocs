# secrets-rotation

## Decision

Every long-lived secret rotates on a documented cadence. Rotation procedures scripted.

## Inventory + cadence

| Secret | Cadence | Trigger | Procedure |
|---|---|---|---|
| Google OAuth client secret | Annual | Calendar reminder + incident | `RUNBOOK.md` "Rotate Google OAuth client secret" |
| Convex deploy key | Quarterly | Calendar + incident | `bunx convex auth rotate-deploy-key` |
| Convex instance admin key | Quarterly | Calendar + incident | Convex CLI rotate |
| Cloudflare API token | Annual | Calendar + incident | Generate new token, update operator secrets root, deploy |
| Sentry DSN (when enabled) | n/a | Public, embedded in client; rotates on Sentry project rotation only | Generate new project, update env, redeploy |
| Plausible API key (when self-host) | Annual | Calendar + incident | Plausible admin UI rotate |
| Bootstrap admin emails | n/a | When admin set changes | Update `BOOTSTRAP_ADMIN_EMAIL` env, redeploy |
| JWT signing keys (if added) | Quarterly | Calendar + incident | Per JWT rotation runbook (drafted when JWT lands) |

## Rotation discipline

- Generate new before revoking old (expand-contract per `book/HARD-RULES.md`)
- Test new in staging before promoting
- Revoke old only after prod has rolled to new for ≥24h
- Calendar reminder per secret-cadence pair; operator's calendar

## Incident rotation

Triggered when a secret is suspected leaked:
1. Generate new immediately
2. Push to staging + prod simultaneously (no expand-contract for compromise)
3. Revoke old immediately
4. Audit logs for usage of old between leak and revocation
5. Postmortem captured in `GOTCHAS.md`

## Automation candidates (deferred)

- Renovate-style auto-rotate for credentials with API support — deferred until first rotation pain.

## Caught by

- Calendar reminders (operator-side)
- Audit log retained per `OBSERVABILITY.md` structured logs
- Smoke after each rotation: signin works, mutation works, deploy works
- Secret-expiry-date warning in dashboard (when observability dashboard exists)
