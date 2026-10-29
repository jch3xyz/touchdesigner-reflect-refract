# Release notes

## MirrorProbe 1.2.0 — 2026-10-28

A feature release for MirrorProbe. GlassPass stays at 1.0.2. Replace
`MirrorProbe.tox` (and the quick-start, which carries its own copy).

### Added

- **Environment > Background Alpha**, 0 to 1, default 1: the alpha of both
  captures where the reflection shows no geometry. Captured objects are always 1.
  At 0 the empty directions are transparent, so a consumer that reads the capture
  can draw it over a sky of its own. Water's `probe` reflection is the first: it
  layers the probe's objects over its own sky map and wants Background Alpha 0 on
  the probe it reads. The mirror itself looks the same at any value.

### Changed

- **The mirror material draws its capture over the Background colour.** A
  TouchDesigner Render TOP writes its clear colour premultiplied, so a capture with
  Background Alpha 0 holds black where nothing was drawn, not the Background
  colour. The shader now adds `(1 - alpha) × Background` to what it reads, so the
  mirror's own reflection shows the Background colour whatever the alpha. At
  alpha 1 the added term is exactly zero: the quick-start renders pixel-identical
  to 1.1.2 (maximum difference 0.0 over the 1920×1080 output, the cube hold and the
  planar capture).
- **Clones of an upgraded master take the defaults of new parameters.** Until now
  a clone gained a master's new parameter at the style's zero, not at the schema's
  default: See Mirrors arrived off on every clone when it was added, and
  Background Alpha would have arrived at 0, which is transparent. The component
  now applies the default of any parameter newer than its recorded Version when
  its code initialises, then records the new Version. No manual step on clones.
- The About page reads **1.2.0**.

### A GlassPass reading a probe

GlassPass reads the probe's colour only, so a probe at Background Alpha 0 gives
the glass black where the probe captured nothing (the premultiplied clear). Keep
Background Alpha at 1 on a probe that feeds a GlassPass, or give the glass a probe
of its own. At the default nothing changes.

### Verified

macOS (Apple Silicon), TouchDesigner 2025.33230, on the quick-start with its
animation frozen (bit-exact frame to frame, maximum difference 0.0). At the
default, the output, the cube hold and the planar capture match 1.1.2 exactly, and
alpha reads 1.0 on every texel. With Background Alpha 0, and the room excluded
from the Environment so that empty directions exist: 1,022,844 of the cube's
1,572,864 texels read alpha 0 (every background texel) and 550,020 read 1 (every
object texel), nothing in between; the planar capture reads 1,710,951 at 0 and
360,687 at 1 of 2,073,600, with anti-aliased edges in between. The mirror ball and
panel match the alpha-1 picture to within one 8-bit step on 0.48% of the output's
pixels, because the capture stores the Background colour as a 16-bit float while
the shader's uniform is exact. The sketch's five glass passes still report
"reflecting probe glass_probe" with that capture at alpha 1 everywhere. The
exported toxes were loaded back and audited: zero errors, zero warnings.

### Upgrading

Replace the component with the new `.tox` (or replace your master if you keep
clones). Clones pick up Background Alpha at 1 by themselves.

## MirrorProbe 1.1.2 — 2026-10-20

A bug-fix release for MirrorProbe. GlassPass stays at 1.0.2. Replace
`MirrorProbe.tox` (and the quick-start, which carries its own copy).

**The defect.** In a long-running project, a MirrorProbe made other parts of the
scene slower the longer TouchDesigner stayed open, even while the probe sat idle.
In one project, every change to a rigged character in the probe's Environment
went from 25 ms to about 1.9 s after 14 hours. Only a restart cleared it.

**The cause** is the **Cost** readout on the Quality page, combined with a
TouchDesigner behaviour (2025.33230, macOS). Reading an operator's CPU cook time
from Python while TouchDesigner is evaluating an expression or a cook adds a
listener to that operator. The listener is dropped only when that operator cooks,
and the list never gets shorter. Cost read its capture's cook time from a
parameter expression every frame. An idle probe never renders its capture, so the
list grew for as long as TouchDesigner ran, and everything that touched the scene
the capture draws paid for its length.

### Fixed

- **Cost is read once a second, outside any evaluation.** The readout shows the
  same text as before and updates within a second. The parameter re-evaluates
  only when the text changes, so it costs nothing per frame.

### Upgrading

Replace the component with the new `.tox` (or replace your master if you keep
clones). The About page reads **1.1.2**. A probe that still carries 1.1.1's Cost
expression after its code is updated no longer leaks either.

## GlassPass 1.0.2 — 2026-10-11

A bug-fix release for GlassPass, for the TouchDesigner bug described under
MirrorProbe 1.1.1 below. MirrorProbe stays at 1.1.1. Replace `GlassPass.tox` (and
the quick-start, which carries its own copy).

**The defect.** A GlassPass that is not set up yet (no Host Render, Objects or
camera) leaked memory on every frame its tile was on screen in a network pane:
about 290 bytes a frame, never freed. A configured GlassPass did not leak.

**The cause** is the same TouchDesigner bug (2025.33230, macOS): once GPU cook
timing is on, every cook of a Render TOP or Render Pass TOP whose **Render**
toggle is off leaks. An idle GlassPass kept both its glass pass and its 8×8
stand-in render at Render off, and its tile cooked both every frame.

### Fixed

- **No render in GlassPass is ever off.** The glass pass and its 8×8 stand-in
  keep Render on. Until Host Render, Objects and a camera are all set, the pass
  draws nothing over the stand-in, which costs next to nothing.
- **GlassPass's internal cost helper no longer reads GPU cook time**, which would
  switch GPU timing on for the whole project. No parameter calls it, so 1.0.1
  never did; it is fixed so nothing can start to.

Measured with GPU timing on: an idle 1.0.1 on screen leaked 2 × 128 + 32 bytes
per frame; 1.0.2 leaks nothing measurable, idle or configured.

### Changed

- **A GlassPass with a Host Render but no Objects or camera now rests at 8×8**,
  as an idle one does, and does not take the Host Render until it is set up. A
  Render TOP can feed only one Render Pass TOP, so it stays free until then.
- A configured GlassPass looks exactly as before: pixel-identical to 1.0.1 in
  both example scenes.
- The About page reads **1.0.2**.

### Upgrading

Replace the component with the new `.tox` (or replace your master if you keep
clones). Clones pick the fix up from their master.

## MirrorProbe 1.1.1 — 2026-10-11

A bug-fix release for MirrorProbe. GlassPass stays at 1.0.1. Replace
`MirrorProbe.tox` (and the quick-start, which carries its own copy).

**The defect.** A scene with MirrorProbe 1.1.0 slowed down over hours. In one
project it fell from 60+ fps to 10–12 fps after 7.5 hours, and only a restart
cleared it.

**The cause** is a TouchDesigner bug (2025.33230, macOS). A Render TOP whose
**Render** toggle is off still cooks whenever something reads it. Once GPU cook
timing is on, each such cook leaks about 1.5 KB that is never freed, and the
leaked blocks make every GPU operator slower. MirrorProbe 1.1.0 triggered it every
frame, in every mode:

- the material always sampled both captures, and the one not in use sat with
  Render off: the cube capture in Flat mode, the flat capture in Curved mode;
- On Demand kept the capture in use at Render off between captures;
- the probe's own **Cost** readout read the capture's GPU time, which is what
  switches GPU timing on.

### Fixed

- **Neither capture is ever read with Render off.** Both keep Render on. A capture
  not in use is not read at all: the material reads a small constant instead.
- **Every N Frames and On Demand hold the last capture without re-cooking it.**
  Each capture now has a hold: a Feedback TOP whose target is a Switch TOP. On a
  refresh frame the Switch passes the render; on every other frame it passes the
  hold back to itself, and the render does not cook at all. Curved mode already
  read its capture one frame late, so its look and timing are unchanged.
- **Cost no longer reads GPU time.** It shows GPU memory and CPU cook time.
- **An unused capture still gives back its memory** when the Mode changes: it
  renders once at 8×8.
- **Looking at an idle probe in a network no longer cooks its capture.** The
  probe's tile now shows the TOP its material reads.

Measured on the quick-start, with GPU timing on: 1.1.0 leaked one 1.25 KB +
128 + 32 + 2 × 48 byte set per frame on Every Frame and Every N Frames, and two
sets on On Demand. 1.1.1 leaks nothing measurable in any mode, Flat or Curved.

### Changed

- **Flat mode on Every N Frames or On Demand now shows its capture one frame
  late**, through the hold. On Every Frame it still reads the capture directly, so
  a moving camera never sees the reflection slide.
- **The Flat shadow-caster loop only applies on Every Frame**; the slower updates
  read the hold, which breaks the loop.
- **Every N Frames renders only on its frames.** 1.1.0 cooked the capture every
  frame and rendered it on its frames.
- Every Frame looks exactly as before: pixel-identical to 1.1.0 in both example
  scenes.
- The About page reads **1.1.1**.

### Upgrading

Replace the component with the new `.tox` (or replace your master if you keep
clones) and press **Assign**. Clones of an upgraded master pick up everything
except their own **Operator Viewer** (Common page), which still shows the raw
capture and so renders it every frame while the clone's tile is on screen. Copy
the master's Operator Viewer expression to each clone to stop that.

## MirrorProbe 1.1.0 — 2026-10-06

A feature release for MirrorProbe. GlassPass stays at 1.0.1 and needs no change.

### Added

- **Quality > Capture Format** sets the pixel format of both captures, the Flat
  reflection and the Curved cube map (and the one-frame-late copy the material
  reads). Two settings:
  - **16-bit Float (HDR)**, the default. Reflections keep values above 1, so a
    reflected light, flame or sun stays hot for bloom, glare and tone mapping
    downstream. It matches a scene-linear `rgba16float` main render.
  - **8-bit Fixed**, the 1.0 behaviour and the cheaper option. Reflections clip
    at 1.
- **Status shows the format**: "OK: Curved, 512 per face, 1x AA, 16-bit float,
  every frame".

### Changed

- **Captures default to 16-bit float.** In 1.0 and 1.0.1 they were 8-bit, so a
  reflected emitter was capped at 1: an emitter at 6.0 came back as 1.0 from both
  captures, and a GlassPass reading a probe could reflect at most 1.0. At 16-bit
  the same emitter reflects at 5.45 in both modes (6.0 times the mirror's
  reflectance), and glass reflecting a 200 emitter through a probe reads 8.5.
- **Memory:**

  | Capture | 8-bit Fixed | 16-bit Float |
  |---|---|---|
  | Curved, 512 per face, 1x AA | 26 MB | 46 MB |
  | Curved, 1024 per face, 1x AA | 104 MB | 184 MB |
  | Flat, 1920x1080, 2x AA | 44 MB | 70 MB |

  Frame rate was unchanged in testing (60 fps).
- **The look can change in an 8-bit main render.** Anything brighter than 1 that
  a mirror reflects, such as a lit white floor, used to be clipped at the capture
  and then dimmed by the mirror's reflectance, so it read slightly grey. It now
  reflects at its true value, and the main render clips it like the original.
  In the quick-start example this brightens the reflected floor in both mirrors
  (5% of the frame, by 0.07 on average). For the 1.0 look, set Capture Format
  to 8-bit Fixed.
- The About page reads **1.1.0**.

### Upgrading

As for 1.0.1: replace the component with the new `.tox` (or replace your master
if you keep clones) and press **Assign**. Clones of an upgraded master pick up
Capture Format as 16-bit Float.

## 1.0.1 — 2026-09-29

A bug-fix release. Replace both components; the quick-start examples carry the
fixed components too.

**The defect.** In a scene with glass, a curved mirror and a light that casts
shadows, the glass seen in the mirror rendered solid black, and the light reported
"Cook dependency loop detected". Without a shadow-casting light the same scene
could render a whole mirror or its reflected glass black with no warning at all.

**The cause.** A material's sampler is a cook dependency whether or not the shader
reads it. Three such dependencies closed loops, and TouchDesigner breaks a loop
made only of TOPs and materials silently, by giving one material empty textures.

### Fixed

- **GlassPass: the environment-only glass no longer depends on the host render.**
  It bound the host image to a sampler its shader path never reads, so every render
  that drew the glass — a mirror's capture, a light's shadow map — depended on the
  host render, which draws the mirror. It now binds a constant of its own.
- **MirrorProbe: the Curved material reads the capture one frame late**, through a
  Feedback TOP inside the probe (`cube_prev`). A light whose shadow map draws the
  mirror no longer loops with the capture that light illuminates, so **Lights can
  stay at `*` with shadows on**.
- **Mutual capture works.** Two curved mirrors, or a glass probe and a mirror, now
  show each other. 1.0 documented this as "one capture behind"; in fact one of the
  two rendered black.
- **GlassPass `Probe`** reads the same one-frame-late capture.
- **MirrorProbe: an idle Flat capture no longer holds on to the scene's lights**
  while the probe is in Curved mode, which could leave a stale loop warning behind
  after switching modes.

### Added

- **MirrorProbe Status reports a Flat-mode shadow loop** and says what to change:
  "Cook loop: take mirror1 out of light1 Shadow Casters".

### Changed

- **Curved reflections are one frame behind.** Invisible in practice, and the
  capture itself is unchanged; Roughness blurs exactly as before.
- **Memory:** each Curved probe holds a second copy of its cube map — about 8 MB at
  512 per face, 34 MB at 1024. A Flat or idle probe adds a few kilobytes.
- The About page reads **1.0.1**.

### If you worked around 1.0

These are no longer needed and can be undone: `glsl_glassenv`'s first sampler
re-pointed by hand; **Lights** emptied on the probes; the mirror excluded from the
glass probe's **Environment**.

### Upgrading

Replace the component in your network with the new `.tox` and set its parameters
again, or, if you keep a master and clones, replace the master. The material on
your objects is inside the component, so press **Assign** (MirrorProbe) or
**Setup Host** (GlassPass) after replacing it.

### Verified

On macOS (Apple Silicon), TouchDesigner 2025.33230, in a scene with a glass
figure, a curved chrome figure, a glass probe and a soft-shadow light with default
shadow casters, nothing worked around: no warnings; the glass in the mirror's
capture went from 100% black to 0% black; shadows present in the render and in
both captures. The exported files were loaded back into a clean network and tested
together.

## 1.0 — 2026-09-20

First public release of **MirrorProbe** and **GlassPass**, with a quick-start
example for each.

### MirrorProbe

One component for both kinds of mirror.

- **Flat (planar) mode** renders the View Render's camera mirrored across the
  target's plane, with the near plane moved onto that plane so nothing behind the
  mirror shows. Exact and sharp, and correct for that one camera.
- **Curved (cube map) mode** captures a cube map around the target and serves any
  number of cameras at once.
- **Two-sided**, with a Front Only option and a backing colour.
- **Box parallax** with a one-press **Fit Box** that measures the room.
- **Mirror-in-mirror** is on by default: curved probes appear in each other's
  captures, each showing the other one frame behind (working since 1.0.1). Flat mirrors are always
  excluded, because a planar reflection is only correct for its own camera.
- **Shade Targets** can be turned off to make a probe an environment source only —
  how a GlassPass gets a cube map captured at the glass rather than borrowing a
  mirror's.
- **Quality page**: cube face size, anti-aliasing, per-face toggles, and Update
  (Every Frame / Every N Frames / On Demand). Probes on an interval are staggered
  so they never all render on the same frame.

### GlassPass

Physically-shaped refraction drawn over your own render.

- **Chain pattern**: a Render Pass over the host render, so the glass refracts the
  real scene at the real resolution and costs only the pixels it covers.
- **One-button Setup Host** makes the three host edits and reports each in Status.
- **One material per pass**, which is what makes several cameras correct at once,
  plus an environment-only variant the objects wear in mirror captures and other
  cameras' renders.
- **Look**: index of refraction, thickness, dispersion, frost, tint, absorption,
  specular. Two style controls — Lens Zoom and Edge Glow — bring back the older,
  less physical looks on purpose.
- **Environment**: a MirrorProbe, any cube-map TOP, or the component's own
  built-in gradient sky when neither is set, so a dropped-in component reflects
  something plausible immediately.
- **Source: Custom TOP** refracts any image instead of the scene, while still
  using the host render for depth.

### Both

- Self-contained `.tox` files — no external files, no file-sync, nothing to
  install.
- An **About** page carrying version, author, licence, and a Help button.
- Help text on every parameter.
- A quick-start example each, standalone and readable as five numbered bands.

## Platform

**Built and tested on macOS (Apple Silicon) only**, TouchDesigner 2025.33230.
No Windows or discrete-GPU testing was done before this release.

Defaults are sized for a large unified-memory budget. On a smaller GPU, scale down
from the **Quality** page: lower the cube face size, drop anti-aliasing to 1x,
switch Update to Every N Frames or On Demand, and turn off cube faces that never
show — each face you turn off saves a sixth of the capture's cost.

## Known limits

- Screen-space refraction cannot show what is off-screen or behind other glass.
- A flat mirror is correct for one camera, and is left out of other mirrors'
  captures by design; a mirror that must appear inside another needs Curved mode.
- A Curved reflection is one frame behind its capture (since 1.0.1).
- A Flat mirror must be kept out of the Shadow Casters of any shadow-casting light
  that lights its capture; Status says so when it is not.
- An Environment Map supplied to GlassPass is read directly, so a live render used
  there can still loop. Use a Probe for anything captured live.
- Box parallax is exact only when the surroundings really are a box.
- Both components write opaque alpha; there is no transparent output yet.
- `MirrorProbe_QuickStart` draws four renders per frame (a 1080p scene render, a
  1080p planar reflection, a 512 cube map and the palette camera widget's own
  view). It is built to show both mirror modes at once rather than to be cheap.
  Its two scene renders ship at 2x anti-aliasing; 4x costs about 47 MB more and
  buys very little at 1080p. If you want it lighter still, set the flat probe's
  Flat Resolution to Half, or switch the curved probe's Update to Every N Frames.
  Note that while the example is open in the network editor, the editor's own
  node viewers dominate its cost — judge the frame rate with the network closed.

## Licence

MIT. See `LICENSE`.
