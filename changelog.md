---
title: Changelog
layout: default
nav_order: 100
---
# Changelog

{: .note }
> This page is meant to be synced from the repo root's `CHANGELOG.md` by a
> release script (see `docs/product/AUTHORING.md`) — that automation isn't
> written yet, so this copy is hand-maintained until then. Keep it in sync
> with `CHANGELOG.md` manually in the meantime.

### 0.1 (Current — pre-release)
- [ Note ] After Effects must feed the effect **scene-linear** pixels: OCIO colour management (working space = the render's space, e.g. ACEScg), or Adobe colour management with *Linearize Working Space* **on** (any working space). With Adobe colour management and Linearize off — even at working space "None" — AE hands effects gamma-encoded pixels and converts back afterwards: every gain comes out to a power (EV +1 renders as ×5.28, or ×4.3–5.0 non-uniform with sRGB) and Bloom/Streak thresholds see encoded values, so the halos all but vanish. Measured across nine project configurations; documented in Install (with a 10-second self-test), FAQ and Matching Redshift
- [ Fixed ] Bloom and Streak bloomed far too little on real footage: the bright-pass knee had been measured on single pixels, which Redshift averages 2×2 before thresholding. The knee is now measured on flat fields and evaluated on the same 2×2 box luminance Redshift uses; a 1080p production frame now matches Redshift's own bake to 0.002 rms in the added bloom (was 0.136). Threshold semantics are unchanged in the UI, only truer: a pixel at or above Threshold blooms in full, the knee below it is what Softness widens. Verified on single pixels, squares from 2×2 to 8×8, coloured fields, impulses on grey and flat fields (the 2×2 is an even-aligned cell grid, and the plugin reproduces its half-pixel placement)
- [ Added ] Initial RS Photographic Exposure effect for After Effects — SmartFX, 32-bit float, macOS + Windows
- [ Added ] Full Photographic Exposure parameter set: Sensitivity (ISO), Aperture (f/#), Shutter Time, Whitepoint, Vignetting, Highlights, Desaturate Highlights, Blacks, Blacks Threshold, Saturation
- [ Added ] Optical / Tonemapping parameter groups mirror Redshift's RenderView Display panel exactly, including factory defaults and dynamic parameter visibility
- [ Added ] EV exposure mode (Exposure Type: EV Only / Filmic) — sweep-verified to compose identically with every other parameter
- [ Added ] Bloom (first Lens Effect): Intensity, Threshold, Softness, 5-swatch Tint
- [ Added ] Streak (second Lens Effect): Intensity, Threshold, Tail, Softness, Number, Angle
- [ Added ] LUT: load any Redshift `.cube` LUT (the shipped packs are found automatically), with Convert to Log Space and Strength — matched to Redshift's own sampling, including the half-texel lattice convention most implementations get wrong
- [ Note ] With the LUT enabled, values are clamped to Redshift's own Color Correction working range (0.18 × 2^±6.5, i.e. black lifts to ~0.002 and highlights stop at ~16.3). That is measured Redshift behaviour, reproduced on purpose so a difference blend against a render matches; the LUT is also the last stage, after tonemapping and the lens effects
- [ Note ] The LUT menu is built once per After Effects session from the Redshift LUT folder — a LUT added afterwards needs an AE restart, and `RS_POSTFX_LUT_DIR` overrides the folder. A project saves the LUT's **position in that menu, not its file path** (an After Effects dropdown can only store an index), so if the folder's contents change an old project reopens on whatever now sits at that position; "None" is always first, so an unset LUT is stable
- [ Note ] The LUT is calibrated for neutral and near-neutral images. Redshift applies a colour shift around its LUT that this plugin does not reproduce — a fraction of a percent on grey, but up to 0.318 of red out of a zero channel on strongly saturated colour
- [ Added ] Native Windows build (.aex), cross-compiled from macOS
- [ Improvements ] Multithreaded rendering and FFT-accelerated Bloom for large frames
- [ Improvements ] Collapsed parameter groups by default; inactive parameters grey out automatically
- [ Note ] Color Controls (Contrast, Curves) is removed from the v1 interface (measured, deferred): Redshift's contrast is fully measured but not yet matched to standard, and Curves' control-point data cannot be read out of a Cinema 4D scene at all
- [ Added ] Custom LUT file: **Choose .cube File…** loads any `.cube` from disk, stored with the project **by full path** (not by menu position), overriding the built-in list; **Use Built-in List** clears it. The file is validated when you choose it, and a file that later moves is shown as **(MISSING)** rather than silently skipped
- [ Note ] Custom LUT paths are absolute, so a project shared with another machine finds the file only if the same path exists there — keep shared custom LUTs on a stable path, or use the built-in list, which resolves against each machine's own Redshift install
- [ Added ] Settings group (at the bottom of the effect): shows which Redshift folder the LUT list came from, with **Locate Redshift Installation…** and **Reset to Default**. A machine-wide setting, stored per user and never carried inside a project; changing it needs an AE restart to refresh the LUT list, and the plugin says so
- [ Improvements ] The custom-LUT buttons now live in their own **Custom File** sub-group with the status row; Convert to Log Space and Strength stay outside it, since they apply to whichever LUT is active
- [ Fixed ] A custom LUT edited in place while AE is open now re-reads instead of rendering through the copy loaded earlier in the session; a file edited into something unreadable is shown as **(UNREADABLE)** rather than silently skipped
- [ Fixed ] Windows: a custom LUT whose path contains characters outside the system code page now opens correctly
- [ Note ] Flare (third Lens Effect) is removed from the v1 interface (measured, deferred). It was measured in full — six ghosts at exact magnifications about the frame centre, the radial falloff, the bright pass, the Size law, the chromatic dispersion and its position in the chain are all pinned to between two and six decimal places — but the outer edge of a ghost's disk and the Halo's brightness falloff did not reach this project's matching standard, so it ships out of the interface with its real Redshift defaults already in place
- [ Note ] Measured and worth knowing when comparing renders: focal length is **not an input** to Redshift's baked Flare — it is a pure image-space effect. Ten renders spanning f = 17.578–281.250 mm and 7.32–91.36° vertical FOV, each moving the camera without moving a pixel, came back bit-identical. Not a contradiction of Maxon's docs: two of the three behaviours they describe follow from the model without a focal-length term (a shorter lens is a wider FOV, so a highlight subtends fewer pixels and sits nearer the frame centre)
- [ Note ] Build 53: Color Controls (Contrast) and Flare, previously shipped visible-but-disabled in the interface, are now **hidden from the UI entirely, in every host** (After Effects, OFX/Resolve) — not merely greyed. Curves (RGB + per-channel R/G/B) were never buildable as a UI control at all, for the same reason given above, and remain absent. All three keep their real Redshift defaults registered internally, their underlying measurements are retained, and they're candidates for a future update once they clear this project's matching bar
- [ Added ] Diagnostic log per host session (About & Support shows the file; Open Log Folder / Get Support make a bug report a paste plus an attachment); the C4D and Houdini bridges write the same log
- [ Added ] Presets: save and load the whole PostFX look as a portable .json file (Load/Save Preset below Enable PostFX; the folder is configurable in Settings)
- [ Changed ] Copy/Paste/preset messages now go to the diagnostic log instead of a pop-up dialog (all hosts)
- [ Added ] Presets in Cinema 4D, Houdini and Resolve/Fusion — the same .json a preset saved in After Effects produces
