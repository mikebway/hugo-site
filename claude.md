# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Source for [mikebroadway.com](http://mikebroadway.com), a static photography site built with [Hugo](https://gohugo.io/) on the [Newsroom theme](https://github.com/onweru/newsroom) (git submodule at `themes/newsroom`). The site has no server component — it's built to `public/` and synced to an S3 bucket.

## Commands

- **Run locally (with drafts and future-dated posts):** `hugo server -D -F --config config.toml,private.toml`
  - `-F` (`--buildFuture`) is required — Hugo sometimes excludes recently-created pages even when their `date` is not in the future.
- **Build for deployment:** `hugo --config config.toml,private.toml` (outputs to `public/`)
- `private.toml` holds the Google Analytics ID and is not checked in; omit `--config config.toml,private.toml` (falling back to `config.toml` alone) if you don't need it.
- `mikeb.toml` is a third, alternate config (different `baseurl`, for `mikeb.photography`) — pass it in place of/alongside the others if working on that variant.

## Architecture

### Content model (`content/post/*.md`)
Each post is a photo series. Front matter drives both the post page and its placement on the home page:
- `feature`: controls which section of the home page (`layouts/index.html`) lists the post — `true` (top "featured" hero + row), `false` ("Older Series" grid), or `"years"` (by-year catalog grid). `layouts/index.html` filters `.Site.Pages` on this value; the very first `feature: true` post (index 0) becomes the hero banner.
- `image`/`thumb`/`bgpos`/`caption`: the hero/card background image and its caption.
- `imglist`: an ordered YAML list feeding the `{{< gallery >}}` shortcode (`layouts/shortcodes/gallery.html`). Each entry:
  - `root`: image path/stem under `static/images/` (not checked into git — see below). The shortcode derives multiple resolutions from it by suffix: `-0096` (admin thumbnail, unused in gallery), `-0240` (grid thumbnail), `-0450`/`-0900`/`-1200`/`-1920` (responsive `srcset` for the lightbox), and `-0900` again for hero/card backgrounds.
  - `title`: caption shown under the thumbnail and in the lightGallery caption bar.
  - `break: true` (optional): forces the thumbnail grid to start a new row right after this entry, via a zero-height flex spacer (`.post_gallery_break` in `_posts.scss`) — otherwise the flexbox grid just wraps wherever it runs out of width.
  - The `/gen-imglist <static/images/subdir>` skill generates an `imglist` block from `*0096.jpg` files in a directory.

### Home page assembly (`layouts/index.html`)
Three independent `range` passes over `.Site.Pages` filtered by `Params.feature`, rendering `.Summary | truncate 100` for each featured/older-series card. Blockquote-led post bodies need a scoped override (see below) or their summary renders as a narrow, tall column in the card.

### Styling (`assets/scss/`)
Compiled via Hugo Pipes (`resources.Get "scss/main.scss" | css.Sass`, wired in `layouts/partials/head.html`) into `main.css`, loaded *before* the vendored lightGallery CSS files. Files: `_base.scss` (global resets/elements — note `img { margin: 15px auto }` and `ul { list-style: none }` are global and have leaked into places that needed a scoped override), `_components.scss` (article cards, toggles, and all lightGallery overrides — see below), `_posts.scss` (single-post layout, gallery grid), `_nav.scss`, `_utils.scss`, `_variables.scss` (CSS custom properties incl. light/dark mode via `--color-mode`/`data-mode`, `--dark` is always `#000` regardless of mode).

### lightGallery integration
lightGallery (vendored under `assets/js/` and `assets/css/`, GPLv3-licensed — see README for licensing caveat) powers the fullscreen lightbox opened from `.post_gallery` thumbnails. Wired up in `layouts/partials/footer.html`, which also sets the plugin's runtime options (`allowMediaOverlap`, `toggleThumb`, etc.).

**Never edit the vendored lightGallery files** (`assets/css/lightgallery.css`, `assets/css/lg-thumbnail.css`, `assets/js/lightgallery*.js`, `assets/js/plugins/*`). All lightGallery visual customizations for this project live in `assets/scss/_components.scss` as additive overrides, using one of two techniques to win the cascade against the vendor CSS (which loads after `main.css`) without `!important`: either a selector with one more class/ancestor than the vendor's own (e.g. `.lg-container .lg-backdrop` beats the vendor's plain `.lg-backdrop`), or a property the vendor rule never sets at all (e.g. `margin` on `.lg-thumb-item img`). Some vendor *behavior* is coupled to config, not CSS — e.g. the thumbnail toggle button is unconditionally suppressed by the plugin whenever `allowMediaOverlap` is `false` (see `lg-thumbnail.umd.js`), so check JS logic before assuming a visual issue is CSS-fixable.

### Static assets (`static/`)
`static/images` and `static/books` are gitignored (large binaries, backed up separately) — **never modify, delete, or add files under `static/`**; that's the user's job. Deprecated/removed images move to `/deprecated` (also gitignored).

## Workflow

1. Before starting non-trivial work, write a plan to `tasks/todo-###.md` (numbered `001`, `002`, ... — `tasks/` is gitignored, so these are local planning notes only) with a checklist, and check in before beginning.
2. Check off todo items as completed; keep changes minimal — no large or complex diffs.
3. Log every change, timestamped, to `CHANGES.log` (this file *is* checked in).
4. Add a review section to the todo file summarizing what was done once finished.

## git / deployment guardrails

- Check in before committing; never push to remote, under any circumstances.
- Never touch files/directories listed in `.gitignore` (except the `tasks/` planning notes described above, which is an intentional, sanctioned exception).
- Never run `sync.sh` or otherwise deploy to S3 — that's the user's action alone.
