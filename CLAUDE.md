# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal Hugo blog published at https://blog.pakithecat.eu/. Static site, no application code — content is Markdown, presentation comes from the PaperMod theme.

## Commands

- Serve locally with drafts and live reload: `hugo server -D`
- Local preview: http://localhost:1313
- Production build (drafts excluded): `hugo` — output goes to `public/`
- New post (page bundle): `hugo new posts/2026/my-slug/index.md` — the archetype seeds `draft: true`, date, and a title derived from the filename
- Update the theme: `git submodule update --remote themes/PaperMod`
- After a fresh clone: `git submodule update --init --recursive`

## Architecture

- **Config:** `hugo.yaml` (single file, YAML). Holds language definitions, menus, taxonomies, PaperMod params, and the Fuse.js search options.
- **Theme:** PaperMod, pinned as a git submodule at `themes/PaperMod` (see `.gitmodules`). Do not edit files under `themes/` — override via `layouts/`, `static/`, or `hugo.yaml` params instead.
- **Multilingual (en + ro):** English is the default language; Romanian translations live beside each file with a `.ro.md` suffix (`index.md` / `index.ro.md`). Each language has its own menu, homeInfoParams, and taxonomy names (`categories`/`tags`/`series` vs `categorii`/`taguri`/`serii`). When adding or changing a post, update both language files.
- **Content layout:** Posts are page bundles grouped by year — `content/posts/<year>/<slug>/index.md` — with the cover image and any other assets co-located in the same folder. Top-level section pages (`archives.md`, `search.md`) also come in `.md` / `.ro.md` pairs.
- **Search:** The home page emits a JSON output (`outputs.home` in `hugo.yaml`) that feeds client-side Fuse.js search on `/search/`.
- **Rendering:** Goldmark is configured with `unsafe: true`, so raw HTML in Markdown is allowed.
- **Mermaid:** ```` ```mermaid ```` fences render as diagrams. `layouts/_default/_markup/render-codeblock-mermaid.html` emits `<pre class="mermaid">`; `layouts/partials/extend_head.html` carries a deferred module script that looks for those elements and dynamically `import()`s mermaid.js from jsDelivr only when the page has one, redrawing on the PaperMod light/dark toggle. `assets/css/extended/mermaid.css` strips the code-block styling. **Do not make PaperMod extension points page-conditional** — `baseof.html` calls `head.html` and `footer.html` through `partialCached` with a key of `.Layout`/`.Kind` and no page identity, so a `.Store.Get` condition inside `extend_head.html` or `extend_footer.html` is evaluated once and that result is reused site-wide. Decide at runtime in the browser instead. Hugo 0.146 moves render hooks to `layouts/_markup/` — this repo pins 0.140.1 in `deploy.yml`, so the `_default` path is correct for now.

## Front matter conventions

Match existing posts (e.g. `content/posts/2026/wsl-basic-commands/index.md`):

- `draft: false` on new posts — they are written to be published, not staged. Note the archetype in `archetypes/default.md` still seeds `draft: true`, so a post created with `hugo new` needs this flipped by hand.
- `tags` (list) and `categories` (usually `["tech"]`)
- `description` — used for SEO and list summaries
- `cover.image` (filename relative to the bundle) and `cover.alt`
- `showToc: true`, `author: "Me"`, and the `Show*` flags as seen in siblings

## Style

- NEVER put a top-level `#` heading in content — the `title` front matter renders it. Sections start at `##`.
- Always provide alt text for images (`cover.alt` and `![alt](...)`).
- Use relative links for internal references.
- Code blocks use triple backticks with an explicit language.

## Deployment

`.github/workflows/deploy.yml`: every push to `main` builds with Hugo Extended (version pinned in the workflow) and pushes the generated site to the `gh-pages` branch, which GitHub Pages serves. Pull requests build but do not deploy. `public/` and `resources/` are gitignored.
