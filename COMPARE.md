# COMPARE

Side-by-side two-instruction comparison view.

## Layout

- Split-pane vertical, left and right
- Each pane renders the full 3D datapath scene, identical aesthetic, independent camera
- Top: instruction picker per pane (select from any encodable instruction)
- Bottom: synchronized step controls (single play/step affects both)

## Synchronization

- Step controls drive both panes simultaneously
- When stepping, both panes advance to the same step (IF, ID, EX, MEM, WB)
- Camera can be:
  - Locked (mirrored — moving one moves both)
  - Independent (each pane has its own camera)

## Diff highlighting

- Components active in left-only: accent-blue
- Components active in right-only: accent-amber
- Components active in both: accent-neutral
- Control signals table: per-signal row colored to show match/diff

## Use cases

- `add` vs `addi` — see how ALUSrc flips the second ALU operand source
- `lw` vs `sw` — see MemRead vs MemWrite, MemToReg vs RegWrite difference
- `beq` vs `bne` — see BranchAnd path lit differently
- Same instruction, different operands — see operand routing identical, values differ

## Sharing

Compare-mode snapshot includes both pane states (selected instr, operands, current step, camera). Shareable as one permalink. Receiver opens compare mode with both panes pre-loaded.

## Substrate vs product

Compare mode is product-side, consuming two `sim-engine` traces side-by-side with cross-pane diff visualization.

## Caught by

- E2E smoke: load compare with `add` vs `addi`, step through, assert documented diff highlights at each step
- Synchronization test: step-once advances both panes' step counter exactly once
