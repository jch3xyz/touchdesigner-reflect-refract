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

Quality lives on its own page: cube face size, anti-aliasing, capture format,
clipping, and how often the capture refreshes (every frame, every N frames, or on
demand).

**Capture Format** defaults to 16-bit Float, so reflections stay HDR: a reflected
light or flame keeps its values above 1 for bloom and other post effects, matching
a scene-linear `rgba16float` main render. 8-bit Fixed clips reflections at 1 and
uses a little over half the memory.

**Near and Far** (Quality page) clip the Curved capture, measured from the capture
point. Far defaults to 20000, enough for hills and mountains in an outdoor scene;
anything beyond it is missing from the reflection and shows the Background
instead. Raise it if your scene is bigger. A Flat mirror uses the View Render
camera's own Near and Far, so it shows what that camera could see along the
reflected path.

**What an empty direction shows.** Where the reflection holds no geometry, such as
the sky, **Background** (Environment page) decides:

- **Colour** (default): the **Background Colour**. Match your main render's
  background.
- **Environment Map (Cube)** or **(Equirectangular)**: the TOP you set in
  **Environment Map**, a sky picture or an HDRI, looked up along the reflected ray.
  It is infinitely far away, so it has no parallax. A cube map can be a Projection
  TOP or a Cube Map TOP. An equirectangular map is a 2:1 image (u 0 looks along +X,
  0.25 along +Z, the layout TouchDesigner's Projection TOP uses), for instance a Sky
  in Equirectangular projection. Turn it with a Projection TOP's Rotate.
- **Transparent**: the mirror lets the sky your render is composited over show
  through. Use it when the sky is a picture behind your render rather than
  something in it, with the render's background alpha at 0. It shows the sky behind
  the mirror on screen, not the reflected one, so it is an approximation that reads
  well at a distance.

**Background Alpha** (with Background on Colour, default 1) is for anything that
reads the probe's capture rather than the mirror itself. At 0 the capture is
transparent wherever nothing was drawn, so a consumer can lay the captured objects
over a sky of its own (TD Water's `probe` reflection does this). The mirror looks
the same either way. The other Background choices always leave the capture
transparent there. Keep Background on Colour with Background Alpha 1 on a probe
that feeds a GlassPass: GlassPass reads colour only and would reflect black where
the probe captured nothing.

**Glass in a mirror is opaque.** In a mirror's capture a GlassPass object wears
its environment-only material, which shows the environment map through the glass,
not what is inside it. A flame inside a glass tube is therefore missing from the
reflection (seen in Flat mode; Curved captures use the same material). If the
contents matter more than the glass, leave the glass out of the probe's
**Environment** (`* ~geo_glass`): the reflection then shows the flame, without
the glass.

**Cube faces you turn off** (Quality page) show the Background. Turning off a
face nothing reflects saves a sixth of a Curved capture's cost each. Before 1.3.0
a reflected ray that landed on such a face showed whatever TouchDesigner left
there: the Background colour, red or pink.

## Mirrors inside mirrors: the Room Probe

A Curved mirror is exact for the point it was captured from. For a flat panel you
can make it exact for your camera. Set the panel's probe to **Capture Point**
Origin and **Parallax** Infinite, and set **Capture Offset** so the capture stands
at the camera's mirror image across the panel. For a panel whose plane passes
through point *c* with normal *n*, a camera at *e* has its mirror image at
*e* − 2((*e* − *c*)·*n*)*n*; the offset is that point minus the panel's origin.
Only the faces that view needs have to be on.

A ring of such panels (an infinity room) shows each panel's first reflection
exactly, but the mirrors inside the mirrors do not recede. Each capture stands
outside the room and holds it only through its own panel's window. Another panel's
capture looks into this one from somewhere else, misses that window, and sees
mostly Background.

**Room Probe** fixes that (Environment page, Curved only):

1. Add one more MirrorProbe for the room: **Targets** an empty Geometry COMP at
   the room's centre, **Shade Targets** off, **Parallax** Box with **Box Min** and
   **Box Max** on the walls, all six faces on.
2. On every panel's probe, set **Room Probe** to it and **Main Camera** to the
   camera the panels' captures are placed for.

The Main Camera still sees each panel's own exact capture. Every other viewer sees
the Room Probe's cube map, box-projected from the panel's surface: the other
panels' captures, above all. Inside that cube the panels show the room cube again,
so the reflections recede. Each level is one capture refresh older than the one in
front of it. On Every Frame the depth fills in within a few frames, and moving
things lag a frame per level.

- With **Room Probe** empty nothing changes: every viewer reads the probe's own
  cube map, as before 1.3.0.
- With a Room Probe and no Main Camera, every viewer reads the Room Probe, the
  main camera included: one shared, box-projected environment for all the panels.
- The Main Camera's **Translate** is taken as where it stands, which keeps the
  material from re-reading a transform every frame. Keep that camera at the top
  level of its network, without a pivot. **Status** says when it does not stand
  at its Translate.
- **Status** also names a Room Probe that cannot serve: a Flat probe, a COMP that
  is not a MirrorProbe, the probe itself, or a probe with no Targets. Until it is
  fixed, every viewer reads the probe's own cube map.
- Box projection is exact only for a box-shaped room. In a polygonal ring, set the
  box to the walls, and the deeper reflections are approximate.

## GlassPass in three steps

1. Drop it in, set **Host Render** to the Render TOP that draws your scene, and
   **Objects** to the geometry that should become glass.
2. Press **Setup Host**. It makes the three host edits for you and reports each one
   in **Status**: takes the glass out of the host render's Geometry, gives the
   objects the environment-only glass material, and routes whatever read the host
   render through this component instead.
3. Shape it on the **Look** page.

The environment-only material is what the glass wears everywhere except its own
pass: in mirror captures, in other cameras' renders and in a light's shadow map.
It never touches the host render, so the glass can sit in a scene with mirrors and
shadow-casting lights.

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
  needs Curved mode. For mirrors facing mirrors (a ring of panels), see
  "Mirrors inside mirrors: the Room Probe".
- A Flat mirror whose Geometry COMP sits inside another one that the Environment
  includes (a pane on a door inside a wardrobe) reads its reflection one frame
  late on Every Frame, and **Status** says so. That is exact for a still camera;
  for a moving one, keep the mirror's Geometry COMP at the top level.
- A Curved reflection is one frame behind its capture. That is what lets mirrors
  reflect each other (See Mirrors, on by default), lets glass and mirrors capture
  each other, and lets a shadow-casting light shine on all of them without a cook
  loop. It costs a second copy of the cube map: about 16 MB at 512 per face and
  64 MB at 1024 at 16-bit Float, half that at 8-bit Fixed.
- A Flat mirror on Every Frame reads its capture directly, so a light that casts
  shadows must not have the mirror among its **Shadow Casters** (`* ~ mirror1`). Otherwise
  TouchDesigner reports a cook dependency loop on the light. The probe's **Status**
  names the light and the target when this happens.
- An **Environment Map** you supply to GlassPass is read directly too. If that map
  is itself a render that draws the glass, or is lit by a light whose shadow map
  draws the glass, it loops; use a **Probe** for anything captured live.
- Box parallax is exact when the surroundings really are a box. For a dome, an
  L-shaped room or an open scene, use Infinite and a well-placed capture point.
- Both components write opaque alpha, except a MirrorProbe whose Background is
  Transparent.
- An Environment Map is read directly. It must not be a render that draws the
  mirror. If the map is animated, the mirror follows it every frame.
- MirrorProbe's Background menu names the map's layout (Cube or Equirectangular),
  because a TOP does not say from Python whether it is a cube map. A 2D image
  given as Cube, or the reverse, leaves the mirror black there; within a second
  **Status** names the map and the layout to pick.

## Other machines

Defaults are sized for an Apple Silicon Mac with a large unified-memory budget.
They were not tested on Windows or on a discrete GPU. If frame rate suffers, the
**Quality** page is where to scale down: lower the cube face size, drop
anti-aliasing to 1x, set Capture Format to 8-bit Fixed if nothing downstream needs
HDR reflections, switch **Update** from Every Frame to Every N Frames or On
Demand, and turn off cube faces that never show (the −Y face under an object
standing on a floor, for instance — each face you turn off saves a sixth of the
capture's cost).

## Licence

MIT — see `LICENSE`. Use it for anything, commercial work included, as long as the
copyright notice travels with it.

Made by Studio John Holmes.
