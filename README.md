# MirrorProbe and GlassPass

Two drop-in TouchDesigner components for reflective and refractive surfaces.

- **MirrorProbe** turns any geometry into a mirror — a flat panel, floor or wall
  by planar reflection, or a curved body, sphere or chrome figure by cube-map
  probe. One component, one material, a Mode switch between them.
- **GlassPass** draws your objects as glass over your own render, refracting the
  real scene at the real resolution. Index of refraction, thickness, dispersion,
  frost, tint and absorption, with screen-space depth so anything in front still
  hides the glass.

They are independent: use either alone. They also cooperate — point GlassPass's
`Probe` at a MirrorProbe and the glass takes that probe's cube map, capture point
and parallax box automatically.

Each component ships with a quick-start example that teaches it in five numbered
bands you can read in the network.

| File | What it is |
|---|---|
| `MirrorProbe.tox` | The mirror component |
| `MirrorProbe_QuickStart.tox` | A room with a flat mirror panel and a curved chrome ball |
| `GlassPass.tox` | The glass component |
| `GlassPass_QuickStart.tox` | A glass sphere over a procedural backdrop |

## Requirements

TouchDesigner 2025 or newer (built and tested on 2025.33230).

**Built and tested on macOS, Apple Silicon, only.** See *Other machines* below.

## Install

Drop the `.tox` into your network, or put it in your palette
(`Palette` → right-click a folder → *Import Component*) so it is one drag away.
The components are fully self-contained: no external files, no dependencies,
nothing to install.

The quick-start examples are standalone too — they contain their own copy of the
component, so you can open one without the component `.tox` beside it.

## MirrorProbe in three steps

1. Drop it in, set **Targets** to the geometry that should become a mirror.
2. Press **Assign**. The Targets now wear the probe's mirror material.
3. Pick **Mode**:
   - **Flat (planar)** for floors, walls and panels. Set **View Render** to the
     Render TOP that draws the mirror. Exact and sharp, correct for that one camera.
   - **Curved (cube map)** for bodies, spheres and anything not flat. Correct for
     any number of cameras at once.

If the mirror sits in a room, set **Parallax** to Box and press **Fit Box** — the
reflection then lines up with the walls instead of sliding as the camera moves.

Quality lives on its own page: cube face size, anti-aliasing, and how often the
capture refreshes (every frame, every N frames, or on demand).

## GlassPass in three steps

1. Drop it in, set **Host Render** to the Render TOP that draws your scene, and
   **Objects** to the geometry that should become glass.
2. Press **Setup Host**. It makes the three host edits for you and reports each one
   in **Status**: takes the glass out of the host render's Geometry, gives the
   objects the environment-only glass material, and routes whatever read the host
   render through this component instead.
3. Shape it on the **Look** page.

Use one GlassPass per camera view. Each pass owns its own material and samples its
own background, which is what makes several cameras correct at once.

**What the glass reflects**, in order of precedence: a **Probe** (a MirrorProbe),
an **Environment Map** (any cube-map TOP), or — when neither is set — the
component's own small built-in gradient sky, so a freshly dropped-in GlassPass
reflects something plausible straight away.

## Known limits

- Screen-space refraction: the glass can only refract what is in the host image.
  Anything off-screen, or behind other glass, does not show through.
- A flat (planar) mirror is correct for one camera only, and is deliberately left
  out of other mirrors' captures. A mirror that must appear inside another mirror
  needs Curved mode.
- Curved mirrors reflect each other one capture behind (See Mirrors, on by default).
- Box parallax is exact when the surroundings really are a box. For a dome, an
  L-shaped room or an open scene, use Infinite and a well-placed capture point.
- Both components write opaque alpha.

## Other machines

Defaults are sized for an Apple Silicon Mac with a large unified-memory budget.
They were not tested on Windows or on a discrete GPU. If frame rate suffers, the
**Quality** page is where to scale down: lower the cube face size, drop
anti-aliasing to 1x, switch **Update** from Every Frame to Every N Frames or On
Demand, and turn off cube faces that never show (the −Y face under an object
standing on a floor, for instance — each face you turn off saves a sixth of the
capture's cost).

## Licence

MIT — see `LICENSE`. Use it for anything, commercial work included, as long as the
copyright notice travels with it.

Made by Studio John Holmes.
