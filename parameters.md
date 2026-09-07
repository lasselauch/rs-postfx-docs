---
title: 🎛️ Parameters
layout: default
nav_order: 2
---
# Parameters

Every parameter {{ site.title }} ships today, generated from the plugin's
own parameter definitions (`engine/rsp_param_table.c`, the single source of
truth for label/range/default as of the multi-host-engine campaign's Task 7,
cross-checked against `plugin/src/RSPostFX.h`'s
position/persistent-id enums) — labels, ranges and defaults are exactly
what's in the build, not an aspirational spec. As of the C4D-camera-UI
regroup, the panel's top-level structure
mirrors the **C4D RS camera UI's own tabs** rather than RenderView's flat
Display panel: **Optical**, then **Color Correction** (nesting
Tone-Mapping and LUT), then **Lens Effects** (nesting Bloom and Streak),
then **Output** (an AE-plugin-only compositing group, added in build 38,
moved here directly before Settings in a user revision round the same day
— see its own section below), then **Settings**, then the two bare **Copy
for C4D** / **Paste from C4D** buttons last (their "C4D Bridge" wrapper
twirl was removed in build 23). Every individual parameter's contents,
labels, ranges and defaults are unchanged by this regroup — only the
grouping and nesting changed.

Build 53 additionally hid **Color Controls** and **Flare** from the
interface entirely, in every host — not merely greyed, not drawn at all.
Both were candidates the regroup above still nested (Color Controls inside
Color Correction, Flare inside Lens Effects), but neither reached this
project's matching bar for v1, so as of build 53 the panel only shows
**Bloom and Streak** under Lens Effects, and **Tone-Mapping and LUT** under
Color Correction. Their parameters stay registered in the plugin (saved
projects and param counts are unaffected) with their real Redshift defaults
intact — see
[Not in v1 (measured, deferred)](#not-in-v1-measured-deferred) below.

Above all of that, since build 24, an **About & Support** twirl (collapsed by
default) shows the plugin version and build, the host's effect-API version
(the same "AE 13.28"-style number other plugins show), your OS, and this session's log file —
the facts a support request needs (your CPU travels in the copied support block) — with **Product Page**, **Documentation**,
**Get Support** and **Open Log Folder** buttons beside them. The fourth row
names this session's diagnostic log file (`rs_postfx.<host>.<pid>.log`);
**Open Log Folder** opens the folder it lives in (`<app-data>/RS-PostFX/logs/`,
one level below the data-folder root where the C4D bridge exchange file
itself sits), and **Get Support** first copies a support-info block —
version, host, OS, CPU, plugin path, log folder and file, LUT root, settings
file — to the clipboard and then opens the support page, so a bug report is
a paste plus an attached log. These rows are never greyed, even with Enable
PostFX off.
{: .fs-5 .fw-300 }

Directly below that sits **Enable PostFX** (the plugin's own bypass switch), then, as of 2026-09-06, a
**Presets** twirl — save or load the whole look as one portable file, see [Presets](#presets) below — and only
then does the panel proper begin with **Optical**.

Defaults marked **RS factory** match a freshly-created Redshift Camera
object's own factory value, measured directly — applying {{ site.title }}
untouched is a mathematical no-op end to end. See
[Matching Redshift]({{site.baseurl}}/matching-redshift) for what that means
in practice.

- - -

## Enable PostFX

| Parameter | Type | Default |
|:----------|:-----|:--------|
| Enable PostFX | checkbox | On |

This is the plugin's own bypass toggle, not a Redshift parameter — it has
no "factory default" to match.

- - -

## Presets

Added 2026-09-06, directly below **Enable PostFX** and above **Optical** (it briefly shipped at the very top of
the panel for one build, then moved here the same day for clarity of ownership). Save or load the whole PostFX
look as one portable file.

| Parameter | Type |
|:----------|:-----|
| Load Preset… | button — opens a native "open" dialog in the presets folder |
| Save Preset… | button — opens a native "save" dialog in the presets folder |
| *(status row)* | the preset in use, by name; after a Load, also names the host it came from |

**A preset is a plain JSON file** — the same `rs-postfx-bridge` document the **Copy**/**Paste** buttons at the
bottom of the effect exchange, plus one small additive block:

```json
"preset": { "name": "Warm night exterior", "created_at": "2026-09-06T21:40:12", "description": "" }
```

It carries the **whole** PostFX state — every Optical / Color Correction / Lens Effects value — plus the
[Output](#output) block (Show and the Multi-Pass Mixer sliders), since `output` has been part of every Copy since
Task L13; a preset saved from the C4D/Houdini side of the bridge instead won't carry it, the same way Copy
doesn't there (no matching Redshift camera field to read it from). A preset carries no path specific to the
machine that saved it: a custom LUT keeps the same relative-to-your-Redshift-install fallback Paste already
uses, so a teammate's preset resolves against LUTs on *their* machine, not yours. Being plain JSON, a preset file
is safe to rename, email, drop on a shared drive, or commit to a repo alongside a project.

**Save Preset…** opens a save dialog already pointed at your presets folder, pre-filled with the current
preset's name (or `Untitled.json` the first time). Type a name and save; the status row updates to show it.

**Load Preset…** opens an open dialog in the same folder. Loading **merges** — exactly like Paste, any field the
file doesn't mention is left exactly as it was, so a preset that only sets exposure and Bloom will not reset
Streak or the LUT you already had dialed in. Picking a file that isn't an RS PostFX preset (the wrong or a
missing `format`/`version`, or not valid JSON at all) is refused outright, and nothing in the effect changes.

**Where presets live.** By default:

| Platform | Path |
|:---------|:-----|
| macOS | `~/Library/Application Support/RS-PostFX/presets` |
| Windows | `%APPDATA%\RS-PostFX\presets` |

**Settings ▸ Locate Presets Folder…** (see [Settings](#settings) below) points the plugin at a different folder
instead — a synced or shared drive, so a whole team reads and writes the same presets — and **Reset Presets
Folder** returns to the default above. Like the Redshift-install path beside it, this is a **machine setting**,
not a project setting: stored once per user, shared by every project, and it does not travel inside a project
file.

**No more pop-ups, as of 2026-09-06.** Save/Load — and Copy/Paste, and both Settings folder buttons — no longer
show a confirmation or error dialog: After Effects has no status bar for an effect to write to, so every outcome
(a successful save, a load's applied/skipped count, a refused file, a folder that couldn't be written) is
recorded to the **diagnostic log** only. Use the **Open Log Folder** button in the About & Support twirl above to
find it — see [Settings](#settings) below for the folder both the log and presets live under.

**DaVinci Resolve / Fusion (OFX).** The OFX build mirrors the same three rows (Load Preset…, Save Preset…, a
status row) at the top of its own parameter list, and reads/writes the exact same portable `.json` file — a
preset saved from AE loads unchanged in Resolve and back. The one visible difference: OFX's status row is a
read-only field that shows the preset's **name** directly, where the AE/Premiere row instead stores the file's
absolute path and draws the name from it — same information either way, just held differently under the hood.

**Cinema 4D and Houdini.** Both have their own **Save Preset**/**Load Preset** commands — C4D in the Extensions
menu next to **RS PostFX: Copy**/**Paste**, Houdini on the `rspostfx` shelf next to **Copy Settings**/**Paste
Settings** — reading and writing the exact same portable `.json` file this plugin does, in the exact same shared
presets folder. A preset saved from Cinema 4D's RS Camera loads straight into this effect's **Load Preset…**, and
vice versa; the C4D/Houdini side has no `Output` block to save (no matching Redshift parameter), so a preset
saved there simply leaves this effect's Output settings untouched on Load, same as every other field a source
host doesn't have. Full detail: `bridge/README.md`'s own **Presets** section.

**Known limitation — Premiere Pro (v1).** **Presets and Copy/Paste are verified in After Effects.** In Premiere
Pro the same **Load Preset…**/**Save Preset…** buttons (and Copy/Paste) are present and their dialogs and files
work identically, but *applying* a loaded/pasted value into the visible Effect Controls panel relies on an
After-Effects-only mechanism (the AEGP suite) that Premiere does not expose — Premiere has no AEGP. Whether Load/
Paste visibly updates Premiere's own panel is **not yet verified for this release**; a Premiere-native fix (the
queued legacy `PF_Cmd_RENDER` path) is planned but not yet built. The file itself is always written/read
correctly either way — only the live panel refresh in Premiere is in question.

- - -

## Optical

Mirrors Redshift RenderView's **Optical** group.

| Parameter | Type | Range | Default |
|:----------|:-----|:------|:--------|
| Exposure Type | popup: EV Only / Filmic | — | EV Only (RS factory) |
| Exposure (EV) | slider | -20 .. 20 | 0 (RS factory) |
| Sensitivity (ISO) | slider | 25 .. 102400 | 100 (RS factory) |
| Whitepoint | color | — | white / (1,1,1) (RS factory) |
| Vignetting | slider | 0 .. 10 | 0 (RS factory) |
| Camera H-FOV | slider (degrees) | 10 .. 175 | 54.43° ¹ |
| Aperture (f/#) | slider | 0.5 .. 64 (UI: 1 .. 22) | 8 (RS factory) |
| Shutter Type | popup | Still / Movie ² | Still |
| Shutter Time (1/s) | slider | 1 .. 2000 | 60 (RS factory) |
| Shutter Angle (deg) | slider | 1 .. 360 | 180 (RS factory) |

¹ **Plugin-only parameter.** Redshift derives this from the actual camera
object; AE footage has no camera to read it from, so it's a manual control
for v1. A planned "Read Metadata from Footage" button (v1.5) will fill it
from the source EXR's `rs/camera/fov` key instead. 54.43° matches this
project's own calibration-render setup, not a Redshift default.

² Mirrors Redshift's Still / Movie popup. **Still** exposes with Shutter
Time (1/s). **Movie** exposes with the shutter angle over the frame
duration — measured bit-identical to Still at `360 × fps ÷ angle` (180° @
24 fps = 1/48 s, exactly Maxon's documented formula). The plugin takes
`fps` from the **AE comp's frame rate**, because Redshift takes it from the
C4D **document** frame rate and AE has no other equivalent — keep the two
equal (the C4D bridge warns on paste when they differ; that ratio is
exactly the exposure error). Whichever control is inactive is greyed. This
only matters in Filmic exposure; it never touches motion blur.

Exposure Type's third Redshift option, **Automatic**, is intentionally not
offered: it's renderer-side auto-metering with no deterministic per-pixel
formula to replicate.

- - -

## Tone-Mapping

Mirrors Redshift RenderView's **Tonemapping** group (label renamed to
"Tone-Mapping" to match the C4D camera UI's own spelling; nested inside
**Color Correction** in the panel).

| Parameter | Type | Range | Default |
|:----------|:-----|:------|:--------|
| Enabled | checkbox | — | Off (RS factory) |
| Highlights | slider | 0 .. 1 | 0.2 (RS factory) |
| Desaturate Highlights | checkbox | — | On (RS factory) |
| Blacks | slider | 0 .. 1 | 0 (RS factory) |
| Blacks Threshold | slider | 0 .. 1 | 0.25 (RS factory) |
| Saturation | slider | -1 .. 1 | 0 (RS factory) |

With Tonemapping **off**, the tone curve and saturation stage are skipped
entirely — exposure gain, whitepoint and vignetting still apply.

- - -

## Bloom

The first of Redshift's Lens Effects. Off by default so a fresh apply of
the effect stays a true no-op — bloom is the one stage that's expensive as
well as visible.

| Parameter | Type | Range | Default |
|:----------|:-----|:------|:--------|
| Bloom | checkbox | — | Off |
| Intensity | slider | 0 .. 4 | 1.0 (100%) |
| Threshold | slider | 0 .. 100 ¹ | 20.0 |
| Softness | slider | 0 .. 1 | 0 |
| Tint | checkbox + 5 swatches | — | Off, all white |

¹ An absolute HDR value, not a 0–1 fraction. A pixel whose luminance is at
or above the Threshold passes into the bloom in full; below it the knee falls
off, and **Softness** widens that knee (0 = a cliff, 1 = a very gentle
roll-off). Redshift compares the threshold against the average of each
**2×2 pixel cell** (an even-aligned grid, like a half-resolution copy of the
frame), so a single bright pixel needs four times the value of a bright
patch to bloom the same, and the halo is centred on the cell — the plugin
does exactly the same. The knee and its softness are measured on flat fields
(luminance/Threshold from 1/50 to 2, seventeen softness values) and
cross-checked on single-pixel impulses, squares from 2×2 to 8×8, coloured
fields and impulses on a grey field (docs/reports/bloom-fit-notes.md §12-13);
a 1080p production frame matches Redshift's own bake to 0.002 rms
(docs/reports/bloom-realscene-audit.md).
Redshift's own control clamps writes to 50; 50–100 here is extrapolation
along the table's last (flat) slope.

Tint swatch 1 tints the smallest bloom radius, swatch 5 the largest.

- - -

## Streak

The second Lens Effect, computed independently of and added to Bloom.

| Parameter | Type | Range | Default |
|:----------|:-----|:------|:--------|
| Streak | checkbox | — | Off |
| Intensity | slider | 0 .. 4 ¹ | 1.0 |
| Threshold | slider | 0 .. 200 (UI slider tops out at 100) ⁴ | 20.0 |
| Tail | slider | 0 .. 1 | 0.5 |
| Softness | slider | 0 .. 1 ² | 0 |
| Number | slider (lines) | 1 .. 8 ¹ | 3 |
| Angle | dial (degrees) ³ | — | 20° |

¹ Redshift clamps Intensity writes at 4.0 and Number at 8 — those are the
largest figures the renderer itself can be put into.

² Unlike Bloom's Softness, Streak's Softness blurs the streak field itself
(isotropically), not the bright pass. It's the model's weakest-fit
parameter (~4% fit) — flagged "partial" in the underlying reference header.

⁴ Same rule as Bloom's Threshold: compared against the 2×2 cell's average
luminance, full pass at or above it, a knee below (re-measured on flat
fields, docs/reports/streak-fit-notes.md §15). Streak's knee has no softness
control — see ² for what Streak's Softness does instead.

³ A dial (`PF_ADD_ANGLE`), not a slider — AE draws its needle at 0 = 12
o'clock while Redshift's own zero is +x, so the needle sits 90° off the arm
it names. The typed VALUE is still Redshift's own screen-space angle
unmodified, which is what matching a render depends on; matching the
number was judged to matter more than matching the needle. See the streak
fit notes for detail.

**Number** is lines, not arms: N lines through each bright pixel = 2N arms,
spaced 180°/N apart. **Angle** is verified to high accuracy on the raster
axes (0°/90°/180°/270°); off-axis — including the 20° default — Redshift's
own diagonal rasterization leaves a texture near each arm's core that this
model doesn't reproduce pixel-for-pixel, though arm directions, per-arm
profile and total energy all remain correct. See the streak fit notes for
the full breakdown.

## LUT

Redshift's PostFX **LUT** block. All defaults are off/neutral, so switching
the group on with no file chosen changes nothing.

| Parameter | Type | Range | Default |
|:----------|:-----|:------|:--------|
| Enabled | checkbox | — | Off |
| LUT File | dropdown | *None* + your Redshift LUT folder | None |
| **Custom File** ▸ | *sub-group* | | |
| &nbsp;&nbsp;Choose .cube File… | button | — | — |
| &nbsp;&nbsp;*(status row)* | text | the chosen file's name | *(none)* |
| &nbsp;&nbsp;Use Built-in List | button | — | — |
| Convert to Log Space | checkbox | — | Off |
| Strength | slider | 0 .. 1 ¹ | 1.0 |

Convert to Log Space and Strength sit **outside** the Custom File group
because they apply to whichever LUT is active — the one from the list or the
one you chose from disk.

¹ Redshift's own control clamps writes to [0, 1], so this is the real range
and not a convenience subset.

**How the file list works.** The menu is built **once per After Effects
session** from the `.cube` files in your Redshift LUT folder (found
automatically; override it with the `RS_POSTFX_LUT_DIR` environment
variable). A LUT added to that folder later appears only after an AE
restart. And what a project saves is the **position in the list, not the
path** — After Effects dropdowns can only store an index — so if the
folder's contents change, an old project reopens on whatever now occupies
that position. *None* is always first, so "no LUT" is stable regardless.

**Any LUT on disk: “Choose .cube File…”.** The dropdown can only offer what
your Redshift LUT folder held when the effect was first applied in this AE
session. The button opens a normal file dialog, and the file you pick is
stored **by its full path**, which is the important difference:

- while a custom file is set it **overrides** the dropdown, and the dropdown
  greys out to say so;
- a project stores the **path**, not a menu position — so it survives the
  folder's contents changing, and it points at the same file when the
  project is reopened;
- **Use Built-in List** clears it and hands control back to the dropdown.

The file is read the moment you choose it: if it isn't a valid `.cube`, the
plugin says so straight away and keeps whatever was set before. If the file
is later moved or deleted, the row shows its name followed by **(MISSING)**
in orange and the LUT stage is skipped for those frames — it will not fail
your render, and it will not pretend to be grading either.

**Sharing projects across machines.** Paths are absolute. A project opened on
another computer finds the LUT only if the same absolute path exists there —
so for a shared project, keep custom LUTs on a path that is the same
everywhere (a mounted volume with a fixed mount point, for instance), or use
the built-in list, which resolves against each machine's own Redshift
install. There is no “relative to the project” option: After Effects gives an
effect no reliable way to resolve one.

**Convert to Log Space** applies Redshift's Cineon log encode before the
lookup, for LUTs authored against log footage.

**Strength** blends linearly, in linear light, between the untouched image
and the LUT's output.

**Two things to expect.**

- **Enabling the LUT lifts blacks and clips extreme highlights** — to about
  0.002 and 16.3 in scene-linear terms. That is Redshift's own Color
  Correction working range, reproduced deliberately so a difference blend
  against a render matches.
- **Saturated colour is a known gap.** Redshift applies a colour shift
  around its LUT that this plugin doesn't reproduce. On grey and near-grey
  images it is a fraction of a percent; on strongly saturated ones it is
  not — a pure green pushed through an *identity* LUT comes back with up to
  0.318 of red. Neutral and near-neutral content matches; heavily saturated
  content will show a visible difference.

The LUT is the **last** stage in the chain — after tonemapping and after
Bloom and Streak — which is where Redshift applies it.

## Output

Added in build 38, moved here (directly before Settings, right after Lens
Effects) in a user revision round the same day -- it started out directly
under Enable PostFX. Unlike every other topic on this page, **Output is
not a Redshift parameter at all** — it's a pair of AE-plugin
**compositing** controls, so it carries no "RS factory" default to match
and it does **not** round-trip through the C4D bridge's Copy/Paste buttons
(there's no Redshift camera-object field for it to read from or write to).

| Parameter | Type | Range | Default |
|:----------|:-----|:------|:--------|
| Show | popup: Composite / Lens Effects only | — | Composite |
| **Multi-Pass Mixer** ▸ | *sub-group* | | |
| &nbsp;&nbsp;Bloom | slider | 0 .. 100 | 1.0 |
| &nbsp;&nbsp;Streak | slider | 0 .. 100 | 1.0 |

A Flare row exists in this mixer internally, but as of build 53 it's hidden
along with the rest of Flare — see
[Not in v1 (measured, deferred)](#not-in-v1-measured-deferred) below.

**Show = Composite** (the default) is the ordinary graded output, exactly
as every build before this one produced it — with every Multi-Pass Mixer
slider at its default 1.0, applying the effect is unchanged from build 37.

**The Multi-Pass Mixer** scales each lens effect's own pre-tonemap
*additive* term — not the whole image — before it's summed into the
composite. 1.0 is a pure pass-through of what Redshift itself would
produce; 0.0 behaves as if that effect's Enabled checkbox were off, without
touching the checkbox itself. The range runs to 100, well past 1.0, on
purpose: amplifying a lens effect beyond what Redshift itself would render
is the point of a mixer, not just attenuating it. Each visible slider greys
with its own effect's Enabled (Bloom's mixer with Bloom Enabled, Streak's
with Streak Enabled).

**Show = Lens Effects only** renders an **isolated glow on black** instead
of the composite (build 49): the combined additive contribution of every
active lens effect (`planes − c0`, linear), graded **once** through the
exact same vignette → tone curve/saturation → LUT pipeline the composite
uses, clamped to non-negative, and placed on a **black** background —
never composited against a Lens-Effects-off render. Alpha is a
**premultiplied glow-coverage value**, `clamp(luma(graded_glow), 0, 1)`
(Rec.709 luma), not the source alpha — this makes the pass drop straight
onto the user's own footage in **Add** or **Screen** (or even **Normal**,
since it's already premultiplied over black). With no lens effect active
on a given frame, this pass is always **fully transparent black** (RGB 0,
alpha 0) — a no-glow frame has zero coverage.

*(Before build 49, this popup rendered a difference pass instead — the
graded image *with* every active lens effect's contribution minus the
graded image with **none** of them, alpha = source alpha. That design
carried the source's own alpha and could hue-shift, since it subtracted
two independently-tonemapped/LUT'd images rather than grading one linear
glow. See `docs/reports/l12-output-passes-notes.md` for that design's own
notes.)

**The Flare mixer exists in the plugin for interface parity, not because
Flare is live.** Flare ships hidden (see
[Not in v1 (measured, deferred)](#not-in-v1-measured-deferred) below) — no
effect term is ever computed for it — so it **never contributes to the
Lens Effects only pass, at any mixer value**, and its mixer row is hidden
right alongside the rest of the Flare topic. It will start contributing the
moment Flare itself ships enabled.

## Settings

At the very bottom of the effect, because it configures the plugin rather
than the image. Two things live here: **where Redshift is installed**,
which is what the LUT File list is built from, and (added alongside
[Presets](#presets) above) **where presets are saved and loaded**.

| Parameter | Type |
|:----------|:-----|
| *(status row)* | the Redshift folder in use, marked *default* or *custom*, with the number of LUTs found |
| Locate Redshift Installation… | button — opens a folder picker |
| Reset to Default | button |
| Locate Presets Folder… | button — opens a folder picker, points [Presets](#presets)' Save/Load at it |
| Reset Presets Folder | button |

The plugin looks for Redshift in the usual install locations, so most people
never need this. If your Redshift lives somewhere else the LUT list comes up
empty, and this is the fix: point it at your Redshift folder (either the
install root or its `Data/LUT` folder — both work).

**Both are machine settings, not project settings.** Where Redshift is
installed, and where presets live, are properties of your computer, so each
is stored once per user and shared by every project — a project you send to
someone else will not carry either path into their setup. Both are kept in
one small file: `~/Library/Application Support/RS-PostFX/settings.json` on
macOS and `%APPDATA%\RS-PostFX\settings.json` on Windows.

**Changing the Redshift folder needs an AE restart to take effect** — the
LUT menu is built once per session and After Effects gives an effect no way
to refill a dropdown afterwards. (The `RS_POSTFX_LUT_DIR` environment
variable still works and still wins over this setting, for anyone who
scripts their setup.) **Changing the presets folder takes effect
immediately** — the very next Save/Load dialog opens there, no restart
needed.

**No dialogs, log only.** As of 2026-09-06, none of these four buttons pop
up a confirmation or error message — the outcome (the new folder, or why a
change failed) is written to the diagnostic log instead. See the About &
Support twirl above (**Open Log Folder**) to find this session's log file,
under the same `<app-data>/RS-PostFX/` folder as `settings.json` above (in
its own `logs/` subfolder).

## Not in v1 (measured, deferred)

**Contrast, Curves (RGB + per-channel R/G/B) and Flare are hidden from the
interface in every host as of build 53** — not merely greyed, not drawn at
all. {{ site.title }} ships a control only once it matches Redshift 1:1
against real renders, and none of these three cleared that bar for v1.
Every row below stays registered in the plugin (saved projects and param
counts are unaffected) and carries its real Redshift default internally,
and every measurement behind them is kept — they're candidates for a future
update. See
[Matching Redshift's Known gaps]({{site.baseurl}}/matching-redshift#known-gaps-v1)
for the full why and the numbers behind each one.

| Parameter | Type | Range | Default |
|:----------|:-----|:------|:--------|
| Color Controls ▸ Enabled | checkbox (hidden) | — | Off |
| Color Controls ▸ Contrast | slider (hidden) | −1 .. +1 | 0 |
| Flare ▸ Enabled | checkbox (hidden) | — | Off |
| Flare ▸ Intensity | slider (hidden) | 0 .. 1 ¹ | 0.2 |
| Flare ▸ Threshold | slider (hidden) | 0 .. 50 | 20 |
| Flare ▸ Softness | slider (hidden) | 0 .. 1 | 0 |
| Flare ▸ Chromatic | slider (hidden) | 0 .. 10 | 4 |
| Flare ▸ Size | slider (hidden) | 0 .. 1 | 0.5 |
| Flare ▸ Halo | slider (hidden) | 0 .. 1 | 0.6 |
| Flare ▸ Tint | checkbox (hidden) | — | Off |
| Flare ▸ Tint 1 … Tint 6 | colour (hidden) | — | white |
| Output ▸ Multi-Pass Mixer ▸ Flare | slider (hidden) | 0 .. 100 | 1.0 |

¹ Unlike Bloom's and Streak's, Redshift clamps Flare's Intensity at 1.0.

**Curves has no row above** because it was never buildable as a UI control
in the first place: Cinema 4D's Python API doesn't expose the PostFX curve
data, so there was never control-point data to wire a parameter to, and no
row for it exists in the plugin's own parameter table either.

**Focal length is not an input to Redshift's baked Flare** — a detail worth
knowing whether or not Flare is on screen, since it affects how you read an
A/B comparison. See
[Matching Redshift's Known gaps]({{site.baseurl}}/matching-redshift#known-gaps-v1)
for the measurement.

<link rel="stylesheet" href="{{ '/assets/css/general.css' | relative_url }}">
