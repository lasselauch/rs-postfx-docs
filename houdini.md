---
title: 🎬 Houdini to After Effects
layout: default
nav_order: 4
---
# Houdini to After Effects

The complete loop for a Redshift-for-Houdini shot: render a **clean** beauty EXR, copy the camera's PostFX
settings with one shelf click, paste them into {{ site.title }} in After Effects, and check that the composite
matches what Redshift shows. Cinema 4D users: the same steps apply, with the **RS PostFX: Copy** command in the
Extensions menu in place of the shelf tool.
{: .fs-5 .fw-300 }

{: .important }
> Redshift keeps **every** PostFX value on the camera in Houdini — the Redshift ROP holds none of them. Whatever
> you dial in on the camera's *Redshift Camera* tab (Optical, Color Correction, Lens Effects) is what the shelf
> tool copies, and what the plugin then reproduces on the linear plate.

## 1. Install the shelf

In the download, open `Houdini/install/` and double-click **`Install RS PostFX for Houdini.bat`** (Windows) or
**`Install RS PostFX for Houdini.command`** (macOS). No administrator rights are needed. The installer

- copies the shelf and its Python payload to `%APPDATA%\RS-PostFX\rspostfx-houdini` (Windows) or
  `~/Library/Application Support/RS-PostFX/rspostfx-houdini` (macOS), and
- writes a `packages/rspostfx.json` into **every** Houdini preferences folder it finds
  (`Documents\houdini21.0`, `houdini22.0`, … — OneDrive-synced, redirected and localized Documents folders such as
  `OneDrive\Dokumente` included),

so one install covers every Houdini version on the machine. Restart Houdini. An **RS PostFX** shelf appears with
four tools: **Copy Settings**, **Paste Settings**, **Save Preset**, **Load Preset**. If a shelf set doesn't show
it, add it with the shelf set's **+** tab.

**Versions.** This release supports Houdini **19.5 through 22.0** — for 19.5 its standard Python 3.9 build; the
optional Python 3.7 build of 19.5, and anything older, is not covered. The Windows
installer uses a `python` from the PATH if there is one and otherwise Houdini's own, so it runs on a plain
workstation without any extra install.

## 2. Render a clean plate

{{ site.title }} applies the PostFX itself, so the EXR it reads must **not** already contain them. Three things
decide what Redshift for Houdini writes into the file:

1. **The Redshift ROP's *Post Effects* tab.** Each output has its own row of checkboxes — *Viewport IPR Preview*,
   *MPlay Preview*, *HDR Files* and *LDR Files* — each with **Photographic Exposure**, **Bloom/Glare/Streak** and
   **Color/LUT/Controls**. For the EXR you composite, untick **Photographic Exposure** and **Bloom/Glare/Streak**
   under **HDR Files** (both are on by default): that keeps the tone curve, Bloom and Streak out of the file. Leave
   **Color/LUT/Controls** off there, as it is by default — ticking it bakes the camera's display transform into the
   file, which is no longer a linear plate. Keep the IPR and MPlay rows as they are, so the viewport still shows
   the look you are dialing in.
2. **The exposure gain itself is not a post effect in Houdini.** EV, ISO, f-stop and shutter are applied while the
   frame renders, so they are in the EXR no matter what you untick. Two ways to handle that:
   - **Exact:** click **Copy Settings** first, so the clipboard holds the exposure you want, then set the camera's
     Exposure Type to *EV Only* at **EV 0** for the file render. The plate comes out unexposed and the values you
     copied are applied in After Effects.
   - **Shortcut for EV-only setups:** leave the camera as it is, paste in After Effects, then set the plugin's
     Exposure (EV) to **0**. The gain is the first step of Redshift's chain, so a plate that already carries it
     composites the same.
3. **The camera's *Lens Effects* section.** *Enable Lens Effects* and *Apply to File Output* also gate Bloom, Flare
   and Streak, and both default on. Unticking *Apply to File Output* is an alternative to the ROP's
   Bloom/Glare/Streak box, but it does not cover the exposure above: the ROP boxes plus a neutral exposure are the
   combination that covers everything.

Everything else about the render is unchanged: 32-bit or 16-bit float EXR, the colour space your pipeline uses
(ACEScg or linear sRGB/Rec.709), any AOVs you like — the plugin only ever touches RGB and passes alpha through.
Verified on Redshift 2026.3 for Houdini 21.0.

{: .tip }
> **Want a reference to compare against?** Render the same frame a second time with the camera's exposure restored
> and the two HDR boxes ticked again. That EXR carries Redshift's own bake of the PostFX and is the ground truth
> for the check in step 6. Keep this reference **below 1200 px on its shorter side**: above that, Redshift's own
> file output can silently drop Bloom, Streak and Flare (see
> [Matching Redshift]({{site.baseurl}}/matching-redshift#known-gaps)).

## 3. Copy Settings in Houdini

Select the Redshift camera — or the Redshift ROP, the tool follows its *Camera* parameter to the camera it
renders — and click **RS PostFX: Copy Settings**. With nothing selected the tool uses the camera the current
viewport looks through, then the first Redshift camera or ROP it finds in the scene.

The tool puts one small block of text on the clipboard **and** writes it to
`%APPDATA%\RS-PostFX\rs_postfx_bridge.json` (`~/Library/Application Support/RS-PostFX/…` on macOS). There is no
pop-up; the outcome and any warnings go to the diagnostic log in `%APPDATA%\RS-PostFX\logs`.

What travels: every Optical value (Exposure Type, EV, ISO, f-stop, shutter type, time and angle, Whitepoint,
Vignetting), all of Tone-Mapping, Bloom and Streak, the LUT (name plus *Convert to Log Space* and *Strength*),
and two values Houdini **derives** rather than stores — the camera's horizontal field of view (from Focal Length
and Aperture) and the scene's frame rate. The plugin's **Camera H-FOV** slider, which drives the vignette, is
filled from the first; the second lets the plugin warn you when a Movie-shutter exposure would differ because the
After Effects comp runs at another frame rate. Flare and Color Controls are carried along but not applied: the
plugin does not offer either yet (see [Parameters]({{site.baseurl}}/parameters#not-available-yet)).

## 4. Paste in After Effects

Import the EXR, then set the project up for **scene-linear** pixels — this is the one setting that decides whether
anything matches, so check it first:

- **32 bpc**, and either **OCIO** colour management with the working space set to the space the EXR was rendered
  in, or **Adobe** colour management with *Linearize Working Space* **on**. The full explanation and a ten-second
  self-test are under [Colour pipeline]({{site.baseurl}}/install#colour-pipeline).
- Interpret the footage's alpha as **Straight**, not Premultiplied — otherwise After Effects divides every
  semi-transparent pixel's RGB by its alpha before the plugin sees it and the lens effects pump
  (see the [FAQ]({{site.baseurl}}/faq)).

Apply **Effect → RS PostFX → RS PostFX** to the layer and click **Paste from C4D** at the bottom of the effect —
the button is named for the first bridge host, and it reads exactly what the Houdini shelf produces. The plugin
reads the **clipboard first**, then the exchange file above, so on the same machine it simply works. Across two
machines, send the text through any channel — a chat message is enough — and copy it to the clipboard on the
After Effects side before clicking.

Paste **merges**: values the text carries are applied, everything else stays as it was. Messages go to the
plugin's log (About & Support → *Open Log Folder*) rather than a dialog. Two checks worth knowing about: a
**frame-rate mismatch** between the Houdini scene and the comp is reported when the shutter type is Movie (that
ratio is exactly the exposure error you would see), and a LUT the Houdini side referenced is resolved against the
Redshift installation on the After Effects machine — set it once with **Settings ▸ Locate Redshift
Installation…** if the plugin didn't find it.

## 5. Presets instead of clipboard

**RS PostFX: Save Preset** on the shelf writes the same settings as a named `.json` into the shared presets folder
(`%APPDATA%\RS-PostFX\presets`); **Load Preset…** in the effect's *Presets* group opens there. A preset saved
from a Houdini camera loads unchanged in After Effects, Cinema 4D and Resolve, and back. Details:
[Parameters → Presets]({{site.baseurl}}/parameters#presets).

## 6. Check the match

With the reference render from step 2 in the comp:

1. Put the clean plate with {{ site.title }} on it above the reference, blend mode **Difference**.
2. The frame should be black. Read a few pixels in the Info panel: neutral areas within a few thousandths, bloom
   and streak halos within about 0.002 rms on a 1080p production frame, which is what the plugin is verified to.
3. What is **expected** to differ, and is not a setup error: strongly saturated colour through a LUT (Redshift
   applies a colour shift around its LUT that the plugin does not), a reference rendered at 1200 px or more on
   the shorter side (Redshift's own drop-out), and Flare or Contrast, which the plugin does not apply.

If the difference is a uniform gain instead — everything a bit brighter or darker — the plate already carries the
camera's exposure and the plugin applies it again (step 2), or the project is not linear (step 4). If the tone
curve or bloom shows up in the difference, the HDR boxes were still ticked when the plate was rendered.

## What does not transfer

- **Curves** (Color Controls) never travel, on any host: their data cannot be read from a scene.
- **Bloom tint on/off** has no Houdini equivalent — Houdini tints are always live; a white tint is a no-op.
- The plugin's **Output** group (*Show* Composite / Lens Effects only, the Multi-Pass Mixer) is a compositing
  control with no Redshift parameter behind it; it is never written on Copy and never applied on Paste.
- **Camera H-FOV** and the **frame rate** are copy-only in every direction: a paste onto a Houdini camera never
  rewrites its focal length or the scene's frame rate.

[Back to top](#top){: .btn .float-right}

<link rel="stylesheet" href="{{ '/assets/css/general.css' | relative_url }}">
