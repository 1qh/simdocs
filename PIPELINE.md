# PIPELINE

Stage-time diagram + hazard overlay. Sits on top of single-cycle execution traces per `adr/pipeline-shape.md`.

## Stages

5 classic stages: IF, ID, EX, MEM, WB.

## Diagram shape

- Rows: instructions in program order
- Columns: clock cycles
- Cells: stage occupied that cycle, color-coded per stage
- Each cell tooltip: instruction mnemonic, fields, stage, control signals active

## Per-stage latches (conceptual)

| Latch | Carries |
|---|---|
| IF/ID | Next PC, fetched instruction word |
| ID/EX | Read data 1, read data 2, sign-extended immediate, rt, rd, control signals (EX/MEM/WB groups) |
| EX/MEM | ALU result, Zero, RF read 2, write register (post-RegDst), control signals (MEM/WB groups) |
| MEM/WB | DM read data, ALU result, write register, control signals (WB group) |

Latches rendered as labels on the diagram cells, never as discrete 3D components in the datapath scene.

## Hazards detected

| Type | Detection |
|---|---|
| RAW (read-after-write) | Instr N writes reg R; instr N+1, N+2, or N+3 reads R before WB |
| WAW (write-after-write) | Two instructions write the same register; in-order pipeline → trivially handled |
| WAR (write-after-read) | Instr N reads R; instr N+M writes R; in-order pipeline → trivially handled |
| Control | Branch taken in EX → IF and ID stages flush (instructions speculatively fetched are discarded) |
| Structural | Locked single-cycle has separate IM + DM + 2-read-1-write RF → no structural hazards under this datapath |

## Forwarding overlay

When enabled:
- **EX/MEM → EX**: ALU result of cycle N's EX is forwarded to ALU input of cycle N+1's EX. Arrow drawn from producing cell to consuming cell, accent-colored.
- **MEM/WB → EX**: DM read (or ALU result) of cycle N's MEM is forwarded to ALU input of cycle N+1's EX (when N+2's WB hasn't fired yet).
- **Load-use hazard**: `lw $t0, ...` then `add $t1, $t0, ...` — forwarding cannot resolve (load result isn't available until MEM). One bubble inserted; consumer stalls one cycle.

## Stalls / bubbles

- Bubble cells rendered as gray with `bubble` label
- Stall reason in tooltip
- CPI counter updates live

## Counters

| Counter | Definition |
|---|---|
| Cycle count | Total cycles elapsed |
| Instruction count | Instructions reaching WB |
| CPI | cycles / instructions |
| IPC | instructions / cycles |
| Forwarded-RAW | RAW hazards resolved by forwarding |
| Stalled-RAW | RAW hazards requiring stall |
| Branch-flush | Stages flushed due to taken branches |

## Controls

- Step by cycle (not by instruction)
- Scrub timeline forward / backward
- Toggle forwarding on/off
- Toggle hazard-stall-insertion on/off (without forwarding)
- Click cell → highlight producing instruction trace + relevant control signals

## What is NOT in this view

- Per-cycle datapath topology with forwarding muxes — the 3D datapath scene stays single-cycle; this is a 2D diagram sitting beside it
- Branch prediction (deferred per `NON-GOALS.md`)
- Out-of-order, speculation, superscalar
- Exception-pipeline-flush (no exception model in floor)

## Substrate vs product

`sim-engine` exposes `arrangeStageMatrix(executions[], stages)` helper — generic, takes a list of per-instruction step traces and arranges them in a stage-time matrix with optional hazard detection. Used by this feature; reusable for any 5-stage-pipeline-shaped sim.

`apps/web/features/pipeline/` is the product surface. Consumes single-cycle traces + the substrate helper.

## Sharing

Saved pipeline state includes:
- Source program (asm or encoded words)
- Forwarding toggle state
- Stall-insertion toggle state
- Current scrub cycle position

Shareable via tier-1 or tier-2 per `adr/share-content-addressed.md`.

## Caught by

- Golden-trace test: documented programs with documented hazard patterns produce expected diagram cells + counters
- Stall-count test: load-use program produces expected single bubble
- Forwarding test: RAW-resolvable-by-EX-EX program shows 0 stalls forwarding-on, ≥1 stall forwarding-off
- Branch-flush test: taken branch produces expected flush cells
