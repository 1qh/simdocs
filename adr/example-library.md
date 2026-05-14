# example-library

## Decision

20 MIPS asm example programs + 20 K-map exercises committed under `apps/web/content/examples/`. Curated by agent. MDX frontmatter carries title, description, tags, expected outcomes.

Inventory locked in `LEARN.md` "Example library" section.

## Content shape

```
apps/web/content/examples/
├── mips/
│   ├── hello.mdx
│   ├── sum-1-n.mdx
│   ├── array-sum.mdx
│   ├── gcd.mdx
│   ├── factorial.mdx
│   ├── fibonacci.mdx
│   ├── string-length.mdx
│   ├── bubble-sort.mdx
│   ├── power-of-two.mdx
│   ├── memory-copy.mdx
│   ├── branch-heavy.mdx
│   ├── load-use-hazard.mdx
│   ├── forwarding-resolvable.mdx
│   ├── critical-path-worst.mdx
│   ├── slow-alu-experiment.mdx
│   ├── all-r-type.mdx
│   ├── all-i-type.mdx
│   ├── all-branch.mdx
│   ├── pseudo-zoo.mdx
│   └── alu-only.mdx
└── kmap/
    ├── 2var-xor.mdx
    ├── 2var-and.mdx
    ├── 3var-majority.mdx
    ├── 3var-xor.mdx
    ├── 4var-bcd-7seg-a.mdx
    ├── 4var-bcd-7seg-g.mdx
    ├── 4var-full-adder-carry.mdx
    ├── 4var-full-adder-sum.mdx
    ├── 4var-mux-selector.mdx
    ├── 4var-dont-cares.mdx
    ├── 4var-hazard.mdx
    ├── 5var-parity.mdx
    ├── 5var-majority.mdx
    ├── 6var-demo.mdx
    ├── multi-bcd-7seg.mdx
    ├── multi-2bit-comparator.mdx
    ├── multi-half-adder.mdx
    ├── multi-full-adder.mdx
    ├── cross-link-regdst.mdx
    └── cross-link-alusrc.mdx
```

## Frontmatter shape

```yaml
title: "Sum 1..N"
slug: sum-1-n
description: "Loop summing integers via addi + beq"
tags: [loop, control-flow, basic]
difficulty: 1
expected:
  finalRegisters: { $v0: 5050 }   # for n=100
  cycles: 1502
```

Frontmatter validated by Zod at build time. Per `book/HARD-RULES.md` "Zero fallback" — invalid frontmatter throws at build, never silently default.

## MDX body

```mdx
# Sum 1..N

Compute the sum 1 + 2 + ... + N using a simple loop.

<MipsExample slug="sum-1-n" autoplay={false} />

The loop body executes N times. Each iteration adds the counter to the accumulator and decrements...

<TruthTable rows={[...]} />
```

## Curation policy

- Each example demonstrates one concept clearly
- Code is canonical asm, not contrived
- Expected outcomes are pre-computed and committed (golden trace fixture)
- Examples are linkable from learn pages and from the example picker

## Caught by

- Build-time frontmatter validation
- Golden-trace test: each example's expected outcome matches executor output
- Link-check: every example referenced from any learn page exists
- Snapshot test: rendering each example MDX produces stable output
