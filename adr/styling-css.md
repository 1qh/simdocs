# styling-css

## Decision

Tailwind CSS latest (utility-first, CSS-first config, JIT) + CSS Modules as escape valve.

Per `STYLING.md`.

## Why Tailwind

- Utility-first composes with design-system component shape
- Active maintenance + latest stable per `book/PHILOSOPHY.md`
- JIT keeps shipped CSS minimal
- CSS-first config integrates with `packages/design-tokens` via CSS variables
- Matches operator's existing pm4ai-standard stack
- Zero runtime cost

## Rejected alternatives

- **CSS-in-JS (Emotion, styled-components, stitches)** — runtime cost, hydration cost, SSR fight
- **Vanilla Extract** — TypeScript styles are nice but bundle gen is slower than Tailwind JIT; no current operator-stack precedent
- **Panda CSS** — promising but smaller ecosystem; OSS-import-first scan ranks Tailwind ahead
- **Tailwind + UnoCSS hybrid** — extra moving parts
- **Pure CSS Modules only** — utility composition is the design-system pattern; pure modules would re-derive utilities

## CSS Modules role

Escape valve for:
- Complex keyframe animations not expressible as utilities
- State-driven complex selectors
- 3D overlay positioning where utility classes hurt readability

## Substrate constraint

`packages/three-kit`, `hud`, `design-tokens`, `editor` ship Tailwind-preset-compatible. No package-internal styling-library lock-in. Substrate-consumable from any Tailwind-using app.

`packages/design-tokens` emits both:
- `tokens.css` (raw CSS variables) — consumable by any styling system
- `tailwind.preset.ts` — opt-in for Tailwind users

Non-Tailwind apps can still consume design-tokens.

## Class composition

`cn()` is the only allowed composition utility — defined once in `packages/design-tokens` (substrate) as `twMerge(clsx(...))` wrapper. Lintmax rule enforces this. No `cva`, no bare `clsx`, no bare `twMerge`, no template-literal class assembly. Per `STYLING.md`.

## Caught by

- `tools/lint/no-css-in-js.ts` greps for emotion / styled-components / stitches imports
- `tools/lint/no-arbitrary-tailwind.ts` greps for `[#...]` / `[123px]` arbitrary values
- `tools/lint/tokens-only.ts` asserts CSS Modules use only `var(--token)` references for color / spacing / typography
- Lintmax `cn()`-only rule (operator-existing) — enforces single composition path
