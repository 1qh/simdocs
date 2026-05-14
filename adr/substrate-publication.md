# substrate-publication

## Decision

Substrate packages are public-OSS source code. Product app is private.

| Surface | Visibility |
|---|---|
| `packages/three-kit`, `hud`, `design-tokens`, `sim-engine`, `editor`, `bits`, `boolean` | Public OSS source |
| `apps/web` | Private |
| `apps/backend` (Convex functions) | Private |
| `simdocs` repo | Private (operator decision; substrate-shaped ADRs may extract into the public substrate repo when one materializes) |
| `tools/lint/*` generic ones | Public OSS (move to substrate where reusable across operator's other projects) |
| `tools/lint/*` product-specific | Private (no-school-refs, atemporal-docs scoped to this product) |

## License

Substrate packages: MIT, matches operator's existing pattern across pm4ai-standard projects. Product app: proprietary, no LICENSE file shipped.

## Repo host

Public substrate mirrors to GitHub under operator's account (precedent: operator's existing OSS tooling — paths in agent memory — already on GitHub). Private product app + simdocs on operator's primary git host per the reference deploy project pattern in memory (Forgejo or GitHub-private, confirmed at deploy phase by reading that project's config).

## Publishing as installable artifact

Substrate packages stay source-only inside this monorepo until a second consumer exists. Per `book/SUBSTRATE.md` "Premature publication anti-pattern" — extract-publish-on-second-use, not speculative future-consumer. `package.json` carries `"private": true` until extraction.

## Discipline forcing function

Per `book/SUBSTRATE.md` "Substrate stays clean BECAUSE it is published" — visible publication is the forcing function. Every substrate commit reviewed against "is this generic enough to ship to strangers". A diff that fails the test belongs in `apps/web`, not in a package.

## Caught by

- Substrate-boundary lint scans `packages/*` source for domain vocab (MIPS, K-map, datapath, kmap, opcode-specific names) — must be zero hits per `book/HARD-RULES.md` "Never reference our-codebase identifiers in docs" generalized to source.
- `package.json` `"private": true` enforced on substrate packages until publish-trigger fires (second-consumer evidence).
- Public mirror push triggered on every substrate commit to main.
