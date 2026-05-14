# GLOSSARY

Domain term inventory. Substrate-boundary discipline helper: if a term appears here, it lives in product code, never substrate. Substrate code that imports or references any term below = violation.

## MIPS / datapath

| Term | Meaning |
|---|---|
| ALU | Arithmetic Logic Unit; 32-bit op block |
| ALU Control | Derives ALU op from `ALUOp` + funct |
| ALUOp | 2-bit signal class from Control unit |
| ALUSrc | Mux select: RF read-2 vs sign-extended immediate |
| Branch | Control signal enabling BEQ |
| BranchAdder | Adds (PC+4) and (sign-ext-immediate << 2) for branch target |
| BranchNE | Control signal enabling BNE |
| Control unit | Decodes opcode → 9 base control signals |
| CPI | Cycles Per Instruction |
| Datapath | The combinational + sequential circuit executing instructions |
| DM | Data Memory |
| EX | Execute stage (single-cycle phase) |
| Funct | 6-bit field of R-type instruction selecting ALU op |
| ID | Instruction Decode stage |
| IF | Instruction Fetch stage |
| IM | Instruction Memory |
| Immediate | 16-bit field of I-type instruction |
| IPC | Instructions Per Cycle |
| IR | Instruction Register (implicit in single-cycle, field decomposition of fetched word) |
| MEM | Memory Access stage |
| MemRead | Control signal enabling DM read |
| MemToReg | Mux select: ALU result vs DM read for RF write |
| MemWrite | Control signal enabling DM write |
| Mnemonic | Asm symbol for an instruction (`add`, `lw`, etc.) |
| Opcode | 6-bit field [31..26] of instruction word |
| PC | Program Counter |
| PCSrc | Mux select: PC+4 vs branch target |
| Pseudo | Asm-level abstraction expanded into one or more real instructions |
| RegDst | Mux select: rt vs rd as write register |
| RegWrite | Control signal enabling RF write |
| RF | Register File |
| RS | Source register field [25..21] |
| RT | Source/destination register field [20..16] |
| RD | Destination register field [15..11] |
| Shamt | Shift amount field [10..6] |
| Sign-extend | 16 → 32 bit signed extension |
| WB | Write-Back stage |
| Zero (flag) | ALU output is zero |

## Pipeline

| Term | Meaning |
|---|---|
| Hazard | Data or control dependency that prevents trivial pipelining |
| Forwarding | Routing a stage's output to a later stage's input to resolve a RAW hazard without stalling |
| Stall / bubble | A NOP inserted into the pipeline to resolve an unforwardable hazard |
| Load-use hazard | RAW hazard from a load instruction; not resolvable by forwarding within standard 5-stage pipeline |
| Control hazard | Hazard caused by a taken branch invalidating speculatively fetched instructions |
| RAW | Read-After-Write hazard |
| WAW | Write-After-Write hazard |
| WAR | Write-After-Read hazard |

## K-map / Boolean

| Term | Meaning |
|---|---|
| K-map | Karnaugh map; 2D / 3D grid for Boolean minimization |
| Minterm | Product term covering exactly one row in the truth table |
| Maxterm | Sum term covering all rows except one |
| Don't-care | Truth table output marked X; can be 0 or 1 to optimize grouping |
| Gray code | Encoding where adjacent values differ by one bit |
| Group | Rectangle of adjacent 1s (or 0s) of size 2^k on the K-map |
| Prime Implicant (PI) | A group not contained in a larger group |
| Essential PI | A PI covering at least one minterm no other PI covers |
| Cover | Set of PIs covering every minterm |
| Minimal SOP | Sum-of-Products expression with fewest literals + fewest terms |
| Minimal POS | Product-of-Sums equivalent |
| Quine-McCluskey | Tabular algorithm finding all PIs deterministically |
| Petrick's method | Selects minimum cover from PI chart |
| Espresso | Heuristic minimization for large functions |
| Static hazard | Glitch when transitioning between minterms in different groups |
| Hazard cover | Redundant PI added to eliminate a static hazard |

## Critical path

| Term | Meaning |
|---|---|
| Critical path | Longest signal path through the datapath for a given instruction |
| Propagation delay | Time for a signal to traverse a component |
| Clock period | Time per cycle; bounded by worst-case critical path across all instructions |

## Auth / persistence

| Term | Meaning |
|---|---|
| Anonymous | No identity; default state |
| Claim | Linking an anonymous-saved snapshot to a user after signin |
| Permalink | URL pointing at a specific sim state; content-addressed or URL-fragment-encoded |
| Tier-1 | URL-fragment-encoded share, ≤1KB canonical bytes, no backend hit |
| Tier-2 | Content-addressed share via Convex, >1KB canonical bytes |
| Hash | blake3 truncated 16-hex-char identifier of canonical body |
| Snapshot | Serialized sim state |
| Snapshot version | Schema version embedded in snapshot body |

## Banned in substrate code

Every term above. Substrate packages stay domain-agnostic. CI lint `tools/lint/substrate-boundary.ts` greps `packages/*` source for any of these tokens; zero hits required.

Allowed in product code: `apps/web/features/*` consumes these freely; foundation demos under `apps/web/learn/foundation/*` use generic vocabulary only (no MIPS, no K-map).

## Caught by

- `tools/lint/substrate-boundary.ts` token grep
- `tools/lint/glossary-coverage.ts` asserts every domain term used in product code appears in this glossary
