# KMAP

Karnaugh map tool spec. Geometry split per `adr/kmap-geometry.md` — 2D for ≤4 vars, 3D toroidal for 5-6 vars, solver-only for ≥7.

## Inputs

| Mode | Source |
|---|---|
| Truth table | Cell-by-cell entry, paste tab-separated, paste CSV |
| Boolean expression | Live-parsed via custom grammar in Monaco editor — supports `*`, `+`, `'`, `!`, `&&`, `\|\|`, `~`, `XOR`, `XNOR`, parens, var names `A..Z`, `x0..xN` |
| Minterm list | `Σm(0,2,5,...)` or `f = m(0,2,5,...)` |
| Maxterm list | `ΠM(1,3,4,...)` |
| Don't-care list | `d(0,2,...)` or per-cell X |
| Multi-output | One column per output; shared variable headers |

Live two-way sync — editing any representation updates all others within the same input panel.

## 2D grid (2-4 vars)

- Layout: Gray-code on rows and columns so adjacency = single-variable change
- 2-var: 2×2, vars on row + col headers
- 3-var: 2×4, vars on row + col headers (row = 1 var, col = 2 vars)
- 4-var: 4×4, vars on row + col headers (2+2)
- Don't-care cells: `X` marker
- Wrap edges shown as dashed border + tooltip on hover

## 3D toroidal (5-6 vars)

- 5-var: torus with 4 cells × 8 cells around the major axis
- 6-var: torus with 8 cells × 8 cells
- Cells: raised blocks on torus surface; height encodes output
  - 0 → flat
  - 1 → tall, accent-emissive
  - X → mid-height, stripe shader
- Wrap edges: rendered as actual torus continuity (geometry inherently wraps); subtle highlight on the "where 2D would split" axes
- Camera bookmarks:
  - Outside-orbit (whole torus visible)
  - Inside-tube (looking outward along the major axis)
  - Flat-unrolled (Gray-code rectangle on a plane, equivalent to textbook split-map presentation, for users who want the familiar view)
- Slicing planes: drag to slice the torus along either Gray-axis to isolate a 4×4 sub-grid for precise grouping

## Grouping interaction

Both 2D and 3D modes:
- Click-drag rectangle (or click-drag on torus surface in 3D) to select cells
- Snap to 2^k cell sizes (1, 2, 4, 8, 16, 32, 64)
- Wraparound-aware — group can span the wrap edge
- Overlapping groups allowed
- Color-coded per group, palette of 12 distinct accent variants
- Click a group → highlight + show its product term
- Right-click → delete group
- Undo / redo (full history)

## Outputs

Always computed live as user groups:
- **All prime implicants** — list of every PI for the current function, regardless of whether user has grouped them
- **Essential prime implicants** — PIs that cover a minterm no other PI covers; highlighted with thicker outline
- **Minimal SOP** — solver's canonical minimal sum-of-products
- **Minimal POS** — solver's canonical minimal product-of-sums (computed by grouping 0s)
- **User's current cover** — expression formed from user's selected groups, compared against minimal; "minimal" badge when matched
- **Hazard analysis** — pairs of groups that don't share a PI; highlighted with amber, hazard-cover PI suggested
- **Gate-level circuit** (optional toggle) — render the minimal SOP as AND-OR network, AND-NAND-only, or NOR-only, with gate counts

## Solver

`packages/boolean` provides:
- Quine-McCluskey for ≤6 vars, deterministic, returns all PIs + selects essential
- Petrick's method for cover selection when multiple equivalent covers exist
- Espresso heuristic for ≥7 vars (solver only, no geometry)
- Multi-output minimization with shared-term sharing across outputs

## Step-through QM mode

Toggle to step through the QM tabular method:
- Step 1: list minterms in binary, grouped by 1-count
- Step 2: combine adjacent minterms, mark "combined"
- Step 3: identify PIs (uncombined entries)
- Step 4: build prime-implicant chart
- Step 5: identify EPIs
- Step 6: Petrick selection if needed

Each step animated on a side panel showing the tabular work.

## Reveal solution toggle

When active: solver's chosen cover overlays the user's current cover. User compares without losing their work.

## Datapath cross-link

Each of the 9 base control signals in `MIPS-DATAPATH.md` is a Boolean function over the opcode bits. Click "Load from datapath: <signal>" → truth table populated automatically with `(opcode → signal value)` for the 10 locked datapath instructions. User minimizes; the result is the actual logic in the Control unit. The headline pedagogy of the product.

## Sharing

Saved K-map state includes:
- All input modes' current values
- Don't-care set
- User's grouping list
- Selected solver options (multi-output, hazard analysis, gate style)
- 3D mode camera bookmark

Snapshot serialized via `sim-engine` codec, shareable via tier-1 URL-fragment if small, tier-2 Convex hash otherwise.

## Caught by

- Property-based test (fast-check): random truth table → solver → manual PI verification
- Golden-test fixtures: documented truth tables with documented minimal covers
- 2D ↔ 3D parity test: same 5-var truth table grouped identically in both modes produces identical result
- Wrap-edge test: 5-var function whose minimal cover requires wraparound is correctly minimized
- Cross-link test: loading `RegDst(opcode)` from datapath produces the expected truth table
