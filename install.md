---
title: 📦 How to Install
layout: default
nav_order: 1
---
# How to install

{: .important }
> **Early access.** {{ site.title }} is not on the aescripts + aeplugins manager yet. Every host has a one-click
> installer inside the download; run the ones for the hosts you use and restart those hosts.

## After Effects & Premiere Pro

Premiere Pro loads the **identical plugin** built for After Effects — there is no separate Premiere build. Both
hosts load it from the same **shared** plug-ins folder:

```
macOS:   /Library/Application Support/Adobe/Common/Plug-ins/7.0/MediaCore/
Windows: C:\Program Files\Adobe\Common\Plug-ins\7.0\MediaCore\
```

This "Common MediaCore" folder is the one location **both** After Effects and Premiere Pro scan. After Effects'
own private `Plug-ins` folder would work for After Effects alone; Premiere never looks there.

### One-click installer (recommended)

1. Quit After Effects and Premiere Pro.
2. Open the **After Effects & Premiere** folder of the download and double-click the installer for your OS:
   - **macOS**: `install.command`
   - **Windows**: `install.bat`

   It asks for your administrator password once (the MediaCore folder is owned by the system) and copies the
   plugin in, replacing any previous version and removing any stray copy in After Effects' own `Plug-ins` folder so
   the effect is never loaded twice.
3. Start After Effects or Premiere Pro. The effect appears as **Effect → RS PostFX → RS PostFX**.

{: .note }
> **macOS:** if the installer refuses to open because it comes from an unidentified developer, right-click it and
> choose **Open** once.

### Manual install

Copy `RSPostFX.plugin` (macOS) or `RSPostFX.aex` (Windows) into the MediaCore folder above yourself, then restart
the host. On macOS a file downloaded from the web carries a quarantine flag that stops After Effects from
loading it, which is why the installer is the recommended route.

{: .note }
> **Premiere Pro** has no collapsible parameter groups — every group in the Effect Controls panel shows
> force-collapsed. That is how Premiere draws every effect, not something the plugin can change.

## DaVinci Resolve (OFX)

{{ site.title }} also ships as an OpenFX plug-in built from the same rendering engine. Open the **OpenFX (Resolve)**
folder of the download, quit Resolve, and double-click **`install.command`** (macOS) or **`install.bat`**
(Windows). The installer asks for administrator rights and copies `RS PostFX.ofx.bundle` into the shared OFX
plug-ins folder, replacing any previous copy:

```
macOS:   /Library/OFX/Plugins/
Windows: C:\Program Files\Common Files\OFX\Plugins\
```

Restart Resolve. The effect appears under **OpenFX → RS PostFX → RS PostFX** in the Color page's OpenFX list.

In Resolve, bracket the node with **Color Space Transform** nodes — into scene-linear ahead of it, back out of
scene-linear after it — so the plugin sees the same linear data it does in After Effects (see
[Colour pipeline](#colour-pipeline) below).

## Houdini and Cinema 4D (the bridge)

The Houdini and Cinema 4D parts of the download are not effects — they are the **bridge**: shelf tools (Houdini)
and Extensions-menu commands (Cinema 4D) that copy a Redshift camera's PostFX settings so the plugin above can
paste them, and that save and load the same presets.

- **Houdini:** double-click the installer in the **Houdini/install** folder. No administrator rights are needed
  and one install covers every Houdini version on the machine. The full render-to-composite workflow is on
  [Houdini to After Effects]({{site.baseurl}}/houdini).
- **Cinema 4D:** copy the whole `rspostfx-c4d` folder into a `plugins` folder Cinema 4D scans
  (for example `~/Library/Preferences/Maxon/<your Cinema 4D version>/plugins/` on macOS) and restart Cinema 4D.
  **Copy**, **Paste**, **Load Preset** and **Save Preset** appear under **Extensions › RS PostFX**.

## Colour pipeline

{{ site.title }} works on **scene-linear** pixel values in every host — the same place Redshift applies its own
PostFX — and has no colour-space transform of its own. In After Effects, one project setting decides whether the
effect receives linear pixels (Project Settings › Color):

* **OCIO colour management (recommended).** Choose the OCIO system and a config (the shipped *ACES 1.3 CG* is
  fine). Set the **working colour space to the space your EXR was rendered in** — ACEScg for a Redshift ACES
  render, *Linear Rec.709 (sRGB)* for a Redshift render in linear sRGB — and check that the footage is
  interpreted as that same space (the file rules usually get an EXR right; Interpret Footage › Color shows it).
  Matching the working space to the render space matters for *colour*: Bloom and Streak thresholds are computed
  from luminance in the working space, as Redshift computes them in its render space, so a different working
  space shifts the threshold slightly on saturated sources. Nothing else to set.
* **Adobe colour management (classic ICC).** Any working space is fine — sRGB, Rec.709, ACEScg — **as long as
  *Linearize Working Space* is ON.** With it off (the default), After Effects hands every effect gamma-encoded
  pixels and converts back afterwards, even when the working space is "None": every exposure gain comes out
  raised to a power (EV +1 renders as about ×5 instead of ×2) and the Bloom/Streak thresholds are compared against
  encoded values, so the halos all but vanish. Measured across nine project configurations.

**Self-test (10 seconds):** on a linear plate, apply {{ site.title }} with Exposure Type *EV Only*, Exposure (EV)
**+1** and everything else off. The result must be exactly **twice** the plate everywhere (read a pixel in the
Info panel). If it is around 5×, or varies across the frame, the project is feeding the effect encoded pixels.

**Premiere Pro** is different: in its default Rec. 709 working space it hands effects the scene-linear render raised
to the power 1/2.4, and none of its working spaces is linear. Until the plugin handles that itself, results in
Premiere Pro do not match Redshift — see [Matching Redshift's known gaps]({{site.baseurl}}/matching-redshift#known-gaps).

## GPU acceleration

{{ site.title }} renders on the graphics card by default in After Effects (both platforms) and in DaVinci Resolve
on Windows with NVIDIA graphics. There is nothing to switch on:

| Host | macOS | Windows |
|:-----|:------|:--------|
| After Effects | Metal | NVIDIA (CUDA) |
| DaVinci Resolve | CPU in this beta | NVIDIA (CUDA) |
| Premiere Pro | CPU in this beta | CPU in this beta |

- **Same pixels.** A frame rendered on the graphics card matches the CPU render to within about 5×10⁻⁶ of its peak
  value — float precision, black in a Difference blend. A difference you can see is a bug: please report it.
- **Whichever is faster, in After Effects.** On every Mac — Apple silicon and Intel with AMD graphics alike — After
  Effects renders Bloom and Bloom+Streak frames on the CPU, and Streak frames below about 5.5 megapixels on the CPU,
  because the CPU is faster for them there; everything else renders on the GPU. On an NVIDIA card, After Effects
  renders everything on the GPU. Resolve renders every frame on the GPU — it does not route individual frames yet.
- **Premiere Pro and Resolve on macOS render on the CPU in this beta.** Premiere Pro's GPU path has not yet been
  checked pixel-for-pixel in a running Premiere Pro, and on Apple silicon Resolve's GPU renders Bloom and
  Bloom+Streak frames up to 2.8× slower than the CPU.
- **A failed GPU frame still renders.** If the graphics card cannot finish a frame, for example because it ran out
  of memory, After Effects renders that frame on the CPU itself, and Resolve re-renders it on its own CPU path. If
  the card fails for good, After Effects renders the rest of that session on the CPU.
- **AMD and Intel graphics on Windows** render on the CPU, with the same results.
- **The host has to offer the GPU.** After Effects does so only with *File › Project Settings › Video Rendering and
  Effects* set to *Mercury GPU Acceleration*. With *Mercury Software Only* the plugin renders on the CPU.
- **The first GPU frame after launching a host** can take about a second longer, while its GPU kernels compile
  once; every frame after that runs at full speed.
- **After Effects' disk cache** can hand back a frame rendered before a change: purge it (*Edit › Purge › All
  Memory & Disk Cache*) after installing a new build and after switching between GPU and software rendering.
- **Windows: a keyframed Sensitivity (ISO)** makes After Effects stop offering the GPU to the effect, so those
  frames render on the CPU instead — the pixels are unaffected, only the speed. The cause isn't known yet.

### Turning it off

Use the host's own renderer setting. Rendering the same frame both ways is the quickest way to tell whether a
problem comes from the GPU path.

- **After Effects:** *File › Project Settings › Video Rendering and Effects › Use:* **Mercury Software Only**. The
  effect then renders on the CPU. It is a project setting, so it also moves every other GPU effect in that project
  to the CPU, and `aerender` follows it, so it covers render farms too. Choose *Mercury GPU Acceleration* again to
  turn the GPU back on.
- **Premiere Pro:** *File › Project Settings › General › Renderer:* **Mercury Playback Engine Software Only**. In
  this beta the effect renders on the CPU in Premiere Pro either way.
- **DaVinci Resolve** has no such switch.

**Purge After Effects' disk cache before you compare** (*Edit › Purge › All Memory & Disk Cache*), then render the
same frame with each setting. A Difference blend between the two should be black.

### Which path rendered a frame

The [diagnostic log]({{site.baseurl}}/faq#where-are-the-log-files-and-what-should-i-attach-to-a-bug-report) —
`~/Library/Application Support/RS-PostFX/logs/` on macOS, `%APPDATA%\RS-PostFX\logs\` on Windows — records the
graphics card as soon as the plugin sets it up, for example:

```
GPU device: metal 'Apple M2' 16384 MB unified, kernels ready in 1.3 ms
```

A card the plugin does not use gets a `GPU not used: … -- rendering on the CPU` line with the reason instead. A GPU
frame that fails and is rendered on the CPU is logged right away, and so is a card that fails for the rest of the
session. A summary counts the frames rendered on the GPU and on the CPU, with a `routed to cpu` line giving the
reasons; it is written every few minutes and when the host quits, so quit the host before you send the log.

## Licensing and trial

- **Trial:** without a license every feature works, and a red X marks the frame. There is no time limit.
- **Register:** open **About & Support** and click **License...** (After Effects also has a **Register** link at the top of the effect). Paste your aescripts license code and click **Activate**. The aescripts + aeplugins manager app will be able to install a trial or your license for you too, once the product is available in it.
- **One license, every host:** the same license unlocks After Effects, Premiere Pro and DaVinci Resolve on that machine.
- **After registering:** frames rendered during the trial keep their X in the cache — purge it (After Effects: *Edit › Purge › All Memory & Disk Cache*; Premiere Pro: delete render files; Resolve: the frame refreshes on the next change).
- **Moving to another machine:** click **License...** → **Deactivate** first, so the activation is freed.
- **Floating licenses:** with the aescripts Floating License Server configured, enter `@REMOTE` as the license code.
- **Render-only licenses** work in the command-line renderer (`aerender`) only; in the interface they show the trial X.
- **Windows on ARM** is not available yet: the licensing framework has no ARM64 build.

## Coming later

- Installing through the [aescripts + aeplugins manager app](https://aescripts.com/learn/aescripts-aeplugins-manager-app/)

[Back to top](#top){: .btn .float-right}

<link rel="stylesheet" href="{{ '/assets/css/general.css' | relative_url }}">
