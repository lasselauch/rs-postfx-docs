---
title: Changelog
layout: default
nav_order: 100
---
<!-- Generated from the repo root's CHANGELOG.md by tools/sync_changelog.py -- edit that file, not this page. -->
# Changelog

## 0.1.2 · September 10, 2026 · Pre-release

Windows on ARM and Houdini 19.5 join the list, and the macOS plug-ins now say who they really are.

### ✨ New

- **Windows on ARM**: an ARM64 build of the After Effects / Premiere plug-in — not yet tested on real hardware
- **Houdini 19.5**: the bridge now ships for its standard Python 3.9 build, which also covers the Python 3.9 builds of 20.0 and 20.5

### 💎 Improvements

- **Windows installer**: picks the x64 or ARM64 build for the machine by itself; `install.bat -Arch x64` forces x64 for an emulated host

### 🐞 Fixes

- **macOS**: the plug-ins report their real identity, version and build number in Get Info and crash reports
- **macOS**: saved projects are unaffected — hosts recognise the effect by its internal name, not the bundle identifier

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
