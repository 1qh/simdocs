# command-palette

## Decision

Global `Cmd+K` / `:` (colon) palette in every route. Searches across:
- Routes (jump to `/datapath`, `/kmap`, `/pipeline`, `/compare`, `/learn/*`)
- Learn pages
- Example asm programs
- Example K-map exercises
- Instructions (jump to instruction's anatomy page in learn)
- Datapath components (jump to component description)
- Current sim state (registers, memory addresses, active signals) — only when sim is mounted
- Settings + commands (toggle theme, toggle reduced motion, view keyboard cheatsheet, etc.)

## Why

- Keyboard-first product per `UX-DOCTRINE.md`
- Pro-tool surface (every world-class developer tool has Cmd-K) — signals premium quality
- Single search-and-act surface reduces UX surface clutter
- Accessibility win (single shortcut accesses every action)

## Search index

Built at build time:
- Static index of routes + learn pages + examples (server-side, JSON file shipped with app)
- Dynamic index of sim state (client-side, regenerated per state change)

## UX

- Modal overlay, near-black background, blur backdrop, palette card centered
- Single input field, results list below
- Per-result: icon + title + breadcrumb path
- Arrow keys navigate, Enter selects, Esc closes
- Empty input shows "recent" + "suggested next actions" based on current route
- Fuzzy search via `mini-search` (locked per `adr/oss-import-audit.md`)

## Search index entry shape

```ts
type CommandItem = {
  id: string;
  title: string;
  description?: string;
  kind: 'route' | 'learn' | 'example' | 'instruction' | 'component' | 'signal' | 'register' | 'setting';
  breadcrumb: string;
  action: () => void;
  keywords?: string[];
};
```

## Performance

- Index search < 16ms (60fps budget)
- Fuzzy match on up to ~500 items easily under budget
- Lazy-mounted (only loaded when palette opens)

## Caught by

- E2E: open palette → search "add" → select → land on datapath with add loaded
- Performance test: 500-item search returns in < 16ms p99
- A11y test: palette is keyboard-only navigable, screen-reader announces result count
