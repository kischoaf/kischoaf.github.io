# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Install dependencies (first time only)
bundle install --path vendor/bundle

# Serve locally with live reload
bundle exec jekyll serve
```

The site runs at `http://localhost:4000` by default.

## Architecture

This is a Jekyll-based personal portfolio site for Kieran Schoaf. It uses no external CMS — all content is managed through YAML data files and Markdown.

**Content lives in two places:**
- `_data/` — YAML files that drive the main sections: `projects.yml`, `experience.yml`, `education.yml`
- `projects/` — Individual Markdown files for project detail pages (linked via `page-name` field in `projects.yml`)

**Page structure:**
- `index.html` assembles the single-page portfolio by composing `_includes/` partials in order: navbar → hero → projects → experience → education → footer
- `_layouts/default.html` wraps standalone content pages (project detail pages use `layout: page` which maps to `_layouts/page.html`)
- `_config.yml` holds all site-wide metadata (name, bio, nav links, social handles)

**Styling:**
- Sass files in `_sass/` are compiled via `assets/css/all.scss`
- Each major section has its own `.scss` partial; inline `<style>` blocks also appear directly in some includes (e.g., `_includes/projects.html`)

**Assets:**
- Project thumbnails go in `assets/img/projects/` and are referenced by filename in `_data/projects.yml`
- Company/institution logos go in `assets/img/` and are referenced by filename in `experience.yml` / `education.yml`

## Adding a New Project

1. Add an entry to `_data/projects.yml` with `title`, `category`, `date`, `page-name`, `thumbnail`, and `desc`
2. Place the thumbnail image in `assets/img/projects/`
3. Create `projects/<page-name>.md` with front matter `title` and `layout: page`
