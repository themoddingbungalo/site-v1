# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Jekyll site (the wiki for The Modding Bungalo Discord) using the gem-based [just-the-docs] theme, deployed to GitHub Pages at the custom domain `themoddingbungalo.com`. Content is Markdown; there is no application code and no test suite.

`README.md` describes this site for contributors arriving via GitHub. It overlaps with this file by design — if you change how the nav, theming, or deploy works, update both.

## Commands

```bash
bundle install                # first-time setup / after Gemfile changes
bundle exec jekyll serve      # local preview at http://localhost:4000 with rebuild-on-save
bundle exec jekyll build      # one-shot build into _site/ (what CI runs)
```

CI (`.github/workflows/ci.yml`) only runs `bundle exec jekyll build` on push/PR — a green check means the site compiled, nothing more. Pushing to `main` also triggers `.github/workflows/pages.yml`, which builds with `JEKYLL_ENV=production` and deploys to GitHub Pages.

## Site structure

Three Jekyll collections, each declared twice in `_config.yml` — once under `collections:` (permalink + output) and once under `just_the_docs.collections:` (nav display name + fold behavior). Adding a collection requires both entries.

| Directory | Nav section | Contents |
| --- | --- | --- |
| `_lists/` | Lists | Per-modlist docs (Lorerim, CSVP, Ghoulified, DNGG, SpeedTweeks, LoreOut) — Skyrim lists plus LoreOut for Fallout 4 |
| `_ngvo/` | NGVO | The NGVO visual baseline list |
| `_modding_guides/` | Modding Guides | Wabbajack, xEdit, LOD generation, Creation Kit guides |

Standalone pages at the repo root (`index.md`, `contribute.md`, `biggie-boss.md`, `themoddingbordello.md`) form the top-level nav alongside the collections.

## Navigation is front matter, not directories

just-the-docs builds the sidebar from front matter, and it links children to parents **by matching the parent's `title` string** — the on-disk folder layout is only for human organization. A page nested two levels deep needs both keys:

```yaml
---
layout: default
title: Colloquy's Guide
parent: Guides          # must exactly match the parent page's `title:`
grand_parent: CSVP      # must exactly match the grandparent's `title:`
nav_order: 1
---
```

Consequences worth remembering:

- Renaming a page's `title` silently orphans every child that referenced it as `parent`/`grand_parent`. Update the children in the same change.
- A parent page must set `has_children: true` (see `_lists/csvp/index.md`) or its children won't render in the nav.
- `nav_order` is scoped within its nav section, so numbers restart per collection and per parent.
- Only two levels of nesting are supported (`parent` + `grand_parent`); there is no `great_grand_parent`.

## `baseurl` must stay empty

`_config.yml` sets `baseurl: ""` and `url: https://themoddingbungalo.com`, and `pages.yml` deliberately does **not** pass `--baseurl` or use `actions/configure-pages`. The site is served from the root of the custom domain (`CNAME`); a derived baseurl bakes in `/site-v1` and 404s every asset (commit 519995c). Reference assets as `{{ site.baseurl }}/assets/...` so links keep working either way.

`aux_links` in `_config.yml` is the Discord invite shown in the header; `url` is the site's own canonical base — don't conflate them.

## Theme customization

The theme is a gem, so there is no theme source in-repo. Everything custom lives in four places:

- `_sass/custom/setup.scss` — imported **before** any color scheme, for every stylesheet the theme builds (default/light/dark). Shared values that both the color scheme and `custom.scss` need go here; currently `$bungalo-red`.
- `_sass/color_schemes/bungalo-dark.scss` — activated by `color_scheme: bungalo-dark` in `_config.yml`. It imports the theme's `dark` scheme, overrides its SCSS color variables, and also carries the site's custom CSS classes: `.youtube-container` (responsive 16:9 iframe wrapper), `.important` (amber callout), `.warning` (red callout).
- `_sass/custom/custom.scss` — imported **last**, after the theme's own modules.
- `_includes/nav_footer_custom.html` — overrides the theme include of the same name for the sidebar footer.

Import order is the thing to get right (the sequence is in the gem's `_includes/css/just-the-docs.scss.liquid`):

```
support → custom/setup → color_schemes/light → color_schemes/bungalo-dark → modules → callouts → custom/custom
```

Because the color scheme is imported *before* the theme's modules, a plain CSS rule there loses to any theme rule of equal specificity. Setting a property the theme already sets — e.g. `font-weight` on `.nav-category`, which `navigation.scss` sets to 600 — must go in `_sass/custom/custom.scss` instead. Declaring a *new* property the theme never touches (like the `.nav-category` color) works fine from the color scheme.

`_sass/custom/setup.scss` and `_sass/custom/custom.scss` are shared by all three built stylesheets, so they must not reference variables that only `bungalo-dark.scss` defines — the `just-the-docs-light`/`-dark` builds would fail with "Undefined variable". That is what `setup.scss` is for.

To override any other piece of theme markup, copy the theme's include/layout into `_includes/` or `_layouts/` under the same filename.

## Markdown conventions used across pages

- Callouts: put `{: .important}` or `{: .warning}` on its own line **before** the paragraph it styles.
- Buttons: `[Join the Discord](https://discord.gg/bungalo){: .btn }`.
- YouTube embeds: wrap the `<iframe>` in `<div class="youtube-container">` with the inline absolute-position style used in `_modding_guides/wabbajack/index.md`; the class supplies the aspect ratio.
- Every content page sets `layout: default`.

## Dependencies

`Gemfile` pins `just-the-docs` to an exact version (currently 0.12.0) and Jekyll to `~> 4.4.1`; Dependabot opens the bump PRs. just-the-docs minor bumps have needed migration work before (e.g. `nav_footer_custom` in 0.12.0), so build and eyeball the nav after any theme bump. The `sass.silence_deprecations: [import]` block in `_config.yml` exists because the theme still uses `@import`; remove it only once upstream migrates to `@use`.

[just-the-docs]: https://just-the-docs.github.io/just-the-docs/
