---
title: 🎯 Matching Redshift
layout: default
nav_order: 3
---
# Matching Redshift

{{ site.title }}'s whole reason to exist: dial in the same Photographic Exposure numbers in After Effects that
you'd dial into Redshift, on a render that never had them baked in, and get the same pixels back.
{: .fs-5 .fw-300 }

- - -

## Defaults = a fresh Redshift camera

Every default in {{ site.title }} was measured directly off a **freshly created Redshift camera** — not copied from
documentation. Apply the effect with every parameter left at its default and it changes nothing at all: Exposure
Type EV Only at EV 0 is unit gain, Whitepoint (1,1,1) is neutral, Vignetting 0 is off, Tone-Mapping defaults to
**off**, and Bloom and Streak default **off**. See [Parameters]({{site.baseurl}}/parameters) for the full table.

This matters for the A/B workflow below: starting from defaults means you are always adding exactly the amount of
Photographic Exposure you dial in — never a hidden baseline.

## The A/B workflow

1. **Render the beauty pass without PostFX baked in** — a linear EXR written with the camera's PostFX left out of
   the file output. Houdini users: the exact switches are on [Houdini to After Effects]({{site.baseurl}}/houdini).
2. **Render the same frame with Redshift's own PostFX** dialed in however you like — this is your reference image.
   Keep it below 1200 px on its shorter side (see [Known gaps](#known-gaps)).
3. In After Effects, apply {{ site.title }} to the clean render from step 1, in a 32 bpc project that feeds effects
   **scene-linear** pixels — see [Colour pipeline]({{site.baseurl}}/install#colour-pipeline) and its ten-second
   self-test. That is the same position Redshift applies PostFX in its own pipeline, before any view transform.
4. Dial in the **same values** you used in step 2 — or paste them with the [bridge]({{site.baseurl}}/houdini).
5. Compare against the reference with a **Difference** blend. It should be black to within the tolerances below.

## How close, really

{{ site.title }}'s math is not guessed from Redshift's documentation, which describes what each control does but
never publishes a formula. Every component was fit against real Redshift renders and then re-verified against
fresh renders it had never seen:

| Component | Result |
|:----------|:-------|
| Core Photographic Exposure (exposure, whitepoint, vignette, highlight rolloff, black crush, saturation) | Maximum relative error about 6×10⁻⁵ across more than 80 verification renders — the float32 noise floor, not a fit residual |
| EV exposure mode combined with every other parameter | Matches to about 5×10⁻⁵ |
| Tone-Mapping off, combined with non-neutral gain, whitepoint and vignette | Matches to about 3×10⁻⁶ |
| Bloom | Matches on every single-pixel, patch and coloured-field configuration tested, and on real content: a 1080p production frame (eleven area lights, a mirror ball, a colour checker) matches Redshift's own bake to 0.002 rms in the added bloom, with the halo peak within 0.1 % and the far field within 1 % |
| Streak | Matches on horizontal and vertical arms (angles that are multiples of 90°); at other angles, including the 20° default, a fine texture near each arm's core differs while arm directions, profile and total energy match — see the note on the [Parameters]({{site.baseurl}}/parameters#streak) page |
| LUT | Matches Redshift's own sampling, including the half-texel lattice convention most implementations get wrong, on neutral and near-neutral content |

These are our own verification results, not an independent audit — read them as "this is how it was tested",
not as a certification.

- - -

## Known gaps

- **Flare, Contrast and Curves are not in the interface yet.** {{ site.title }} ships a control only once it matches
  Redshift 1:1 against real renders; these three have not cleared that bar. Their Redshift defaults are kept
  internally so the rest of the composite behaves exactly as if they were present and neutral. Why each one is
  held back is on the [Parameters]({{site.baseurl}}/parameters#not-available-yet) page.
- **Redshift's Flare does not depend on focal length** — it is a pure image-space effect. Renders spanning
  17.6 mm to 281 mm, each moving the camera without moving a pixel of the image, came back identical. A shorter
  lens looks different only because it is a wider field of view: a highlight covers fewer pixels and sits nearer
  the frame centre.
- **The LUT is calibrated for neutral and near-neutral images.** Redshift applies a colour shift around its LUT
  that this plugin does not reproduce, and on **saturated** colour it is large: a pure green through an *identity*
  LUT comes back with up to **0.318** of red in a channel that was zero. On grey and near-grey content the same
  shift is about 0.26 % on red and under 0.05 % on green and blue, inside the LUT stage's overall accuracy. So a
  difference blend on a normal, mostly neutral render will be black; on a heavily saturated one it will not, and
  that is a known gap rather than a problem with your setup.
- **Redshift's "Apply color management before LUT" is not reproduced.** With that checkbox ticked, Redshift applies
  the LUT to the image as converted for your display, then converts it back — a step that belongs to your project's
  colour management, not to this effect. Leave it off in Redshift for a clean comparison; the
  [FAQ]({{site.baseurl}}/faq#apply-color-management-before-lut) explains why.
- **At large or square frames, Redshift's own file output can silently drop Bloom, Streak and Flare.** In our
  measurements, Redshift's offline render path stops producing all three lens effects once a frame's shorter
  side reaches **1200 px**, and behaves unpredictably right at that boundary — a 1600×1600 render came back with
  the effect present in some runs and absent in others. {{ site.title }} has no such boundary and renders all three
  at every resolution. So an A/B against a Redshift reference exported at 1200 px or larger on its shorter side
  can show the plugin adding lens effects over an empty-looking reference; that is Redshift's own behaviour, not
  the plugin inventing anything. Export the reference below 1200 px on its shorter side for a clean comparison.
- **GPU acceleration** is not implemented yet — rendering is CPU-only, multithreaded, with FFT-accelerated bloom
  on large frames.
- **Reading camera metadata from the EXR** (to fill Camera H-FOV automatically) is planned but not built. Until
  then, type it in or paste it from the [bridge]({{site.baseurl}}/houdini).

[Back to top](#top){: .btn .float-right}

<link rel="stylesheet" href="{{ '/assets/css/general.css' | relative_url }}">
