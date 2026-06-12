# nucleus — Protons + Nucleus Stability Digital Model

Standalone single-file sim deployed to `ghostoutfit.github.io/nucleus/`.
Derived from the canonical `v11/index.html` in the `protons` repo.
Vanilla HTML + CSS + JS only. No build step — edit `index.html` directly.

To run locally: `python3 -m http.server` in this directory, then open `localhost:8000/`.

## Differences from canonical v11

- Image paths use `images/` (no `../images/` parent prefix)
- Hash routes `#nucleus` and `#stability` both open the Investigate Stability tab; `#protons` opens the Protons tab
- `#all` hash route opens a full-screen hub overlay (`#allModal`) with hotspot links to all four sims

## Hash routing

| Hash | Effect |
|------|--------|
| `#protons` | Protons tab |
| `#nucleus` or `#stability` | Investigate Stability tab |
| `#all` | Full-screen hub overlay |

`applyHash()` is bound to both init and `hashchange`. The `#all` overlay is driven by a separate IIFE at the bottom of the file listening to `hashchange`.

## Hub overlay (`#all`)

- `#allModal` — full-screen fixed div, shown when `location.hash === '#all'`
- Background: `images/TitleFull.png`; hotspot image: `images/TitleText.jpg`
- ✕ button calls `history.back()`
- Hotspot `<a>` links (absolutely positioned over `TitleText.jpg`):

| Row | URL |
|-----|-----|
| Protons | `ghostoutfit.github.io/nucleus/` |
| Investigate Stability | `ghostoutfit.github.io/nucleus/#stability` |
| Fission | `ghostoutfit.github.io/fission/` |
| Chain Reaction | `ghostoutfit.github.io/chain-reaction/` |
| Temperature | `ghostoutfit.github.io/fusion/` |
| Fusion | `ghostoutfit.github.io/fusion/#fusion` |

`positionAllHotspots()` repositions the hotspot container on resize.

## Tabs

- **Protons** — Coulomb-only. `currentTabIsProton = true`. Nuclear force, stability table, and force panel nuclear branch all disabled.
- **Investigate Stability** — Coulomb + nuclear force. Stability table (`NUCLEUS_TABLE`) active.

Tab switching: `applyTab(isProtonOnly)` — boolean arg, not a string.

## Physics constants

```javascript
const PARTICLE_RADIUS    = 10;
const SUBSTEPS           = 20;
const coulombStrength    = 139;
let   strongForceStrength = 0.6;
let   strongForceBaseline = 0.6;
let   strongForceRange   = 1.2;   // cutoff = PARTICLE_RADIUS * 2 * range = 24px
const COULOMB_CUTOFF_SQ  = (PARTICLE_RADIUS * 40) ** 2;
```

## History ring buffer

4000-frame ring buffer; each frame is a compact `Float32Array` snapshot (`{ n, data, t }` — 5 floats per particle: x, y, vx, vy, type; plus wall-clock timestamp for timestamp-based replay).

```javascript
const MAX_HISTORY = 4000;   // sim auto-stops when full
_histGet(idx)               // logical index → physical slot
_histPush(frame)            // O(1) ring write
_histReset()                // clear on new run
```

Replay is timestamp-based: each frame stores `t: performance.now()`. During playback the loop seeks to the frame matching elapsed wall-clock time scaled by `speed / recordedSpeed`.

## Speed slider

`#speedSlider` — integer steps 0–20. Default index 13 (`SPEED_DEFAULT_IDX`).
Blue tick mark at default position; snaps within ±2 steps (`SPEED_SNAP_RANGE`).

## Force panel

Draggable, resizable overlay (`#forcePanel`). Canvas: `#forcePanelCanvas`.

Key behaviors:
- Canvas **width** is synced to panel width on every `drawForcePanel()` call (fixes right-aligned text before first drag). Height is NOT synced per-frame (would cause feedback loop growth).
- On canvas resize (drag handle), `drawForcePanel()` is called immediately to prevent flash.
- Three modes: `arrowMode` 0 (hidden), 1 (All Forces), 2 (Total Force).
- Two selection modes: single particle (`!groupSelectMode`) and group (`groupSelectMode`).

**Unscaled overlays** (drawn before `scale(fpZoom)`, so never zoomed):
- "click a particle to see forces on it from other particles" — bottom-center, 13px, hidden in group mode, shrinks to fit narrow panels (min 7px)
- "total arrow length not to scale" — top-right, 8px, visible whenever `arrowMode === 2`
- Both switch to dark gray in light mode

**Group counter label** (drawn after `restore()`, screen coords):
- Format: `"N protons  N neutrons"` — no comma, no total
- Protons: pink `rgba(255,100,150,0.85)` / dark rose in light mode
- Neutrons: gray `rgba(170,170,170,0.8)` / dark gray in light mode

**"Click and drag" hint** (group mode, no selection): cyan in dark mode, dark gray in light mode.

## Images

```
images/
  favicon.png / favicon.svg
  Turtle.png / Rabbit.png / Stopwatch.png
  TitleFull.png      — hub overlay background
  TitleText.jpg      — hub overlay hotspot image
  logo-placeholder.png
```

No BSCS logo in this repo (bscs-logo.png lives in `protons/images/`).

## Deployment

GitHub Pages from `main` branch root. Push to `origin` to deploy.
Remote: `https://github.com/ghostoutfit/nucleus.git`

## Cross-sim search

```bash
# Compare with canonical v11
grep -n "functionName" index.html /path/to/protons/v11/index.html
```

Changes to canonical `v11/index.html` in the protons repo should generally be ported here. They are not auto-synced.
