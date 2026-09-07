---
title: FAQ
layout: default
nav_order: 101
---
# FAQ

## Is this an official Redshift or Maxon product?

No. {{ site.title }} is an independent plugin that replicates
Redshift's Photographic Exposure tonemapping by measurement — it isn't
built or endorsed by Maxon. The product name itself is still being
finalized, partly because of this.

## The lens effects pump/flicker on my animation — why?

Check the footage's alpha interpretation (Interpret Footage → Alpha).
When an EXR sequence is interpreted as **Premultiplied**, After Effects
un-premultiplies on decode — every pixel's RGB is divided by its alpha.
Renders are full of semi-transparent pixels with small, per-frame-noisy
alpha (anti-aliased edges, depth of field, motion blur, volumetrics), and
dividing by a tiny noisy alpha manufactures huge HDR spikes that change
every frame. Bloom and Streak then faithfully integrate those spikes into
a frame-wide glow, and the whole image pumps.

The fix: interpret the footage as **Straight** for back-to-beauty work
(the plugin's math then sees the pixel values Redshift actually rendered),
or, if you genuinely need matting removal, apply it *after* {{ site.title }}.
The plugin itself never reads alpha — RGB in, RGB out, alpha passed
through — and renders bit-identically for identical input (measured:
same frame, repeated renders, max difference 0.0).

## Why not just use After Effects' own Exposure effect?

AE's native exposure tools don't match Redshift's specific tone curve,
highlight rolloff, black crush, vignette falloff or the way Photographic
Exposure composes ISO/f-stop/shutter into a single gain. If you need a
Redshift beauty composite to match the render buffer exactly, those
differences show up. See [Matching Redshift]({{site.baseurl}}/matching-redshift).

## Why do I have to type in a Camera H-FOV — can't the plugin read my Redshift camera?

After Effects footage doesn't carry a camera object the way a Redshift
scene does, so there's nothing to read it from automatically today. A
planned "Read Metadata from Footage" feature will fill it in from the
source EXR's embedded Redshift metadata instead — see
[Matching Redshift]({{site.baseurl}}/matching-redshift#known-gaps-v1).

## Which hosts and platforms are supported?

After Effects and Premiere Pro (macOS `.plugin` / Windows `.aex`, 32-bit
float / SmartFX), plus DaVinci Resolve and other OpenFX hosts (the OFX
build). A specific minimum host version hasn't been pinned yet — see
[How to Install]({{site.baseurl}}/install) for the current pre-release
build status.

## How accurate is it, really?

See [Matching Redshift]({{site.baseurl}}/matching-redshift) for the
verification numbers per parameter group.

## What's not supported yet?

Contrast, Curves, and Flare (not in the v1 interface at all — see the next
question), GPU acceleration, and automatic metadata reading — see
[Matching Redshift's Known gaps]({{site.baseurl}}/matching-redshift#known-gaps-v1).

**LUT is supported** as of build 13, and build 14 adds a custom `.cube` file
picker to it.

## Why aren't Contrast, Curves, and Flare in v1?

{{ site.title }} only ships a control once it matches Redshift 1:1 against
real renders, and these three didn't clear that bar in time for v1 — so
they're kept out of the interface rather than shipped as approximations.
**Curves** can't be measured at all: Cinema 4D's Python API doesn't expose
the PostFX curve data, so there's nothing to read control points from or
check against a render. **Contrast** is fully measured but not yet
matched — Redshift's contrast is a saturation-dependent colour transform
with a near-step response at high values that a fixed transform can't
reproduce. **Flare** is measured in full except for two pieces — a ghost's
exact disk edge and the Halo's off-axis shape. All three keep their real
Redshift defaults internally, their measurements are kept, and they're
candidates for a future update. See
[Matching Redshift's Known gaps]({{site.baseurl}}/matching-redshift#known-gaps-v1)
for the full numbers.

## Can I use a LUT that isn't in my Redshift folder?

Yes, since build 14. **Choose .cube File…** in the LUT group opens a file
dialog and stores the file's full path with the project, which overrides the
dropdown (the dropdown greys out while it does). **Use Built-in List** clears
it again.

Two things worth knowing:

- the file is validated when you choose it, so a file that isn't a readable
  `.cube` is refused there and then rather than quietly doing nothing at
  render time;
- if the file later moves or is deleted, the row shows its name followed by
  **(MISSING)** and the LUT stage is skipped for those frames. Your render
  still completes.

Paths are absolute, so a project shared with another machine finds the LUT
only if the same path exists there. For shared projects, keep custom LUTs
somewhere with a stable absolute path, or use the built-in list — that one
resolves against each machine's own Redshift install.

## How does the LUT picker work?

The **LUT File** menu lists the `.cube` files in your Redshift LUT folder,
which the plugin finds automatically on both platforms. Two things about it
are worth knowing before you build a project around it:

- **The menu is built once per After Effects session.** After Effects can't
  rebuild an effect's dropdown after it's created, so a LUT you drop into
  the folder appears only after you restart AE.
- **What a project saves is the position in that list, not the file path.**
  If you add, remove or rename files in the LUT folder and reopen an old
  project, the effect will load whatever now sits at that position. "None"
  is always first, so a project with no LUT chosen is safe either way. If
  you rely on specific LUTs across a team, keep the folder's contents fixed
  — or copy the LUTs you use into a dedicated folder and point
  `RS_POSTFX_LUT_DIR` at it.

Set the `RS_POSTFX_LUT_DIR` environment variable to use a folder other than
Redshift's own.

## My composite comes out far too bright (or far too dark), and the bloom halos are missing — what's wrong?

Almost certainly the project is feeding the effect gamma-encoded pixels.
That happens with Adobe (classic) colour management whenever **Linearize
Working Space** is off — even with the working space set to "None", After
Effects then applies its 2.4 working gamma before effects and undoes it
afterwards — so every gain the plugin applies comes out raised to a power:
EV +1 renders as ×5.28 instead of ×2 (×4.3–5.0 with an sRGB working space),
Filmic settings land stops away from Redshift, and the Bloom/Streak
thresholds are compared against encoded values, so the halos all but vanish.
Two fixes, both measured to give exactly ×2 at EV +1 and a bloom error of
0.0015 rms against Redshift's bake: switch the project to **OCIO** colour
management with the working space your EXR was rendered in (ACEScg for an
ACES render), or keep Adobe colour management and turn **Linearize Working
Space on** (any working space). Quick check: EV Only, EV +1, everything else
off, must be exactly twice the plate. See [How to Install]({{site.baseurl}}/install)'s
colour-pipeline note.

## Does it matter which OCIO working space I pick, as long as it is linear?

For the exposure math, no — any scene-linear working space gives the same
result. For Bloom and Streak it matters a little: their thresholds are
computed from Rec.709 luminance in the working space, the way Redshift
computes them in its render space, so a working space with different
primaries (ACEScg vs. linear Rec.709) shifts the threshold slightly on
strongly saturated sources. Pick the space your render was made in and
interpret the EXR as that space, and the plugin sees exactly what Redshift
saw.

## Why does enabling the LUT lift my blacks?

Because Redshift does. Its Color Correction stage clamps to a fixed working
range — roughly 0.002 to 16.3 in scene-linear terms (13 stops centred on
middle grey) — and everything that passes through the LUT lands inside it.
The plugin reproduces that on purpose: the whole point is that a difference
blend against a Redshift render comes out black. If you want the LUT's look
without the clamp, apply it after this effect with something else.

## I see bloom/streak/flare from the plugin but my Redshift reference at 4K/UHD has none — is that a bug?

No — see [Matching Redshift's Known gaps]({{site.baseurl}}/matching-redshift#known-gaps-v1):
above `min(w,h) = 1200px` Redshift's own baked render can silently (and
non-deterministically) drop those effects; the plugin has no such boundary.

## Where are the log files, and what should I attach to a bug report?

{{ site.title }} writes one small diagnostic log per host process —
`rs_postfx.<host>.<pid>.log` — into `<app-data>/RS-PostFX/logs/`
(macOS `~/Library/Application Support/RS-PostFX/logs`, Windows
`%APPDATA%\RS-PostFX\logs`). Each file starts with a header naming the plugin
build, host, OS and CPU, then records the LUT folder it found, every
Copy/Paste outcome and every error — never per-frame data, never your
footage. Files roll over at 1 MB and stale ones are pruned after 14 days.
The About & Support twirl shows the current file's name; **Open Log Folder**
opens the folder, and **Get Support** copies the same facts to your clipboard.
For a bug report: click **Get Support**, paste, and attach the `rs_postfx.*.log`
files. Set `RSPE_LOG_LEVEL=debug` (or `trace`) before launching the host if
support asks for more detail.

[Back to top](#top){: .btn .float-right}

## Where do my presets live, and how do I share them with the team?

By default, `<app-data>/RS-PostFX/presets/` — macOS
`~/Library/Application Support/RS-PostFX/presets`, Windows
`%APPDATA%\RS-PostFX\presets` — right next to the `logs/` folder above. **Load
Preset…**/**Save Preset…** (in the effect's **Presets** group, just below
Enable PostFX) open there by default.

A preset is a plain `.json` file — the same portable format Copy/Paste
already use, plus a small `preset` block naming it — so it's safe to email,
drop on a shared drive, or commit to a repo. To share one folder across a
whole team, point everyone's plugin at the same synced/shared location with
**Settings ▸ Locate Presets Folder…** (and **Reset Presets Folder** to go
back to the default above); like the Redshift-install path beside it, this
is a per-machine setting, not something a project file carries. See
[Presets]({{site.baseurl}}/parameters#presets) for the full picture,
including what Load does when a file isn't a valid preset.

**Premiere Pro note:** Presets and Copy/Paste are verified in After Effects. In Premiere the same buttons exist
and read/write the same files correctly, but applying a loaded/pasted value into Premiere's own visible panel is
not yet verified for this release — see the limitation note in
[Presets]({{site.baseurl}}/parameters#presets).

**It's not just After Effects.** Cinema 4D and Houdini each have their own **Save Preset**/**Load Preset**
commands — C4D in the Extensions menu, Houdini on the `rspostfx` shelf, both next to their existing Copy/Paste
tools — and DaVinci Resolve/Fusion (via the OFX build) shows the same **Load Preset…**/**Save Preset…** rows this
plugin does. All four read and write the exact same portable `.json` file in the exact same shared presets
folder, so a preset saved from a Cinema 4D camera, a Houdini shelf tool, or Resolve loads unchanged in After
Effects, and back — see `bridge/README.md`'s **Presets** section for exactly what does and doesn't transfer
per host (C4D/Houdini presets carry no `Output`/Multi-Pass-Mixer block, since neither host has that Redshift
parameter to read it from).

[Back to top](#top){: .btn .float-right}

## Where do I report a bug?

{{ site.title }} is pre-release and not yet publicly distributed, so there's
no public issue tracker yet — this section will be filled in alongside the
first public release. When you do, attach the `rs_postfx.*.log` files (About
→ Open Log Folder) and paste what About → Get Support put on your clipboard.

[Back to top](#top){: .btn .float-right}

<link rel="stylesheet" href="{{ '/assets/css/general.css' | relative_url }}">
