---
title: 🎯 Matching Redshift
layout: default
nav_order: 3
---
# Matching Redshift

{{ site.title }}'s whole reason to exist: dial in the same Photographic
Exposure numbers in After Effects that you'd dial into Redshift, on a
render that never had them baked in, and get the same pixels back.
{: .fs-5 .fw-300 }

- - -

## Defaults = a fresh Redshift camera

Every default in {{ site.title }} was measured directly off a **freshly
created Redshift Camera object** — not copied from documentation. Apply
the effect with every parameter left at its default and it is a
mathematical no-op end to end: Exposure Type EV Only at EV 0 is unit gain,
Whitepoint (1,1,1) normalizes to a no-op, Vignetting 0 is gated off,
Tonemapping defaults to **off**, and Bloom/Streak default **off**. See
[Parameters]({{site.baseurl}}/parameters) for the full table.

This matters for the A/B workflow below: starting from defaults means
you're always adding exactly the amount of Photographic Exposure you dial
in — never a hidden baseline.

## The A/B workflow

1. **Render the AOV/beauty pass without Photographic Exposure baked in** —
   either a RAW/linear EXR, or a render where the camera's Photographic
   Exposure block is at its neutral factory settings.
2. **Render the same frame with Redshift's own Photographic Exposure**
   dialed in however you like (RenderView PostFX or the camera object) —
   this is your reference/target image.
3. In After Effects, apply {{ site.title }} to the linear render from step
   1, on a 32bpc project that feeds effects **scene-linear** pixels: OCIO
   colour management with the working space the render was made in
   (ACEScg for an ACES render), or Adobe colour management with *Linearize
   Working Space* **on** (any working space) — **before** any view/OCIO
   transform, the same position Redshift applies it in its own pipeline.
   With Adobe colour management and Linearize off, After Effects feeds
   the effect gamma-encoded pixels and every gain comes out to a power
   (EV +1 → ×5.28) — see [How to Install]({{site.baseurl}}/install#finding-it-once-installed)'s
   colour-pipeline note and its 10-second self-test.
4. Dial in the **same values** you used in step 2's Redshift settings.
5. Compare against step 2's reference render. They should match to within
   the tolerances below.

## How close, really

{{ site.title }}'s math isn't guessed from Maxon's documentation (Redshift
is closed source and the docs don't publish formulas) — every component was
fit against real Redshift renders and then re-verified against fresh
sweeps. Headline numbers from the verification campaign:

| Component | Result |
|:----------|:-------|
| Core Photographic Exposure (exposure, whitepoint, vignette, highlight rolloff, black crush, saturation) | Max relative error ≈ 6×10⁻⁵ across 80+ calibration sweeps — effectively the float32 noise floor, not a fit residual |
| EV exposure mode composed with every other parameter | 7/7 sweeps MATCH (~5×10⁻⁵) |
| Tonemapping-off composed with non-neutral gain/whitepoint/vignette | 6/6 sweeps MATCH (~3×10⁻⁶) — one sweep float-exact |
| Bloom | MATCH on every impulse configuration (20/20 inside the envelope), and on real content: a 1080p production frame (eleven area lights, mirror ball, colour checker) matches Redshift's own bake to 0.002 rms in the added bloom, halo peak within 0.1 %, far field within 1 %; the bright pass is measured on flat fields and evaluated on the even-aligned 2×2 cells Redshift thresholds, verified on a ladder of squares from 2×2 to 8×8, coloured fields and impulses on grey (18/18 MATCH, docs/reports/bloom-fit-notes.md §13) |
| Streak | 17/17 MATCH on raster-axis angles (multiples of 90°), bright pass re-measured on flat fields like Bloom's; off-axis angles (including the 20° default) carry higher error concentrated near each arm's core — see the caveat on the [Parameters]({{site.baseurl}}/parameters#streak) page |

These are internal calibration-campaign results, not independently
audited third-party benchmarks — treat them as "this is how we tested it,"
not a certification.

- - -

## Known gaps (v1)

- **Why some controls aren't in v1 at all.** {{ site.title }} ships a
  control only once it matches Redshift 1:1 against real renders. Three
  controls didn't reach that bar for v1 — rather than ship them as visible
  approximations, they're kept out of the interface entirely: their
  Redshift defaults are preserved internally (so the rest of the composite
  behaves exactly as if they were present and neutral), the underlying
  measurements are kept, and all three are candidates for a future update
  once they clear the bar.
- **Flare is not in the v1 interface (hidden).** It was measured
  thoroughly — six ghosts at exact magnifications about the frame centre, a
  radial falloff verified to four digits, a bright pass identical to Streak's
  to six, exact Intensity linearity, a Size law good to 0.5 %, the chromatic
  dispersion's mechanism and coefficients, and its position in the chain
  (after the exposure gain, before the vignette, summing in parallel with
  Bloom and Streak to float precision). Two pieces did not reach this
  project's matching standard: the outer edge of a ghost's disk carries a
  ~7 % elongation along the radius that the model cannot express (it needs a
  2-D kernel this model doesn't have), and the Halo's brightness falls six
  decades across the frame in a way no closed form fits to better than 23 %.
  Rather than ship something visibly close and measurably wrong, it's
  removed from the interface for v1, with every control already carrying its
  real Redshift default so enabling it later won't disturb projects saved
  now. Full numbers: `docs/reports/flare-fit-notes.md`.
- **Focal length is not an input to Redshift's baked Flare** — it is a pure
  image-space effect on the frame buffer. Ten renders spanning
  f = 17.578 .. 281.250 mm and vertical field of view 7.32 .. 91.36°, each
  moving the camera without moving a pixel of the image, came back
  bit-identical. This does not contradict Maxon's documentation: their page
  describes what you *see* when you change the lens, and two of its three
  described behaviours follow from the measured model without any
  focal-length term — a shorter lens is a wider field of view, so a highlight
  subtends fewer pixels (its ghosts look relatively larger) and sits nearer
  the frame centre (ghost spacing shrinks with it).
- **Contrast and the RGB / per-channel R/G/B Curves are not in the v1
  interface (hidden).** Neither reached the matching bar in time for v1:
  - **Curves** can't be measured at all. Cinema 4D's Python API does not
    expose the PostFX curve data, so a scene's control points can't be read
    out of it — which rules out both measuring curves against renders (the
    method behind every other control on this page) and carrying them
    through the Copy/Paste bridge. A "close-enough" spline would break the
    measured-1:1 guarantee this project is built around, so curves stay out
    until that data is exposed.
  - **Contrast** is fully measured, just not yet matched to standard.
    Redshift applies contrast as a saturation-dependent colour transform — a
    bright primary desaturates toward gray — combined with a near-step
    response at high values, neither of which a fixed colour transform can
    reproduce. A contrast control that's wrong on saturated colour is worse
    than no contrast control at all.
- **The LUT is calibrated for neutral and near-neutral images.** Redshift's
  Color Correction stage applies a colour shift around the LUT that this
  plugin does not reproduce, and on **saturated** colour it is large: feed
  Redshift a pure green through an *identity* LUT and up to **0.318** of red
  comes back out of a channel that was zero. On grey and near-grey content
  the same shift is about 0.26 % on red and under 0.05 % on green and blue,
  which is inside the LUT stage's overall accuracy. So: a difference blend
  on a normal, mostly-neutral render will be black; a difference blend on a
  heavily saturated one will not be, and that is a known gap rather than a
  bug in your setup.
- **At large or square frames, Redshift's own baked render can silently drop
  Bloom, Streak and Flare — this is Redshift's divergence, not a plugin
  bug.** On the hardware/build this campaign measured, Redshift's own
  offline render path (the headless `RenderDocument()` mechanism this
  campaign's calibration renders use, which shares its code path with
  batch/network rendering) stops producing Bloom, Streak and Flare once a
  frame's shorter side (`min(w,h)`) reaches **1200 px**, and is
  non-deterministic right at that boundary — a 1600×1600 render came back
  with the effect present in some solo runs and silently absent in others.
  This was confirmed independently for all three effects:
  `docs/reports/bloom-fit-notes.md` §7.1, `docs/reports/streak-fit-notes.md`
  §8, `docs/reports/flare-fit-notes.md` §11. {{ site.title }} has no such
  boundary — RS PostFX renders all three deterministically at every
  resolution. So an A/B against a Redshift reference exported at 1200 px or
  larger on its shorter side can show the plugin adding lens effects over
  what looks like an empty reference frame; that's Redshift's own
  render-path divergence surfacing, not the plugin inventing anything. (The
  campaign measured this on Redshift's offline/batch render path; it has
  not independently cross-checked the interactive RenderView display at
  these sizes — the bloom and streak reports above record that cross-check
  as still open.) If you hit this, re-export the Redshift reference below
  1200 px on its shorter side for a clean A/B, or expect the mismatch at
  UHD/square frames.
- **GPU acceleration** isn't implemented yet — v1 is CPU-only.
- **Metadata auto-read** (filling Camera H-FOV etc. from the source EXR's
  `rs/camera/*` keys) is planned but not built — v1 is manual-parameter
  only.

<link rel="stylesheet" href="{{ '/assets/css/general.css' | relative_url }}">
