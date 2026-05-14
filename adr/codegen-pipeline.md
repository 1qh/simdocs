# codegen-pipeline

Codegen owns every fact derivable from another fact. Hand-typed duplicates are violations.

## Pipelines

```mermaid
flowchart LR
    DatapathMD[simdocs/MIPS-DATAPATH.md tables] --> DatapathGen[bun run gen.datapath]
    DatapathGen --> DatapathTS[apps/web/features/datapath/generated/topology.ts]

    ISAMD[simdocs/MIPS-ISA.md tables] --> ISAGen[bun run gen.isa]
    ISAGen --> ISATS[apps/web/features/datapath/generated/isa.ts]
    ISAGen --> GoldenFixtures[tools/test/golden-traces/*.json]

    ConvexSchema[apps/backend/convex/schema.ts] --> ConvexGen[bunx convex codegen]
    ConvexGen --> ConvexAPI[apps/backend/convex/_generated/api.ts]
    ConvexGen --> ConvexDataModel[apps/backend/convex/_generated/dataModel.ts]
    ConvexGen --> ConvexServer[apps/backend/convex/_generated/server.ts]

    DesignTokensSrc[packages/design-tokens/src/*] --> DesignTokensGen[bun run gen.tokens]
    DesignTokensGen --> DesignTokensCSS[packages/design-tokens/dist/tokens.css]
    DesignTokensGen --> DesignTokensJSON[packages/design-tokens/dist/tokens.json]
```

## Properties

- All codegen outputs land under `generated/` paths and are committed per `book/HARD-RULES.md` "Generated artifacts committed; CI verifies regeneration".
- Pre-commit hook regenerates and diffs; staleness fails CI.
- Hand-editing a `generated/` file = violation.
- Regeneration is idempotent — running `make gen` twice produces zero diff.

## `make gen` orchestration

```
make gen:
  bun run gen.datapath
  bun run gen.isa
  bunx --filter=backend convex codegen --typecheck disable
  bun run gen.tokens
```

`make gen.check` regenerates into a temp dir and diffs against committed outputs; non-zero diff = stale = CI fail.

## Caught by

- `gen.check` CI gate.
- Pre-commit hook fires `gen` + adds resulting changes.
- Hand-editing-generated lint scans `generated/` paths for non-codegen-shape edits.
