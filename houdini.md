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

In the release bundle, open `Houdini/install/` and double-click **`Install RS PostFX for Houdini.bat`**
(Windows) or **`Install RS PostFX for Houdini.command`** (macOS). No admin rights are needed. The installer

- copies the payload to `%APPDATA%\RS-PostFX\rspostfx-houdini` (Windows) or
  `~/Library/Application Support/RS-PostFX/rspostfx-houdini` (macOS), and
- writes a `packages/rspostfx.json` into **every** Houdini preferences folder it finds
  (`Documents\houdini21.0`, `houdini22.0`, … — OneDrive-synced Documents folders included),

so one install covers every Houdini version on the machine. Restart Houdini. An **RS PostFX** shelf appears with
four tools: **Copy Settings**, **Paste Settings**, **Save Preset**, **Load Preset**. If a shelf set doesn't show
it, add it with the shelf set's **+** tab.

**Versions.** This build ships Python bytecode for Houdini **20.0 through 22.0** (Python 3.10 to 3.13). Houdini
19.x is not covered by this release. The Windows Houdini installer needs a `python` on the PATH or falls back to
Houdini's own `hython`, so it runs on a plain workstation without any extra install.

## 2. Render a clean plate

{{ site.title }} applies the PostFX itself, so the EXR it reads must **not** already contain them. Two switches
decide that in Redshift for Houdini:

1. **The Redshift ROP, *Post Effects* tab.** Twelve checkboxes gate which output context applies the camera's
   Exposure, Color and Lens-Effects chain: IPR, MPlay, the **HDR file** output and the LDR file output. Their
   defaults apply exposure and lens effects to **all four**, including the EXR you write to disk
   (parameters `RS_PFX_HDR_exposure`, `RS_PFX_HDR_effects`, `RS_PFX_HDR_color`). For the plate, **untick the
   three HDR-file boxes**. Leave the IPR/MPlay boxes on — that keeps the viewport showing the look you are
   dialing in while the file on disk stays clean.
2. **The camera, *Lens Effects* section.** *Enable Lens Effects* (`RS_campro_enablePFX`) and *Apply to File
   Output* (`RS_campro_applyPFX`) also gate Bloom, Flare and Streak, and both default on. Turning *Apply to
   File Output* off is an alternative way to keep lens effects out of the EXR, but it does not cover exposure
   or the LUT — the ROP boxes above are the switch that covers everything.

Everything else about the render is unchanged: 32-bit float EXR, the colour space your pipeline uses (ACEScg or
linear sRGB/Rec.709), any AOVs you like — the plugin only ever touches RGB and passes alpha through.

{: .tip }
> **Want a reference to compare against?** Render the same frame a second time with the HDR-file boxes **on**.
> That EXR carries Redshift's own bake of the PostFX and is the ground truth for the A/B in step 6. Keep this
> reference **below 1200 px on its shorter side**: above that, Redshift's own file output can silently drop
> Bloom/Streak/Flare (a Redshift-side divergence, see [Matching Redshift]({{site.baseurl}}/matching-redshift#known-gaps-v1)).

## 3. Copy Settings in Houdini

Select the Redshift camera — or the Redshift ROP, the tool follows its *Camera* parameter to the camera it
renders — and click **RS PostFX: Copy Settings**. With nothing selected the tool uses the camera the current
viewport looks through, then the first Redshift camera or ROP it finds in the scene.

The tool puts one small JSON document on the clipboard **and** writes it to
`%APPDATA%\RS-PostFX\rs_postfx_bridge.json` (`~/Library/Application Support/RS-PostFX/…` on macOS). There is no
pop-up; the outcome and any warnings go to the diagnostic log `%APPDATA%\RS-PostFX\logs\rs_postfx.houdini.<pid>.log`
and, for a copy, to the Houdini Python shell.

What travels: every Optical value (Exposure Type, EV, ISO, f-stop, shutter type/time/angle, Whitepoint,
Vignetting), all of Tone-Mapping, Bloom and Streak, the LUT (name plus *Convert to Log Space* and *Strength*),
and two values Houdini **derives** rather than stores — the camera's horizontal FOV (from Focal Length and
Aperture) and the scene fps. The plugin's **Camera H-FOV** slider, which drives the vignette, is filled from the
first; the second lets the plugin warn you when a Movie-shutter exposure would differ because the AE comp runs at
a different frame rate. Flare and Color Controls are copied too but land under an `unsupported` key: the plugin
does not expose either yet (see [Matching Redshift]({{site.baseurl}}/matching-redshift#known-gaps-v1)).

## 4. Paste in After Effects

Import the EXR, set the project up for **scene-linear** pixels — this is the one setting that decides whether
anything matches, so check it first:

- **32 bpc**, and either **OCIO** colour management with the working space set to the space the EXR was
  rendered in, or **Adobe** colour management with *Linearize Working Space* **on**. The full explanation, and a
  ten-second self-test, is in [How to Install]({{site.baseurl}}/install#davinci-resolve-ofx) under *Colour
  pipeline*.
- Interpret the footage's alpha as **Straight**, not Premultiplied — otherwise After Effects divides every
  semi-transparent pixel's RGB by its alpha before the plugin sees it and the lens effects pump
  (see the [FAQ]({{site.baseurl}}/faq)).

Apply **Effect → RS PostFX → RS PostFX** to the layer and click **Paste from C4D** at the bottom of the effect
(the button is named for the first bridge host; it reads exactly the JSON the Houdini shelf produces). The plugin
reads the **clipboard first**, then the exchange file above, so on the same machine it simply works. Across two
machines, paste the JSON text through any channel — a chat message is enough — and copy it to the clipboard on
the AE side before clicking.

Paste **merges**: fields the JSON carries are applied, everything else stays as it was. Messages go to the plugin's
log (About & Support → *Open Log Folder*) rather than a dialog. Two checks worth knowing about: a **fps
mismatch** between the Houdini scene and the AE comp is reported when the shutter type is Movie (that ratio is
exactly the exposure error you would see), and a LUT that the Houdini side referenced is resolved against the
Redshift installation on the AE machine — set it once with **Settings ▸ Locate Redshift Installation…** if the
plugin didn't find it.

## 5. Presets instead of clipboard

**RS PostFX: Save Preset** on the shelf writes the same document as a named `.json` into the shared presets folder
(`%APPDATA%\RS-PostFX\presets`); **Load Preset…** in the effect's *Presets* group opens there. A preset saved
from a Houdini camera loads unchanged in After Effects, Cinema 4D and Resolve, and back. Details:
[Parameters → Presets]({{site.baseurl}}/parameters#presets).

## 6. Check the match

With the reference render from step 2 in the comp:

1. Put the clean plate with {{ site.title }} on it above the reference, blend mode **Difference**.
2. The frame should be black. Read a few pixels in the Info panel: neutral areas within a few 0.001, bloom and
   streak halos within about 0.002 rms on a 1080p production frame (the number the plugin is verified to).
3. What is **expected** to differ, and is not a setup error: strongly saturated colour through a LUT (Redshift
   applies a colour shift around its LUT that the plugin does not), a reference rendered at 1200 px or more on
   the shorter side (Redshift's own drop-out), and Flare or Contrast, which the plugin does not apply.

If the difference is a uniform gain instead — everything a bit brighter or darker — the plate was rendered with
the HDR-file boxes still on (step 2) and carries the exposure twice, or the AE project is not linear (step 4).

## What does not transfer

- **Curves** (Color Controls) never travel, on any host: their data is not readable from a scene.
- **Bloom tint on/off** has no Houdini equivalent — Houdini tints are always live; a white tint is a no-op.
- The plugin's **Output** group (*Show* Composite/Lens Effects only, the Multi-Pass Mixer) is an AE-side
  compositing control with no Redshift parameter behind it; it is never written on Copy and never applied on
  Paste.
- **Camera H-FOV** and **fps** are copy-only in every direction: a paste onto a Houdini camera never rewrites
  its focal length or the scene's frame rate.

[Back to top](#top){: .btn .float-right}

<link rel="stylesheet" href="{{ '/assets/css/general.css' | relative_url }}">
