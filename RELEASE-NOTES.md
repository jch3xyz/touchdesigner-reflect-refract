# Release notes

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
