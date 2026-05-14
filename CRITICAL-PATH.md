# CRITICAL-PATH

Per `adr/critical-path-semantics.md`. Two modes, both ship: structural longest path, timing-weighted accumulated delay.

## Structural mode

- Walks directed graph of enabled paths (paths gated by control signals active for the current instruction)
- Finds longest chain of enabled components from PC to WB completion
- Highlights every component on the chain with accent color
- Reports component count
- Path enumeration uses `MIPS-DATAPATH.md` paths table; gating uses control signal table for the current instruction

## Timing-weighted mode

- Same path graph, edges weighted by per-component propagation delay
- Default delay table (ps), configurable in side panel:

| Component | Delay (ps) |
|---|---|
| PC clock-to-Q | 30 |
| Instruction memory read | 200 |
| Register file read | 150 |
| Register file write setup | 20 |
| ALU 32-bit add | 200 |
| ALU 32-bit logic (and/or/nor/slt) | 120 |
| Data memory read | 250 |
| Data memory write | 200 |
| Mux 2-to-1 | 25 |
| Sign-extend | 20 |
| Left-shift-2 | 5 |
| Branch adder | 200 |
| Control unit | 100 |
| ALU control | 30 |
| Zero detect | 15 |
| AND gate | 10 |
| OR gate | 10 |
| NOT gate | 8 |

- Returns total accumulated delay along longest path
- Component highlight intensity proportional to delay contribution

## Per-instruction critical path

Different instructions exercise different control signals → different enabled paths → potentially different critical path. Critical-path overlay updates per current instruction / per current step.

## Worst-case across instruction set

Toggle "show worst-case" — iterates every instruction in the locked subset, returns the max critical path. Determines the clock period this datapath would run at.

For default delay table + locked subset, `lw` is worst-case (memory-load latency: PC → IM → RF read → ALU add → DM read → mux → RF write).

## Configurable delays

Side panel exposes the delay table. User edits a value → critical path recomputes live → worst-case updates. Pedagogy: see how memory latency dominates; see how a faster ALU shifts the bottleneck.

## Render

Two overlays on the 3D datapath scene:
- **Path highlight**: every component on the critical path glows with accent color
- **Delay heatmap** (timing mode): components colored by their delay contribution to the path; tooltip shows component delay and path delay

In structural mode, all components on path glow at the same intensity. In timing mode, gradient.

## Substrate vs product

`packages/sim-engine` exposes `findLongestPath(graph, weights, gatedEdges)` — generic longest-weighted-path algorithm. Used by this feature; reusable for any graph longest-path problem.

`apps/web/features/critical-path/` is the product surface. Wires the path graph + control gating + default delay table.

## Caught by

- Unit test: known instruction produces known structural path length + known timing total against default table
- Worst-case test: iterating locked subset returns `lw` under default delays
- Configurability test: doubling DM read delay shifts worst-case characteristics as expected
- Render test: highlighted-component set matches solver output
