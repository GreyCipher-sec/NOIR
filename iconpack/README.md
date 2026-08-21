# iconpack/

Curated subset of two icon packs, bundled at the source-tree level (not
under `static/`) so `load_data(..., format="plain")` can inline the SVG
markup directly into rendered HTML at build time, only the icons a site
actually references (via `icon = "slug"` in `zola.toml`) end up in the
published output. Nothing here is copied wholesale into `public/`.

- `social/` from [Simple Icons](https://simpleicons.org) (CC0 1.0, see `social/LICENSE.md`).
  38 common social/contact/publishing platforms.
- `crypto/` from Simple Icons as well (CC0 1.0, see `crypto/LICENSE.md`).
  20 common cryptocurrencies.
- `generic/` from [Lucide](https://lucide.dev) (ISC, see `generic/LICENSE`).
  Fallback icons for concepts Simple Icons doesn't cover as a "brand"
  (a PGP key, a generic email address, a generic link, a generic coin).

## Adding an icon not in this subset

Both packs are far larger than what's bundled here. If you need a slug
that isn't in `social/` or `crypto/`, grab the single SVG file from the
upstream repo and drop it in the matching folder, named `<slug>.svg`:

- Simple Icons (thousands of brands, incl. more cryptocurrencies):
  https://github.com/simple-icons/simple-icons/tree/develop/icons
- Lucide (generic UI glyphs):
  https://github.com/lucide-icons/lucide/tree/main/icons

No build step or registration needed beyond adding the file, set
`icon = "<slug>"` in `zola.toml` and the template picks it up.
