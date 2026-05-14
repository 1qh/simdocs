# REQUIREMENTS

Locked floor for the product. Every item ships. Items outside the floor live in `NON-GOALS.md` with trigger.

## MIPS visualizer

```mermaid
mindmap
  root((MIPS))
    Input
      Asm editor with syntax highlight and inline errors
      Instruction picker quick select
      Encoding view machine code hex and binary
      Pseudo instruction expansion visible
    Datapath
      Single-cycle topology fixed
      Per-instruction step animation IF ID EX MEM WB
      Active components lit per step
      Control signal values shown per step
      Signal propagation visualized as light travelling buses
      Critical path overlay structural and timing weighted
    Execution
      Step forward
      Step back
      Run to cursor
      Run to breakpoint
      Reset
      Speed control
    State
      32 GPRs with named aliases
      PC HI LO
      Data memory hex plus ASCII plus disasm
      Diff highlight per step
    Pipeline
      Stage time diagram instructions versus cycles
      Hazard detection RAW WAW WAR control
      Forwarding lines lit when active
      Stalls and bubbles shown
      CPI counter cycle count instruction count
    Comparison
      Side by side two instructions
      Synchronized step
      Diff highlighting
    Debug
      Breakpoints on PC mem reg signal
      Watchpoints reg write mem write
      Trace log
    Syscalls
      Print int print string read int read string exit
    Persistence
      Save snapshot
      Content addressed permalink
      Anonymous claim on later signin
```

### Instruction set

- **Datapath-animated floor**: `add addi and beq bne lw slt or sw sub`
- **Encodable floor**: + `andi j lui nor ori sll srl`
- **Full MIPS32 ratchet**: every official instruction grinds in per "only more never less"

### Datapath topology

Single-cycle MIPS datapath, CS2100-variant (no school refs in product surface). Topology, paths, segments, value ids, control signals frozen against the reference implementation at `~/mips/ref/src/core/mips/single-cycle/`. See `MIPS-DATAPATH.md`.

### Control signals

`RegDst`, `ALUSrc`, `MemToReg`, `RegWrite`, `MemRead`, `MemWrite`, `Branch`, `BranchNE`, `ALUOp ∈ {00,01,10}`, `PCSrc` (derived). See `MIPS-DATAPATH.md`.

## K-map tool

```mermaid
mindmap
  root((Kmap))
    Input
      Truth table values
      Boolean expression
      Minterm list sigma m
      Maxterm list pi M
      Don't cares supported
      Multi output supported
    Geometry
      2D for less than or equal four variables
      3D toroidal for five or more variables
      Cells as raised blocks output height
      Wrap edges rendered as actual geometry
    Interaction
      Click drag rectangle grouping
      Snap to power of two sizes
      Wraparound aware groups
      Overlap allowed
    Output
      All prime implicants
      Essential prime implicants
      Minimal SOP
      Minimal POS
      Hazard analysis static hazards highlighted
      Hazard cover suggested
      Gate level circuit AND OR NAND only NOR only
    Solver
      Quine McCluskey deterministic
      Petrick selection
      Espresso heuristic for larger fns
      Multi output minimization shared terms
    Modes
      Practice user groups
      Reveal solution toggle
      Step through QM table
    Persistence
      Save exercise
      Permalink
      Anonymous claim on later signin
```

### Variable range

2 to 6 variables. Beyond 6 → Espresso heuristic, no K-map geometry (out of scope, see `NON-GOALS.md`).

## Cross-sim integration

The MIPS datapath's Control unit and ALU Control are themselves Boolean functions. The K-map tool can ingest the truth table of any control signal as a function of `(opcode, funct)` and visualize its minimization. This link is the headline pedagogy of the product. See `KMAP.md` "Datapath cross-link".

## Persistence

- Anonymous-first, all features available without signin
- Save / share / permalink for every sim state, content-addressed
- Optional signin claims anonymous saves
- See `AUTH.md`, `PERSISTENCE.md`

## Deploy

- Dokploy VM + Cloudflare DNS + Convex self-host instance
- Local-first hostability invariant — same stack runs in compose on a MacBook
- See `DEPLOY.md`

## Quality floor

- 60 fps on mid-tier laptop hardware
- WebGPU primary, WebGL fallback
- Keyboard-first, every action invocable without mouse
- WCAG AA, reduced-motion respected
- Deterministic sim engine, golden-trace tested
- Substrate published OSS with foundation-app demos
