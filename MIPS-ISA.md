# MIPS-ISA

Instruction set encoding spec. Spec-of-code per `adr/ssot-precedence.md` — CI lint diffs this doc against `apps/web/features/datapath/generated/isa.ts`.

## Formats

```mermaid
flowchart LR
    R[R-type: opcode 6 | rs 5 | rt 5 | rd 5 | shamt 5 | funct 6]
    I[I-type: opcode 6 | rs 5 | rt 5 | immediate 16]
    J[J-type: opcode 6 | address 26]
```

All instructions are 32 bits. Word-aligned. Big-endian byte order.

## Opcodes

| Mnemonic | opcode | format |
|---|---|---|
| `add` | 0x00 | R |
| `sub` | 0x00 | R |
| `and` | 0x00 | R |
| `or` | 0x00 | R |
| `nor` | 0x00 | R |
| `slt` | 0x00 | R |
| `sll` | 0x00 | R |
| `srl` | 0x00 | R |
| `addi` | 0x08 | I |
| `andi` | 0x0c | I |
| `ori` | 0x0d | I |
| `lui` | 0x0f | I |
| `beq` | 0x04 | I |
| `bne` | 0x05 | I |
| `lw` | 0x23 | I |
| `sw` | 0x2b | I |
| `j` | 0x02 | J |

## Funct codes (R-type)

| Mnemonic | funct |
|---|---|
| `add` | 0x20 |
| `sub` | 0x22 |
| `and` | 0x24 |
| `or` | 0x25 |
| `nor` | 0x27 |
| `slt` | 0x2a |
| `sll` | 0x00 |
| `srl` | 0x02 |

## Encoding rules

R-type:
```
word = (opcode << 26) | (rs << 21) | (rt << 16) | (rd << 11) | (shamt << 6) | funct
```
Constraints:
- `sll` and `srl` set `rs = 0`, use `shamt` field for shift amount
- All others set `shamt = 0`

I-type:
```
word = (opcode << 26) | (rs << 21) | (rt << 16) | (immediate & 0xffff)
```
Constraints:
- `lui` sets `rs = 0`
- `immediate` sign-extended on use (except `andi`, `ori`, `lui` which zero-extend)

J-type:
```
word = (opcode << 26) | (address & 0x03ffffff)
```

## Decoding

| Field | Bit range |
|---|---|
| opcode | [31..26] |
| rs | [25..21] |
| rt | [20..16] |
| rd | [15..11] |
| shamt | [10..6] |
| funct | [5..0] |
| immediate | [15..0] |
| address | [25..0] |

## Pseudo-instructions

| Pseudo | Expansion |
|---|---|
| `li $rd, imm` (32-bit) | `lui $at, imm[31..16]` + `ori $rd, $at, imm[15..0]` |
| `li $rd, imm` (signed 16) | `addi $rd, $zero, imm` |
| `la $rd, label` | `lui $at, label[31..16]` + `ori $rd, $at, label[15..0]` |
| `move $rd, $rs` | `add $rd, $rs, $zero` |
| `nop` | `sll $zero, $zero, 0` |
| `blt $rs, $rt, label` | `slt $at, $rs, $rt` + `bne $at, $zero, label` |
| `bgt $rs, $rt, label` | `slt $at, $rt, $rs` + `bne $at, $zero, label` |
| `ble $rs, $rt, label` | `slt $at, $rt, $rs` + `beq $at, $zero, label` |
| `bge $rs, $rt, label` | `slt $at, $rs, $rt` + `beq $at, $zero, label` |
| `beqz $rs, label` | `beq $rs, $zero, label` |
| `bnez $rs, label` | `bne $rs, $zero, label` |

User can toggle "pseudo expanded" vs "pseudo opaque" view in the editor.

## Register aliases

| Name | Number | Use |
|---|---|---|
| `$zero` | 0 | Hardwired zero |
| `$at` | 1 | Assembler temp |
| `$v0`, `$v1` | 2-3 | Return values |
| `$a0`-`$a3` | 4-7 | Args |
| `$t0`-`$t7` | 8-15 | Temps |
| `$s0`-`$s7` | 16-23 | Saved |
| `$t8`-`$t9` | 24-25 | Temps |
| `$k0`-`$k1` | 26-27 | Kernel reserved |
| `$gp` | 28 | Global pointer |
| `$sp` | 29 | Stack pointer |
| `$fp` | 30 | Frame pointer |
| `$ra` | 31 | Return address |

Both `$N` numeric and `$name` aliases accepted in asm source.

## Encoding view in product

Per-instruction display in the encoding panel:
- Full 32-bit binary, color-coded by field
- 8-hex-digit machine code
- Per-field breakdown with binary, hex, decimal
- Tooltip on each field showing bit positions

## Assembler

Hand-written recursive-descent assembler in `apps/web/features/datapath/core/assembler/`. Two-pass: pass 1 builds symbol table, pass 2 emits machine code. Errors surface as Monaco diagnostics with squigglies.

Parse errors are typed and never silent. Per `book/HARD-RULES.md` "Zero fallback".

## Caught by

- Encoder round-trip property test: random valid instruction → encode → decode → re-encode produces identical bytes.
- Spec-of-code lint diffs the opcode + funct tables here against `apps/web/features/datapath/generated/isa.ts`.
- Pseudo expansion test: each pseudo expands to documented expansion exactly.
