---
title: 👋 Welcome
layout: default
nav_order: 0
---
<h3 class="index-headline"><span class="hl-primary">{{ site.title }}</span> matches <span class="hl-accent">Redshift's Photographic Exposure</span> inside AE, Premiere &amp; Resolve</h3>

**{{ site.subtitle }}** — calibrated 1:1, measured, not approximated.
{: .fs-5 .fw-300 }

- - -

[How to Install]({{site.baseurl}}/install){: .btn .btn-coral}
[Parameters]({{site.baseurl}}/parameters){: .btn .btn-coral-outline }

## What is {{ site.title }}?

{{ site.title }} is a native effect plugin for After Effects, Premiere Pro and DaVinci Resolve that replicates
**Redshift's Photographic Exposure tonemapping** — the PostFX / RenderView /
camera exposure block — so a back-to-beauty composite of Redshift AOVs
matches the render buffer.

Every parameter in this plugin is calibrated against real Redshift renders,
not derived from documentation or guesswork: Redshift is closed source, so
the underlying math was reverse-engineered and then measured against
hundreds of Redshift renders until it matched to within float32
precision. See [Matching Redshift]({{site.baseurl}}/matching-redshift) for
the verified accuracy numbers.

## What's included?

* [**Optical**]({{site.baseurl}}/parameters#optical) — Exposure Type (EV / Filmic), Sensitivity (ISO), Aperture, Shutter Time, Whitepoint, Vignetting
* [**Tone-Mapping**]({{site.baseurl}}/parameters#tone-mapping) — Highlights, Desaturate Highlights, Blacks, Blacks Threshold, Saturation
* [**Bloom**]({{site.baseurl}}/parameters#bloom) — Intensity, Threshold, Softness, 5-swatch Tint
* [**Streak**]({{site.baseurl}}/parameters#streak) — Intensity, Threshold, Tail, Softness, Number, Angle
* [**LUT**]({{site.baseurl}}/parameters#lut) — any Redshift `.cube` LUT, with Convert to Log Space and Strength
* [**Output**]({{site.baseurl}}/parameters#output) — show the composite or the lens effects alone, and mix each lens effect
* [**Presets**]({{site.baseurl}}/parameters#presets) — save and load the whole look as one portable file, shared with Cinema 4D, Houdini and Resolve
* [**Houdini and Cinema 4D bridges**]({{site.baseurl}}/houdini) — *Copy Settings* on the Redshift camera, *Paste* in After Effects: every PostFX value travels as one small block of text
* Works entirely on **scene-linear float** data, before any view/OCIO transform — exactly where Redshift applies it (your project needs a linear working space: 32 bpc + OCIO, or *Linearize Working Space* on — see [How to Install]({{site.baseurl}}/install#colour-pipeline))
* 32-bit float, multi-frame rendering ready

- - -

**Early access.** {{ site.title }} is under active development. See
[How to Install]({{site.baseurl}}/install) to get it running and the
[Changelog]({{site.baseurl}}/changelog) for what's new.

<div class="footer-info">
  <span class="connection-status">Measured, not guessed.</span>
  <p class="legal-disclaimer">
    Redshift® and Cinema 4D® are registered trademarks of Maxon Computer
    GmbH / Maxon Computer, Inc. After Effects® is a registered trademark
    of Adobe Inc. {{ site.title }} is an independent product developed by
    Lasse Lauch and is <strong>not affiliated with, endorsed by, sponsored
    by, or supported by</strong> Maxon or Adobe. All trademarks are the
    property of their respective owners.
  </p>
</div>

<link rel="stylesheet" href="{{ '/assets/css/general.css' | relative_url }}">
