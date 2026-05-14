# browser-support-matrix

## Decision

Latest stable Chrome / Edge / Firefox / Safari (desktop + mobile). Older = polite "browser too old" page.

Per `BROWSER-SUPPORT.md` for full matrix + capabilities.

## Why latest-only

- Per `book/PHILOSOPHY.md` "Latest only" rule applies to browser targets symmetrically with dep targets
- Modern browsers auto-update; lagging users are increasingly rare
- Polyfilling old browsers compounds bundle weight + test matrix
- WebGPU + modern features unavailable in old browsers anyway; 3D experience degrades silently below threshold

## Detection

`tools/detect-browser.ts` runs at app boot, gates the React tree:
- If browser version too old → render minimal HTML page with the line "this app requires a current browser"
- Else → render the app

Detection uses feature checks (per `BROWSER-SUPPORT.md` "Capability detection") not user-agent strings.

## Old-browser page

Single HTML file shipped statically. No JS required to render. Contains:
- One sentence ("this app requires a current browser")
- Link to Chrome / Edge / Firefox / Safari download pages
- Nothing else

## Why not polyfill aggressively

- Polyfills add bundle weight permanently
- Test matrix multiplies per supported version
- World-class endpoint targets capable hardware/software per `GOAL.md`

## CI matrix

Playwright cross-browser:
- Chromium (Chrome / Edge)
- Firefox
- WebKit (Safari)

Run on every PR + nightly.

## Caught by

- Capability-detection unit tests
- Cross-browser Playwright CI
- Old-browser page smoke
