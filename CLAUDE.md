# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Jekyll Garden** site - a Jekyll-based static site for the "安卓程序设计与应用" (Android Programming and Application) course. It publishes Obsidian-compatible markdown notes as a website with wiki-style linking, full-text search, and backlinks.

## Development Commands

### Local Development

```bash
# Install dependencies
bundle install

# Serve locally (default: http://localhost:4000)
bundle exec jekyll serve

# Serve with live reload
bundle exec jekyll serve --livereload

# Build site to _site/
bundle exec jekyll build
```

### Docker Development

```bash
# Build and run with Docker Compose
docker-compose up

# Site available at http://localhost:4000
```

## Architecture

### Content Collections

Content is organized in `_collections/`:

- **`_collections/_lectures/`** - Lecture notes (Chinese language)
  - Permalink format: `/lectures/:name`
  - Layout: `Post`
  - Frontmatter: `title`, `date`, `permalink`, `show`

- **`_collections/_assignments/`** - Assignment files
  - Permalink format: `/assignments/:name`
  - Same layout and frontmatter as lectures

### Wiki-Style Linking System

The site implements Obsidian-compatible wiki links via client-side JavaScript in `_includes/Content.html`:

- **Note links**: `[[Note Title]]` or `[[Note Title|Display Text]]`
- **External links**: `[[Title::URL]]`
- **Image embeds**: `![[image.png]]` or `![[image.png|300x200]]` for dimensions
- **File links**: `[[filename.pdf]]` links to static files

The JavaScript processes these after DOM load, resolving titles against `availableNotes`, `availableImages`, and `availableFiles` arrays generated at build time.

### Search Implementation

- **Lunr.js** powers client-side search (`assets/js/Search.js`)
- Search data generated from `SearchData.json` (Jekyll template)
- Indexes both `lectures` and `assignments` collections
- Keyboard shortcut: `Shift+S` focuses search box
- Search results show highlighted excerpts with title/content matches

### Layout System

The main layout (`_layouts/Post.html`) conditionally renders:

- **Homepage** (`/`): Content + Search + Feed (when `preferences.homepage.enabled: false`)
- **Lecture/Assignment index**: List with search
- **Individual content**: Content + Backlinks (if enabled)
- **Static pages**: Content only

### Key Configuration

`_config.yml` controls features via `preferences`:

```yaml
preferences:
  search: { enabled: true }
  backlinks: { enabled: true }
  pagepreview: { enabled: true }
  wiki_style_link: { enabled: true }
```

Site deployed to subdirectory: `baseurl: "/apda"`

## Important File Locations

| Purpose | Location |
|---------|----------|
| Lecture content | `_collections/_lectures/` |
| Assignment content | `_collections/_assignments/` |
| Static pages | `pages/` |
| Layouts | `_layouts/` |
| Includes (components) | `_includes/` |
| CSS | `assets/css/style.css` |
| Search JavaScript | `assets/js/Search.js` |
| Wiki-link processor | `_includes/Content.html` |
| Search index template | `SearchData.json` |

## Content Guidelines

When creating or editing content:

1. Use YAML frontmatter with `title`, `date`, and `permalink`
2. Use wiki links `[[title]]` to connect related content
3. Store images in `assets/img/`
4. Store public files (PDFs, PPTs) in `assets/public/`
5. Use Chinese titles for lectures to match course context

## Dependencies

- **Ruby**: 3.1.1 (see Dockerfile)
- **Jekyll**: ~> 4.0.0
- **Key gems**: jekyll-feed, jekyll-sitemap, jekyll-tidy, kramdown-math-katex
- **Frontend**: Lunr.js (search), KaTeX (math), Inter font
