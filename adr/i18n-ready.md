# i18n-ready

## Decision

Single-language English ships per `NON-GOALS.md`. Code patterns make i18n flip mechanical when trigger fires.

## Patterns enforced

### String extraction

All user-visible strings live in a single registry: `apps/web/src/i18n/strings.ts`.

```ts
export const strings = {
  nav: {
    home: 'Home',
    datapath: 'MIPS datapath',
    kmap: 'K-map',
    pipeline: 'Pipeline',
  },
  // ...
} as const;
```

Components consume via typed accessor:
```ts
const t = useStrings();
return <Button>{t.actions.save}</Button>;
```

### Banned

- Hard-coded strings in component source (except dev-only logs)
- String concatenation building user-visible text (`'Hello ' + name`) — use templated entries with interpolation slots
- Date / number / list formatting without locale-awareness (use `Intl.DateTimeFormat`, `Intl.NumberFormat`, `Intl.ListFormat`)
- RTL-hostile CSS (always use `start` / `end` instead of `left` / `right`; logical properties)

### Allowed today

- Single-locale `Intl.*` calls (en-US by default)
- ASCII-only strings
- Single string registry serving as the en-US baseline

### When trigger fires (i18n lands)

- Replace `strings.ts` with locale-keyed entries
- `useStrings()` becomes locale-aware via React context provider
- Add `next-intl` or equivalent for routing locale prefixes (`/en/`, `/es/`, etc.)
- Add per-locale Intl formatters
- Add hreflang metadata per route

The flip is mechanical because the extraction discipline is enforced from day one.

## Domain symbols

Math + Boolean operators stay as-is (`∧`, `∨`, `¬`, `⊕`, `Σ`, `Π`, etc.) — they're symbols not language. Asm mnemonics stay as-is — domain language.

## Caught by

- `tools/lint/no-hardcoded-strings.ts` greps JSX for string literals; flags any not in `strings.ts`
- `tools/lint/rtl-safe.ts` greps CSS for `left:` / `right:`; recommends `inline-start:` / `inline-end:`
- Date / number formatting smoke: assert `Intl` used at every formatting site
