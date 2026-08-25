# The Modding Bungalo Wiki

Source for **[themoddingbungalo.com](https://themoddingbungalo.com)** — the community wiki for
[The Modding Bungalo](https://discord.gg/bungalo) Discord, covering the server's Wabbajack
modlists and general Skyrim/Bethesda modding guides.

It's a [Jekyll] site using the gem-based [Just the Docs] theme, deployed to GitHub Pages.
Everything here is Markdown — there's no application code and no test suite.

## Contributing

You don't need to run anything locally to suggest a change. The quickest route is to
[open an issue](https://github.com/themoddingbungalo/site-v1/issues/new/choose) describing what
needs adding or fixing (a GitHub account is required).

If you'd rather write the change yourself, fork the repo, edit the Markdown, and open a pull
request. Read [Adding a page](#adding-a-page) first — the sidebar is built from front matter, not
from the folder layout, so a new file won't appear in the nav until it's wired up.

## Running it locally

Requires Ruby and [Bundler].

```bash
bundle install                # first time, and after any Gemfile change
bundle exec jekyll serve      # preview at http://localhost:4000, rebuilds on save
bundle exec jekyll build      # one-shot build into _site/
```

## Site structure

Content lives in three Jekyll collections plus a handful of standalone pages at the repo root:

| Path | Nav section | Contents |
| --- | --- | --- |
| `_lists/` | Lists | Per-modlist docs — Lorerim, CSVP, Ghoulified, Do Not Go Gentle, Speed Tweaks |
| `_ngvo/` | NGVO | The NGVO visual baseline — overview, read me, addons, forks, FAQs, showcase |
| `_modding_guides/` | Modding Guides | Wabbajack, xEdit, LOD generation, Creation Kit, building a modlist |
| `index.md`, `contribute.md`, `biggie-boss.md`, `themoddingbordello.md` | (top level) | Standalone pages |

Adding a whole new collection means adding it to `_config.yml` **twice** — once under
`collections:` (permalink and output) and once under `just_the_docs.collections:` (nav display
name and fold behavior).

## Adding a page

Just the Docs builds the sidebar from front matter, and it links a child to its parent by
**matching the parent page's `title` string**. Directory nesting is for humans only. A page two
levels deep needs both keys:

```yaml
---
layout: default
title: Colloquy's Guide
parent: Guides          # must exactly match the parent page's title:
grand_parent: CSVP      # must exactly match the grandparent's title:
nav_order: 1
---
```

Things that bite:

- Renaming a page's `title` silently orphans every child pointing at it. Update the children in
  the same commit.
- A parent page must set `has_children: true`, or its children won't render in the nav.
- `nav_order` is scoped to its nav section, so the numbering restarts per collection and parent.
- Only two levels of nesting exist (`parent` + `grand_parent`). There is no `great_grand_parent`.
- Every content page sets `layout: default`.

### Markdown conventions

- Callouts — put `{: .important}` (amber) or `{: .warning}` (red) on its own line **before** the
  paragraph it styles.
- Buttons — `[Join the Discord](https://discord.gg/bungalo){: .btn }`.
- YouTube embeds — wrap the `<iframe>` in `<div class="youtube-container">`; the class supplies
  the 16:9 aspect ratio. See `_modding_guides/wabbajack/index.md` for the pattern.
- Assets — reference as `{{ site.baseurl }}/assets/...`.

## Theming

The theme is a gem, so there's no theme source in this repo. The custom layer is four files:

| File | Role |
| --- | --- |
| `_sass/custom/setup.scss` | Shared values, imported before everything. Currently `$bungalo-red`. |
| `_sass/color_schemes/bungalo-dark.scss` | The `bungalo-dark` scheme — theme color variables, plus `.youtube-container`, `.important`, `.warning`. |
| `_sass/custom/custom.scss` | Imported **last**, so it can override the theme's own rules. |
| `_includes/nav_footer_custom.html` | Sidebar footer credit. |

Import order matters, because the color scheme is pulled in *before* the theme's modules:

```
support → custom/setup → color_schemes/light → color_schemes/bungalo-dark → modules → callouts → custom/custom
```

So overriding a property the theme already sets (e.g. `font-weight` on `.nav-category`) has to go
in `custom/custom.scss`. Declaring a property the theme never touches works fine from the color
scheme. Note that `setup.scss` and `custom.scss` are compiled into the `just-the-docs-light` and
`-dark` stylesheets too, which never load `bungalo-dark.scss` — so they must not reference
variables defined only there. That's what `setup.scss` is for.

To override any other piece of theme markup, copy the theme's include or layout into `_includes/`
or `_layouts/` under the same filename.

### Palette

As implemented in `_sass/`:

| Role | Hex |
| --- | --- |
| Accent red (`$bungalo-red`) — nav categories, buttons, Discord link | `#b94733` |
| Link / logo cream | `#f5ecd6` |
| Background & sidebar | `#252a2d` |
| Body text | `#e6e6e6` |

The logo SVGs in `assets/` are cropped to their artwork. They were originally exported from
Illustrator on an A4 artboard, so their `viewBox` framed the logo in a page of empty space —
don't re-export over them without re-cropping.

## Deployment

Two workflows in `.github/workflows/`:

- `ci.yml` — runs `bundle exec jekyll build` on every push and pull request. A green check means
  the site compiled, nothing more.
- `pages.yml` — on push to `main`, builds with `JEKYLL_ENV=production` and deploys to GitHub
  Pages. Changes are usually live within a minute or two.

### `baseurl` must stay empty

`_config.yml` sets `baseurl: ""` and `url: https://themoddingbungalo.com`, and `pages.yml`
deliberately does **not** pass `--baseurl` or use `actions/configure-pages`. The site is served
from the root of the custom domain (see `CNAME`); a derived baseurl bakes in `/site-v1` and 404s
every asset on the site. This has broken the site before — see commit `519995c`.

`aux_links` in `_config.yml` is the Discord invite shown in the header. `url` is the site's own
canonical base. They are not interchangeable.

## Dependencies

`Gemfile` pins `just-the-docs` to an exact version (currently 0.12.0) and Jekyll to `~> 4.4.1`;
Dependabot opens the bump PRs. Theme minor bumps have needed migration work before (for example
`nav_footer_custom` in 0.12.0), so build and eyeball the nav after any theme upgrade.

## Attribution

This repo started from the [just-the-docs template], and the [MIT License] file is retained from
it. The site content is by The Modding Bungalo community.

[Jekyll]: https://jekyllrb.com
[Just the Docs]: https://just-the-docs.github.io/just-the-docs/
[Bundler]: https://bundler.io
[just-the-docs template]: https://github.com/just-the-docs/just-the-docs-template
[MIT License]: LICENSE
