# NOIR

A minimal, dark, monospace-accented [Zola](https://www.getzola.org) theme for research notebooks, technical archives, and personal engineering blogs. No JavaScript, no tracking, no build step beyond Zola itself.

![NOIR theme screenshot](screenshot.png)

## Features

- Dark, high-contrast color scheme designed for long-form reading
- Six built-in content sections: **posts**, **projects**, **notes**, **ctf**, **wargames**, **about**
- Homepage digest showing latest posts, active projects, recent notes, recent CTF entries, and recent wargame logs
- Per-project metadata (stack, repository, license, status) via `extra`
- Tag and category taxonomies
- Built-in RSS 2.0 feed (`/rss.xml`), with `<link rel="alternate">` auto-discovery in `<head>`
- Pagination on all list sections
- SEO out of the box: dynamic meta descriptions, canonical URLs, Open Graph, Twitter Cards, JSON-LD structured data (see [SEO](#seo) below)
- Collections for multi-part series, ongoing projects, and investigative dossiers, any post in any section can opt in (see [Collections](#collections) below)
- Modular by default: only **posts** and **about** are required, the other five sections (projects, notes, collections, ctf, wargames) are discovered dynamically and can be addded or skipped entirely at any point without a single template (see [Content structure](#content-structure) below)
- Optional Contact / crypto donations / PGP sections on the About page, each with an inline SVG icon from a bundled, curated icon pack (see [Contact, donations & PGP](#contact-donations--pgp) below)
- Optional webring footer widget: previous/next links computed from your position in a configured ring (see [Webring](#webring) below)
- No JavaScript, no external requests, no cookies
- Fully static, single CSS pass, self-hosted fonts

> Content is managed by hand exactly as you would with any other Zola site, creating files, editing front matter, running `zola build/serve`. See [Managing content](#managing-content) below.

## Installation

Inside your Zola site, add NOIR as a submodule (or plain clone) under `themes/`:

```
git submodule add https://github.com/GreyCipher-sec/NOIR.git themes/NOIR
```

Or without submodules:

```
git clone https://github.com/GreyCipher-sec/NOIR.git themes/NOIR
```

Then enable it in your site's `zola.toml`:

```
theme = "NOIR"
```

Three things need to exist at your site's root, not just inside `themes/NOIR/`, copy them over from the theme:

```
cp themes/NOIR/zola.toml.example zola.toml   # then edit base_url, title, etc.
cp -r themes/NOIR/content.example content
cp -r themes/NOIR/iconpack .
```

- **`content.example/`** has the two required `_index.md` stubs (`posts` and `about`, see [Content structure](#content-structure) below, everything else is optional and can be added later, see [Adding an optional section later](#adding-an-optional-section-later).
- **`icontpack/`** is required regardless of whether you use Contact / Donate / PGP: the RSS link in the navbar loads an icon from it unconditionally on every page (see [Icons](#icons) below). Zola's `load_data()` always resolves path against your site's root, never the theme's directory, so the theme's own copy of `iconpack/` isn't reachable from your site unless you copy it over, without this step, every page fails to build, not just the About page.

## Content structure

NOIR's only two required sections are **posts** and **about**, everyting else (projects, notes, collections, ctf, wargames) is entirely optional and discovered dynamically at build time. The navbar and homepage digest check which section folders actually exist under `content/` (via Zola's `subsections`, not a hardcoded list), so a section you haven't created yet simply doesn't appear anywhere, no placeholder link, no empty digest block and adding one later needs zero template changes:

```
content/
├── about/
│   └── _index.md
├── posts/
│   ├── _index.md
│   └── my-post.md
```

RSS needs no section of its own `generate_feed = true` in `zola.toml` is enough on top of whatever content exists.

The fastest way to get this minimal base right is to copy `content.example/` (at this theme's repository root) into your site's `content/`, it has both required `_index.md` files with correct front matter and zero entries, ready to build immediately.

### Adding an optional section later

Each of the five optional sections lives, pre-built and ready to copy under `optional-sections.example/` at the theme's repository root:

```
cp -r themes/NOIR/optional-sections.example/projects content/projects
```

Each `_index.md` needs a `template` pointing to the matching section template, e.g.:

```
+++
title = "Posts"
sort_by = "date"
template = "posts_section.html"
page_template = "page.html"
paginate_by = 10
+++
```

Repeat the same pattern for `projects/_index.md` (`projects_section.html`), `notes/_index.md` (`notes_section.html`), `collections/_index.md` (`collections_section.html`, `page_template = "collection_single.html"`, see [Collections](#collections) below), `ctf/_index.md` (`ctf_section.html`), and `wargames/_index.md` (`wargames_section.html`). `about/_index.md` uses `about_section.html` and just renders its own body, it has no child pages.

## Navigation

The navbar in the header is generated dynamically, not hardcoded. `base.html` first asks Zola which section folders actually exist under `content/`, then, for the ones that do, checks whether they have any pages yet:

```
{% set home = get_section(path="_index.md") %}
{% if "posts/_index.md" in home.subsections %}
  {% set posts_section = get_section(path="posts/_index.md") %}
  {% if posts_section.pages | length > 0 %}<a href="/posts/">posts</a>{% endif %}
{% endif %}
```

The outer `in home.subsections` check is what makes this safe to run even when a section (say `projects/`) doesn't exist at all yet, not just when it exists but is empty. Calling `get_section()` directly on a path that has no folder at all is a hard build error in Zola, checking membership in the root section's `subsections` list first avoids ever making that call for a path that isn't there. This is the mechanism behind [modularity](#content-structure): the five optional sections can be entirely absent from `content/`, and every page still builds.

Once a section folder exists, its link still only appears after it has at least one page beyond its own `_index.md`, so creating `projects/_index.md` alone (e.g. via `optional-sections.example/`) doesn't clutter the nav until you've actually added a project.

This isn't limited to the five sections NOIR ships templates for, either. After the known five, both the navbar and the homepage digest loop over `home.subsections` a second time and pick up **any other section folder**, under any name, the same way, using that section's own `title` as the link/heading text and generic title+date rendering on the homepage (since a name NOIR's never heard of has no bespoke fields like a project's `stack` or a CTF entry's `difficulty` to show). A section named `content/gists/` with its own `_index.md` (`template` pointing at any listing template the theme ships, e.g. `posts_section.html`) shows up in the nav as "gists", gets its own "GISTS" block on the homepage once it has an entry, and its pages appear in `/rss.xml` automatically, RSS is a global feed over every page site-wide already, with no per-section awareness to update in the first place. Removing the section (and its content) makes all three disappear again on the next build, with nothing left over: NOIR only ever reports what currently exists on disk, it doesn't remember a section that used to be there.

`about` is the one exception and always shows: it's a single-body page, not a listing, so it never has "pages" to count, and it's one of the two sections assumed to always exist.

**Note:** `get_section()` must be called without `metadata_only=true` for the page-count check to work, that flag causes Zola to return an empty `pages` array regardless of actual content, which would hide every section permanently. This isn't documented clearly upstream, worth knowing if you extend this pattern further.

### Post / note front matter

```
+++
title = "Post title"
date = 2026-08-02T15:12:38+02:00
description = "Optional one-line summary, used in RSS."

[taxonomies]
categories = ["systems"]
tags = ["linux", "kernel"]
+++

Your content here.
```

### Project front matter

```
+++
title = "packet-loom"
date = 2026-08-02T16:09:17+02:00
description = "A minimal packet inspection toolkit for constrained hosts."

[extra]
status = "Active"
stack = "Rust, eBPF, libpcap"
repository = "https://github.com/you/packet-loom"
license = "MIT"
+++

Project write-up here.
```

### CTF write-up front matter

```
+++
title = "picoCTF 2026 - pwn/baby-rop"
date = 2026-08-05T10:00:00+02:00
description = "A basic ROP chain to bypass NX on a stripped x86_64 binary."

[taxonomies]
categories = ["pwn"]
tags = ["rop", "x86_64", "nx-bypass"]

[extra]
event = "picoCTF 2026"     # competition or CTF event name
category = "pwn"           # pwn / web / crypto / rev / forensics / misc
difficulty = "Easy"        # Easy / Medium / Hard / Insane
points = 100                # challenge point value, optional
solved = true                # shows a SOLVED / UNSOLVED badge on the listing page
+++

Write-up content here.
```

### Wargame log front matter

```
+++
title = "OverTheWire: Narnia - Level 4"
date = 2026-08-06T10:00:00+02:00
description = "Exploiting a format string vulnerability to overwrite a return address."

[taxonomies]
tags = ["format-string", "narnia"]

[extra]
platform = "OverTheWire - Narnia"   # wargame platform / series name
level = "4"                          # level, floor, or challenge number
difficulty = "Medium"                # Easy / Medium / Hard / Insane
+++

Write-up content here.
```

## Configuration

Minimal `zola.toml` for a site using NOIR:

```
base_url = "https://example.com"
title = "Your Site"
description = "Your tagline."
theme = "NOIR"

generate_feeds = true
feed_filenames = ["rss.xml"]

[[taxonomies]]
name = "tags"
feed = true
paginate_by = 10

[[taxonomies]]
name = "categories"
feed = true
paginate_by = 10

[extra]
author = "Your Name"
social_image = "images/social-preview.png"  # optional, used for Open Graph / Twitter previews
```

## SEO

NOIR ships with sensible SEO defaults so you don't have to think about it. All of this is generated automatically from your content and `zola.toml`, no extra setup required.

**On every page:**
- Unique `<title>` per page (`Post Title - Site Title`)
- `<meta name="description">` uses the page's `description` front matter, falls back to the auto-generated summary, then to the site description
- `<link rel="canonical">` prevents duplicate-content issues
- Open Graph tags (`og:title`, `og:description`, `og:type`, `og:image`, `og:url`) controls how links look when shared on Slack, Discord, etc.
- Twitter Card tags (switches to `summary_large_image` automatically if a social image is configured)
- `sitemap.xml` and `robots.txt` generated by Zola itself, no configuration needed
- `<link rel="alternate" type="application/rss+xml">` in `<head>` so feed readers and browsers can auto-discover the RSS feed without a visibile link

**On posts, projects, and notes specifically:**
- JSON-LD structured data (`schema.org/Article`) with publish date, author, and canonical URL, helps search engines understand your content without guessing
- Semantic `<time datetime="...">` for publish dates

**On the homepage:**
- JSON-LD `WebSite` block so search engines can identify the site

### Setting a default social preview image

To make shared links show a preview image instead of a blank card, add an image (1200×630px recommended) under `static/` and reference it in `zola.toml`:

```
[extra]
social_image = "images/social-preview.png"
```

### Writing good descriptions

Every `_index.md` and content page accepts a `description` field. Keep it under ~160 characters, that's roughly what shows up in Google search results and link previews:

```
+++
title = "Bypassing Signature Checks on Embedded Bootloaders"
date = 2026-08-02T15:12:38+02:00
description = "A walkthrough of a bootloader signature bypass on a common ARM SoC, and what it means for supply-chain trust."
+++
```

If you skip `description`, NOIR falls back to an auto-generated summary of the page content, then to the site-wide description in `zola.toml`, so nothing is ever left empty, but writing one by hand is worth the two minutes it takes.

## Collections

Some write-ups don't fit in a single post, a multi-part series, a long-running project, or a CTF/wargame dossier made of several linked entries. NOIR supports this without turning into a wiki or a CMS: any post in **any** section (posts, ctf, wargames, notes) can optionally belong to a collection, while continuing to behave like a completely normal, standalone article, it still shows up in its section listing, the homepage, RSS, tags/categories, and the sitemap exactly as before.

### Concepts

- **Collection** a themed group of entries: a `series` (linear, e.g. a tutorial), a `project` (open-ended, e.g. a tool that keeps evolving), a `dossier` (investigative/technical, e.g. a CTF case), or any other type you invent. Each collection is its own real, indexable page at `/collections/<id>/`.
- **Entry** a normal post that opts into a collection via two small front matter additions. It keeps its own URL, its own section, its own tags, the collection is a relationship, not a container.

This is deliberately built on Zola's native taxonomy system (the same mechanism behind `tags`/`categories`) rather than a bespoke system, so it stays static, git-versionable, and framework-native. It does **not** use Tera's list-manipulation filters (`concat`, `filter`, `map`) because this Zola build doesn't ship them, cross-section aggregation is instead handled by Zola's taxonomy engine itself, which is more robust anyway.

### How to create a new collection

Create one file: `content/collections/<id>.md`

```
+++
title = "HTB Sherlock"
date = 2026-08-01        # started
updated = 2026-08-11      # last updated - bump this whenever you add an entry

[extra]
collection_type = "dossier"   # series | project | dossier | anything you want
status = "active"              # planned | active | paused | completed | archived
+++

Optional longer intro shown at the top of the collection page.
```

`<id>` must be lowercase-hyphenated (e.g. `htb-sherlock`) it's used as-is to link entries to this collection, so keep it consistent.

The `status` field controls what the position indicator shows on entry pages: `completed`/`archived` collections show a fixed total (`3 / 5`); anything else shows an open-ended total (`3 / ?`), since more entries might still be coming.

### How to add a post to a collection

In any post, in **any** section, add:

```
[taxonomies]
collection = ["htb-sherlock"]   # must match the collection's <id> exactly

[extra]
collection_part = 3              # required - controls ordering within the collection
```

That's it. The entry automatically appears on the collection page, in the right position, with working Previous/Next navigation, no manual index to update, no total to maintain.

**`collection_part` is required for every entry in a collection.** Templates sort and link entries by this value, so a missing `collection_part` on an entry that has a `collection` taxonomy will break template rendering at `zola build` time, Zola will fail with an error pointing at the offending page, since the field the template expects isn't there. This is Zola's own behavior, not a feature NOIR adds on top; today, nothing checks for this *before* you run the build. See [Managing content](#managing-content) for a planned tool that would catch this earlier, with a clearer message.

Standalone posts (no `[taxonomies] collection`) are entirely unaffected and need no `collection_part`.

Numbering can have gaps (`1, 2, 4, 5`), the system only sorts by whatever values exist, it doesn't require consecutive numbers or a known final count.

### How the navigation works

Any post with a `collection` taxonomy automatically gets, with zero extra template work:

- An eyebrow line above the title: `SERIES / BUILDING A CLI IN RUST · 3 / 5`, linking to the collection page
- A contextual nav block at the end of the article: `← Previous entry`, `↑ Collection`, `→ Next entry` omitting Previous on the first entry and Next on the last, and never appearing at all on standalone posts

Both are computed at build time from `collection_part`, not from publish date, so you can write and correct entries out of order without breaking the sequence.

### How to create a standalone post

Nothing to do, just don't add a `collection` taxonomy. Every post is standalone by default; collections are opt-in.

## Contact, donations & PGP

The About page can optionally show three sections, in this order": **Contact** (socials/email/etc.), **Donate** (crypto addresses) and **PGP** (fingerprint + public key link). Each one is entirely config-driven, from `zola.toml` and each renders only if configured, an empty or absent field means that section simply doesn't appear, no blank boxes.

### Contact

```
[[extra.socials]]
label = "Email"
value = "you@example.com"
url = "mailto:you@example.com"
icon = "mail"

[[extra.socials]]
label = "GitHub"
value = "@you"
url = "https://github.com/you"
icon = "github"
```

One entry per channel, in the order you want them displayed. `label` is the fixed text shown before the link; `value` is the human-readable link text (a handle, an address); `url` is where it points.

### Donations

```
[[extra.donations]]
coin = "BTC"
address = "bc1q..."
icon = "bitcoin"

[[extra.donations]]
coin = "XMR"
address = "4..."
icon = "monero"
```

One entry per coin you accept, in display order. Add or remove entries freely, nothing else needs updating.

### PGP

```
[extra]
pgp_fingerprint = "AAAA BBBB CCCC DDDD EEEE  FFFF 0000 1111 2222 3333"
pgp_key_url = "/pgp-key.asc"
```

`pgp_key_url` should point at your exported public key, a file under `static/` (linked as an absolute path like above) or an external keyserver URL. Export one with:

```
gpg --armor --export you@example.com > static/pgp-key.asc
```

The section only appears if `pgp_fingerprint` is set; `pgp_key_url` is optional on top of that (omit it and you just get the fingerprint with no download link).

### Icons

Both Contact and Donate entries take an `icon` field, a slug rendered as an inline SVG next to the entry, styled to inherit the surrounding text color rather than showing a fixed brand color. The icons live in `iconpack/` at the theme root (not under `static/`), split into `iconpack/social/`, `iconpack/crypto/`, and `iconpack/generic/`:

- **`iconpack/social/`** and **`iconpack/crypto/`** a curated ~58-icon subset of [Simple Icons](https://simpleicons.org) (CC0 1.0), covering common platforms (GitHub, Mastodon, Matrix, Discord, RSS, and more) and cryptocurrencies (Bitcoin, Monero, Ethereum, and 17 others). See `iconpack/social/` and `iconpack/crypto/` for the exact file list, each `<slug>.svg` matches the `icon` value you'd write in config.
- **`iconpack/generic/`** five fallback icons from [Lucide](https://lucide.dev) (ISC) for concepts Simple Icons doesn't cover as a brand: `mail`, `key`, `link`, `coins`, `globe`. Use these for a generic email address, or leave `icon` unset entirely and the template falls back to `link` (Contact) or `coins` (Donate) automatically.

Icons are inlined at build time via Zola's `load_data(path=..., format="plain")`, only the icons a site actually references end up in the published HTML; the rest of the bundled pack adds to the theme repo's size but never ships to visitors. **This requires `iconpack/` to exist at your site's root** (see [Installation](#installation) above), `load_data()` cannot reach it inside `themes/NOIR/` since Zola resolves these paths against the site being built, not the theme providing the template.

**If you set `icon` to a slug that isn't bundled, the build fails** with a clear `doesn't exist` error naming the missing file, it does not fall back silently. This is intentional (consistent with how a missing `collection_part` fails loudly elsewhere in this theme) so a typo doesn't quietly ship a broken page. To add an icon that isn't in the curated set, grab the single SVG from upstream and drop it in the matching folder:

- More Simple Icons (thousands of brands and cryptocurrencies): https://github.com/simple-icons/simple-icons/tree/develop/icons
- More Lucide glyphs: https://github.com/lucide-icons/lucide/tree/main/icons

No registration or build step beyond adding the file, see `iconpack/README.md` for details.

## Webring

The footer can optionally show a webring widget: `← previous site · ring name · next site →`, computed from your position in a configured member list.

```
[extra.webring]
enabled = true
name = "Example Webring"
url = "https://example-webring.example/"

[[extra.webring.sites]]
name = "alice's blog"
url = "https://alice.example"

[[extra.webring.sites]]
name = "This site"
url = "https://example.com"
current = true

[[extra.webring.sites]]
name = "bob's blog"
url = "https://bob.example"
```

`sites` is the full membership list **in ring order**, including your own site, mark exactly one entry `current = true` so the template knows where you are and can compute your neighbors. `name`/`url` on `[extra.webring]` itself describe the ring (optional, `url` usually points at the ring's member directory or homepage).

`enabled = false` hides the widget while leaving the rest of the config intact, useful if you've listed a reciprocal link but haven't confirmed the other site actually links back yet, or you just want to pause it temporarily without deleting the list. Omit `enabled` (or leave it `true`) to show it normally.

Position is resolved at build time (`set_global` plus modulo arithmetic over the list), so reordering or growing the ring is just editing this list, nothing to recompute by hand.

**Limitations, both deliberate:**
- Only previous/next are rendered, not the classic "random member" link some rings offer. A static site with no JavaScript and no server can't pick randomly at request time, only in advance, so a "random" link here would just be a fixed, non-random choice wearing a misleading label.
- If `sites` has fewer than 2 entries, none is marked `current`, or `enabled` is `false`, the widget doesn't render, no error, it just silently doesn't appear. If you're not in a webring, leave `[extra.webring]` out entirely, the footer stays otherwise empty (no fallback text is shown in its place).
- **With exactly 2 sites in the ring**, previous and next resolve to the same site, the wraparound math has nowhere else to go. Cosmetically redundant (`← friend · ring · friend →`) but not a bug.

## Managing content

Everything above, creating files, filling in front matter, bumping `updated` on a collection, running `zola build`/`zola serve`, is done by hand today, the same way as on any other Zola site. There is no bundled tooling in this repository.

## Templates

| Template | Purpose |
|---|---|
| `base.html` | Shell: header, nav, footer, shared styles |
| `index.html` | Homepage, latest posts, active projects, recent notes |
| `page.html` | Single post / project / note page |
| `posts_section.html` | Posts listing with pagination |
| `projects_section.html` | Project dossiers listing with pagination |
| `notes_section.html` | Notes grid with pagination |
| `ctf_section.html` | CTF write-ups listing,  shows event, category, difficulty, points, solved/unsolved badge |
| `wargames_section.html` | Wargame logs listing, shows platform, level, difficulty |
| `collections_section.html` | Index of all series/projects/dossiers |
| `collection_single.html` | Single collection reference page,  status, entry count, ordered entry list |
| `about_section.html` | Renders the About page body, plus optional Contact / Donate / PGP sections |
| `taxonomy_list.html` | Index of tags or categories |
| `taxonomy_single.html` | Pages under one tag/category |
| `404.html` | Not found page |
| `rss.xml` | Custom RSS 2.0 feed (overrides Zola's default Atom feed) |

## Local development

This repository is itself a working Zola site used to preview the theme. It contains a self-referencing symlink so you can develop and preview NOIR without a separate demo site, already included in this repo (`themes/NOIR`, pointing at the repository root):

```
zola serve
```

Because the symlink makes the theme root and the site root the same directory here, `zola.toml`, `content/` and `iconpack/` at the repository root all resolve correctly without any copying, this is *not* representative of a real consuming site, where those three live separately (see [Installation](#installation) above for an actual site needs to copy over).

If you're setting it up from scratch elsewhere (or the symlink didn't survive a zip/copy, see note below), recreate it from the repository root with:

```
mkdir -p themes && ln -s .. themes/NOIR
```

`content/`, `static/`, `templates/` at the repository root **are** the theme; `zola.toml` and `content/` at the root are only used for local preview and are not required by sites consuming the theme.

## License

MIT - see `LICENSE`.

## Credits

Design and layout by GreyCipher.
