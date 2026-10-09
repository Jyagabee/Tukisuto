# CLAUDE.md

This file provides guidance to Claude Code when working with this Hugo-based blog project.

## Project Overview

**じゃがびぃのサイト** (Jyagabee's Site) - A Japanese tech blog focused on smartphones, hardware, and technology topics, built with Hugo and deployed on Cloudflare Pages.

- **Framework**: Hugo v0.155.3 (extended)
- **Theme**: [hugo-blog-awesome](https://github.com/hugo-sid/hugo-blog-awesome)
- **Language**: Japanese (ja)
- **Deployment**: Cloudflare Pages
- **Analytics**: Google Analytics (G-3FGTMKYNHZ)

## Directory Structure

```
.
├── archetypes/          # Content templates
│   ├── default.md       # Default post template
│   └── weekly/          # Weekly report template
├── assets/              # SCSS, JS, and other assets
├── config/              # Hugo configuration files
│   └── _default/
│       ├── hugo.toml    # Main Hugo configuration
│       ├── params.toml  # Theme parameters
│       ├── languages.ja.toml  # Japanese language settings
│       └── menus.ja.toml      # Navigation menus
├── content/             # Markdown content files
│   ├── posts/           # Blog posts (tech articles)
│   ├── weekly/          # Weekly reports
│   └── about/           # About page
├── data/                # Data files (JSON, YAML)
├── layouts/             # Custom HTML templates (override theme)
├── static/              # Static files (images, fonts, etc.)
├── Templates/           # Content templates for authors
│   ├── Blowfish_article.md
│   └── WeeklyReport.md
├── themes/              # Hugo themes (git submodule)
│   └── hugo-blog-awesome/  # hugo-blog-awesome theme
└── public/              # Generated site (build output)
```

## Build & Deployment

### Local Development

```bash
# Start Hugo development server
hugo server -D

# Build for production
hugo --gc --minify
```

### Cloudflare Pages Configuration

**IMPORTANT**: Cloudflare Pages must be configured with the following settings:

| Setting | Value |
|---------|-------|
| Framework preset | `Hugo` |
| Build command | `hugo --gc --minify` |
| Build output directory | `public` |
| Root directory | `/` (project root) |

**Environment Variables**:
```
HUGO_VERSION = 0.155.3
```

### Git Submodules

The hugo-blog-awesome theme is managed as a git submodule:

```bash
# Initialize submodules
git submodule update --init --recursive

# Update theme to latest version
git submodule update --remote themes/hugo-blog-awesome
```

## Content Management

### Creating New Posts

```bash
# Create a new blog post
hugo new posts/my-post-name/index.md

# Create a new weekly report
hugo new weekly/YYYYMMDD/index.md
```

### Content Structure

All content files should include frontmatter:

```yaml
---
title: "記事タイトル"
date: 2026-02-13T12:00:00+09:00
draft: false
tags: ["tag1", "tag2"]
categories: ["category"]
description: "記事の説明"
---
```

### Content Types

1. **Posts** (`content/posts/`): Main blog articles about technology, smartphones, hardware
2. **Weekly** (`content/weekly/`): Weekly reports and updates
3. **About** (`content/about/`): About page

## Theme Configuration

The hugo-blog-awesome theme is configured in `config/_default/params.toml`.

### Customization

- Custom layouts: Place in `layouts/` to override theme templates
- Custom shortcodes: Place in `layouts/shortcodes/`
- Custom styles: Write them in `assets/sass/_custom.scss`. The theme's `main.scss`
  ends with `@import "custom"`, which resolves to `sass/_custom.scss`, and a file
  of the same path in the project overrides the theme's empty one. Do not edit
  files under `themes/`.
- Layout widths (`$narrow-size` 720px, `$wide-size` 890px) are hardcoded in the
  theme's SCSS and cannot be changed by variables, so override `.wrapper` /
  `.navbar` in `_custom.scss` instead.
- `assets/scss/custom.scss` (note: `scss`, not `sass`) is NOT imported by the
  theme, so it has no effect. `assets/css/extended/` is a PaperMod convention
  and is not loaded either.

## Common Tasks

### Adding a New Post

1. Create post: `hugo new posts/post-name/index.md`
2. Edit content in `content/posts/post-name/index.md`
3. Add images to the same directory
4. Set `draft: false` when ready to publish
5. Build and deploy

### Updating Configuration

- **Site settings**: Edit `config/_default/hugo.toml`
- **Theme settings**: Edit `config/_default/params.toml`
- **Menu items**: Edit `config/_default/menus.ja.toml`
- **Language settings**: Edit `config/_default/languages.ja.toml`

### Theme Updates

```bash
git submodule update --remote themes/hugo-blog-awesome
git add themes/hugo-blog-awesome
git commit -m "chore: Update hugo-blog-awesome theme"
```

## Troubleshooting

### Build Fails on Cloudflare Pages

**Symptom**: `npm ERR! enoent ENOENT: no such file or directory, open '/opt/buildhome/repo/package.json'`

**Cause**: Cloudflare Pages incorrectly detects the project as a Node.js project due to `package-lock.json`

**Solution**:
1. Ensure Framework preset is set to `Hugo` (not auto-detect)
2. Verify environment variable `HUGO_VERSION` is set
3. Consider removing `package-lock.json` if not needed
4. Ensure git submodules are properly initialized

### Theme Not Found

**Symptom**: `Error: module "hugo-blog-awesome" not found`

**Solution**:
```bash
git submodule update --init --recursive
```

### Changes Not Reflected

**Symptom**: Changes don't appear on the site

**Solution**:
1. Check if `draft: true` in frontmatter
2. Clear Hugo cache: `hugo --gc`
3. Restart development server

## File Conventions

- **Configuration files**: TOML format in `config/_default/`
- **Content files**: Markdown with YAML frontmatter
- **Image files**: Place with content (e.g., `posts/my-post/image.jpg`)
- **Static assets**: Place in `static/` (available at site root)

## Important Notes

- Always initialize git submodules before building
- Use `hugo --gc --minify` for production builds
- Keep `draft: false` for published content
- Images should be optimized before adding
- All user-facing content should be in Japanese
- Configuration and technical documentation should be in English

## Communication

- Respond to users in Japanese for all communication
- Technical documentation and CLAUDE.md should remain in English
- Code comments can be in English or Japanese as appropriate
