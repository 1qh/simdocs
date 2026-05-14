# critical-path-semantics

## Decision

Two modes, both ship in the floor:
- **Structural** — longest enabled signal path through the datapath for the current instruction, measured in component-count.
- **Timing-weighted** — same path graph, edges weighted by per-component propagation delay (configurable table), accumulator returns longest total delay.

User toggles between modes. Both render as overlay on the 3D datapath scene.

## Structural mode

- Walks the directed graph of enabled paths (paths gated by control signals active for the current instruction)
- Finds the longest chain of enabled components from PC to WB completion
- Highlights every component on the chain with the accent color
- Reports component count

## Timing-weighted mode

- Same graph walk, but each edge carries a weight from a `componentDelays` table
- Default delay table sourced from Patterson & Hennessy reference values (configurable):
  - PC clock-to-Q: 30 ps
  - Instruction memory read: 200 ps
  - Register file read: 150 ps
  - ALU 32-bit add: 200 ps
  - Data memory read: 250 ps
  - Mux 2-to-1: 25 ps
  - Sign-extend: 20 ps
  - Left-shift-2: 5 ps
  - Branch adder: 200 ps
  - Control unit: 100 ps
  - ALU control: 30 ps
- Returns total accumulated delay along the longest path
- Highlights every component on the chain with accent color, intensity proportional to delay contribution

## Configurable delay table

`packages/sim-engine` exposes the delay table as a typed configuration shape. Default values committed under `apps/web/features/critical-path/defaultDelays.ts`. User can edit delays in a side panel and see critical-path recompute live.

## Per-instruction critical path

The "longest path" depends on which control signals are active, which depends on the instruction. Different instructions have different critical paths. Critical-path overlay updates per step / per selected instruction.

## Whole-program critical path

The clock period for the locked datapath is determined by the LONGEST critical path across the WORST-case instruction (typically `lw` for memory-load latency). User can request "show worst-case critical path" which iterates every instruction in the locked subset and returns the max.

## Caught by

- Unit test: known instruction produces known structural path length and known timing total against committed delay table.
- Worst-case test: iterating subset returns `lw` as worst-case under default delay table.
- Configurability test: changing delay table shifts worst-case as expected.
