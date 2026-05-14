# isa-scope

## Decision

Two tiers, both in the locked floor:

- **Datapath-animated tier (10)**: `add`, `addi`, `and`, `beq`, `bne`, `lw`, `slt`, `or`, `sw`, `sub` — each gets full 3D step animation through the locked datapath.
- **Encodable tier (17)**: above + `andi`, `j`, `lui`, `nor`, `ori`, `sll`, `srl` — assembled + executed in the machine state model. Animation lives only on the datapath subset for the locked floor; encodable-only instructions execute correctly with state changes shown in the register/memory HUDs but no per-stage 3D animation.

## Ratchet path

Floor never ceiling. Full MIPS32 instruction set grinds in over time — each additional instruction is a code commit + golden-trace fixture + (if datapath-animated) topology mapping. Specifically tracked grind targets:

- R-type arithmetic + logical: `addu`, `subu`, `xor`, `xori`, `slti`, `sltiu`, `sltu`, `mult`, `multu`, `div`, `divu`, `mfhi`, `mflo`, `mthi`, `mtlo`
- R-type shift: `sllv`, `srlv`, `sra`, `srav`
- Branch: `blez`, `bgtz`, `bltz`, `bgez`, `bltzal`, `bgezal`
- Jump: `jal`, `jr`, `jalr`
- Memory: `lb`, `lbu`, `lh`, `lhu`, `lwl`, `lwr`, `sb`, `sh`, `swl`, `swr`
- Conditional move: `movz`, `movn`

Each addition lands as ADR amendment (`adr/isa-scope.md` table grows) + golden-trace fixture committed + spec-of-code lint verifies the addition.

## Out of scope

Per `NON-GOALS.md`:
- FP instructions (`.s`, `.d` suffix)
- Atomic instructions (`ll`, `sc`)
- CP0 instructions (`mfc0`, `mtc0`, `eret`)
- Trap instructions (`teq`, `tne`, `tge`, `tlt`)
- Exception-shaped semantics

## Pseudo-instruction expansion

Pseudos visible to user, expansion visible in the encoding view. Floor pseudos:
- `li $rd, imm` → `lui` + `ori` for 32-bit constants; `addi` for small
- `la $rd, label` → `lui` + `ori`
- `move $rd, $rs` → `add $rd, $rs, $zero`
- `nop` → `sll $zero, $zero, 0`
- `blt`, `bgt`, `ble`, `bge` → `slt` + conditional branch
- `beqz`, `bnez` → `beq`/`bne` against `$zero`

User can toggle "pseudo expanded" vs "pseudo opaque" view.

## Caught by

- Encoder round-trip test: assemble → encode → decode → re-assemble produces identical canonical asm.
- Executor golden-trace test: each instruction in encodable tier produces correct state transitions against committed golden trace.
- Spec-of-code lint diffs the table here against `apps/web/features/datapath/core/instructionSet.ts` codegen output.
