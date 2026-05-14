# OBSERVABILITY

Self-host, aggregate-only, privacy-respecting. Per `adr/observability-policy.md`.

## Logs

Structured JSON to stdout, captured by Caddy + Dokploy log pipeline. Every log line:

```json
{
  "ts": "2026-05-14T09:12:33.123Z",
  "level": "info | warn | error",
  "service": "next-app | convex-backend | caddy",
  "op": "saveSnapshot | loadSnapshot | render | ...",
  "duration_ms": 12,
  "outcome": "ok | error",
  "errCode": "INVALID_INPUT | RATE_LIMITED | ..." | null,
  "redacted_fields": ["..."]
}
```

PII never logged. No emails, no user IDs in plaintext (hash if identification needed for correlation).

Errors quoted exact in dev; redacted in prod.

## Metrics

Self-host Plausible Analytics (privacy-respecting, no cookies, DNT-respecting):
- Page view per route (aggregate counts only)
- Event: snapshot saved (no content, no identity)
- Event: example loaded
- Event: signin (count only)

No per-user funnel. No conversion tracking. No retargeting pixels. No third-party scripts.

## Error tracking

Self-host Sentry OR none (per `adr/observability-policy.md`).

When enabled:
- Source maps uploaded at build (private, not served to browser)
- PII scrubbed at boundary
- Rate-limited to prevent spam
- Connected to Plausible event for "error class X occurred at frequency Y"

## Real-user performance

Web Vitals (LCP, INP, CLS, FCP, TTFB) reported via Route Handler `/api/rum` with no identifiers attached. Aggregated server-side, exposed via internal dashboard.

## Internal dashboards

Self-host Grafana + Prometheus OR none. When enabled, exposed only on internal network (Caddy basic-auth or VPN-only).

Dashboards:
- Page view + sim usage aggregate
- Error rate per route
- Convex mutation latency p50/p95/p99
- Snapshot save volume (count, not content)
- CDN cache-hit ratio at Cloudflare

## Tracing

Distributed tracing via OpenTelemetry, self-host Tempo / Jaeger. Request spans across Caddy → Next → Convex. Optional, disabled in floor.

## Banned

- Third-party analytics with non-bearer persistence (Google Analytics, Mixpanel, Segment, Amplitude, Hotjar, FullStory, Heap)
- Per-user behavioral tracking
- Session recording
- Cookie-based tracking (Plausible is cookieless)
- Cross-domain tracking
- Marketing pixels
- A/B testing infrastructure that targets individuals

## Caught by

- `tools/lint/no-third-party-trackers.ts` greps for known tracker script URLs + SDK package names; zero hits
- DNT-header smoke: requests with `DNT: 1` produce no Plausible event
- Privacy budget: server logs scrub-test asserts no PII tokens slip through redactor
