# visual-regression

## Decision

Playwright snapshot baselines per known fixture. Baselines committed under `tools/test/visual-baselines/`. Drift fails CI.

## Coverage

| Surface | Baselines |
|---|---|
| Datapath scene per locked instruction × per step | 10 instructions × 5 steps = 50 baselines |
| K-map 2D per representative truth table | ~20 baselines (covers example exercises) |
| K-map 3D toroidal per representative 5/6-var function | ~10 baselines |
| Pipeline diagram per representative program | ~5 baselines |
| Compare mode per documented instruction pair | ~5 baselines |
| Learn page hero render | per published learn page |
| OG image per fixture snapshot | per OG variant |
| Foundation demo render per substrate package | 7 baselines |

## Determinism prerequisites

Snapshots only stable when rendering is deterministic:
- Wall-clock-free animation (per `DETERMINISM.md`)
- Frame-N rendering matches across browsers / GPUs
- Antialiasing tolerance: max 0.1% pixel diff before failure
- Font rendering: subpixel positioning disabled in test config

WebGPU vs WebGL rendering may have minor diffs — separate baselines per renderer if needed; CI runs both.

## Update policy

Visual diff failure → developer reviews diff image → if intentional, runs `make visual.update` to refresh baselines + commits.

No silent baseline updates. Every refresh has a commit message explaining the visual change.

## CI integration

| Gate | Threshold |
|---|---|
| `visual.datapath` | 0% pixel diff > 0.1% tolerance |
| `visual.kmap` | same |
| `visual.pipeline` | same |
| `visual.compare` | same |
| `visual.learn` | same |
| `visual.og` | same |
| `visual.foundation` | same |

## Caught by

- Playwright `toHaveScreenshot()` per fixture
- CI fails on any unmatched baseline
- `make visual.update` regenerates baselines under operator control
