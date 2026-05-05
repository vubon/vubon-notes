# Project Guidelines

## Overview

This is a Hugo static blog hosted on GitHub Pages at https://vubon.me/. All content lives under `content/posts/`. The theme is `typo` (located in `themes/typo/`).

## Content Conventions

### Front Matter

Every post must include these fields:

```yaml
---
title: "Your Post Title"
date: 2026-05-04T10:00:00Z
draft: false          # true while writing, false when ready to publish
tags:
  - "Tag One"
categories:
  - "Category"
description: "One sentence summary shown in previews."
---
```

For articles that belong to a series, also include:

```yaml
series:
  - "Series Name"
```

### File Naming

Use lowercase kebab-case slugs: `my-post-title.md`. Place all posts in `content/posts/`.

### Writing Style

- Conversational and direct — write like you're explaining to a colleague
- Lead with the real-world problem before introducing the concept
- Use ASCII diagrams (not images) for architecture/flow illustrations
- Use fenced code blocks with a language identifier (` ```python `, ` ```go `, ` ```sql `)
- Strong answers include trade-offs, not just "how it works"

### Series

The "System Design Interview Prep" series uses `series: ["System Design Interview Prep"]` in front matter. Only add a topic to the table in `content/posts/system-design-interview-series.md` **after** the article is published (i.e., `draft: false`).

## Taxonomies

Three taxonomies are configured in `hugo.toml`: `tags`, `categories`, `series`.

## Build & Preview

```bash
# Preview (including drafts)
hugo server -D

# Production build
hugo --gc --minify
```

Hugo version pinned at **0.138.0** (extended) — see `.github/workflows/deploy.yml`.

## Deploy

Pushing to `main` triggers the GitHub Actions workflow which builds and deploys to the `gh-pages` branch. Do not manually edit the `public/` directory.

To publish changes:

```bash
# 1. Build first to verify no errors
hugo --gc --minify

# 2. Stage and commit
git add .
git commit -m "post: add <slug>"

# 3. Push to main — this triggers the deploy workflow automatically
git push origin main
```

The GitHub Actions workflow (`.github/workflows/deploy.yml`) handles the rest — no manual deploy step needed.

## Static Assets

- Images → `static/images/`
- Referenced in markdown as `/images/filename.png`
- Custom JS → `static/js/`
