---
title: Changelog
layout: default
nav_order: 100
---
<!-- Generated from the repo root's CHANGELOG.md by tools/sync_changelog.py -- edit that file, not this page. -->
# Changelog

## 0.2.0 · September 11, 2026 · Pre-release

Licensing arrives: register with your aescripts license, or try every feature with a watermark first.

### ✨ New

- **Licensing**: register with your aescripts license under About & Support › License...; one license covers After Effects, Premiere Pro and DaVinci Resolve
- **Trial**: every feature works without a license, and a red X marks the frame until you register
- **32 bpc warning**: After Effects flags an 8 or 16 bpc project, where the effect passes frames through untouched, with a one-click Use 32 bpc button

### 💎 Improvements

- **Windows**: built with Microsoft's own C++ runtime, the one After Effects, Premiere Pro and Resolve already install
- **About & Support**: Product Page and the info button in After Effects' Effects & Presets panel open the aescripts product page; Get Support opens aescripts' contact page

### 🐞 Fixes

- **Vignette**: stays centred on layers larger or smaller than their comp instead of drifting off-centre

### 📌 Good to know

- **Windows on ARM**: not in this release — the licensing framework has no ARM64 build yet
- **After registering**: purge the cache so frames rendered during the trial lose the X

## 0.1.2 · September 10, 2026 · Pre-release

Windows on ARM and Houdini 19.5 join the list, and the macOS plug-ins now say who they really are.

### ✨ New

- **Windows on ARM**: an ARM64 build of the After Effects / Premiere plug-in — not yet tested on real hardware
- **Houdini 19.5**: the bridge now ships for its standard Python 3.9 build, which also covers the Python 3.9 builds of 20.0 and 20.5

### 💎 Improvements

- **Windows installer**: picks the x64 or ARM64 build for the machine by itself; `install.bat -Arch x64` forces x64 for an emulated host
- **Cinema 4D**: the four commands now share one submenu in the Extensions menu, with short names and one-line descriptions

### 🐞 Fixes

- **macOS**: the plug-ins report their real identity, version and build number in Get Info and crash reports
- **macOS**: saved projects are unaffected — hosts recognise the effect by its internal name, not the bundle identifier
- **Windows**: user names, presets and LUT folders with non-English characters (Jürgen, 山田, Grün.cube) now work for presets, LUTs, settings and logs
- **macOS**: a LUT chosen from a folder with accented characters is recognised as part of your Redshift list, so projects stay portable
- **Houdini installer**: finds Houdini on Windows even when Documents is localized or redirected, such as `OneDrive\Dokumente`
- **Houdini installer**: runs on PCs without Python, using Houdini's own hython instead of the Microsoft Store placeholder
- **Presets**: files saved as "UTF-8 with BOM" load in Cinema 4D and Houdini too; a file that isn't UTF-8 gets a clear message
- **Cinema 4D**: the four commands use their own registered Maxon plugin IDs, so they can't collide with another plugin's

## 0.1.1 · September 9, 2026 · First release

Redshift's Photographic Exposure, Bloom, Streak and LUT for After Effects, Premiere Pro and DaVinci Resolve —
measured against Redshift's own renders — plus Copy/Paste bridges for Cinema 4D and Houdini.

### ✨ New

- **Photographic Exposure**: ISO, f-stop, shutter, whitepoint, vignetting, highlights, desaturate highlights, blacks, blacks threshold and saturation
- **Exposure modes**: EV Only and Filmic next to the physical camera controls
- **Panel**: Optical and Tone-Mapping groups mirror Redshift's RenderView panel, factory defaults included
- **Bloom**: Intensity, Threshold, Softness and a 5-swatch Tint
- **Streak**: Intensity, Threshold, Tail, Softness, Number and Angle
- **LUT**: any Redshift `.cube`, with Convert to Log Space and Strength, sampled exactly the way Redshift samples it
- **Custom LUT row**: a menu built live from your Redshift LUT folder, *Choose File…* for any `.cube`, and a spinner to scrub through LUTs
- **Resolve**: a LUT File dropdown plus a file-path field, since OpenFX can't draw the Custom LUT row
- **Settings**: see which Redshift folder the LUTs come from — *Locate Redshift Installation…* and *Reset to Default*
- **Presets**: save and load a whole look as a `.json` — the same file in After Effects, Resolve, Cinema 4D and Houdini
- **Bridges**: copy a Redshift camera's PostFX in Cinema 4D or Houdini and paste it onto the effect — and back
- **Diagnostic log**: one per host session; *About & Support* opens it, *Get Support* turns it into a bug report
- **Windows**: native `.aex` build

### 💎 Improvements

- **Performance**: multithreaded rendering and FFT-accelerated Bloom for large frames
- **Panel**: groups start collapsed, and parameters that don't apply grey out
- **Bridges**: a built-in Redshift LUT copied from Cinema 4D or Houdini is recognised from any Redshift install location
- **Messages**: Copy, Paste and preset messages go to the diagnostic log instead of pop-up dialogs

### 🐞 Fixes

- **Bloom & Streak**: bloom as strongly as Redshift on real footage — a 1080p frame now matches Redshift's bake to 0.002 rms (was 0.136)
- **Custom LUT**: a `.cube` edited while After Effects is open is re-read; one that no longer parses shows **(UNREADABLE)**
- **Windows**: LUT paths with characters outside the system code page open correctly

### 📌 Good to know

- **Linear input**: the effect needs scene-linear pixels — OCIO, or Adobe colour management with *Linearize Working Space*. *How to Install* has a 10-second self-test
- **LUT range**: with the LUT on, values clamp to Redshift's own working range (about 0.002 to 16.3), exactly as Redshift does — see *FAQ*
- **LUT colour**: calibrated for neutral and near-neutral images; strongly saturated colour can differ — see *Matching Redshift*
- **Shared projects**: a LUT chosen from outside the Redshift folder needs the same path on every machine — see *Parameters → LUT*
- **Not in this version**: Flare and Color Controls (Contrast, Curves) — measured, but not yet a 1:1 match with Redshift — see *FAQ*
