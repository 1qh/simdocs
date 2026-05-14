# pipeline-shape

## Decision

Pipeline visualizer is a stage-time diagram + hazard overlay sitting on top of the locked single-cycle datapath. Not a separate pipelined-datapath topology.

## Shape

- 5-stage classic: IF, ID, EX, MEM, WB
- Diagram: rows = instructions in program order, columns = cycles, cell = stage occupied that cycle
- Per-stage latches conceptually modelled (IF/ID, ID/EX, EX/MEM, MEM/WB) but rendered as labels on the diagram, not as discrete components in the 3D datapath
- Hazard detection (RAW, WAW, WAR, control) — every detected hazard marked on the diagram with type label and tooltip
- Forwarding overlay — when forwarding resolves a RAW hazard, an arrow connects the producing stage's output to the consuming stage's input
- Stalls / bubbles — inserted NOPs visualized as gray cells with `bubble` label
- Branch resolution: when branch resolved in EX (per locked datapath), the affected IF stages flush, visible as discarded cells

## Counters

- Cycle count
- Instruction count (committed in WB)
- CPI = cycles / instructions
- IPC = 1 / CPI
- Forwarded-RAW count
- Stall-RAW count (load-use)
- Branch-flush count

## Interaction

- Step by cycle (not by instruction)
- Scrub timeline backward and forward
- Click a cell → highlight the producing instruction, the stage it was in, and the relevant control signal
- Toggle forwarding on/off → see CPI delta
- Toggle hazard-stall-insertion on/off (without forwarding) → see ideal vs realistic CPI

## What is NOT in this view

- Per-cycle datapath topology re-rendered with forwarding muxes — the 3D datapath scene stays single-cycle; pipeline view is a separate diagram that consumes per-instruction execution traces and arranges them on a 2D grid
- Branch prediction (deferred to `NON-GOALS.md` until trigger)
- Out-of-order execution, speculation, superscalar — out of scope
- Exception-pipeline-flush — out of scope (no exception model in floor)

## Substrate vs product

The diagram itself is a chart; built on `apps/web/features/pipeline/` consuming `sim-engine` traces. Substrate gain: `sim-engine` exposes "stage-time-arrange" helper that takes a list of executions and arranges them in a pipeline-stage matrix — generic, reusable for any 5-stage-pipeline-shaped sim.

## Caught by

- Golden-trace test: known programs with known hazard patterns produce expected diagram cells and counters.
- Stall-count test: program with load-use hazard produces expected bubble.
- Forwarding test: program with RAW hazard resolvable by EX→EX forwarding produces zero stalls when forwarding on, one stall when off.
