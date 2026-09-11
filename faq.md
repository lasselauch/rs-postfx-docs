---
title: FAQ
layout: default
nav_order: 101
---
# FAQ

## Is this an official Redshift or Maxon product?

No. {{ site.title }} is an independent plugin that replicates Redshift's Photographic Exposure tonemapping by
measurement — it is not built or endorsed by Maxon.

## Why is there a red X over my image?

That's the trial watermark — every feature works without a license, but a red X marks the frame until you
register. Open **About & Support** and click **License...** (After Effects also has a **Register** link at the
top of the effect), paste your aescripts license code and click **Activate**. Frames rendered during the trial
keep their X in the cache, so purge it afterwards (After Effects: *Edit › Purge › All Memory & Disk Cache*;
Premiere Pro: delete render files; Resolve: the frame refreshes on the next change). See
[Licensing and trial]({{site.baseurl}}/install#licensing-and-trial) for the full picture, including floating and
render-only licenses.

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

After Effects and Premiere Pro on macOS and Windows (x64), and DaVinci Resolve (plus other OpenFX hosts) through
the OpenFX version. The Houdini and Cinema 4D bridges run on both platforms as well. Windows on ARM isn't
supported yet — the aescripts licensing framework this plugin now links against has no ARM64 build. See
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

Yes. Click the **Custom LUT** row and choose **Choose File…** from the menu that opens; pick **None** from the same
menu to go back to no LUT. The row works the same way in After Effects and in Premiere Pro.

Two things worth knowing:

- the file is read the moment you choose it, so a file that isn't a readable `.cube` is refused there and then
  (the reason is in the diagnostic log) rather than quietly doing nothing at render time;
- if the file later moves or is deleted, the row shows its name followed by **(MISSING)** and the LUT stage is
  skipped for those frames — your render still completes. If the file is still there but no longer parses (edited
  into something invalid), the row shows **(UNREADABLE)** instead.

**Sharing this across machines** depends on where the file lives. A LUT picked from inside a Redshift LUT folder is
also saved as a path *relative* to that folder, so if its absolute path doesn't exist on another machine, the
plugin retries under that machine's own Redshift LUT folder and still finds it there. A LUT picked from somewhere
else on disk has no such fallback: its absolute path has to exist, unchanged, on every machine that opens the
project, or the row shows **(MISSING)** there too.

**OFX note.** DaVinci Resolve and Natron have no custom rows to draw, so the LUT group there shows two plain
controls instead: the **LUT File** dropdown (built once per session from your Redshift LUT folder) and a
**file-path field** for any `.cube` on disk, which you fill with the host's own file browser. A path in that field
takes charge and greys the dropdown out; empty the field and the dropdown is back in charge.

## How does the LUT picker work?

**Custom LUT is the control.** Click its field and a menu opens that's built right now from your Redshift LUT
folder — one submenu per subfolder — plus **None** and **Choose File…**. Click the **⇅** spinner at its right end
to step to the previous or next LUT, or drag it up or down to scrub through the list with the image updating live.

**What a project saves.** The Custom LUT row saves the file itself — its full path, plus its path inside your
Redshift LUT folder — so it survives folder changes and reopens on the right LUT on another machine, as long as
that LUT lives under a Redshift folder there too.

**OFX note.** DaVinci Resolve and Natron instead show a **LUT File** dropdown, built once per session from what
your Redshift LUT folder held at launch — a LUT you drop into the folder only appears there after a restart — next
to a plain file-path field for picking one from anywhere else on disk.

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

## Why is there no "Apply color management before LUT" checkbox?
{: #apply-color-management-before-lut }

Redshift's LUT settings have this checkbox; {{ site.title }} leaves it out on purpose.

**What it does in Redshift.** Normally the LUT is applied to the linear render, before anything converts it for
your screen. With the box ticked, Redshift first converts the image to what your display shows (using the display
and view from Redshift's colour management), applies the LUT to that, then converts the result back. It exists for
LUTs that were made for display images rather than raw renders.

**Why the plugin doesn't have it.** Copying it would mean the effect doing your project's display conversion
itself. In After Effects, Premiere Pro and Resolve that conversion belongs to the project's colour management, which
runs after the effect — the plugin works on the linear image, the same place Redshift applies its PostFX. A second
copy inside the effect would only be right if it matched your project exactly, and this step is unforgiving: in our
tests, even a LUT that changes nothing came back almost twice as bright in the highlights with the box ticked. So
instead of guessing, the option is left out.

**What this means for you.**

- **Box off in Redshift** (the default): nothing to do. The LUT matches.
- **Box on:** once a LUT is active, a Difference blend against the Redshift render won't be black. That is
  expected, not a setup problem. For an exact match, untick the box in Redshift and render the reference again.
- **Copy/Paste and presets:** a ticked box in Cinema 4D or Houdini is not applied when you paste into After
  Effects, and a paste or preset from After Effects turns the box off on the Cinema 4D or Houdini camera. Between
  Cinema 4D and Houdini the setting carries over normally.

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
