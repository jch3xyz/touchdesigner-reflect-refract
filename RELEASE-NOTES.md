# Release notes

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
  captures, each showing the other one capture behind. Flat mirrors are always
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
