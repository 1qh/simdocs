# LICENSE-INVENTORY

Substrate packages: MIT.
Product app + Convex schema + tools: proprietary (no LICENSE file shipped).

This doc inventories third-party deps + their licenses + the attribution surface.

## Substrate package licenses

| Package | License | LICENSE file |
|---|---|---|
| `packages/three-kit` | MIT | `packages/three-kit/LICENSE` |
| `packages/hud` | MIT | `packages/hud/LICENSE` |
| `packages/design-tokens` | MIT | `packages/design-tokens/LICENSE` |
| `packages/sim-engine` | MIT | `packages/sim-engine/LICENSE` |
| `packages/editor` | MIT | `packages/editor/LICENSE` |
| `packages/bits` | MIT | `packages/bits/LICENSE` |
| `packages/boolean` | MIT | `packages/boolean/LICENSE` |

LICENSE file body (standard MIT, year and holder template-filled at scaffold time per operator-local credential):

```
MIT License

Copyright (c) [year] [operator]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## Third-party deps used by substrate

Compatibility matrix — every substrate dep must be license-compatible with MIT redistribution.

Locked compatible licenses for substrate-consumed deps:
- MIT
- BSD-2-Clause, BSD-3-Clause
- Apache-2.0
- ISC
- 0BSD
- Unlicense
- CC0-1.0
- MPL-2.0 (file-level copyleft; allowed for unmodified consumption)

Banned in substrate (would force substrate to be GPL):
- GPL-2.0 / GPL-3.0
- LGPL-2.1 / LGPL-3.0 (file-level only OK for unmodified consumption; flag for review)
- AGPL-3.0
- SSPL
- Commons Clause modifications

## Inventory generation

`tools/license-inventory.ts` walks `packages/*/package.json` direct + transitive deps, queries each for license, asserts compatibility, emits inventory report under `LICENSE-INVENTORY-GENERATED.md`.

Generated file committed; CI diffs on every push.

## Product app licenses

Product app (`apps/web`, `apps/backend`, `tools/`) is proprietary. Third-party deps in product can be any license compatible with internal proprietary use (most permissive + copyleft). Inventory still generated for audit.

## Attribution surface

Substrate distribution (via public OSS repo + future npm publish) includes:
- Per-package LICENSE file
- README listing third-party dep licenses (auto-generated section)
- `NOTICE` file for Apache-2.0 deps that require attribution

## Caught by

- CI license-inventory diff on every push
- License-compatibility lint asserts no banned license in substrate transitive deps
- Substrate publish blocked until LICENSE files present
