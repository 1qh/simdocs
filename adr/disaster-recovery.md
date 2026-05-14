# disaster-recovery

## Decision

RPO (Recovery Point Objective): 24 hours. Data loss bounded by daily backup.
RTO (Recovery Time Objective): 4 hours. From total infrastructure loss to operational.

Backup, restore, tested-restore protocol.

## Backup strategy

| Asset | Frequency | Retention | Destination |
|---|---|---|---|
| Convex backend data | Daily | 30 days rolling + monthly snapshot retained 1 year | Self-host MinIO or operator's backup destination (per claude2b pattern) |
| Caddy state (TLS certs, cache cert keys) | Weekly | 30 days | Same |
| Operator-original secrets root | Manual after every change | Operator's personal backup | Per `book/HARD-RULES.md` "Single secrets root" |
| Repo state | Push to remote | Forever | Git remote (host per `adr/repo-layout.md`) |

## Convex backup mechanism

```bash
# Scheduled daily at 03:00 UTC via cron in dokploy
cd /opt/sim/apps/backend
bunx convex export --path /backups/convex/$(date -Iseconds).tar.zst
# Sync to off-site
rclone copy /backups/convex/ <backup-destination>:sim/convex/
```

## Restore protocol (tested quarterly)

```bash
# 1. Fresh VM provisioned (dokploy or new VM)
# 2. Restore secrets root from operator backup
unzip <operator-backup>.zip -d <secrets-root>

# 3. Clone + bootstrap
git clone <sim-repo-url> /opt/sim
git clone <simdocs-repo-url> /opt/simdocs
cd /opt/sim
sh tools/bootstrap-mac.sh   # works on Linux too

# 4. Restore Convex
bunx convex import <latest-backup>.tar.zst

# 5. Smoke
make verify.bearer
make smoke.deploy
```

## Tested restore (game day)

Quarterly. Operator simulates total infrastructure loss:
1. Spin up a new VM
2. Execute restore protocol from operator backup + git remote only
3. Time to operational measured against RTO
4. Data loss measured against RPO

Failure → restore-time gap landed in `GOTCHAS.md`, addressed in next backup-procedure update.

## Failure scenarios + responses

| Scenario | Response |
|---|---|
| Convex container crash | Dokploy auto-restart; if persistent, `dokploy logs convex-backend` → fix |
| Convex data corruption | Restore latest daily backup via `convex import` |
| VM hardware failure | Provision new VM, execute restore protocol |
| Cloudflare account compromise | Re-point DNS to direct origin; revoke API token; restore once new DNS provider in place |
| Git remote loss | Push to alternative remote; substrate repo is OSS-mirrored so canonical history survives |
| Operator-original secrets loss | Catastrophic — re-establish all OAuth client credentials, regenerate all keys; not recoverable from system, only from operator's personal backup |

## Banned

- Backup destination same as production data location
- Untested-for-≥6-months restore procedure
- Backup retention shorter than 30 days
- Plaintext credentials in backup archives without encryption

## Caught by

- Cron schedule for daily Convex export
- Quarterly tested-restore game day on operator's calendar
- Restore-time measurement landed in operations ledger per `book/PHILOSOPHY.md` "Ledger consultation"
- Backup-size + recency monitored via daily smoke (alerting if backup hasn't appeared)
