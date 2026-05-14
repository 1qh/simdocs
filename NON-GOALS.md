# NON-GOALS

Project-specific scope defenses with explicit triggers. Cross-project scope defenses live in `book/NON-GOALS.md`. Items here are filed-and-tracked; the agent does not re-list them in proactive "what's left" surfaces per `book/HARD-RULES.md`.

## Multi-cycle MIPS datapath variant

Datapath topology is locked single-cycle CS2100-variant per `MIPS-DATAPATH.md`. Multi-cycle datapath (shared ALU/memory across cycles, FSM controller, variable CPI) is out of scope.

Trigger: never. The locked datapath is the contract.

## Fully pipelined datapath with forwarding muxes

Pipeline visualizer is a stage-time diagram + hazard overlay sitting on top of single-cycle execution traces per `PIPELINE.md`. A separate pipelined datapath topology (IF/ID/EX/MEM/WB latches as physical registers, forwarding multiplexers wired into the datapath, hazard detection unit as a discrete component) is out of scope.

Trigger: req for "pipelined datapath with forwarding visualized as a separate datapath topology, not just a stage-time diagram" lands as a new requirement.

## Branch delay slot

Locked datapath has no delay-slot semantics. Out of scope.

Trigger: explicit requirement for delay-slot ISA variant.

## Floating-point unit + FP registers

`$f0..$f31`, FP ops (`add.s`, `mul.d`, `c.lt.s`, etc.), CP1 register file are out of scope.

Trigger: req for FP visualization or a second sim that needs FP semantics.

## Exceptions, interrupts, syscalls beyond minimal

Minimal syscalls covered per `REQUIREMENTS.md` MIPS section (print int, print str, read int, read str, exit). Full exception model — EPC, Cause, BadVAddr, Status register, exception handler dispatch viz, trap-on-overflow, timer interrupt, keyboard interrupt — out of scope.

Trigger: req for exception viz lands.

## Cache simulator

L1 I/D cache, associativity policies, replacement policies, TLB, MMIO. Out of scope.

Trigger: second sim adoption that includes cache hierarchy.

## Memory-mapped I/O surfaces

Bitmap display MMIO, keyboard MMIO, console MMIO. Out of scope.

Trigger: req for embedded-system-shaped MMIO sim.

## Real-time multiplayer / collaboration

Two users editing the same sim state simultaneously. Out of scope.

Trigger: req for classroom collaboration shape.

## Instructor dashboards / grading / curriculum sequencing

Per-user analytics, lesson sequencing, auto-grading of student work, classroom rosters. Out of scope. Product is generic learning tool, not curriculum platform.

Trigger: separate product adoption — would extract substrate primitives and build classroom product as a sibling delta per `book/SUBSTRATE.md` growth loop.

## K-map beyond 6 variables

Quine-McCluskey + Espresso solvers cover up to 6 variables in K-map geometry. Beyond 6 → solver still runs, geometry not rendered.

Trigger: req for ≥7-var K-map visualization shape.

## Mobile / tablet full interactivity

Read-only experience on screens narrower than ~1024px. Full sim interactivity (asm editor, K-map drag-grouping, 3D camera controls) requires precision input and screen real estate that small touch screens do not provide. Mobile gets the share-permalink read view, the explainer copy, the static OG cards, the snapshot preview — no editing.

Trigger: pinch-zoom + multi-touch grouping for K-map proves cleanly designable AND req for it lands.

## Internationalization

Single-language English. UI copy + math symbols + Boolean operators are language-shaped; full i18n adds translation infrastructure and re-layout per language without proportional pedagogical gain.

Trigger: substantial non-English user demand evidence.

## Custom ISA / MIPS variants beyond CS2100-locked datapath

MIPS R3000, R4000, MIPS64, RISC-V, ARMv7, custom-academic ISAs. Out of scope.

Trigger: second sim adoption with different ISA — extracts substrate ISA-engine primitive, builds new ISA as sibling product per substrate growth loop.

## User accounts beyond minimum-viable auth

Profile pages, user-to-user follows, comment threads, public profile galleries, leaderboards. Out of scope.

Trigger: explicit social product pivot.

## Telemetry / per-user analytics

Beyond aggregate counts (total sims run, total snapshots saved), no per-user tracking, no behavior analytics, no funnel analysis. Out of scope.

Trigger: privacy-respecting aggregate-only mode if surface ever needs them; never per-user.

## Paid tier / paywalls / subscription

Product is free, anonymous-first, no paid features. Substrate is OSS. Out of scope.

Trigger: substrate sustainability requires it — separate ADR with rejection rationale for current model.

## Native mobile / desktop apps

Web-only. No iOS, no Android, no Electron, no Tauri.

Trigger: req for offline-first installable shell.

## WebXR / VR mode

VR session for 3D scenes (datapath + K-map toroidal) is scoped, substrate-compat-locked, but not in floor.

Trigger: drei `xr` package stable on locked R3F version + user demand signal + operator decision. Per `adr/webxr.md`.

## Native mobile / desktop apps

Web-only. No iOS, no Android, no Electron, no Tauri. PWA service-worker + installable shell per `adr/offline-pwa.md` covers offline-installable use cases.

Trigger: req for offline-first native shell beyond PWA capability.

## Phased "MVP first, polish later" framings

Per `book/PHILOSOPHY.md` "Unlimited rework pre-launch" + "Locked is floor, only more never less". Every feature ships at world-class quality on first land; rework freely pre-launch.

Trigger: never. Banned framing.
