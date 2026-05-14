# repo-layout

## Decision

Two repos: `sim` (code, monorepo) + `simdocs` (this docs corpus, flat).

```
sim/
├── apps/
│   ├── web/                  # Next.js product app
│   └── backend/              # Convex schema + functions (self-host instance)
├── packages/
│   ├── three-kit/
│   ├── hud/
│   ├── design-tokens/
│   ├── sim-engine/
│   ├── editor/
│   ├── bits/
│   └── boolean/
├── tools/
│   ├── lint/                 # stack-presence, staleness, agent-first-output, zero-fallback, no-school-refs
│   ├── bootstrap/
│   ├── smoke/
│   └── ledger/
├── compose.yaml              # full operator zoo, local-first
├── Dockerfile.web
├── Caddyfile
├── Makefile
├── turbo.json
├── package.json              # bun workspaces
├── lintmax.config.ts
├── tsconfig.json
├── up.sh                     # pre-commit driver (per byerag pattern)
├── clean.sh
├── CLAUDE.md                 # short pointer to simdocs/CLAUDE.md
└── README.md                 # 4-line quickstart + book + simdocs pointers

simdocs/                       # flat docs, this corpus
├── *.md                       # foundationals + spec + surface
└── adr/                       # one file per decision
```

## Why

- Two repos enforce `book/PHILOSOPHY.md` "One import graph, one home per concept" — docs never import code; code consumes specs by name (string match validated by CI lint).
- Monorepo in `sim` enforces substrate-vs-product split physically — substrate (`packages/*`) vs product (`apps/web/**`) lives on disk, commit path lint catches mixed-mode.
- Flat `simdocs` matches operator's existing docs-repo pattern (`byerag-docs`, `eximdocs`, `vadocs`).
- ADRs under `adr/` topical kebab-case, no numbering per `book/HARD-RULES.md` no-numbering rule.

## Required components

Every entry above is mandatory. Missing one = `book/HARD-RULES.md` violation. Bootstrap order per `adr/foundation-bootstrap-order.md`.

## Caught by

- Repo-layout-presence lint at root of `sim` walks the inventory above and asserts each path exists.
- Commit-path lint asserts staged paths fit one of: `packages/*`, `apps/*`, `tools/*`, root-config files. Mixed substrate+product commits = ADR-approved or split.
