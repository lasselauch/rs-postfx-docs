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
  **RS PostFX: Copy** and **RS PostFX: Paste** appear in the Extensions menu.

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

Premiere Pro's colour management is a separate system and has not been verified for this yet.

## Coming later

- Installing through the [aescripts + aeplugins manager app](https://aescripts.com/learn/aescripts-aeplugins-manager-app/)
- Licensing

[Back to top](#top){: .btn .float-right}

<link rel="stylesheet" href="{{ '/assets/css/general.css' | relative_url }}">
