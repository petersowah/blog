[![Netlify Status](https://api.netlify.com/api/v1/badges/88d3a42c-0fc5-41bd-ab2a-b59af790d50d/deploy-status)](https://app.netlify.com/sites/petersowah/deploys)

# petersowah.dev

Personal site and blog — [petersowah.dev](https://petersowah.dev)

Jekyll, with a custom theme. No CSS framework, no build step beyond Jekyll's own Sass.

## Design

- **Palette** — kente-derived: green-black ground, kente gold, clay. Dark by default, with a sage-paper light mode. Both are defined once as token mixins in `_sass/_tokens.scss`.
- **Type** — Bricolage Grotesque (display), Newsreader (body), JetBrains Mono (dates, tags, labels and code).
- **Adinkra glyphs** — section markers in `_includes/glyph.html`, each chosen for its meaning: Nkyinkyim (versatility) on the home hero, Nkyimu (precision) over Writing, Fihankra (the enclosed house) on About, Sankofa (go back and fetch it) on the Archive, Adinkrahene as the logo mark.

## Running it

```bash
bundle install
bundle exec jekyll serve
```

## Layout

```
_sass/          tokens, base, and one partial per region
_layouts/       default, post, page, about, archive
_includes/      header, footer, glyph, marker, social, meta tags
_data/          author.json, social.json
_posts/         posts, front matter takes title, description, tags, feature_image
```

Site-wide values — navigation, description, pagination — live in `_config.yml`.
Bio, stack list and social links live in `_data/`, not in templates.
