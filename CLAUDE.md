# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal Hugo blog ("寺子屋" / terakoya) published to GitHub Pages at https://kiwamizamurai.github.io/. Built with Hugo 0.91.2 (extended) using the **PaperMod** theme, which is vendored as a git submodule at `themes/PaperMod`.

## Common commands

```sh
# Create a new post — slug is today's date, content lives at content/posts/YYYY-MM-DD/index.md
make new

# Local preview (requires Hugo extended ≥ 0.91.2 installed locally; not declared in repo)
hugo server -D       # include drafts
hugo --minify        # production build into ./public (matches CI)

# First clone — themes/PaperMod is a submodule and must be fetched
git submodule update --init --recursive
```

There are no tests, linters, or package manager configs — this is a pure Hugo site.

## Deployment

`.github/workflows/gh-pages.yml` builds on every push to `main` (and PRs) using `peaceiris/actions-hugo@v2` with Hugo `0.91.2` extended, then publishes `./public` via `peaceiris/actions-gh-pages@v3`. Pinning Hugo to 0.91.2 in the workflow means local builds should match that version to avoid template/feature drift.

## Architecture notes

- **Posts as page bundles.** Each post is a directory under `content/posts/<date>/` containing `index.md` plus any co-located images. The `make new` archetype enforces this layout — never create flat `content/posts/foo.md` files, since image paths in posts assume the bundle directory.
- **PaperMod customization is done via two override partials only** (`layouts/partials/`):
  - `comments.html` — giscus widget bound to this repo's Discussions (`Announcements` category, `pathname` mapping). The repo/category IDs are hardcoded; if the repo is forked or renamed, regenerate them at https://giscus.app and update this file.
  - `extend_head.html` — gates KaTeX loading on `.Params.math` / `.Site.Params.math`. Math rendering is enabled site-wide via `params.math: true` in `config.yml`. Both `$$…$$` (display) and `$…$` (inline) delimiters are configured.
- **Search** is client-side via Fuse.js (PaperMod built-in). It requires the JSON output declared under `outputs.home` in `config.yml` — do not remove that block or `/search` breaks.
- **Profile mode is enabled** (`params.profileMode.enabled: true`), so the home page renders the avatar/title/buttons layout rather than the post list. Posts are reached via the `/posts/` menu entry.
- **Comments are opt-in per site** (`params.comments: true`) and the partial is loaded by PaperMod only when this flag is set.

## Editing existing posts

`params.editPost.URL` points to `https://github.com/kiwamizamurai/kiwamizamurai.github.io/blob/main/content` with `appendFilePath: true`, so each post renders a "Suggest Changes" link to its source file on GitHub `main`. Keep filenames stable to avoid breaking those links from external references.
