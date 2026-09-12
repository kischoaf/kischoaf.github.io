# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

Requires the Ruby version in `.ruby-version` (Homebrew Ruby: put `/opt/homebrew/opt/ruby/bin` first on `PATH`; the system Ruby is too old).

```bash
# Install dependencies
bundle install

# Serve locally with live reload
bundle exec jekyll serve

# Production build plus internal link/image check (what CI runs)
JEKYLL_ENV=production bundle exec jekyll build && bundle exec htmlproofer ./_site --disable-external
```

The site runs at `http://localhost:4000` by default. Pushes to `main` deploy through `.github/workflows/pages.yml`.

## Architecture

This is a Jekyll-based personal portfolio site for Kieran Schoaf. It uses no external CMS — all content is managed through YAML data files and Markdown.

**Content lives in three places:**
- `_data/` — YAML files that drive the main sections: `experience.yml`, `projects.yml`, `skills.yml`, `education.yml`, `homelab.yml`
- `projects/` — Individual Markdown files for project detail pages (linked via `page-name` field in `projects.yml`)
- `_posts/` — Field Notes posts (`layout: post`), listed at `/notes/` and previewed on the homepage

**Page structure:**
- `_layouts/default.html` wraps every page: header → navbar → `<main>` → footer → `reveal.html` (scroll reveal script)
- `index.html` (`layout: default`) composes the homepage from `_includes/`: hero (with `_data/stats.yml`) → experience → projects → credentials (skills, certs, education) → homelab → notes-preview (only once there are 3+ posts)
- Section titles use `_includes/section-header.html` (`title`, optional `lede`)
- Project detail pages use `layout: page`; Field Notes posts use `layout: post`; both build on `default`
- `_config.yml` holds all site-wide metadata (name, headline, bio, nav links, social handles) and the build `exclude` list

**Styling:**
- `assets/css/main.scss` `@use`s `_sass/all.scss`, which `@use`s each partial; use the Sass module system (`@use`), not `@import`
- Shared variables (colors, fonts, breakpoints) live in `_sass/_tokens.scss`; every partial starts with `@use 'tokens' as *;`
- `_sass/_base.scss` holds the reset, layout helpers (`.container`, `.section`), buttons (`.btn`), tags, and the `.reveal` animation
- Add `reveal` to an element's classes to fade it in once when it scrolls into view

**Assets:**
- Project thumbnails go in `assets/img/projects/` and are referenced by filename in `_data/projects.yml`
- Company/institution logos go in `assets/img/` and are referenced by filename in `experience.yml` / `education.yml`

## Adding a New Project

1. Add an entry to `_data/projects.yml` with `title`, `category`, `date`, `page-name`, `thumbnail`, and `desc`
2. Place the thumbnail image in `assets/img/projects/`
3. Create `projects/<page-name>.md` with front matter `title` and `layout: page`
