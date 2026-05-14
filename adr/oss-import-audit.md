# oss-import-audit

OSS-import-first scan results per non-trivial concern. Each entry: chosen OSS dep + rejection rationale for alternatives + active-maintenance evidence + license compatibility + bundle-size impact.

Per `book/PHILOSOPHY.md` "OSS-import first, hand-roll only when locked" + `book/HARD-RULES.md` "Active-maintenance dep gate" (every direct dep ships upstream release within 6 months).

## Scan policy

Hand-roll allowed only when:
- Code is ≤ 30 lines AND has no maintained OSS equivalent AND is genuinely trivial
- An OSS scan returned no library meeting active-maintenance gate
- An OSS lib exists but surface mismatch requires more adapter code than the underlying lib (rare)

Hand-roll bans:
- Cryptographic / hash code (`@noble/hashes` for blake3 ✓)
- Compression — `Bun.zstdCompress` / `Bun.zstdDecompress` server, native `CompressionStream` / `DecompressionStream` client. `fzstd` / `pako` / `fflate` banned.
- Validation (`zod` v4 ✓)
- JSON canonicalization (use `safe-stable-stringify`, never hand-roll)
- Parser combinators (use `chevrotain`)
- Worker RPC — hand-rolled thin typed wrapper over `postMessage` / `MessageChannel`. `Comlink` banned per pm4ai
- Worker pool — hand-rolled thin scheduler over native `Worker` / `SharedWorker`. `workerpool` / `threads` / `tinypool` / `piscina` banned per pm4ai
- Fuzzy search (`mini-search` / `fuse.js` / `fzf-for-js`)
- Diff (`microdiff` / `deep-object-diff`)
- Toposort (`toposort`)
- Color manipulation (`culori` / `colord`)
- GPU detection (`detect-gpu`)
- Syntax highlighting (`shiki`)

## Code linting NOT in scope

`lintmax` (operator-owned, source at the path in agent memory) is the single entry point for all code linting — biome + oxlint + eslint orchestrated internally with operator-curated config.

Banned in this project's audit scope:
- Adding eslint plugins / rules / configs directly
- Adding stylelint
- Adding dependency-cruiser, knip, depcheck, size-limit (alternatives to bundlesize2)
- Any code-lint tool overlapping lintmax surface

Per `adr/lint-baseline.md`. If a code-lint pattern not covered by lintmax surfaces during build, file upstream against `lintmax`; do not add tool to this project.

## pm4ai banned-deps conformance

The operator's pm4ai banned-deps catalog at `/Users/o/z/pm4ai/packages/pm4ai/src/banned.ts` is binding across this project. Every dep in `package.json` is asserted NOT to match any entry in `ALL_BANNED`. Mapping highlights:

| Category | pm4ai locked replacement | What was banned |
|---|---|---|
| Worker RPC + pool | Native `Worker` / `MessageChannel` (hand-roll thin RPC) | `comlink`, `workerpool`, `threads`, `tinypool`, `piscina` |
| Forms | `@tanstack/react-form` | `react-hook-form`, `formik`, `final-form`, `informed`, `@hookform/resolvers` |
| Animation | `motion` (framer-motion rebrand) | `react-spring`, `@react-spring/*`, `gsap`, `anime`, `lottie-*`, `react-transition-group` |
| Markdown frontmatter | `Bun.markdown` or `remark-frontmatter` | `gray-matter`, `markdown-it`, `marked`, `react-markdown`, `micromark`, `showdown`, `turndown` |
| i18n | next.js built-in i18n + `Intl` | `next-intl`, `react-i18next`, `react-intl`, `i18next` |
| Error tracking | Error boundary + platform-managed | `@sentry`, `@bugsnag`, `@honeybadger`, `rollbar` |
| Components | `shadcn` + `cnsync` | `@radix-ui` (direct), `@mui`, `@chakra-ui`, `@mantine`, `antd`, `@nextui`, etc. |
| Icons | `lucide-react` | `@fortawesome`, `@heroicons`, `@iconify`, `@tabler/icons-react`, `phosphor-react`, `react-icons` |
| Tables | `@tanstack/react-table` | `ag-grid`, `react-table` |
| Virtualization | `@tanstack/react-virtual` | `react-virtualized`, `react-virtuoso`, `react-window` |
| Drag-and-drop | `@dnd-kit` | `react-beautiful-dnd`, `react-dnd`, `react-draggable`, `react-sortable-hoc` |
| State | `zustand` (or `jotai`) | `redux`, `mobx`, `xstate`, `valtio`, `recoil`, `effector`, `immer`, `nanostores` |
| Utilities | `es-toolkit` + native | `lodash`, `ramda`, `radash`, `remeda`, `underscore` |
| Logging | `consola` or `logtape` | `winston`, `pino`, `debug`, `bunyan`, `log4js`, `signale` |
| Date / time | Native `Intl` | `moment`, `dayjs`, `luxon`, `date-fns` (use `date-fns` only if granular date math beyond Intl) |
| HTTP client | Native `fetch` or `ky` | `axios`, `got`, `node-fetch`, `superagent`, `wretch`, `ofetch` |
| Cookies | `Bun.Cookie` or `better-auth` | `cookie`, `express-session` |
| Build | Bun built-in transpiler + `tsdown` + Turbopack | `webpack`, `rollup`, `tsup`, `esbuild` (direct), `vite`, `@babel`, `@swc` |
| Test | Bun test | `jest`, `vitest`, `mocha`, `ava` |
| Hashing (non-crypto) | `Bun.CryptoHasher` | `crc`, `crc-32`, `md5`, `sha.js`, `hash-wasm`, `xxhash-wasm` |
| Hash (blake3 specifically) | `@noble/hashes` (NOT in pm4ai banned list, not in Bun.CryptoHasher) | banned crypto-libs above |
| Compression | `Bun.zstdCompress`/`Bun.zstdDecompress` (server), native `CompressionStream`/`DecompressionStream` (client) | `pako`, `fflate`, `jszip`, `lz-string`, `fzstd` (transitional) |
| Glob | `Bun.Glob` | `fast-glob`, `glob`, `globby`, `micromatch`, `minimatch`, `tinyglobby` |
| Validation | `zod` | `joi`, `ajv`, `arktype`, `superstruct`, `yup`, `valibot` |
| UUIDs | `crypto.randomUUID()` or `Bun.randomUUIDv7` | `uuid`, `nanoid`, `cuid`, `cuid2`, `ulid`, `ksuid` |
| Shell | `Bun.$` | `execa`, `shelljs`, `zx` |
| Spawn | `Bun.spawn` | `cross-spawn`, `pidtree` |
| Env | `Bun.env` + Zod + `.env` auto-load | `dotenv`, `cross-env`, `envalid`, `@t3-oss/env` |
| TOML / YAML / CSV | `Bun.TOML`, `Bun.YAML`, `Bun.file` + string split | `@iarna/toml`, `js-yaml`, `yaml`, `csv-parse`, `papaparse` |
| File ops | `Bun.file`, `Bun.write`, `node:fs` | `fs-extra`, `graceful-fs`, `mkdirp`, `rimraf`, `del` |
| Cron | `Bun.cron` | `node-cron`, `cron`, `cron-parser` |
| Image opt | `next/image` | `imagemin`, `jimp`, `sharp` (sharp is TEMPORARY-allowed via next-pwa) |

Full source-of-truth: `/Users/o/z/pm4ai/packages/pm4ai/src/banned.ts`. Refresh per quarterly audit.

CI gate: pm4ai itself runs a banned-deps check (`pm4ai status` reports violations). Project also runs `tools/lint/oss-import-first.ts` for project-local OSS-import discipline.

## Per-concern audit

### Boolean minimization (QM / Petrick / Espresso / hazard analysis)

**Scan complete (May 2026)**: NO actively-maintained JS QM library exists. All candidates failed the active-maintenance gate or are class-project-grade with no don't-cares discipline / multi-output / typed Petrick.

| Package | State | Verdict |
|---|---|---|
| `@helander/quine-mccluskey-js` | Jan 2024 release, fails 6-mo gate | Toy-grade. Reject. |
| `quine-mccluskey-js` | ~2020, single-output only | Stale. Reject. |
| `quine-mccluskey` (root) | ~2017 | Dead. Reject. |
| `boolean-exp-minimizer` (emavola) | 2021, single-output | Stale. Reject. |
| `karnaugh-map` (npm) | Not maintained on npm | Reject. |

**Locked picks**:
- **Hand-roll QM + Petrick** (~400 LOC) — bitmask minterms (`Uint32Array` up to 32 vars, BigInt above), exact for ≤8 vars (chart-explosion cap), Petrick branch-and-bound with `AbortSignal` cancellation. Substrate exposes intermediate state (prime-implicant chart, essential-PI flags, group-membership masks) for interactive UI grouping highlight + hazard analysis. No library exposes that.
- **`espresso-iisojs`** for ≥7-var heuristic — pure JS port of Espresso-II by genieacs, MIT, native TypeScript, ~5 KB gz. Single-output only; wrap to handle multi-output via per-output call + greedy term-sharing post-pass.
- **Hand-roll hazard analysis** (~80 LOC) on top of QM output — adjacent-minterm pair check for PI-sharing in cover, consensus-term suggestion when missing.
- **Hand-roll K-map geometry** (~150 LOC) for 2D Gray-code layout + 5/6-var split-panel and 3D toroidal. Tightly coupled to render layer, library would be net-negative.

**WASM SIMD deferred** — for ≤8-var bitmask QM, JS runs <50ms; espresso-iisojs handles 12-var typical inputs <500ms. WASM SIMD (compiling classabbyamp/espresso-logic with Emscripten + `-msimd128`) gives ~3-5x on implicant-expand inner loop but adds toolchain + 80KB WASM + COOP/COEP if threading. Defer until 12+ var profile pain. Adapter shape allows one-file swap.

Substrate `packages/boolean` wraps everything with:
- Typed input/output via Zod
- Cancellation via AbortSignal (checked between Petrick branches)
- Worker-pool integration for truth-table partition strategy
- Cross-link helper `loadFromControl(opcode_bits, signal_name)` — builds truth table by simulating MIPS control ROM for every opcode, feeds minterms into `solve()`

Adapter pattern in `solver/adapter.ts`:
```ts
export async function solve(raw: unknown, signal: AbortSignal): Promise<SolveResult> {
  const input = SolveInput.parse(raw);
  const useExact = input.mode === "exact" || (input.mode === "auto" && input.vars.length <= 8);
  return useExact ? qmExact(input, signal) : espressoHeuristic(input, signal);
}
```

Result shape exposes prime implicants, essential PIs, minimal SOP/POS, hazards (pair + cover PI), stats (algorithm + ms).

### MIPS assembler / disassembler

Scan candidates:
- `mips-assembler` — npm
- `mipsjs` — npm
- `js-mips` — npm
- `mips-cpu` — npm

Locked pick: TBD at scan time. If chosen lib covers our locked instruction set (`MIPS-ISA.md` 17 encodable + ratchet to full MIPS32), import + wrap with our error-catalog typed errors + Monaco diagnostics adapter.

If NO library covers our locked ISA shape + ratchet path: hand-roll the assembler/encoder/decoder is justified (~300 LOC). The ISA is the contract per `MIPS-ISA.md`; codegen from spec.

### MIPS executor

Scan candidates:
- `node-mips` — npm
- `js-mips-simulator` — npm

Locked pick: typically hand-roll because our executor must produce per-step datapath traces (which paths/components activate per cycle), not just final state. Existing libraries return final register/memory state only. Adapter cost > write-from-scratch.

Hand-roll justified per scan rationale. Documented as the canonical executor for the locked datapath; codegen-driven from `MIPS-DATAPATH.md` per `adr/codegen-pipeline.md`.

### Snapshot canonicalize

Locked pick: `safe-stable-stringify` (most-imported, active, MIT). Wraps JSON canonical serialization with deterministic key order, no `undefined` handling surprises, BigInt support.

Fallback option: `canonicalize` (RFC 8785 JCS-compliant) if cross-platform interop with non-JS clients becomes a requirement.

Hand-roll banned per scan rationale (canonicalization correctness is too easy to break subtly).

### State diff (Sim worker → Render worker)

Locked pick: `microdiff` (smallest, fastest, active, MIT). Emits typed diff ops for object trees. Render worker applies diff to its local scene state.

Rejected alternatives:
- `deep-object-diff` — larger bundle, slower, redundant API
- `fast-diff` — string-diff only, wrong shape
- Hand-roll — bug-risk for nested typed arrays

### Boolean expression parser (K-map input)

Locked pick: `chevrotain` for typed AST + Monaco grammar integration. Active, fast, MIT.

Rejected alternatives:
- `peggy` — PEG generator, runtime parser generation, larger bundle
- `parsimmon` — combinator-based, less Monaco-friendly
- Hand-roll recursive-descent — would re-derive Pratt + error recovery + Monaco diagnostics; OSS-import-first scan favors chevrotain

### Fuzzy search (command palette)

Locked pick: `mini-search` (smallest, in-memory, fast, MIT, active). Single-file ≤ 30 KB. Suits ~500-item search index.

Rejected alternatives:
- `fuse.js` — heavier bundle, more features than needed
- `fzf-for-js` — port-quality varies, less maintained
- Native `String.includes` filter — works for trivial case but fuzz tolerance is the point

### Worker pool + work-stealing

Locked pick: hand-rolled thin scheduler over native `Worker` / `SharedWorker`. pm4ai `worker` ban catalogs `comlink`, `workerpool`, `threads`, `tinypool`, `piscina`, `worker-threads-pool` — all worker-pool libraries are banned in operator's portfolio.

Implementation: ~50 LOC pool over native Workers + ~50 LOC work-stealing extension over sibling queues. Hand-roll justified per pm4ai banned-list — no permitted alternative.

Worker RPC: ~30 LOC typed-RPC wrapper over `postMessage` / `MessageChannel` / `Transferable`. No `Comlink` (banned).

Rejected alternatives:
- `Comlink` — banned per pm4ai worker policy
- `workerpool` — banned per pm4ai worker policy
- `threads.js` / `tinypool` / `piscina` — banned

### Device tier detection

Locked pick: `detect-gpu` (npm, MIT, active). Returns GPU tier 0-3 based on benchmark database keyed by GPU model string. Combined with `hardwareConcurrency` + `deviceMemory` for our `high`/`mid`/`low` tier per `adr/adaptive-quality.md`.

### Color manipulation (color-blind palette variants)

Locked pick: `culori` (most-comprehensive color science, supports OKLCH, MIT, active). Used to derive deuteranopia/protanopia/tritanopia accent shifts from base palette deterministically.

Rejected alternatives:
- `colord` — smaller bundle but less complete color-space coverage
- `chroma-js` — older, larger
- Hand-roll — color-space math is exactly the kind of thing OSS-import-first guards against

### Toposort (substrate dependency graph for build / foundation order checks)

Locked pick: `toposort` (npm, MIT, ~ten years active, trivial API). Used by `tools/lint/foundation-order.ts` to verify foundation-bootstrap-order.md phase graph is acyclic + reachable.

### MDX code-block syntax highlighting

Locked pick: `shiki` (microsoft, MIT, active, ships compiled TextMate grammars). Used inside `/learn/*` MDX for `lang="asm-mips"` and `lang="boolean"` code blocks.

Custom grammars for MIPS asm + Boolean expression registered with shiki at build (extends `chevrotain` grammars from K-map parser).

### Markdown frontmatter parsing (MDX examples)

Locked pick: `remark-frontmatter` (allowed via `remark`/`rehype` in pm4ai ALLOWED_STACK) + `remark-parse-frontmatter` for typed parsing.

Rejected alternatives:
- `gray-matter` — banned per pm4ai `markdown` policy
- `Bun.markdown` — server-side option; remark pipeline preferred for `@next/mdx` integration

### Schema-driven form generation (settings panel, K-map input)

Locked pick: `@tanstack/react-form` + Zod resolver. Forms over Zod schemas.

Rejected alternatives:
- `react-hook-form` — banned per pm4ai `forms` policy
- `@hookform/resolvers` — banned (consumed by react-hook-form)
- `formik` / `final-form` / `informed` — banned

### Date/time

Native `Intl.DateTimeFormat`. No library import.

### Number formatting

Native `Intl.NumberFormat`. No library import.

### Animation engine

Locked pick: `motion` (the framer-motion rebrand, present in pm4ai ALLOWED_STACK). Both DOM motion + R3F transitions via `motion`.

For per-frame spring physics inside R3F `useFrame` (where `motion` doesn't fit), hand-rolled critically-damped spring (~30 LOC) using pooled `Vector3` per `adr/runtime-optimization-discipline.md`.

Rejected alternatives:
- `@react-spring/web` / `@react-spring/three` — banned per pm4ai `animation` policy
- `react-spring` — banned
- `gsap` / `anime` / `lottie-web` / `lottie-react` / `react-flip-move` / `react-transition-group` / `@formkit/auto-animate` — all banned

### State machine (sim-engine)

Hand-roll justified — sim is pure-function `(state, action) → state'`, ≤ 100 LOC executor + step driver. XState is event/state-chart-shaped, overkill for monotonic cycle stepping; adapter cost > hand-roll cost. Documented per scan rationale.

### Object pool (Three.js Vector3/Quaternion/Matrix4)

Hand-roll justified — ~30 LOC per pool type. No mature OSS pool sized for this exact shape.

### Bits utilities (two's complement, sign-extend, base conversions)

Hand-roll justified — `~60 LOC`, no mature OSS replacement for this exact shape. `bitwise-buffer` exists but operates on buffers; ours operates on numbers + typed arrays.

### Snapshot tier-1 fragment encoding (base64url)

Native `btoa` / `atob` doesn't handle URL-safe base64 directly. Tiny wrapper (~10 LOC) hand-roll OR `uint8-to-base64` npm. Locked: hand-roll (~10 LOC), trivial threshold.

## CI gate

`tools/lint/oss-import-first.ts`:
- Greps `packages/*/src` + `apps/web/src` + `apps/backend/convex` for hand-rolled patterns that match libraries above (canonicalize, fuzzy search, worker pool, etc.)
- Asserts each hand-roll has matching exception entry in this ADR with rejection rationale
- Asserts each library above appears as a direct dep in the consuming package

## Audit refresh cadence

Quarterly. Re-scan npm + GitHub for new libraries in each category. Update ADR with newer-better choices when found + migrate.

Trigger immediate re-scan when:
- Hand-rolled component exceeds 100 LOC (signal that we're past trivial threshold)
- A locked library fails active-maintenance gate (6-month dep release gate)
- Operator pushback names a specific library we missed
