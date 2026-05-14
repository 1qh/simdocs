# kmap-geometry

## Decision

- 2 to 4 variables: 2D grid in Gray-code layout.
- 5 to 6 variables: 3D toroidal geometry, cells as raised blocks, wrap-edges rendered as actual torus topology.
- > 6 variables: Espresso heuristic solver runs without geometry; visualization shape out of scope per `NON-GOALS.md`.

## Why split at 4

- 2-var (2×2), 3-var (2×4), 4-var (4×4) maps fit a flat grid; wraparound is a single dotted-line annotation; standard textbook presentation works.
- 5-var (4×8) flat presentation has a hidden middle-axis adjacency that students lose; standard split-map workaround (two 4×4 maps side-by-side) obscures the adjacency entirely.
- 6-var (8×8) flat presentation has two hidden adjacency axes; split-map workaround stacks four 4×4 maps with even more invisible wrap.
- The honest geometry of any K-map is a torus (every edge wraps to the opposite edge). For ≤4 vars, the torus and the flat grid are pedagogically equivalent because the wraparound can be conveyed verbally. For ≥5 vars, the torus is the ONLY honest representation.

## 2D shape

- Cells laid out in Gray-code (00, 01, 11, 10) so adjacency = single-variable change
- Variable labels on row/column headers
- Don't-cares as cells with `X` marker
- Group rectangles drag-selected, snap to 2^k sizes, wrap-aware
- Multi-output K-maps: side-by-side stack, shared variable headers
- Reveal toggle shows solver's chosen cover

## 3D shape (5- and 6-var)

- Torus geometry with cells as raised blocks on the torus surface
- Block height encodes output value (0 = flat, 1 = tall, X = mid-height with stripe shader)
- Camera bookmarks: outside-look, inside-cylinder-look, flat-unrolled (Gray-code rectangle on a plane, equivalent to the textbook map)
- Wrap edges rendered with subtle highlight indicating "this edge is the same as the opposite"
- Group selection: drag through cells in 3D; group lives on the torus surface; wrap-aware by construction
- Slicing planes available to isolate a single sub-grid for grouping precision

## Interaction parity

Both 2D and 3D modes share:
- Truth table sync (edit cell ↔ edit row in truth table)
- Boolean expression sync (sum-of-products tracks current groupings)
- Group highlighting, color-coded per prime implicant
- Essential PI marker on the cell(s) only that PI covers
- Reveal solution toggle
- Undo / redo on groupings

## Inputs

| Mode | Source |
|---|---|
| Truth table | Cell-by-cell entry, paste tab-separated text |
| Boolean expression | Live-parsed via custom grammar (Monaco editor) |
| Minterm list | `Σm(0,2,5,…)` notation |
| Maxterm list | `ΠM(1,3,4,…)` notation |
| Don't-care list | `d(0,2,…)` |
| Multi-output | One truth table per output column; shared variable headers |

## Outputs

- All prime implicants
- Essential prime implicants
- Minimal SOP expression
- Minimal POS expression
- Optional gate-level circuit diagram (AND-OR, NAND-only, NOR-only)
- Hazard analysis: static hazards highlighted with amber accent; hazard-cover PI suggested

## Cross-link with datapath

The MIPS Control unit and ALU Control are themselves Boolean functions over (opcode, funct) → 9 control signals. K-map tool ingests the truth table of any control signal as a function of (opcode-bit, funct-bit) and visualizes its minimization. Per `MIPS-DATAPATH.md` "Control cross-link".

## Caught by

- Property-based test (fast-check): random truth table → solver result → manual PI verification.
- Golden-test fixtures: known truth tables with known minimal covers; assertion against committed expected.
- Geometry test: 5-var wraparound assertion — adjacent-on-torus cells are reported adjacent by the grouper.
