# MOTION-CATALOGUE

Concrete timing + easing per transition. `UX-DOCTRINE.md` carries principles; this carries the spec.

Every value here lives in `packages/design-tokens` as a typed token. Hand-typed durations / easings in product code are violations.

## Animation engine

- **DOM motion**: `motion` library (motion.dev — framer-motion rebrand). `motion/react` for declarative `<motion.div>` etc.
- **Discrete 3D state transitions** (camera bookmarks, panel reveals): `motion`'s imperative `animate()` on Three properties. Works because `motion`'s `animate()` accepts any plain object — `Vector3` is `{x,y,z}`.
  ```ts
  import { animate } from 'motion';
  animate(mesh.position, { x: 2, y: 1, z: 0 }, { duration: 0.4, ease: [0.2, 0.8, 0.2, 1] });
  ```
- **Continuous 3D tracking** (camera follow, focus pull, hover): `THREE.MathUtils.damp3` critically-damped springs in `useFrame` with module-level pooled `Vector3`. Frame-rate-independent, no overshoot.
  ```ts
  import { damp3 } from 'three/src/math/MathUtils';
  useFrame((_, dt) => {
    damp3(mesh.position, targetVec, 0.25, dt); // smoothTime 0.25s = snappy; 0.6s = soft
  });
  ```
- **Banned**: `framer-motion-3d` (dead, succeeded by `motion`'s imperative `animate()` on three properties). `@react-spring/*` (pm4ai banned). `gsap` / `anime` / `lottie-*` (banned).

## Easing curves

| Name | Bezier | Use |
|---|---|---|
| `smoothOut` | `cubic-bezier(0.2, 0.8, 0.2, 1)` | Default for every transition; calm, premium |
| `smoothInOut` | `cubic-bezier(0.4, 0, 0.2, 1)` | Two-way transitions (drawer open/close) |
| `quickOut` | `cubic-bezier(0.2, 1, 0.4, 1)` | Snappy responses (button press, focus ring) |
| `quickIn` | `cubic-bezier(0.6, 0, 1, 0.4)` | Departures (dismiss, close) |
| `springSoft` | spring(stiffness=180, damping=22) | Drawers, panels, large surfaces |
| `springSnappy` | spring(stiffness=300, damping=25) | Small interactive feedback (hover lift, drag spring) |
| `signalPulse` | custom TSL — peak at 0.3, exponential falloff | Bus signal traveling along a curve |

## Duration scale

| Token | ms | Use |
|---|---|---|
| `instant` | 0 | Reduced-motion fallback for everything |
| `xs` | 80 | Hover state, focus ring, tooltip show |
| `sm` | 160 | Button press, popover open/close |
| `md` | 240 | Drawer slide, dialog enter/exit |
| `lg` | 400 | Camera dolly bookmark, stage transition focus pull |
| `xl` | 600 | Full scene exploded view, 3D mode flip 2D↔3D |
| `xxl` | 1000 | Onboarding tour step transitions, page navigation |

Per `A11Y.md` — `prefers-reduced-motion: reduce` collapses every duration to `instant` and disables every easing into snap-cuts. No information lost.

## 3D scene motion

### Camera dolly between bookmarks

- Duration: `lg` (400ms)
- Easing: `smoothOut`
- Position: cubic bezier interpolation
- Target: lerp interpolation
- FOV: lerp interpolation
- No spin / yaw acrobatics

### Stage transition (datapath step → step)

- Active components fade in over `md` (240ms) with `smoothOut`
- Inactive components fade out over `md` with `smoothOut`
- Depth-of-field focus pulls to active stage center over `lg` with `smoothOut`
- Signal pulses begin propagating at start of transition
- Total transition completes ≤ `lg` (400ms)

### Signal pulse along bus

- Pulse traverses bus geometry from source to destination
- Duration: derived from path length, base 300ms for shortest segment
- Easing: `signalPulse` TSL node (peak at 30% along path, exponential falloff after)
- Intensity peak: 1.0 at front of pulse, 0.3 trailing tail
- Color: active accent color, tinted slightly toward heat-color for high-frequency signals (control signals shift toward white)

### Component activation glow

- Idle: emissive intensity 0
- Active: emissive intensity 0.6 fade-in over `sm` (160ms), `smoothOut`
- Deactivation: fade-out over `md` (240ms), `smoothOut`

### Hover lift on interactive 3D entities

- Y-offset: +0.5 units over `xs` (80ms), `quickOut`
- Subtle outline glow: emissive intensity 0.1 fade-in
- Cursor: pointer

### Pipeline cell appearance

- Cell fade-in over `sm` (160ms), `smoothOut`, opacity 0 → 1
- New cells appear at next-cycle column
- Hazard overlay arrow draws over `md` (240ms), `smoothOut`, length 0 → full

### K-map group rectangle

- Drag rectangle: live update, no animation (follows pointer)
- Snap-to-group on release: spring `springSnappy` over `sm`
- New group fill: fade-in over `sm` (160ms)
- Group removal: fade-out over `xs` (80ms)

### K-map 3D toroidal cell value

- Block height change: spring `springSoft` over `md`
- Color shift on value change: lerp over `sm`

## DOM motion

### Button press

- Scale: 1 → 0.98 → 1 over `sm` (160ms), `quickOut`
- Background shift: lerp 10% darker over same window

### Focus ring

- Opacity: 0 → 1 over `xs` (80ms), `quickOut`
- Width: 0 → 2px over same window
- Persists while focused; fades out on blur same window

### Tooltip / Popover

- Show: opacity 0 → 1 + scale 0.95 → 1 over `sm` (160ms), `smoothOut`
- Hide: opacity 1 → 0 + scale 1 → 0.95 over `xs` (80ms), `quickIn`

### Dialog (modal)

- Enter: backdrop opacity 0 → 0.7 over `md` (240ms), `smoothOut`; dialog opacity 0 → 1 + translateY 8px → 0 over `md`, `smoothOut`
- Exit: reverse over `sm` (160ms), `quickIn`

### Drawer

- Enter: translate from edge over `md` (240ms), `springSoft`
- Exit: reverse over `md`, `quickIn`

### Toast

- Enter: opacity 0 → 1 + translateY 16px → 0 over `sm` (160ms), `smoothOut`
- Exit: opacity 1 → 0 over `xs` (80ms), `quickIn`

### Skeleton shimmer

- Background-position translation over 1.4s linear infinite
- Disabled under reduced-motion

### Sidebar collapse / expand

- Width transition over `md` (240ms), `springSoft`
- Content fade-in over `sm` (160ms) after width animation completes

### Menu / Select dropdown

- Open: opacity 0 → 1 + translateY 4px → 0 over `sm` (160ms), `smoothOut`
- Close: reverse over `xs` (80ms), `quickIn`

### Tab content switch

- Old content fades out over `xs` (80ms), `quickIn`
- New content fades in over `sm` (160ms) after old completes, `smoothOut`

### Page navigation (Next router)

- Loading bar at top (deterministic, never indefinite spinner)
- Skeleton geometry mounts immediately on navigation start
- Real content fades in over `sm` (160ms) when ready

## Frame-budget compliance

Every animation respects the 60fps frame budget:
- No layout thrash during animation (transform + opacity only where possible)
- GPU-accelerated transforms (`translateZ(0)` or `will-change` where measured beneficial)
- Animations using `requestAnimationFrame` driven by `sim-engine` clock per `DETERMINISM.md`

## Caught by

- Token presence lint: every duration / easing used in product code maps back to a `design-tokens` token
- Frame-budget test: every animation completes within stated duration ± 5ms
- Reduced-motion smoke: `prefers-reduced-motion: reduce` set → animations collapse to instant
