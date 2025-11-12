# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a portfolio website built with MkDocs Material - a static site generator that uses Markdown for content and YAML for configuration. The site showcases case studies, blog posts, and professional information for Miguel Miranda Dias.

## Development Commands

### Local Development (MacOS)

```bash
# Install dependencies (first time setup)
pip install "mkdocs-material[imaging]"
brew install cairo freetype libffi libjpeg libpng zlib pngquant

# Start development server (MacOS M1/M2)
export DYLD_FALLBACK_LIBRARY_PATH=/opt/homebrew/lib
mkdocs serve

# Start development server (Intel Mac)
mkdocs serve
```

### Docker Development (All platforms)

```bash
# Make script executable (first time only)
chmod +x start_server.sh

# Start server
./start_server.sh

# Manual Docker commands
docker build -t mkdocs-site .
docker run --rm -it -p 8000:8000 -v $(pwd):/docs mkdocs-site
```

The development server runs at `http://localhost:8000` with live reload.

### Build and Deploy

```bash
# Build static site (outputs to site/ directory)
mkdocs build

# Deploy to GitHub Pages (manual)
mkdocs gh-deploy --force
```

Automated deployment via GitHub Actions is configured in `.github/workflows/ci.yml` and triggers on pushes to `main` or `master` branches.

## Project Architecture

### Content Organization

```
docs/
├── index.md                    # Homepage with profile and introduction
├── blog/                       # Blog posts
│   ├── .authors.yml           # Author metadata for blog
│   ├── index.md               # Blog landing page
│   └── posts/                 # Individual blog posts
├── portfolio/                  # Case studies and projects
│   ├── index.md               # Portfolio overview
│   └── projects/              # Individual project case studies
├── datavengers/               # Company information
├── youtube/                   # YouTube content page
├── assets/                    # Images, diagrams, and media
│   ├── @miguelmirandadias.jpg # Profile photo
│   ├── favicon.svg            # Site icon
│   └── *.svg, *.png           # Project diagrams and images
├── stylesheets/               # Custom CSS
│   ├── extra.css              # Theme customization and colors
│   └── hero.css               # Profile image and grid layouts
├── javascripts/               # Custom JavaScript
└── robots.txt                 # SEO configuration
```

### Configuration Files

- **mkdocs.yml**: Main site configuration including:
  - Navigation structure (`nav` section)
  - Theme settings (Material theme with custom colors)
  - Plugins: search, social cards, blog
  - Markdown extensions for enhanced formatting
  - Social links and footer configuration

- **pyproject.toml**: Python project dependencies (mkdocs-material[imaging])

- **Dockerfile**: Container configuration with required imaging libraries

### Theme Customization

The site uses custom branding with specific color schemes:

- **Light theme**: Primary `#07091B`, Accent `#6248ff`
- **Dark theme**: Background `#1A1C38`, Accent `#6248ff`
- **Secondary gradient colors**: `#AC50EF`, `#7059FB`, `#2ECFF6`

Custom CSS in `docs/stylesheets/`:
- `extra.css`: Color variables, button styles, footer, testimonials
- `hero.css`: Profile image grid layout with responsive breakpoints

### Social Card Generation

The social plugin automatically generates preview cards for social media sharing. Images use dark background (`#07091B`) and require the imaging dependencies (Cairo, etc.).

### Blog System

Blog posts live in `docs/blog/posts/` with:
- Author metadata defined in `docs/blog/.authors.yml`
- Category support enabled
- Post filenames should follow: `docs/blog/posts/post-title.md`

### Navigation Structure

Navigation is defined in `mkdocs.yml` under the `nav` key with tabs for:
- Home (About)
- Case Studies (Portfolio with project listings)
- Blog
- Datavengers™
- YouTube

## Content Guidelines

### Adding New Case Studies

1. Create markdown file in `docs/portfolio/projects/project-name.md`
2. Add to navigation in `mkdocs.yml` under `Case Studies > Projects`
3. Include architecture diagrams in SVG format in `docs/assets/`
4. Reference template structure from existing projects

### Adding Blog Posts

1. Create markdown file in `docs/blog/posts/post-title.md`
2. Include frontmatter with author, date, categories
3. Author must be defined in `docs/blog/.authors.yml`

### Adding Images/Diagrams

- Store in `docs/assets/`
- Prefer SVG for diagrams (better scaling, smaller file size)
- Use descriptive filenames
- Reference in markdown: `![Alt text](assets/image-name.svg)`

## Deployment

The site deploys automatically to GitHub Pages via GitHub Actions when pushing to `main` or `master`. The workflow:

1. Checks out code
2. Sets up Python 3.x
3. Installs dependencies using `uv` (fast Python package installer)
4. Builds and deploys with `mkdocs gh-deploy`

Custom domain is configured via `docs/CNAME` file.

## Important Notes

- Python 3.11+ required
- MkDocs Material with imaging support is essential for social card generation
- MacOS users need system libraries via Homebrew for image processing
- Docker provides consistent environment across platforms
- Site URL is configured as `https://miguelmirandadias.com`
- Social links point to LinkedIn, GitHub, YouTube, and Datavengers
