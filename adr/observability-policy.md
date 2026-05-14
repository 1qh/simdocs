# observability-policy

## Decision

Self-host aggregate-only privacy-respecting:
- Plausible Analytics for page-view + aggregate event counts (self-host operator)
- Structured JSON logs to stdout, captured by Dokploy log pipeline
- Optional self-host Sentry (deferred unless triggered)
- Optional self-host Grafana + Prometheus (deferred unless triggered)
- Optional OpenTelemetry tracing (deferred unless triggered)

## Why not third-party hosted

- No persistent user identity at any third party
- Per `book/PHILOSOPHY.md` self-host first + bearer-mode-or-self-host invariant
- Aggregate-only respects user privacy without consent banners

## Plausible config

- Self-host Plausible container in operator zoo
- Single site = our domain
- Cookieless (default Plausible behavior)
- DNT-respecting
- Events tracked: page views, snapshot saved (count only), example loaded, signin (count only)
- No custom dimensions tied to user identity

## Log format

Structured JSON to stdout. Caddy log → Dokploy → operator's log pipeline.

Fields:
- `ts`, `level`, `service`, `op`, `duration_ms`, `outcome`, `errCode`, `redacted_fields`

PII redaction at boundary: emails, user IDs, OAuth tokens, request bodies containing such → hash or redact before log emit.

## Banned

- Google Analytics, Mixpanel, Segment, Amplitude, Hotjar, FullStory, Heap, Plausible-Cloud (use self-host)
- Per-user behavioral tracking
- Session recording
- A/B testing infrastructure that targets individuals
- Cross-domain tracking

## Triggers to add deferred layers

- Self-host Sentry: error rate exceeds operator-bearable diagnostic threshold via logs alone
- Self-host Grafana: operator needs visual dashboards
- OpenTelemetry: distributed tracing needed (low-priority for current scope)

Each trigger lands as ADR amendment, never speculative.

## Caught by

- `tools/lint/no-third-party-trackers.ts` greps for known tracker SDKs / script URLs
- Log scrub-test: PII tokens not present in log output
- DNT-header smoke: requests with `DNT: 1` produce no Plausible event
- Self-host Plausible compose service runs in `verify.local`
