# multi-env

## Decision

Three deployment environments: `dev` (operator's laptop compose stack), `staging` (live VM, smoke + Lighthouse-CI target), `prod` (live VM, user-facing).

Each environment runs the identical stack (per `book/PHILOSOPHY.md` local-first-hostability invariant). Only env values, scale, and DNS differ.

## Topology

```mermaid
flowchart LR
    Laptop[Operator laptop — dev<br/>compose.yaml with self-host Convex backend]
    Staging[Staging VM — dokploy<br/>Cloudflare DNS at staging dot domain<br/>own Convex instance own DB]
    Prod[Prod VM — dokploy<br/>Cloudflare DNS at domain<br/>own Convex instance own DB]
```

## Per-environment differences

| Aspect | dev | staging | prod |
|---|---|---|---|
| Domain | `localhost:3000` | `staging.<domain>` | `<domain>` |
| Convex instance | local self-host container | separate self-host instance | separate self-host instance |
| OAuth client | dev-client-id | staging-client-id | prod-client-id |
| Plausible | local self-host or disabled | tracking on, separate site | tracking on |
| Sentry | disabled | enabled if landed | enabled if landed |
| Cache TTLs | minimum (live editing) | production TTLs | production TTLs |
| Rate limits | high (no throttle annoyance for solo dev) | production limits | production limits |
| Logs | verbose, stdout | structured JSON to log pipeline | structured JSON to log pipeline |

## Env variable shape

Per environment loaded from operator's secrets root:
- `<secrets-root>/sim/env.dev.env`
- `<secrets-root>/sim/env.staging.env`
- `<secrets-root>/sim/env.prod.env`

Bootstrap selects via `ENV=<env-name> sh tools/bootstrap-mac.sh`.

## Deploy flow

```mermaid
flowchart LR
    Code[git push origin main] --> CI[CI runs full gate suite]
    CI --> StageDeploy[Deploy to staging]
    StageDeploy --> StageSmoke[Smoke + Lighthouse-CI green]
    StageSmoke --> ProdDeploy[Deploy to prod via make deploy.prod]
    ProdDeploy --> ProdSmoke[Smoke prod]
```

Prod deploy never bypasses staging green.

## Promotion

Staging → prod promotion: `make deploy.prod` which re-tags the staging-tested image and applies the same compose+helm values to prod. No rebuild in prod path.

## Data flow between environments

- Staging Convex instance has fixture data (operator-curated example snapshots)
- Prod Convex instance has real user data
- Dev Convex instance is ephemeral; reset on `make reset.cache`
- Never copy prod data to staging or dev (privacy)

## Banned

- "Quick fix in prod" deploys that skip staging
- Sharing OAuth client between staging and prod
- Sharing Convex instance between staging and prod
- Shared secrets across environments

## Caught by

- CI gate: prod deploy requires staging-green tag
- Env-name lint asserts every deployed env reads its own env file
- Smoke per environment after every deploy
