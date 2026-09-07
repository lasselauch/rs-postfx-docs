---
title: 📦 How to Install
layout: default
nav_order: 1
---
# How to install

{: .important }
> **{{ site.title }} is pre-release.** There's no installer or
> aescripts + aeplugins manager listing yet — until the first public
> release, install a dev build manually as below.

## After Effects & Premiere Pro

Premiere Pro loads the **identical bundle** built for After Effects — there is no separate Premiere build. Both
hosts are installed the same way, into the same **shared** plug-ins folder:

```
macOS:   /Library/Application Support/Adobe/Common/Plug-ins/7.0/MediaCore/
Windows: C:\Program Files\Adobe\Common\Plug-ins\7.0\MediaCore\
```

This "Common MediaCore" folder is the one location **both** After Effects and Premiere Pro scan — unlike After
Effects' own private `Plug-ins` folder (see the dev-only note below), which Premiere never looks in at all.

### Recommended: one-click installer

1. Build (or obtain) `RSPostFX.plugin` (macOS) / `RSPostFX.aex` (Windows).
2. Double-click the installer for your OS, sitting next to the plugin bundle in a release download (or in
   `plugin/mac/` / `plugin/win/` in a repo checkout):
   - **macOS**: `install.command`
   - **Windows**: `install.bat` (a shim for `install.ps1`)

   Each self-elevates (a graphical admin-password prompt appears — the MediaCore folder is root/admin-owned) and
   copies the plugin into MediaCore above, deleting any previous copy first (always overwrite, no prompts). It
   also removes any stray copy left in After Effects' own `Plug-ins` folder (including a `_dev` one, see below) so
   AE never ends up loading two copies of the same effect.
3. Quit **and relaunch** both After Effects and Premiere Pro — plugins are only loaded on startup, and a running
   host holds the previous build in memory.

From a repo checkout, `make -C plugin install-adobe` builds and drives the macOS installer above for you (also
needs the admin password). It appears (see the in-host live-check in docs/reports/p3-premiere-notes.md) as
**RS PostFX → RS PostFX** in both hosts' effect lists.

{: .note }
> Premiere has no concept of collapsible parameter groups — every group in the effect controls panel shows
> force-collapsed, regardless of how it's declared. This is expected Premiere behaviour, not a bug in this plugin;
> see `docs/reports/p3-premiere-notes.md` for the full list of known Premiere differences and the in-host
> verification checklist.

### Manual install

Drop `RSPostFX.plugin` (macOS) or `RSPostFX.aex` (Windows) directly into the MediaCore folder shown above
yourself, then restart both hosts — exactly what the one-click installer does, without the stray-copy cleanup.

### AE-only dev iteration (`_dev`)

During development this project also uses an AE-**only** `_dev` subfolder of After Effects' own private
`Plug-ins` folder — kept separate from your normal plug-ins so a broken dev build can't affect a real project,
and (`make -C plugin install`) fast to iterate on since it needs no admin password:

```
macOS:   /Applications/Adobe After Effects <version>/Plug-ins/_dev/
Windows: C:\Program Files\Adobe\Adobe After Effects <version>\Support Files\Plug-ins\_dev\
```

**Premiere never scans this folder.** It's an After-Effects-only fast loop for AE-only iteration; for Premiere
(or any install both hosts should see), use the MediaCore installer / `make install-adobe` above instead.

## DaVinci Resolve (OFX)

RS PostFX also ships as an OpenFX (OFX 1.4) plug-in, built from the same rendering engine as the After Effects/
Premiere Pro plugin above. Build (or obtain) `RS PostFX.ofx.bundle` and drop it into Resolve's shared OFX plug-ins
folder:

```
macOS:   /Library/OFX/Plugins/RS PostFX.ofx.bundle
Windows: C:\Program Files\Common Files\OFX\Plugins\
```

On **Windows**, a one-click installer ships next to the bundle (staged into `ofx/build/` by `make -C ofx win`):
double-click **`install.bat`** (a shim for `install.ps1`). It self-elevates and copies `RS PostFX.ofx.bundle`
into `C:\Program Files\Common Files\OFX\Plugins\`, deleting any previous copy first (always overwrite, no
prompts). Quit Resolve before running it so the old `.ofx` isn't locked. On **macOS**, a one-click installer ships next to the bundle too (staged into `ofx/build/` by `make -C ofx`):
double-click **`install.command`** in Finder. It shows a graphical admin-password prompt and copies
`RS PostFX.ofx.bundle` into `/Library/OFX/Plugins/`, deleting any previous copy first. Quit Resolve before running it.

Restart Resolve. It appears under **OpenFX → RS PostFX → RS PostFX** in the Color page's OpenFX effects
list — a different menu location than the AE/Premiere "Effect → RS PostFX" path above, since Resolve's own
OFX host groups plug-ins by vendor its own way.

{: .note }
> **Colour pipeline.** RS PostFX renders its tonemap/lens-effects/LUT pipeline on **scene-linear** pixel values in
> every host, including Resolve — it has no colour-space-aware input/output transform of its own.
>
> **After Effects: the effect must receive scene-linear pixels.** Both of AE's colour-management systems can do
> that; only one setting decides it (Project Settings › Color):
>
> * **OCIO colour management (recommended).** Choose the OCIO system and a config (the shipped *ACES 1.3 CG* is
>   fine). Set the **working colour space to the space your EXR was rendered in** — ACEScg for a Redshift ACES
>   render, *Linear Rec.709 (sRGB)* for a Redshift render in linear sRGB — and check that the footage is
>   interpreted as that same space (the file rules usually get an EXR right; Interpret Footage › Color shows it).
>   Matching the working space to the render space matters for *colour*: RS PostFX computes Bloom/Streak thresholds
>   from Rec.709 luminance in the working space, as Redshift does in its render space, so a different working
>   space shifts the threshold slightly on saturated sources. Nothing else to set.
> * **Adobe colour management (classic ICC).** Any working space is fine — sRGB, Rec.709, ACEScg — **as long as
>   *Linearize Working Space* is ON.** With it off (the default), After Effects hands every effect gamma-encoded
>   pixels and converts back afterwards, even when the working space is "None" (it then uses its 2.4 working
>   gamma): every exposure gain comes out raised to a power (EV +1 renders as ×5.28 with None or Rec.709 Gamma 2.4,
>   ×4.3–5.0 and non-uniform with sRGB) and Bloom/Streak thresholds are compared against encoded values, so the halos
>   all but vanish (bloom error 0.13 rms against Redshift's bake instead of 0.0015). Measured on 2026-09-07 across
>   nine project configurations, `docs/reports/bloom-realscene-audit.md`.
>
> **Self-test (10 seconds):** on a linear plate, apply RS PostFX with Exposure Type *EV Only*, Exposure (EV) **+1**
> and everything else off. The result must be exactly **twice** the plate everywhere (read a pixel in the Info
> panel). If it is ~5× or varies across the frame, the project is feeding the effect encoded pixels.
>
> Premiere Pro's colour management is a different system and has not been measured for this yet. In Resolve,
> bracket the RS PostFX node with **Color Space Transform** nodes (into scene-linear ahead of it, back out of
> scene-linear after it) so it sees the same linear data it does in After Effects/Premiere. A working-space popup
> on the plugin itself, so Resolve's own timeline colour space could drive this automatically, is a possible later
> addition — not part of this release. See `docs/reports/p4-ofx-notes.md` for the full OFX write-up, including the
> Resolve live-check.

## Houdini and Cinema 4D (the bridge)

The Houdini and Cinema 4D parts of the bundle are not effects — they are the **bridge**: shelf tools (Houdini) and
Extensions-menu commands (Cinema 4D) that copy a Redshift camera's PostFX settings so the plugin above can paste
them. Houdini's installer is one double-click, needs no admin rights, and covers every Houdini version on the
machine at once — see [Houdini to After Effects]({{site.baseurl}}/houdini) for the install and the full
render-to-composite workflow. Cinema 4D: copy the whole `rspostfx-c4d` folder into a `plugins/` folder Cinema 4D
scans and restart it.

## Finding it once installed

**Effect → RS PostFX → RS PostFX** (After Effects/Premiere); **OpenFX → RS PostFX → RS PostFX**
(Resolve — see above).

{: .note }
> That's the effect's internal, compiled-in name inside After Effects —
> distinct from the "{{ site.title }}{{ site.tagline }}" product name used
> across this documentation, which is still being finalized.

## Once it's released

This page will grow to cover:

- Installing via the [aescripts + aeplugins manager app](https://aescripts.com/learn/aescripts-aeplugins-manager-app/) (recommended)
- Licensing

[Back to top](#top){: .btn .float-right}

<link rel="stylesheet" href="{{ '/assets/css/general.css' | relative_url }}">
