# RUNBOOK

Operator emergency + routine procedures. Concrete commands, not theory.

## Routine

### Deploy a new build

```bash
cd ~/sim
bun run check          # full local check, green required
bun run build          # turbo build
make deploy            # dokploy apply + smoke
```

Smoke fails → roll back via `make rollback` (re-applies previous tagged image).

### Add a new substrate package

```bash
cd ~/sim
bun scripts/new-package.ts <name>
```

Scaffolds `packages/<name>/` with `package.json`, `tsconfig.json`, `src/`, foundation-demo entry under `apps/web/learn/foundation/<name>/`. Per `adr/monorepo-package-split.md`.

### Add a new MIPS instruction to the encodable tier

```bash
# 1. Update simdocs/MIPS-ISA.md opcode + funct table
# 2. cd ~/sim
bun run gen.isa        # regenerates apps/web/features/datapath/generated/isa.ts
# 3. Add golden-trace fixture under tools/test/golden-traces/<mnemonic>.json
bun test golden        # asserts new fixture matches executor output
```

### Add a new K-map example exercise

```bash
# Edit apps/web/content/kmap-examples/<slug>.mdx
# Snapshot test auto-runs on commit
```

### Rotate Google OAuth client secret

```bash
# 1. Generate new secret in Google Cloud Console (founder UI click)
# 2. Update operator-local secrets root:
$EDITOR <secrets-root>/google-oauth-client.env
# 3. Push to Convex env:
cd ~/sim/apps/backend
bunx convex env set GOOGLE_CLIENT_SECRET "$(cat <secrets-root>/google-oauth-client.env)"
# 4. Push to Next env:
make env.sync
# 5. Smoke:
bun scripts/smoke-auth.ts
```

### Restore Convex from backup

```bash
# Backup location per operator reference deploy pattern (path in agent memory): <secrets-root>/convex-backup/<timestamp>.tar.zst
cd ~/sim/apps/backend
bunx convex import --replace <secrets-root>/convex-backup/<timestamp>.tar.zst
```

### Add a snapshot schema version (expand-contract)

```bash
# 1. Add new optional field in apps/backend/convex/schema.ts
# 2. Bump packages/sim-engine version handler
# 3. Commit old-version fixture under tools/test/replay-fixtures/v<N>/
bun test replay        # asserts old fixtures still replay
```

## Emergency

### Flag-then-purge an abusive shared snapshot

```bash
# 1. Flag in Convex
cd ~/sim/apps/backend
bunx convex run admin:flagAbuse '{"hash": "<hash>", "reason": "<reason>"}'

# 2. Purge from Cloudflare cache
curl -X POST "https://api.cloudflare.com/client/v4/zones/<zone-id>/purge_cache" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"files": ["https://<domain>/s/<hash>"]}'

# 3. Verify
curl -I "https://<domain>/s/<hash>"   # should return 410 Gone
```

### Roll back a bad deploy

```bash
cd ~/sim
make rollback          # redeploys previous tagged image via dokploy
make smoke.deploy      # verifies
```

### Convex backend down

```bash
# 1. Check dokploy logs:
dokploy logs convex-backend
# 2. Restart:
dokploy restart convex-backend
# 3. If data corruption suspected:
make convex.restore.latest
```

### Caddy / TLS issues

```bash
# 1. Check Caddy logs:
dokploy logs caddy
# 2. Renew certs manually:
dokploy exec caddy caddy reload
# 3. Verify TLS:
curl -vI https://<domain>
```

### Rate-limit triggered for a legitimate user

```bash
# Per-IP rate limit is in Convex action limits + Valkey sliding-window
# Adjust threshold (avoid in floor; investigate first):
cd ~/sim/apps/backend
bunx convex env set RATE_LIMIT_SNAPSHOT_PER_HOUR <new-value>
```

### Bootstrap a fresh operator machine

```bash
# 1. Restore secrets root from operator backup
unzip <operator-backup>.zip -d <secrets-root>

# 2. Clone repos
git clone <sim-repo-url> ~/sim
git clone <simdocs-repo-url> ~/simdocs

# 3. Bootstrap
cd ~/sim
sh tools/bootstrap-mac.sh

# 4. Verify
make verify.fresh
```

## Diagnostic

### Tail logs from deployed instance

```bash
dokploy logs <service-name> --follow
```

### Check Convex deploy status

```bash
cd ~/sim/apps/backend
bunx convex deploy --dry-run
```

### Inspect a stuck snapshot

```bash
cd ~/sim/apps/backend
bunx convex run snapshots:inspect '{"hash": "<hash>"}'
```

### Check CDN cache hit ratio

```bash
# Cloudflare dashboard or API:
curl "https://api.cloudflare.com/client/v4/zones/<zone-id>/analytics/dashboard" \
  -H "Authorization: Bearer $CF_API_TOKEN"
```

## Caught by

- Each procedure above has a corresponding tracked script under `tools/runbook/<procedure>.sh`
- Smoke tests cover the happy path of each routine procedure
- Emergency procedures exercised quarterly (game day)
