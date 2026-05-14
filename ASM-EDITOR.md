# ASM-EDITOR

Monaco-backed MIPS assembly editor. Per `adr/editor-monaco.md`.

## Features

- Syntax highlighting for opcodes, registers, directives, labels, comments, immediates
- Pseudo-instruction highlighting (different tone from real instructions)
- Live parse-error diagnostics with red squigglies + hover tooltip
- Live encoding view — per-line machine code (hex + binary) in margin gutter
- Auto-format on save (consistent register-name style, immediate base, indentation)
- Hover on register → shows current value (live, synced with sim state)
- Hover on label → shows resolved address
- Hover on opcode → shows brief description + datapath signals it sets
- Completion: opcodes, register names, label names (after first definition)
- Multi-cursor edit (Monaco default)
- Folding: directive blocks, labeled regions
- Find / replace (Monaco default, theme-matched)

## Grammar

Custom Monarch tokenizer + parser:
- Opcodes from MIPS-ISA encodable set
- Pseudo-instructions from MIPS-ISA pseudo set
- Registers in both numeric and aliased forms
- Directives: `.data`, `.text`, `.word`, `.byte`, `.half`, `.asciiz`, `.ascii`, `.space`, `.align`, `.globl`
- Labels: `identifier:`
- Comments: `#` to end of line
- Numbers: decimal, `0x` hex, `0b` binary
- String literals in double quotes for `.ascii` / `.asciiz`

Parser produces typed AST consumed by the assembler.

## Diagnostics

| Class | Surface |
|---|---|
| Lexical error | Inline squiggly |
| Syntax error | Inline squiggly + line-level marker |
| Undefined label | Squiggly on use, gutter marker |
| Duplicate label | Squiggly on second definition |
| Immediate out of range | Squiggly + range hint |
| Pseudo-expansion not yet supported | Info-level squiggly |

All diagnostics typed per `book/HARD-RULES.md` "Maximum typesafety". No silent ignores. Per `book/HARD-RULES.md` "Zero fallback" — missing field = throw, not default.

## Encoding view sync

Each successfully parsed line emits encoded word; the gutter shows it. If a line fails to parse, gutter shows `--` and the line is excluded from runnable code.

## Pseudo expansion view

Toggle "expand pseudos":
- Off: editor shows source as typed (`li $t0, 0x12345678`)
- On: editor shows expansion inline as ghost text below the pseudo line (`lui $at, 0x1234` / `ori $t0, $at, 0x5678`)

## Theme

Industrial dark by default; consumes `design-tokens` palette so colors match the app. Monaco's built-in themes overridden.

## Performance

- Editor loaded as client-only dynamic import — RSC paths skip Monaco entirely
- Parser runs incrementally on text change (debounced 200ms)
- Encoding compute is memoized per source-line content hash

## Sharing

Editor content is part of the datapath sim snapshot. Saved permalink reproduces editor text + cursor + breakpoint positions.

## Caught by

- Parser unit tests: every keyword, every register, every pseudo, every immediate form
- Diagnostics test: each error class produces correct marker at correct position
- Encoding sync test: gutter value matches `encodeMipsInstruction` output for the line
- Bundle-size budget: editor route bundle includes Monaco; non-editor routes do not
