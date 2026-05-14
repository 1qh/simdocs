# editor-monaco

## Decision

Monaco editor wrapped in `packages/editor`. Custom language definitions for:
- MIPS assembly (asm editor in `/datapath` route)
- Boolean expression (truth-table-source editor in `/kmap` route)

## Why Monaco

- Largest TS-ecosystem editor with real Language Server-like extensibility
- Active maintenance (Microsoft, monthly releases)
- Vendor-neutral OSS (MIT)
- Tooling already familiar (VS Code is Monaco)
- Custom language registration is well-documented

## Rejected alternatives

- **CodeMirror 6** — viable, smaller bundle, but Monaco's tokenizer/IntelliSense surface is richer and our asm editor benefits from completion + signature help.
- **Custom textarea + Prism** — reinventing tokenizing + completion + diagnostics; OSS-import-first violation.
- **Ace** — older, less actively maintained.

## Editor package shape

```
packages/editor/
├── src/
│   ├── EditorMount.tsx        # React mount with theme integration
│   ├── language/
│   │   ├── registerLanguage.ts  # Generic registration helper
│   │   └── monarchToTokens.ts   # Helper for Monarch tokenizer definitions
│   ├── theme/
│   │   └── industrialTheme.ts   # Matches design-tokens
│   └── markers/
│       └── setDiagnostics.ts     # Pushes parse errors as squigglies
├── package.json
└── tsconfig.json
```

Substrate-pure: knows nothing about MIPS or Boolean grammars. Product features (`apps/web/features/datapath/asm-grammar.ts`, `apps/web/features/kmap/boolean-grammar.ts`) register their grammars via the editor's helpers.

## Bundle splitting

Monaco is heavy (~2MB). Loaded as client-only dynamic import, gated behind editor mount. RSC paths (landing, learn, `/s/[hash]` view-only) skip Monaco entirely.

## Theme integration

`industrialTheme` consumes `design-tokens` palette so editor colors match the app's color system. Single source of truth = `design-tokens`.

## Caught by

- Bundle-size budget — `apps/web` initial bundle excludes Monaco (only loads on editor route).
- Editor mount smoke test boots Monaco, registers a toy grammar, asserts tokens render.
- A11y test: Monaco respects keyboard nav, screen reader announces text content.
