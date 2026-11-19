# Release notes

## MirrorProbe 1.3.0 — 2026-11-18

A feature release for MirrorProbe: mirrors that recede inside mirrors, and cube
faces you turned off that now show the Background. GlassPass stays at 1.0.2.
Replace `MirrorProbe.tox` (and the quick-start, which carries its own copy).
Nothing changes in a scene that does not set the new parameters.

**Where it came from.** A ring of twelve flat mirror panels (an infinity room),
one Curved probe per panel, each captured at the camera's mirror image across its
panel so that the first reflection is exact. Two things went wrong. The mirrors
inside the mirrors never receded: each panel's capture stands outside the ring
and holds the room only through its own panel's window, so another panel's
capture, looking into it from elsewhere, saw mostly Background. And with only the
faces the first reflection needs turned on, deeper reflections showed pink, the
content TouchDesigner happened to leave in faces nobody rendered.

### Added

- **Environment > Room Probe** (Curved only): another MirrorProbe, captured inside
  the room (Shade Targets off, Parallax Box, all six faces). Every viewer but the
  Main Camera reads its cube map instead of the probe's own, box-projected from
  the mirror's surface, the way the Room Probe reads it itself. In practice that
  means the other probes' captures. The panels inside that cube show the room
  cube again, so reflections recede, each level one capture refresh older than
  the one in front of it.
- **Environment > Main Camera**: the camera the probe's own capture is meant for,
  the one viewer that still reads it. Empty, with a Room Probe set, every viewer
  reads the Room Probe. Its **Translate** is taken as where it stands, which keeps
  the material from re-reading a transform every frame, so keep it at the top
  level of its network, without a pivot.
- **Status** names a Room Probe that cannot serve (a Flat probe, a COMP that is
  not a MirrorProbe, the probe itself, a probe with no Targets) and a Main Camera
  that does not stand at its Translate. On a working setup it adds "room
  *probe* for all but *camera*" to the OK line.
- README: a new section, "Mirrors inside mirrors: the Room Probe", including how
  to place a panel's capture at the camera's mirror image.

### Fixed

- **A cube face that is off shows the Background.** What TouchDesigner leaves in
  a face it never rendered is not defined. In testing it held the Background
  colour, then pure red right after another face was switched on; the infinity
  room showed pink (1.5, 0.5, 0.5). The material now knows which faces were
  rendered, and a reflected ray that lands on any other face shows the
  Background. The same holds for the Room Probe's faces.

### Changed

- The About page reads **1.3.0**.
- The material has a fifth sampler (`sRoomCube`) and five new uniforms
  (`uMainEye`, `uRoomProbe`, `uRoomBoxMin`, `uRoomBoxMax`, `uCubeFaces`). The
  probe writes them itself when it upgrades.

### Verified

macOS (Apple Silicon), TouchDesigner 2025.33230. A new regression scene
(`docs/regression/mirrorprobe_room_probe.py`, 7 cases) has a closed hexagonal
ring of 3 m mirror panels between an unlit floor and ceiling. Each panel's probe
is captured at the camera's mirror image with only its first-bounce faces, and a
Room Probe sits at the centre. Every camera ray is traced in Python, so each pixel
is known to show a first reflection (floor or ceiling) or a reflection of another
panel.

- With the Room Probe, none of the 113,464 second-reflection pixels shows the
  Background (0.0%) and none is pink. First reflections match the floor and
  ceiling colours to 0.0023 (99th percentile, 102,929 pixels): the Main Camera
  still reads its own exact capture.
- Without it (as in 1.2.2), 74.5% of those pixels show the Background.
- With the panels' face masks forced to "nothing rendered", every first-reflection
  pixel falls through to the Background (100%), so the mask is what decides. That
  first reflections are exact with the real masks shows each face bit is the face
  the GPU samples.
- Status names each of the four broken setups it should.
- The room uniforms re-evaluate only when a setting changes: over about 100
  frames of the ring cooking, 0 calls to any of the room methods.
- At the default the quick-start renders bit-exact against 1.2.2: maximum
  difference 0.0 over the output, the planar capture and the cube hold, with
  time frozen.
- `mirrorprobe_far_and_sky.py` still passes 10/10 and `mirrorprobe_own_targets.py`
  11/11.
- The infinity room's scene-local patch (the first version of this feature),
  swapped for the released files on the same probes, upgrades itself to 1.3.0
  and keeps its Room Probe and Main Camera. The two render identically except
  for the deepest reflections at the horizon. There the scene changes by up to
  0.014 from one moment to the next under either version, and the two differ
  by no more than that.
- The exported toxes were loaded back beside 1.2.2's and compared operator by
  operator. The only differences are the two new parameters, Version, the
  extension and shader text, the new sampler and the five new uniforms. They are
  the same on the component and on both quick-start probes. Zero errors, zero
  warnings. With its animation frozen, the released quick-start renders
  identically to 1.2.2's (maximum difference 0.0).

### Upgrading

Replace the component with the new `.tox` (or replace your master if you keep
clones), or copy `MirrorExt.py` and `mirror.glsl` over an older component's: it
upgrades itself, Room Probe and Main Camera empty. A probe that was given Room
Probe and Main Camera by hand before 1.3.0 keeps their values.

## MirrorProbe 1.2.2 — 2026-11-18

A bug-fix release for MirrorProbe, with one new choice for what an empty
direction shows. GlassPass stays at 1.0.2. Replace `MirrorProbe.tox` (and the
quick-start, which carries its own copy).

**The defects.** Found by mirroring a house in an alpine meadow (Curved mode, the
cube map captured inside the house). Forested hills 1.3 to 2.4 km away and a snow
ridge at 8 km were missing from the mirror, and those directions came out as the
flat Background colour, which looked like a shading fault. The same flat colour
stood in for the sky: an outdoor mirror reflected a uniform blue.

**The causes.** The Curved capture camera kept TouchDesigner's default clipping,
Near 0.1 and Far 1000, so anything more than 1 km from the capture point was
clipped away. And where a capture holds nothing, the mirror could only show the
Background colour.

### Fixed

- **Quality > Near and Far** set the Curved capture's clipping. Far defaults to
  20000, which covers outdoor scenes; Near stays 0.1. Existing probes take these
  defaults when they upgrade. A Flat mirror needed no fix: its capture uses the
  View Render camera's own projection, so it already follows that camera's Near
  and Far. It shows what the camera could see along the reflected path.

### Added

- **Environment > Background** chooses what the mirror shows where its capture
  holds no geometry, such as the sky, or anything past Far:
  - **Colour** (the default): the Background Colour, exactly as before.
  - **Environment Map (Cube)** / **(Equirectangular)**: the TOP in the new
    **Environment Map** parameter, looked up along the reflected ray with no
    parallax, so a sky picture or an HDRI shows in the mirror. A cube map can be a
    Projection TOP or a Cube Map TOP. An equirectangular map is a 2:1 image such as
    an HDRI or Sky in Equirectangular projection, in the layout TouchDesigner's
    Projection TOP uses (u 0 looks along +X, 0.25 along +Z). Turn it with a
    Projection TOP's Rotate. Roughness blurs it through its mip levels like the
    capture. The menu names the layout because a TOP does not say from Python
    whether it is a cube map.
  - **Transparent**: the mirror writes alpha where its capture is empty, so the
    sky your render is composited over shows through (alpha is 1 minus the
    pixel's reflective share there). It is the sky behind the mirror on screen,
    not the reflected one, so it reads well at a distance and costs nothing. It
    needs a render with a transparent background over a sky of its own. Reflection
    Tint does not reach that sky.
  - With anything but Colour, both captures are transparent where nothing was
    drawn (Background Alpha 0), which is how the mirror finds those directions.
    Background Colour and Background Alpha are greyed out then.
  - An Environment Map choice with no map set shows the Background Colour, and
    **Status** asks for the map. A map of the other layout (a 2D image named as
    Cube, or the reverse) renders black. Within a second Status names it and the
    layout to pick, from the material's own sampler-type warning.
- Status names the Background when it is not the colour ("background sky
  (equirectangular)", "background transparent").

### Changed

- The Environment page's **Background** colour is now labelled **Background
  Colour**. Its name (`Bgcolor`) is unchanged, so expressions and scripts that set
  it keep working.
- The component's code now also writes the few network settings a release
  changes (`cam_capture`'s Near and Far, the captures' background alpha) when it
  upgrades. Copying `MirrorExt.py` and `mirror.glsl` over an older component
  upgrades it fully.
- The About page reads **1.2.2**.

### A GlassPass reading a probe

As with Background Alpha 0: GlassPass reads the probe's colour only, so with any
Background but Colour the glass reflects black where the probe captured nothing.
Keep Background on Colour for a probe that feeds a GlassPass, or give the glass
an Environment Map of its own.

### Verified

macOS (Apple Silicon), TouchDesigner 2025.33230. A new regression scene
(`docs/regression/mirrorprobe_far_and_sky.py`, 10 cases) has a 5 km ground plane, a
red 600 m box 3 km behind the camera, a flat mirror pane and a curved ball, and an
equirectangular sky whose colour is its own direction.
- On 1.2.1 the box was absent from the ball's cube map (0 red texels, Far 1000).
  On 1.2.2 it is present: 3,249 texels in the cube map and 67 pixels on the ball.
  At Far 1000 it is gone again.
- The flat pane shows the box when the camera's Far is 20000, and not at 1000, in
  both versions.
- Every sky pixel of the pane matches its analytic reflected direction to 0.0028
  with the equirectangular map, and to 0.0028 with the same map converted to a cube
  by a Projection TOP. The two agree to 0.0005 (99th percentile) on the pane and
  on the ball.
- Transparent at Reflectivity 0.5 writes alpha 0.5 and half the base colour on the
  pane's sky, and alpha 1 where the pane and the ball reflect ground.
- At the default the quick-start renders bit-exact against 1.2.1, Far 20000
  included: maximum difference 0.0 over the output, the planar capture and the
  cube hold, with time frozen. A positive control (Far 1) changed 168,387 output
  pixels.
- The 1.2.1 regression test (`mirrorprobe_own_targets.py`) still passes 11/11.
- The exported toxes were loaded back beside 1.2.1's and compared operator by
  operator. The only differences are the new parameters, Version, the extension
  and shader text, the two new samplers, the new uniform, `cam_capture`'s Near and
  Far, and the captures' alpha expressions. They are the same on the component
  and on both quick-start probes. Zero errors, zero warnings. With its animation
  frozen, the released quick-start renders identically to 1.2.1's (maximum
  difference 0.0).

### Upgrading

Replace the component with the new `.tox` (or replace your master if you keep
clones). Clones and upgraded probes take Far 20000 and Background Colour by
themselves. If you raised `cam_capture`'s Far by hand inside a probe, the
component now owns that setting: set **Far** on the Quality page instead.

## MirrorProbe 1.2.1 — 2026-11-09

A bug-fix release for MirrorProbe. GlassPass stays at 1.0.2. Replace
`MirrorProbe.tox` (and the quick-start, which carries its own copy).

**The defect.** A Flat mirror whose target sits inside another Geometry COMP (a
mirror pane on a door, inside a wardrobe, inside a room) could show the wrong
thing: a wall where a figure's reflection belonged. It looked right at some angles
(the mirror turned 8°) and wrong at others (38°). A Curved target that is a closed
body inside another COMP had the same defect: its cube map captured the body from
the inside.

**The cause.** The probe leaves its Targets out of its own capture with a `~` in
the capture's Geometry pattern. But an Environment of `*` lists the target's
outermost parent, a Render TOP draws every listed COMP's children with it, and `~`
removes only what the pattern itself matched. So a nested target drew itself into
its own capture. A flat pane lies exactly on the plane the reflection is clipped
at, so whenever rounding let it through it covered everything seen in the mirror.
A target at the top level of the network was never affected.

### Fixed

- **A mirror never draws into its own capture.** The mirror material drops its
  geometry for the probe's own capture camera, and only for that camera: every
  other camera, and shadow maps, draw it as before. On the defect's scene the
  reflection now lands within 0.6 px of where it geometrically belongs, with or
  without a camera Window Y shift, at every door angle tested.

### Changed

- **A nested Flat target on Every Frame reads its reflection one frame late.**
  The capture still has to list the target's parents to draw them, and the
  target's material reads the capture, so reading it directly is a cook loop.
  TouchDesigner resolves it by blanking the reflection on about half the frames
  once anything moves. The probe reads such a capture through its hold instead:
  always present, exact for a still camera, one frame behind a moving one. Status
  says when this applies ("every frame, one frame late (mirror is inside
  wardrobe)"). If the camera moves and that frame matters, keep the mirror's
  Geometry COMP at the top level. Top-level targets read the capture directly, as
  before.
- **Status names geometry that shows in a reflection when it should not**: a
  Target with Shade Targets off, or another probe's mirror, nested inside a COMP
  the Environment includes. Leave that COMP out with `~`, or move the object out
  of it.
- The About page reads **1.2.1**.

### Verified

macOS (Apple Silicon), TouchDesigner 2025.33230, on a regression scene with the
defect's geometry (a pane on a door turned 30–40° inside a wardrobe turned 8°, a
red sphere where the figure stood), 11 cases. On 1.2.0 the nested pane at 30° and
40° and a nested sphere in Curved mode changed their own capture (maximum
difference 0.80–0.96) and the reflection at 30° was missing. On 1.2.1 every
capture is identical with its target shown or hidden, and the reflection is within
0.6 px of the analytic position, with a Window Y shift and with a moving camera on
a top-level mirror. The quick-start, whose mirrors are top level, renders
bit-exact with the fix on and off (maximum difference 0.0 over the output, the
planar capture and the cube hold). The exported toxes were loaded back beside
1.2.0's and compared operator by operator: the only differences are the Version,
the extension and shader text, and the material's new uniform. Zero errors, zero
warnings; with its animation frozen, the released quick-start renders identically
to 1.2.0's (maximum difference 0.0).

### Upgrading

Replace the component with the new `.tox` (or replace your master if you keep
clones). Clones pick up the fix and the material's new uniform by themselves.

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
