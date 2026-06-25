# Law-Gallery — AGENTS.md

## What this is

Hugo static site serving a Chinese-language law-education image gallery. Deployed to GitHub Pages.

## Quick start

```powershell
# Dev server with drafts
hugo server --buildDrafts --disableFastRender

# Production build (matches CI)
hugo --theme=gallery --buildDrafts --gc --minify
```

Hugo Extended >= 0.123.0 required (v0.153.2 installed locally). Theme requires the **extended** edition for Hugo Pipes (Sass/JS bundling).

## Architecture

### Theme

[nicokaiser/hugo-theme-gallery](https://github.com/nicokaiser/hugo-theme-gallery) v4 — git submodule at `themes/gallery/`.

- No local theme overrides exist (`layouts/`, `static/`, custom CSS/JS are all absent)
- Theme provides: justified album grids, PhotoSwipe lightbox, OpenGraph tags, dark theme, i18n
- **Never use WebP images** — Hugo's Go-based WebP encoder has a color-level bug. Use PNG or JPEG only.
- Theme dependencies are bundled (no Node.js needed to build).

### Content model

```
content/
  _index.md              # Homepage (page bundle with cover image)
  about.md               # Prose page (layout: prose, not listed in albums)
  imprint.md             # Footer prose page
  featured-album/        # Private + featured album (homepage only)
  CivilLaw/              # Leaf bundle → single gallery page
  CriminalLaw/           # Leaf bundle
  ConstitutionLaw/       # Leaf bundle
  Theory/                # Leaf bundle
  CivilProcedureLaw/     # Leaf bundle
  CommercialLaw/         # Leaf bundle
  CriminalProcedureLaw/  # Leaf bundle
```

- **Leaf bundles** (`index.md` + images) → gallery page. All current albums are leaf bundles.
- **Branch bundles** (`_index.md` + child bundles) → album list page. Not used yet; if sub-albums are needed, the parent dir needs `_index.md`.

### Frontmatter conventions (all albums)

```yaml
---
date: 2023-04-01
title: 民法                    # Chinese title
sort_by: Name                 # Sort images by filename (explicit in every album)
categories: ["民法"]           # Single Chinese category per album
resources:
  - src: ai-logo.png
    params:
      cover: true             # Album thumbnail
      hidden: true            # Hidden from gallery lightbox
  - src: civil-rights.png     # Visible gallery image
---
```

- `cover: true` + `hidden: true` is the standard pattern for cover images (thumbnail but not shown in gallery).
- Image titles from `resources[].title` show in the lightbox (overrides EXIF `ImageDescription`).

## CI/CD

GitHub Actions at `.github/workflows/hugo.yaml`:

| Trigger | Condition |
|---|---|
| Push to `main` | Unless path matches `images/**`, `LICENSE`, or `README.md` |
| `workflow_dispatch` | Manual trigger |

Pipeline: `checkout (submodules: true)` → `git submodule update --init --recursive` → `git submodule update --remote` → `peaceiris/actions-hugo@v3 (extended, latest)` → `hugo --theme=gallery --buildDrafts --gc --minify` → `peaceiris/actions-gh-pages@v3` deploys `./public` to `gh-pages` branch.

**Key**: `--buildDrafts` is used in production — drafts are published. Theme is force-updated to latest commit on every build via `submodule update --remote`.

## Config

Only `hugo.toml` at repo root (no `config/_default/`):

```toml
baseURL = 'https://hanguangwu.github.io/Law-Gallery'
languageCode = 'zh-cn'
title = 'Law-Gallery'
theme = 'gallery'
```

## Image workflow

- All images are PNG (keep this consistent).
- To add a new album: create a directory under `content/` with `index.md` and images. Follow existing frontmatter conventions (cover+hidden for thumbnail, `sort_by: Name`).
- Adding images to an existing album: drop PNG files in the album directory and list them under `resources` in `index.md`.
- To reorder images in an album: change `sort_by` (default `Name`, supports `Date`, `Params.weight`).

## i18n

Chinese translations at `themes/gallery/i18n/zh.yaml`. The site is fully Chinese — frontmatter titles, descriptions, and categories use Chinese throughout.

## Git

No commits yet — this is a fresh repository. No branch conventions established.

## Gotchas

1. **WebP forbidden** — Hugo's WebP resize produces washed-out images. PNG/JPEG only.
2. **No local theme overrides** — if you need custom CSS, create `assets/css/custom.css`. If you need custom JS, create `assets/js/custom.js`. Theme's exampleSite shows the pattern.
3. **Categories taxonomy** is enabled (used by all albums). No `disableKinds` configuration means `taxonomy` and `term` pages are active.
4. **Theme auto-update** in CI — the submodule is updated to latest remote commit each deploy. This means theme changes can break the build without notice.
5. **`--buildDrafts` always on** — drafts are published. Don't rely on `draft: true` to hide content.
