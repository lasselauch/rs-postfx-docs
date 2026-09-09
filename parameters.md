---
title: 🎛️ Parameters
layout: default
nav_order: 2
---
# Parameters

Every control in {{ site.title }}, with its range and default. The panel follows the layout of the Redshift
camera's own PostFX tabs: **Optical**, then **Color Correction** (Tone-Mapping and LUT), then **Lens Effects**
(Bloom and Streak), then **Output**, then **Settings**, with **Copy for C4D** / **Paste from C4D** at the very
bottom. Above the panel proper sit **About & Support**, the **Enable PostFX** switch and **Presets**.
{: .fs-5 .fw-300 }

Defaults marked **RS factory** are the values a freshly created Redshift camera has, measured directly —
applying {{ site.title }} untouched changes nothing at all. See
[Matching Redshift]({{site.baseurl}}/matching-redshift) for what that means in practice.

- - -

## About & Support

A collapsed group at the top of the effect. Its four rows show the plugin version, the host version, your
operating system and this session's log file, with a button beside each:

| Button | What it does |
|:-------|:-------------|
| Product Page | opens the product page |
| Documentation | opens this documentation |
| Get Support | copies a support block — version, host, OS, CPU, plugin location, log file, LUT folder — to the clipboard, then opens the support page. A bug report is that paste plus the attached log |
| Open Log Folder | opens the folder that holds the diagnostic logs |

These rows always stay active, even with Enable PostFX off.

## Enable PostFX

| Parameter | Type | Default |
|:----------|:-----|:--------|
| Enable PostFX | checkbox | On |

The plugin's own bypass switch — not a Redshift parameter, so it has no factory default to match.

- - -

## Presets

Save or load the whole PostFX look as one portable file.

| Parameter | Type |
|:----------|:-----|
| Load Preset… | button — opens a file dialog in the presets folder |
| Save Preset… | button — opens a save dialog in the presets folder |
| *(status row)* | the preset in use, by name; after a Load, also the host it came from |

**A preset is a plain JSON file** — the same document the **Copy**/**Paste** buttons at the bottom of the effect
exchange, plus a name. It carries the **whole** PostFX state — every Optical, Color Correction and Lens Effects
value — plus the [Output](#output) settings. A preset carries no path specific to the machine that saved it: a
custom LUT is resolved relative to each machine's own Redshift installation, so a teammate's preset finds the LUT
on *their* machine, not yours. Being plain text, a preset is safe to rename, email, drop on a shared drive or
keep next to a project.

**Save Preset…** opens a save dialog already pointed at your presets folder, pre-filled with the current preset's
name (or `Untitled.json` the first time). Type a name and save; the status row updates to show it.

**Load Preset…** opens an open dialog in the same folder. Loading **merges** — exactly like Paste, any value the
file doesn't mention is left as it was, so a preset that only sets exposure and Bloom will not reset the Streak or
the LUT you already had dialed in. A file that isn't an RS PostFX preset is refused, and nothing in the effect
changes.

**Where presets live.** By default:

| Platform | Path |
|:---------|:-----|
| macOS | `~/Library/Application Support/RS-PostFX/presets` |
| Windows | `%APPDATA%\RS-PostFX\presets` |

**Settings ▸ Locate Presets Folder…** (see [Settings](#settings)) points the plugin at a different folder instead
— a synced or shared drive, so a whole team reads and writes the same presets — and **Reset Presets Folder**
returns to the default. Like the Redshift-installation path beside it, this is a **machine setting**, not a
project setting: it is stored once per user, shared by every project, and never travels inside a project file.

**No pop-ups.** Save, Load, Copy, Paste and the Settings buttons never show a confirmation or error dialog. Every
outcome — a successful save, how many values a load applied, a refused file, a folder that couldn't be written —
is recorded in the diagnostic log. **Open Log Folder** in About & Support takes you there.

**DaVinci Resolve / Fusion.** The OpenFX version has the same three rows at the top of its parameter list and reads
and writes the same files — a preset saved in After Effects loads unchanged in Resolve, and back. The one visible
difference: Resolve's status row shows the preset's name in a read-only field.

**Cinema 4D and Houdini.** Both have their own **Save Preset** / **Load Preset** commands — Cinema 4D in the
Extensions menu next to **RS PostFX: Copy** / **Paste**, Houdini on the **RS PostFX** shelf next to
**Copy Settings** / **Paste Settings** — reading and writing the same files in the same shared presets folder. A
preset saved from a Cinema 4D or Houdini camera loads straight into this effect, and vice versa. Those hosts have
no Output settings to save, so a preset from there leaves this effect's Output group untouched.

{: .note }
> **Premiere Pro.** Presets and Copy/Paste are verified in After Effects. In Premiere Pro the same buttons exist
> and read and write the files correctly, but the Effect Controls panel may not yet refresh to show the loaded or
> pasted values. A Premiere-specific fix is planned.

- - -

## Optical

Mirrors Redshift's **Optical** group.

| Parameter | Type | Range | Default |
|:----------|:-----|:------|:--------|
| Exposure Type | popup: EV Only / Filmic | — | EV Only (RS factory) |
| Exposure (EV) | slider | -20 .. 20 | 0 (RS factory) |
| Sensitivity (ISO) | slider | 25 .. 102400 | 100 (RS factory) |
| Whitepoint | color | — | white / (1,1,1) (RS factory) |
| Vignetting | slider | 0 .. 10 | 0 (RS factory) |
| Camera H-FOV | slider (degrees) | 10 .. 175 | 54.43° ¹ |
| Aperture (f/#) | slider | 0.5 .. 64 (slider 1 .. 22) | 8 (RS factory) |
| Shutter Type | popup | Still / Movie ² | Still |
| Shutter Time (1/s) | slider | 1 .. 2000 | 60 (RS factory) |
| Shutter Angle (deg) | slider | 1 .. 360 | 180 (RS factory) |

¹ **A plugin-only control.** Redshift takes the horizontal field of view from the camera itself, which drives how
the vignette falls off; After Effects footage carries no camera, so you enter it here — or let the
[Houdini / Cinema 4D bridge]({{site.baseurl}}/houdini) fill it in on Paste. 54.43° is a starting value, not a
Redshift default. Reading it automatically from the EXR's embedded Redshift metadata is planned.

² Mirrors Redshift's Still / Movie popup. **Still** exposes with Shutter Time (1/s). **Movie** exposes with the
shutter angle over the frame duration — identical to Still at `360 × fps ÷ angle` (180° at 24 fps = 1/48 s,
exactly Redshift's documented formula). The plugin takes the frame rate from the **composition**, because Redshift
takes it from the scene's frame rate and After Effects has no other equivalent — keep the two equal (the bridge
warns on Paste when they differ; that ratio is exactly the exposure error). Whichever control is inactive is
greyed. This only matters with Filmic exposure, and it never touches motion blur.

Redshift's third Exposure Type, **Automatic**, is intentionally not offered: it is renderer-side auto-metering
with no fixed per-pixel formula to reproduce.

- - -

## Tone-Mapping

Mirrors Redshift's **Tonemapping** group (the panel uses the Redshift camera's own "Tone-Mapping" spelling, nested
inside **Color Correction**).

| Parameter | Type | Range | Default |
|:----------|:-----|:------|:--------|
| Enabled | checkbox | — | Off (RS factory) |
| Highlights | slider | 0 .. 1 | 0.2 (RS factory) |
| Desaturate Highlights | checkbox | — | On (RS factory) |
| Blacks | slider | 0 .. 1 | 0 (RS factory) |
| Blacks Threshold | slider | 0 .. 1 | 0.25 (RS factory) |
| Saturation | slider | -1 .. 1 | 0 (RS factory) |

With Tone-Mapping **off**, the tone curve and saturation stage are skipped entirely — exposure gain, whitepoint
and vignetting still apply.

- - -

## Bloom

The first of Redshift's Lens Effects. Off by default so a fresh apply of the effect stays a true no-op — bloom is
the one stage that is expensive as well as visible.

| Parameter | Type | Range | Default |
|:----------|:-----|:------|:--------|
| Bloom | checkbox | — | Off |
| Intensity | slider | 0 .. 4 | 1.0 (100%) |
| Threshold | slider | 0 .. 100 ¹ | 20.0 |
| Softness | slider | 0 .. 1 | 0 |
| Tint | checkbox + 5 swatches | — | Off, all white |

¹ An absolute HDR value, not a 0–1 fraction. A pixel whose luminance is at or above the Threshold passes into the
bloom in full; below it the response falls off along a knee, and **Softness** widens that knee (0 = a cliff, 1 = a
very gentle roll-off). Redshift compares the threshold against the average of each **2×2 pixel cell** (an
even-aligned grid, like a half-resolution copy of the frame), so a single bright pixel needs four times the value
of a bright patch to bloom the same, and the halo is centred on the cell — the plugin does exactly the same. The
knee, its softness and the cell behaviour are all measured against Redshift renders; on a 1080p production frame
the added bloom matches Redshift's own to 0.002 rms. Redshift's own control stops at 50; 50–100 here continues the
same curve.

Tint swatch 1 tints the smallest bloom radius, swatch 5 the largest.

- - -

## Streak

The second Lens Effect, computed independently of Bloom and added to it.

| Parameter | Type | Range | Default |
|:----------|:-----|:------|:--------|
| Streak | checkbox | — | Off |
| Intensity | slider | 0 .. 4 ¹ | 1.0 |
| Threshold | slider | 0 .. 200 (slider to 100) ² | 20.0 |
| Tail | slider | 0 .. 1 | 0.5 |
| Softness | slider | 0 .. 1 ³ | 0 |
| Number | slider (lines) | 1 .. 8 ¹ | 3 |
| Angle | dial (degrees) ⁴ | — | 20° |

¹ Redshift itself clamps Intensity at 4.0 and Number at 8 — those are the largest values the renderer accepts.

² Same rule as Bloom's Threshold: compared against the 2×2 cell's average luminance, full pass at or above it, a
knee below. Streak's knee has no softness control — see ³ for what Streak's Softness does instead.

³ Unlike Bloom's Softness, Streak's Softness blurs the streaks themselves, evenly in every direction, not the
bright pass. It is the least precisely matched control in the plugin (within a few percent).

⁴ A dial, not a slider. After Effects draws a dial's needle at 0 = 12 o'clock while Redshift's zero points to the
right, so the needle sits 90° off the arm it names. The **value** you type is Redshift's own screen-space angle,
unchanged — that is what matching a render depends on.

**Number** is lines, not arms: N lines through each bright pixel means 2N arms, spaced 180°/N apart. **Angle** is
matched to high accuracy on the horizontal and vertical (0°, 90°, 180°, 270°). At other angles — including the
20° default — Redshift's own diagonal rasterization leaves a fine texture near each arm's core that the plugin
does not reproduce pixel for pixel, although arm directions, each arm's profile and the total energy all match.

## LUT

Redshift's PostFX **LUT** block. All defaults are off or neutral, so switching the group on with no file chosen
changes nothing.

| Parameter | Type | Range | Default |
|:----------|:-----|:------|:--------|
| Enabled | checkbox | — | Off |
| Custom LUT | menu + spinner | *None*, your Redshift LUT folder (a submenu per subfolder), *Choose File…* | *(none)* |
| Convert to Log Space | checkbox | — | Off |
| Strength | slider | 0 .. 1 ¹ | 1.0 |

Convert to Log Space and Strength apply to whichever LUT is active.

¹ Redshift's own control accepts 0 to 1, so this is the real range.

**Custom LUT is the control.** Click its field and a menu opens that is built **right now** from your Redshift LUT
folder — one submenu per subfolder — plus *None* and *Choose File…*. Click the **⇅** spinner at its right end to step
to the previous or next LUT, or **click-and-drag it up or down to scrub** through the list with the image updating
live as you go.

**OFX note.** DaVinci Resolve and Natron have no custom rows to draw, so there the plugin keeps a **LUT File**
dropdown instead — built once per session from what your Redshift LUT folder held at launch — next to a plain
**file-path field** for any `.cube` on disk, which the host fills with its own file browser. A path in that field
takes charge and greys the dropdown out; clearing the field hands control back to the dropdown.

**What a project saves.** The Custom LUT row saves the **file itself**: its full path plus its path relative to your
Redshift LUT folder — so a project made on a Mac reopens on a Windows machine with the same LUT, as long as that
LUT exists under its Redshift folder too.

**Choose File…** opens a normal file dialog for any `.cube` on disk. The file is read the moment you choose it: if it
isn't a valid `.cube`, the plugin refuses it (the reason is in the diagnostic log) and keeps whatever was set
before. If a chosen file is later moved or deleted, the row shows its name followed by **(MISSING)** and the LUT
stage is skipped for those frames — it will not fail your render, and it will not pretend to be grading either.

**Sharing projects across machines.** The Custom LUT row tries the stored absolute path first; if that doesn't
exist on the machine that opens the project, it falls back to the stored path **relative to a Redshift LUT
folder** — but only for a LUT that came from inside one (the folder it was picked under, or another machine's
equivalent). A LUT you pick via **Choose File…** from somewhere else on disk carries no such fallback: its absolute
path has to exist, unchanged, on every machine that opens the project, or the row shows **(MISSING)**. In OFX, the
**LUT File** dropdown has no fallback of any kind — it stores a position in its list, not a path, and reopens on
whatever now occupies that position on the machine that opens it.

**Convert to Log Space** applies Redshift's Cineon log encode before the lookup, for LUTs authored against log
footage.

**Strength** blends linearly, in linear light, between the untouched image and the LUT's output.

**Two things to expect.**

- **Enabling the LUT lifts blacks and clips extreme highlights** — to about 0.002 and 16.3 in scene-linear terms.
  That is Redshift's own Color Correction working range, reproduced deliberately so a difference blend against a
  render matches.
- **Saturated colour is a known gap.** Redshift applies a colour shift around its LUT that this plugin does not
  reproduce. On grey and near-grey images it is a fraction of a percent; on strongly saturated ones it is not — a
  pure green pushed through an *identity* LUT comes back with up to 0.318 of red. Neutral and near-neutral content
  matches; heavily saturated content will show a visible difference.

The LUT is the **last** stage in the chain — after tonemapping and after Bloom and Streak — which is where Redshift
applies it.

## Output

Unlike every other group on this page, **Output is not a Redshift parameter** — it is a pair of compositing
controls that exist only in the plugin. It has no factory default to match, and it does not travel through the
Copy/Paste bridge (there is no Redshift camera setting for it).

| Parameter | Type | Range | Default |
|:----------|:-----|:------|:--------|
| Show | popup: Composite / Lens Effects only | — | Composite |
| **Multi-Pass Mixer** ▸ | *sub-group* | | |
| &nbsp;&nbsp;Bloom | slider | 0 .. 100 | 1.0 |
| &nbsp;&nbsp;Streak | slider | 0 .. 100 | 1.0 |

**Show = Composite** (the default) is the ordinary graded output. With every Multi-Pass Mixer slider at 1.0, the
effect renders exactly what Redshift would.

**The Multi-Pass Mixer** scales each lens effect's own *additive* contribution — not the whole image — before it is
summed into the composite. 1.0 passes through exactly what Redshift itself produces; 0.0 behaves as if that
effect's checkbox were off, without touching the checkbox. The range runs to 100 on purpose: amplifying a lens
effect beyond what Redshift renders is the point of a mixer, not just attenuating it. Each slider greys out with
its own effect's checkbox.

**Show = Lens Effects only** renders the **glow alone, on black**: the combined contribution of every active lens
effect, graded once through the same vignette, tone curve and LUT pipeline the composite uses. Alpha is the glow's
own coverage (premultiplied), not the source alpha, so the pass drops straight onto your own footage in **Add**,
**Screen** or even **Normal**. On a frame where no lens effect is active, the pass is fully transparent black.

A Flare mixer will appear alongside Bloom and Streak when Flare ships — see
[Not available yet](#not-available-yet).

## Settings

At the very bottom of the effect, because it configures the plugin rather than the image: **where Redshift is
installed**, which is what the LUT list is built from, and **where presets are saved and loaded**.

| Parameter | Type |
|:----------|:-----|
| *(status row)* | the Redshift folder in use, marked *default* or *custom*, with the number of LUTs found |
| Locate Redshift Installation… | button — opens a folder picker |
| Reset to Default | button |
| Locate Presets Folder… | button — opens a folder picker; [Presets](#presets)' Save/Load use it from then on |
| Reset Presets Folder | button |

The plugin looks for Redshift in the usual install locations, so most people never need this. If your Redshift
lives somewhere else, the LUT list comes up empty, and this is the fix: point it at your Redshift folder (either
the install root or its `Data/LUT` folder — both work).

**Both are machine settings, not project settings.** Where Redshift is installed, and where presets live, are
properties of your computer, so each is stored once per user and shared by every project — a project you send to
someone else will not carry either path into their setup. Both are kept in one small file:
`~/Library/Application Support/RS-PostFX/settings.json` on macOS and `%APPDATA%\RS-PostFX\settings.json` on
Windows.

**Changing the Redshift folder takes effect immediately in the Custom LUT row** (its menu rescans every time it
opens); in OFX, the **LUT File** dropdown is built once per session and only picks the change up after a
Resolve/Natron restart. The Settings status row names both folders when they differ. (The `RS_POSTFX_LUT_DIR`
environment variable still works and still wins over this setting, for anyone who scripts their setup.)
**Changing the presets folder takes effect immediately** — the very next Save/Load dialog opens there.

**No dialogs, log only.** None of these buttons pop up a confirmation or error message — the outcome (the new
folder, or why a change failed) is written to the diagnostic log. **Open Log Folder** in About & Support opens the
folder; it sits next to `settings.json` above, in a `logs` subfolder.

## Not available yet

**Contrast, Curves (RGB and per-channel R/G/B) and Flare are not in the interface.** {{ site.title }} ships a
control only once it matches Redshift 1:1 against real renders, and none of these three has cleared that bar yet.
Their Redshift defaults are kept internally, so projects saved now will not shift when they arrive, and all three
are candidates for a future update. The reasons:

- **Flare** is measured in full — the six ghosts, their magnifications, the radial falloff, the bright pass, the
  Size law, the chromatic dispersion and its place in the chain — but two details are not yet matched to this
  product's standard: the outer edge of each ghost's disk and the Halo's brightness falloff across the frame.
- **Contrast** is fully measured but not yet matched: Redshift applies contrast as a saturation-dependent colour
  transform with a near-step response at high values, which a fixed transform cannot reproduce. A contrast control
  that is wrong on saturated colour is worse than none.
- **Curves** cannot be measured at all: Redshift does not expose the curve data to scripting, so there is nothing
  to read control points from, nothing to verify against a render, and nothing for the bridge to carry.

Worth knowing when you compare renders: **focal length is not an input to Redshift's Flare** — it is a pure
image-space effect. A shorter lens looks different only because it is a wider field of view, so a highlight covers
fewer pixels and sits nearer the frame centre.

[Back to top](#top){: .btn .float-right}

<link rel="stylesheet" href="{{ '/assets/css/general.css' | relative_url }}">
