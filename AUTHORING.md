# Authoring these docs

This mirrors the workflow used for the user's other aescripts.com products
(AEC4D-PRO, C4D2HOU): a Jekyll + [just-the-docs](https://just-the-docs.com)
site living at `docs/product/` in this (private) source repo, auto-synced
to a separate **public** docs repo on push, which is what GitHub Pages
actually serves.

## How the real workflow works (once wired up)

1. Edit pages under `docs/product/*.md` and the root `CHANGELOG.md` in this
   repo, on a normal feature branch.
2. Before cutting a release, run a release script (not written yet for this
   repo — see `c4d-aec4dpro/__create_release.py` / `c4d2hou/__create_release.py`
   for the pattern to copy) so it can:
   - sync `VERSION.txt` into the plugin source (`RSPostFXVersion.h`),
   - sync `CHANGELOG.md` → `docs/product/changelog.md` — **done, as its own
     step:** `python3 tools/sync_changelog.py` writes the front matter plus the
     changelog's body verbatim, and `tests/test_changelog.py` fails when the two
     drift,
   - sync `CHANGELOG.md`'s content into `README.md`'s changelog section, if
     that README grows a mirrored one the way AEC4D-PRO's does.
3. Push to `main`. `.github/workflows/sync-docs.yml` (repo root) notices
   `docs/product/**` changed, builds the Jekyll site as a link-check gate,
   then mirrors `docs/product/`'s content verbatim into a separate public
   repo (`DOCS_REPO` secret, pushed over SSH with `DOCS_DEPLOY_KEY`). That
   public repo's GitHub Pages is set to *deploy from branch* `main`, so
   GitHub's own Jekyll builder renders what was pushed —
   `docs/product/.github/workflows/pages.yml` is unused (and, being a
   dotfile, never synced).

**Wired up 2026-09-07:** the public repo `lasselauch/rs-postfx-docs`, its write
deploy key, the `DOCS_REPO` / `DOCS_DEPLOY_KEY` secrets and GitHub Pages (deploy
from branch `main`, root — the c4d2hou-docs pattern) all exist; every push of
`main` that touches `docs/product/**` now publishes. Runbook, verification
commands and troubleshooting: `docs/reports/docs-site-go-live.md`. Still not
written: the README sync step (the changelog page is generated, see above).

## Changelog style

Modeled on Raycast's (raycast.com/changelog): short, scannable, what changed for the user.
`tests/test_changelog.py` enforces the mechanical parts.

- One `## X.Y.Z · Month D, YYYY` heading per release, optionally `· Pre-release` or `· First release`.
  Newest first; the newest is always `VERSION.txt`'s version.
- One or two plain sentences under it: what this release is about.
- Then only these sections, in this order, each optional: `### ✨ New`, `### 💎 Improvements`,
  `### 🐞 Fixes`, `### 📌 Good to know` (caveats a user must know, not trivia).
- Every bullet is `- **Area**: fragment` — a short lead-in (Bloom, LUT, Houdini, Windows installer…), then
  what changed for the user, **30 words at most**. No measurement essays, build numbers or internal file
  names: the evidence lives in `docs/reports/`, the details on a docs page — point to it by page title
  (*FAQ*, *Matching Redshift*), never by URL, because the URL carries the product name.
- Describe what a release ships, not how it got there: no task ids, commit hashes or intermediate steps that
  never reached a user. Credit a bug reporter by name when it helps ("Thanks, …!").
- Never name the product (see Naming below).
- After editing, run `python3 tools/sync_changelog.py`.

## Naming — research-settled, but still rename-friendly

**Current name: "RS PostFX"** (sidebar shows the wordmark only — no tagline
subline), positioning subtitle **"Redshift-matched PostFX for AE, Premiere &
Resolve"** (used wherever a
subtitle fits — see `_config.yml`'s `subtitle`). This is the outcome of a
dedicated trademark/positioning research pass, not a placeholder guess —
see `docs/research/product-positioning.md` section 3 for the full
evidence, risk ladder and decision. Naming history, for context:

1. **"RSPostFX"** — earliest working name.
2. **"Redshift PostFX"** — working name during early docs scaffolding.
3. **"RS PostFX"** — current, research-settled (2026-08-30). "Redshift
   PostFX" was rejected as the highest-risk option checked: it leads with
   a registered Maxon trademark (`REDSHIFT`, US Reg. 6047288) *and*
   reproduces Maxon's own "PostFX" feature name verbatim, has zero
   aescripts catalog precedent, and would permanently collide with
   Maxon's own docs page in search. "RS PostFX" has none of that (an
   abbreviation, not the mark), direct aescripts catalog precedent
   (`C4D2HOU`, `AEC4D PRO`, `XML2AE`), and costs nothing to ship since its
   logo already exists.

The name could still move again in principle (e.g. if aescripts declines
it at listing time — see the research's §3.2 recommendation to ask them in
writing before launch), so the rename mechanism below stays worth keeping
up to date rather than hardcoding the name everywhere.

**Descriptive/referential use of "Redshift" in body prose is fine and
expected** ("replicates Redshift's Photographic Exposure tonemapping",
"matching Redshift", etc.) — what the research flags is specifically the
product's own head wordmark, not mentioning Redshift at all. Don't scrub
descriptive mentions; just don't make them the product name.

**Non-affiliation notice:** a short disclaimer (Redshift/Cinema 4D are
Maxon trademarks, After Effects is an Adobe trademark, RS PostFX is
independent and not affiliated with/endorsed by either) lives in
`index.md`'s footer and in the root `README.md`, per the research's §3.2
recommendation to place it in a handful of visible spots. The exact
wording is theirs, adapted — see that section before changing it.

### Renaming it (in one edit, where Liquid can reach)

- `docs/product/_config.yml` → `title:`, `tagline:` and `subtitle:` (plus
  `description:`). `{{ site.title }}` / `{{ site.tagline }}` /
  `{{ site.subtitle }}` are used everywhere the name appears in a Liquid
  context: the sidebar logo + tagline (`_includes/title.html`),
  `index.md`'s headline, and any other page body that uses those
  variables instead of a literal string.

### Everything else — a manual pass

SCSS variables, CSS class names, filenames and plain prose can't read
`_config.yml`. Rename-point list:

| File | What to change |
|:-----|:----------------|
| `docs/product/_config.yml` | `title`, `tagline`, `subtitle`, `description`, `baseurl` (currently `/rs-postfx-docs` — matches the eventual public repo name), `aux_links` comment |
| `docs/product/_sass/color_schemes/foo.scss` | the three brand hex values (coral / soft indigo / dark background) and the `$rspfx-*` variable prefix (cosmetic only — safe to leave, but matches the other products' convention of a branded prefix) |
| `docs/product/_sass/custom/custom.scss` | comment header only — class names (`.product-logo`, `.product-tagline`, `.hl-primary`, `.hl-accent`) are deliberately generic and never need to change on a rename |
| `docs/product/README.md` | "RS PostFX" / "for AE, Premiere & Resolve" in the heading and body |
| `README.md` (repo root) | "RS PostFX" / "for AE, Premiere & Resolve" in the heading and body, and the non-affiliation notice's product name |
| `docs/product/install.md`, `faq.md` | the literal name/tagline mentions outside `{{ site.title }}` |
| `docs/product/index.md` | the non-affiliation notice's product name (currently `{{ site.title }}`, already rename-safe) |
| `CHANGELOG.md` / `docs/product/changelog.md` | nothing — the changelog deliberately never names the product, since it's the thing most likely to still change |
| `.github/workflows/sync-docs.yml` (repo root) | comment references to the product name / `rs-postfx-docs`; the `DOCS_REPO` **value** lives in a GitHub secret, not this file |
| `docs/product/.github/workflows/*.yml` | nothing — these are generic |
| Public repo name itself | `lasselauch/rs-postfx-docs` exists since 2026-09-07 and is live; a rename means renaming the GitHub repo, then updating `baseurl`, the `DOCS_REPO` secret and `engine/RSPAboutInfo.h`'s three URLs to match |
| Logo / favicon assets | not committed yet — see `docs/product/assets/img/PLACEHOLDERS.txt` for the expected filenames (a finished logo exists for "RS PostFX": hexagonal red mark + wordmark + tagline, navy/red/white on dark) |

Everything under `plugin/`, `reference/`, `verify/`, `tests/`, `calibration/`
is out of scope for a docs rename — see those areas' own naming (e.g. the
`RSP_*` param prefixes and the compiled-in AE effect name "RS Photographic
Exposure" in `plugin/src/RSPostFX.h`) if a rename ever needs
to go that deep. That compiled effect name is a separate, already-fixed
technical fact from the marketing product name and doesn't need to move in
lockstep with it — though note the research (§3.5) separately recommends
freezing a brand-neutral **match name** (`AE_Effect_Match_Name`, distinct
from the display name) before any external build ships, so that a future
display-name change never orphans saved projects. That's a plugin-side
change, out of scope for this docs pass.

## Page front matter

Every page needs:

```yaml
---
title: <emoji> <Title>
layout: default
nav_order: <n>
---
```

`nav_order` convention: `0` welcome, `1` install, `2..9` core guides,
`100` changelog, `101` FAQ.

## Images

Reference them as `{{site.baseurl}}/assets/img/<file>` from `docs/product/`
pages. Nothing has been captured yet —
`docs/product/assets/img/PLACEHOLDERS.txt` lists what each stub page
expects; drop the real file in and the `![]()` reference already in the
page will pick it up.

## Deliberate deviations from the reference repos' pattern

- Reference repos hardcode branded CSS class names (`.aec4d-logo`,
  `.hl-c4d`). This site uses generic ones (`.product-logo`,
  `.product-tagline`, `.hl-primary`, `.hl-accent`) instead — the name
  survived one rename already and could in principle move again (see
  Naming above), so the class names stay decoupled from it regardless.
- Reference repos' page footers read "Built with 💙🧡 in Hamburg, Germany" /
  "Built with 💜💙 in Lüneburg, Germany" (the author's location). This site's
  footer says "Measured, not guessed." instead — a location flourish wasn't
  specified for this product, so one wasn't invented.
- `Gemfile.lock` is committed (matches the reference repos), generated by
  running `bundle install` from `docs/product/`; `bundle exec jekyll build`
  was also run once here as a smoke test — it built cleanly (the only
  warnings are pre-existing `darken()` deprecation notices from
  just-the-docs' own `buttons.scss`, not from this site's files). Re-run
  `bundle install` and re-commit the lockfile after bumping any gem
  version in the `Gemfile`.
