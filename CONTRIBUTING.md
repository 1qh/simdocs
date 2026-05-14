# CONTRIBUTING

Substrate packages are public OSS. This guide covers how to contribute.

(Product app is private; contributions to `apps/web` not accepted via PR.)

## Substrate package scope

Open packages accepting contributions:
- `three-kit`
- `hud`
- `design-tokens`
- `sim-engine`
- `editor`
- `bits`
- `boolean`

Each package's `README.md` describes its concern. Contributions must fit the package's stated concern.

## Code of conduct

Be technical, terse, kind. No personal attacks, no harassment. Disagreements stay on the substance.

Violations: reach out to maintainer (security disclosure address per `SECURITY.md`).

## Issue templates

### Bug

```markdown
**Package**: <package-name>
**Version**: <version>
**Description**: <one-paragraph>
**Repro**:
1. ...
2. ...
**Expected**: <what should happen>
**Actual**: <what happens>
**Environment**: browser, OS
```

### Feature

```markdown
**Package**: <package-name>
**Concern fit**: <how this fits the package's stated concern>
**Use case**: <what the contribution enables>
**Proposed API**: <code sketch>
**Alternatives considered**: <other shapes evaluated>
```

### Question

GitHub Discussions, not Issues. Issues are for actionable bugs + features.

## Pull request shape

- One concern per PR
- Conventional commit message in PR title
- Body explains why
- Linked issue if applicable
- Tests added or updated
- Foundation demo updated if API surface changed
- Substrate-boundary lint green (no domain vocab leak)
- All CI gates green

### PR template

```markdown
**Concern**: <which package + which concern>
**Change**: <one-paragraph what changed>
**Why**: <why this change>
**Tests**: <what tests added>
**Foundation demo**: <updated, or n/a>
```

## Development setup

```bash
git clone <substrate-repo>
cd <substrate-repo>
bun install
bun run dev  # foundation app for testing
```

## CI gates

Every PR runs:
- `bun run check` (lintmax pass)
- `bun test` (unit + property tests)
- Stryker mutation (≥80% per substrate package)
- Substrate-boundary lint
- License-inventory check
- Foundation-demo build

## Substrate philosophy

Substrate stays generic. Contributions that bake in domain assumptions get redirected to a product-side discussion. Per `SUBSTRATE-VS-PRODUCT.md`.

## Releases

Substrate packages are source-only until a second consumer materializes per `adr/substrate-publication.md`. Versioning + npm publish flow lands at that trigger.

## Security disclosure

Per `SECURITY.md` — responsible disclosure address provided privately, never public issue.

## Style

Per `book/PHILOSOPHY.md` "Agent-first docs" + `simdocs/AGENT-DOCTRINE.md`. Caveman docs, atemporal, only-what-shapes-decisions.

## Caught by

- PR template enforced via repo settings
- CI gates above must pass before merge
- License inventory diff
