---
title: FAQ
layout: default
nav_order: 101
---
# FAQ

## Is this an official Redshift or Maxon product?

No. {{ site.title }} is an independent plugin that replicates Redshift's Photographic Exposure tonemapping by
measurement — it is not built or endorsed by Maxon.

## The lens effects pump or flicker on my animation — why?

Check the footage's alpha interpretation (Interpret Footage → Alpha). When an EXR sequence is interpreted as
**Premultiplied**, After Effects un-premultiplies on decode — every pixel's RGB is divided by its alpha. Renders
are full of semi-transparent pixels with small, per-frame-noisy alpha (anti-aliased edges, depth of field, motion
blur, volumetrics), and dividing by a tiny noisy alpha manufactures huge HDR spikes that change every frame. Bloom
and Streak then faithfully turn those spikes into a frame-wide glow, and the whole image pumps.

The fix: interpret the footage as **Straight** for back-to-beauty work (the plugin then sees the pixel values
Redshift actually rendered), or, if you genuinely need matting removal, apply it *after* {{ site.title }}. The
plugin itself never reads alpha — RGB in, RGB out, alpha passed through — and renders identically for identical
input, frame after frame.

## Why not just use After Effects' own Exposure effect?

After Effects' native exposure tools don't match Redshift's specific tone curve, highlight rolloff, black crush,
vignette falloff or the way Photographic Exposure composes ISO, f-stop and shutter into a single gain. If you need
a Redshift beauty composite to match the render buffer exactly, those differences show up. See
[Matching Redshift]({{site.baseurl}}/matching-redshift).

## Why do I have to type in a Camera H-FOV — can't the plugin read my Redshift camera?

After Effects footage doesn't carry a camera the way a Redshift scene does, so there is nothing to read it from
automatically. Two ways around typing it: paste it with the [Houdini / Cinema 4D bridge]({{site.baseurl}}/houdini),
which computes it from the camera's focal length, or save a preset from the camera. Reading it from the EXR's
embedded Redshift metadata is planned.

## Which hosts and platforms are supported?

After Effects and Premiere Pro on macOS and Windows, and DaVinci Resolve (plus other OpenFX hosts) through the
OpenFX version. The Houdini and Cinema 4D bridges run on both platforms as well. See
[How to Install]({{site.baseurl}}/install).

## How accurate is it, really?

See [Matching Redshift]({{site.baseurl}}/matching-redshift) for the verification numbers per parameter group.

## What's not supported yet?

Contrast, Curves and Flare (not in the interface — see the next question), GPU acceleration, and reading camera
metadata from the EXR automatically. See [Matching Redshift's known gaps]({{site.baseurl}}/matching-redshift#known-gaps).

## Why aren't Contrast, Curves and Flare in the plugin?

{{ site.title }} only ships a control once it matches Redshift 1:1 against real renders, and these three haven't
cleared that bar yet — so they stay out of the interface rather than shipping as approximations. **Curves** cannot
be measured at all: Redshift does not expose the curve data to scripting, so there is nothing to read control
points from or check against a render. **Contrast** is fully measured but not yet matched — Redshift's contrast is
a saturation-dependent colour transform with a near-step response at high values that a fixed transform can't
reproduce. **Flare** is measured in full except for two details, a ghost's exact disk edge and the Halo's falloff
across the frame. All three keep their Redshift defaults internally and are candidates for a future update. See
[Parameters → Not available yet]({{site.baseurl}}/parameters#not-available-yet).

## Can I use a LUT that isn't in my Redshift folder?

Yes. **Choose .cube File…** in the LUT group opens a file dialog and stores the file's full path with the project,
which overrides the dropdown (the dropdown greys out while it does). **Use Built-in List** clears it again.

Two things worth knowing:

- the file is validated when you choose it, so a file that isn't a readable `.cube` is refused there and then
  rather than quietly doing nothing at render time;
- if the file later moves or is deleted, the row shows its name followed by **(MISSING)** and the LUT stage is
  skipped for those frames. Your render still completes.

Paths are absolute, so a project shared with another machine finds the LUT only if the same path exists there.
For shared projects, keep custom LUTs somewhere with a stable path, or use the built-in list — that one resolves
against each machine's own Redshift installation.

## How does the LUT picker work?

The **LUT File** menu lists the `.cube` files in your Redshift LUT folder, which the plugin finds automatically on
both platforms. Two things about it are worth knowing before you build a project around it:

- **The menu is built once per After Effects session.** After Effects can't rebuild an effect's dropdown after it
  is created, so a LUT you drop into the folder appears only after you restart After Effects.
- **What a project saves is the position in that list, not the file path.** If you add, remove or rename files in
  the LUT folder and reopen an old project, the effect will load whatever now sits at that position. "None" is
  always first, so a project with no LUT chosen is safe either way. If you rely on specific LUTs across a team,
  keep the folder's contents fixed — or copy the LUTs you use into a dedicated folder and point
  `RS_POSTFX_LUT_DIR` at it.

Set the `RS_POSTFX_LUT_DIR` environment variable to use a folder other than Redshift's own, or use
**Settings ▸ Locate Redshift Installation…** in the effect.

## My composite comes out far too bright (or far too dark), and the bloom halos are missing — what's wrong?

Almost certainly the project is feeding the effect gamma-encoded pixels. That happens with Adobe (classic) colour
management whenever **Linearize Working Space** is off — even with the working space set to "None" — so every
gain the plugin applies comes out raised to a power: EV +1 renders as about ×5 instead of ×2, Filmic settings land
stops away from Redshift, and the Bloom/Streak thresholds are compared against encoded values, so the halos all
but vanish. Two fixes, both verified to give exactly ×2 at EV +1 and a bloom that matches Redshift's own: switch
the project to **OCIO** colour management with the working space your EXR was rendered in (ACEScg for an ACES
render), or keep Adobe colour management and turn **Linearize Working Space on** (any working space). Quick check:
EV Only, EV +1, everything else off, must be exactly twice the plate. See
[Colour pipeline]({{site.baseurl}}/install#colour-pipeline).

## Does it matter which OCIO working space I pick, as long as it is linear?

For the exposure math, no — any scene-linear working space gives the same result. For Bloom and Streak it matters
a little: their thresholds are computed from luminance in the working space, the way Redshift computes them in its
render space, so a working space with different primaries (ACEScg vs. linear Rec.709) shifts the threshold slightly
on strongly saturated sources. Pick the space your render was made in and interpret the EXR as that space, and the
plugin sees exactly what Redshift saw.

## Why does enabling the LUT lift my blacks?

Because Redshift does. Its Color Correction stage clamps to a fixed working range — roughly 0.002 to 16.3 in
scene-linear terms (13 stops centred on middle grey) — and everything that passes through the LUT lands inside it.
The plugin reproduces that on purpose: the whole point is that a difference blend against a Redshift render comes
out black. If you want the LUT's look without the clamp, apply it after this effect with something else.

## I see bloom/streak from the plugin but my Redshift reference at 4K/UHD has none — is that a bug?

No — see [Matching Redshift's known gaps]({{site.baseurl}}/matching-redshift#known-gaps): from 1200 px on the
shorter side, Redshift's own file output can silently drop its lens effects; the plugin has no such limit. Export
the reference smaller for a clean comparison.

## Where are the log files, and what should I attach to a bug report?

{{ site.title }} writes one small diagnostic log per host session into `<app-data>/RS-PostFX/logs/` (macOS
`~/Library/Application Support/RS-PostFX/logs`, Windows `%APPDATA%\RS-PostFX\logs`). Each file starts with a
header naming the plugin version, host, OS and CPU, then records the LUT folder it found, every Copy/Paste and
preset outcome, and every error — never per-frame data, never your footage. Files roll over at 1 MB and stale ones
are pruned after 14 days. The About & Support group shows the current file's name; **Open Log Folder** opens the
folder, and **Get Support** copies the same facts to your clipboard.

For a bug report: click **Get Support**, paste the block it copied, and attach the `rs_postfx.*.log` files. If
support asks for more detail, set the `RSPE_LOG_LEVEL` environment variable to `debug` before launching the host.

## Where do my presets live, and how do I share them with the team?

By default in `<app-data>/RS-PostFX/presets/` — macOS `~/Library/Application Support/RS-PostFX/presets`, Windows
`%APPDATA%\RS-PostFX\presets` — right next to the `logs` folder above. **Load Preset…** and **Save Preset…** (in
the effect's **Presets** group, just below Enable PostFX) open there by default.

A preset is a plain `.json` file — the same portable format Copy/Paste use, plus a name — so it is safe to email,
drop on a shared drive, or keep next to a project. To share one folder across a whole team, point everyone's plugin
at the same synced or shared location with **Settings ▸ Locate Presets Folder…** (and **Reset Presets Folder** to
go back to the default); like the Redshift-installation path beside it, this is a per-machine setting, not
something a project file carries. See [Presets]({{site.baseurl}}/parameters#presets) for the full picture,
including what Load does when a file isn't a valid preset.

**It's not just After Effects.** Cinema 4D and Houdini each have their own **Save Preset** / **Load Preset**
commands — Cinema 4D in the Extensions menu, Houdini on the **RS PostFX** shelf, both next to their Copy/Paste
tools — and DaVinci Resolve / Fusion shows the same **Load Preset…** / **Save Preset…** rows this plugin does. All
four read and write the same file in the same shared presets folder, so a preset saved from a Cinema 4D camera, a
Houdini shelf tool or Resolve loads unchanged in After Effects, and back.

## Where do I report a bug?

Use **Get Support** in the effect's About & Support group: it copies everything a report needs to your clipboard
and opens the support page. Paste that block, attach the `rs_postfx.*.log` files (About & Support → **Open Log
Folder**), and, if the report is about a mismatch, include the plate and the Redshift reference for one frame.

[Back to top](#top){: .btn .float-right}

<link rel="stylesheet" href="{{ '/assets/css/general.css' | relative_url }}">
