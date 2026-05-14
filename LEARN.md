# LEARN

Pedagogical content layer. MDX pages with embedded 3D islands. The bridge between "sim that works" and "tool that teaches".

## Content shape

```
apps/web/content/learn/
├── intro/
│   ├── 01-what-is-a-datapath.mdx
│   ├── 02-instruction-anatomy.mdx
│   └── 03-control-signals.mdx
├── datapath/
│   ├── r-type-walkthrough.mdx
│   ├── i-type-walkthrough.mdx
│   ├── branch-resolution.mdx
│   ├── load-store-anatomy.mdx
│   └── alu-control-derivation.mdx
├── kmap/
│   ├── 01-truth-tables.mdx
│   ├── 02-grouping-rules.mdx
│   ├── 03-prime-implicants.mdx
│   ├── 04-five-six-var-toroid.mdx
│   └── 05-hazard-analysis.mdx
├── pipeline/
│   ├── 01-stages.mdx
│   ├── 02-hazards.mdx
│   └── 03-forwarding.mdx
└── cross-link/
    ├── derive-control-in-kmap.mdx       # The killer demo
    └── alu-control-derivation.mdx
```

## Tone

- Direct, accurate, terse
- No infantilizing
- No "Welcome!", no "Let's get started!"
- Lead with the diagram, then the prose
- Every paragraph has a visual companion (3D widget, table, mermaid, or equation)
- Domain language only (no school refs per `AGENT-DOCTRINE.md`)

Reference: Bartosz Ciechanowski's essays — atomic interactives, calm motion, prose that earns its place. Translated to our 3D substrate.

## Killer demo — "Derive Control in K-map"

The headline pedagogical sequence. Linked from landing page.

```
1. Page open at /learn/cross-link/derive-control-in-kmap
2. Inline truth table of opcode → RegDst signal value
3. Inline K-map (auto-loaded from the truth table)
4. User groups the cells; minimal SOP appears live
5. Click "show this function in datapath"
6. Right side of page splits — datapath scene appears
7. Stepping any instruction lights the Control unit's RegDst output;
   the K-map highlights the cell corresponding to the current opcode
8. The link is explicit, visible, and operable in both directions
```

The "I just understood why the Control unit looks like that" moment.

## Embedded interactive islands

MDX components available in learn pages:

| Component | Use |
|---|---|
| `<DatapathView instruction="add $t0,$t1,$t2" step="EX" />` | Snapshot of the datapath at a chosen step |
| `<DatapathStep instruction="lw $t0,0($t1)" autoplay />` | Animated full instruction trace |
| `<KmapView vars={4} minterms={[0,2,5,7]} />` | Snapshot of a K-map |
| `<KmapInteractive vars={4} minterms={[0,2,5,7]} />` | Full interactive K-map |
| `<PipelineDiagram program="..." cycles={10} />` | Stage-time diagram for a program |
| `<TruthTable rows={[...]} />` | Tabular truth-table widget with value-sync to surrounding components |
| `<Signal name="RegDst" instruction="add" />` | Single signal value pill with tooltip |
| `<RegisterValue name="$t0" sim={ref} />` | Live register readout from a sim instance |

Each component:
- Renders server-side as static markup with placeholder
- Hydrates to interactive island on client
- Shares state across components on the same page (one sim instance per page)

## Onboarding tour

First-visit tour, dismissible, never reappears:

1. Hover step — "this is your asm program"
2. Hover step — "this is the datapath; instructions move through it"
3. Hover step — "click step to advance, or run for autoplay"
4. Hover step — "every component lights when active; signals show in the control panel"
5. Hover step — "share button gives you a permalink to the exact state you see"

After dismissal, hint never returns. Re-accessible via `?` overlay > "show tour again".

## Mobile read-only experience

Per `adr/mobile-read-only.md`:
- All MDX learn pages fully readable on mobile
- Interactive islands → static screenshot fallback below ~1024px (rendered server-side via Playwright or equivalent at build time; cached at edge)
- Read snapshot permalink renders the static view + state summary in tabular form
- Editor + grouping interaction not enabled mobile

## Example library

Per `adr/example-library.md`. Curated preset programs.

### MIPS asm examples (~20)

- Hello (simple `add` chain)
- Sum 1..N (loop with `addi` + `beq`)
- Array sum (`lw` loop)
- GCD (Euclidean)
- Factorial (recursion via stack pseudo)
- Fibonacci (iterative)
- String length (byte-loop pseudo — once `lb` lands)
- Bubble sort
- Power of 2 (`sll`-based)
- Memory copy
- Branch-heavy (test cases)
- Load-use hazard demo (pipeline focus)
- Forwarding-resolvable demo
- Critical-path worst-case (`lw`)
- Slow-ALU experiment (configurable delay, see Critical Path)
- All-R-type
- All-I-type
- All-branch
- Pseudo-instruction zoo
- ALU-only (no memory)

### K-map example exercises (~20)

- 2-var XOR
- 2-var AND
- 3-var majority
- 3-var XOR
- 4-var BCD-to-7seg segment A
- 4-var BCD-to-7seg segment G
- 4-var carry-out (full adder)
- 4-var sum (full adder)
- 4-var 2-to-1 mux selector
- 4-var don't-care heavy (e.g. BCD invalid codes)
- 4-var hazard demo
- 5-var parity (toroidal showcase)
- 5-var majority
- 6-var (rarer, demo geometry)
- Multi-output: BCD-to-7seg full
- Multi-output: 2-bit comparator
- Multi-output: half-adder
- Multi-output: full-adder
- Cross-link: derive RegDst from datapath opcodes
- Cross-link: derive ALUSrc from datapath opcodes

Examples in `apps/web/content/examples/<slug>.mdx` with metadata frontmatter.

## Search

Cmd-K palette searches:
- Learn pages
- Example asm programs
- Example K-map exercises
- Instructions (jump to instr's walkthrough page)
- Datapath components (jump to component anatomy page)
- Registers + memory addresses + signals (in active sim)

## Caught by

- Link-check CI: every internal link in MDX resolves
- MDX-compile CI: every page compiles green
- Island-component prop validation: every component called with valid props
- Mobile-snapshot CI: every learn page produces mobile static snapshot
- Example-library presence: locked count (20+20) enforced
