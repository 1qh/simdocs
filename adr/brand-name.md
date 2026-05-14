# brand-name

## Decision

Nameless on product surface. Workspace slug `sim` fills every identifier requirement (package.json `name`, internal CLI, og:title fallback, breadcrumbs, README, npm-scope when extraction triggers).

UI copy uses domain language only:
- "MIPS datapath visualizer"
- "K-map tool"
- "Pipeline view"
- "Compare instructions"
- "Critical path"

No invented product name in nav, heading, OG card, page title, or copy.

## Why

Branding is product-side, founder-owned per `book/SUBSTRATE.md` substrate-vs-product split. Founder-owned items the agent does not self-decide. Naming chosen later when triggering condition fires — domain registration, first public publication, or explicit founder pick.

## Trigger to revisit

- Operator registers a domain for deploy
- Operator explicitly picks a name
- First public substrate or product launch announcement is drafted

When trigger fires: OSS-import-first-shaped scan over candidate names (npm availability, domain availability, trademark check, github-org availability) returns a shortlist with recommendation. Founder picks; ADR amended to lock the pick.

## Substrate package naming

Substrate package names already locked: `three-kit`, `hud`, `design-tokens`, `sim-engine`, `editor`, `bits`, `boolean`. These are concern-named, not product-named — survive any future brand-rename without churn. Per `book/HARD-RULES.md` no-numbering, kebab-case.

## Caught by

- `tools/lint/no-school-refs.ts` already enforces banned vocab; extends to "no invented product brand" once a brand-name is picked (reverse-check: brand name appears in source = `tools/lint/brand-presence.ts`).
- Doc-lint asserts no copy on product pages claims a product name other than the locked one (currently empty — every claim that's not a domain term is suspect).
