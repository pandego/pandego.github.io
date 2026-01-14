# AGENTS.md

## Scope

This guidance applies to the entire repository.
It is intended for coding agents working on this MkDocs Material site.
Follow repo conventions, keep changes minimal, and prefer clarity over novelty.

## Repo Overview

- Static site built with MkDocs Material and Markdown content.
- Core configuration lives in `mkdocs.yml`.
- Content lives in `docs/` with sections for blog, portfolio, and pages.
- Custom styling lives in `docs/stylesheets/`.
- JavaScript (if any) lives in `docs/javascripts/`.
- Build output is generated in `site/` (ignored by git).

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
├── stylesheets/               # Custom CSS
│   ├── extra.css              # Theme customization and colors
│   └── hero.css               # Profile image and grid layouts
├── javascripts/               # Custom JavaScript
└── robots.txt                 # SEO configuration
```

### Configuration Files

- `mkdocs.yml` holds navigation, theme, plugins, and Markdown extensions.
- `pyproject.toml` stores Python dependencies for MkDocs.
- `Dockerfile` configures the container image for local development.

## Commands

### Install Dependencies (macOS)

```bash
uv sync
brew install cairo freetype libffi libjpeg libpng zlib pngquant
```

- Add Python dependencies with `uv add <package>`.
- Do not use `pip install` or edit `pyproject.toml` by hand.

### Local Development Server

```bash
# Apple Silicon
export DYLD_FALLBACK_LIBRARY_PATH=/opt/homebrew/lib
mkdocs serve

# Intel macOS
mkdocs serve
```

### Docker Development

```bash
chmod +x start_server.sh
./start_server.sh

# Manual
docker build -t mkdocs-site .
docker run --rm -it -p 8000:8000 -v $(pwd):/docs mkdocs-site
```

- Development server runs at `http://localhost:8000` with live reload.

### Build

```bash
mkdocs build
```

### Deploy

```bash
mkdocs gh-deploy --force
```

### CI / GitHub Actions

- Automated deployment runs from `.github/workflows/ci.yml` on pushes to `main` or `master`.
- CI installs dependencies with `uv` and runs `mkdocs gh-deploy`.

### Lint / Tests

- No dedicated linting or automated test suite is configured.
- There is no single-test runner in this repo.
- Use `mkdocs build` as a validation step for content and config.
- If you add scripts or automation, document how to run them.

## Theme and Branding

- Light theme primary color `#07091B`, accent `#6248ff`.
- Dark theme background `#1A1C38`, accent `#6248ff`.
- Secondary gradient colors: `#AC50EF`, `#7059FB`, `#2ECFF6`.
- Social card images use a dark background via the `social` plugin.

## Content Conventions

### Markdown Files (`docs/**/*.md`)

- Use YAML frontmatter for blog posts (see `docs/blog/posts/*.md`).
- Frontmatter keys are lower-case with hyphens or underscores.
- Typical post frontmatter includes: `date`, `authors`, `categories`, `description`.
- Use a single top-level `#` heading per page.
- Use sentence case for headings when possible.
- Keep line lengths readable; wrap long paragraphs where helpful.
- Use fenced code blocks with language tags (e.g., ```bash, ```yaml).
- Use `<!-- more -->` to define blog summaries when needed.
- Prefer relative links within `docs/`.
- Images should live in `docs/assets/` and be referenced by relative path.
- Prefer SVG for diagrams and logos.

### YAML (`mkdocs.yml`, frontmatter blocks)

- Use two spaces for indentation.
- Keep keys sorted within the same semantic group when feasible.
- Use consistent quoting for hex colors (existing config uses quotes).
- Avoid inline comments unless they add real clarity.
- Validate YAML after edits by running `mkdocs build`.

### CSS (`docs/stylesheets/*.css`)

- Follow existing formatting: two-space indent, one property per line.
- Use kebab-case for class names (`.profile-image`, `.grid-container`).
- Keep selectors as specific as necessary, not more.
- Group related styles and add a blank line between sections.
- Avoid `!important` unless already used for a specific override.
- Prefer CSS variables and theme tokens already defined in `extra.css`.

### JavaScript (`docs/javascripts/*.js`)

- Use plain JavaScript; no TypeScript configured.
- Prefer small, focused modules with clear function names.
- Use `const` and `let` appropriately; avoid `var`.
- Keep browser-only code behind DOM-ready hooks when needed.
- Avoid new dependencies unless explicitly requested.

## Naming Conventions

- File names for blog posts: `docs/blog/posts/post-title.md`.
- File names for portfolio projects: `docs/portfolio/projects/project-name.md`.
- Asset filenames should be descriptive and lowercase.
- Use kebab-case for new CSS classes and file names.
- Use snake_case for frontmatter keys when matching existing usage.

## Content Workflows

### Adding New Case Studies

1. Create markdown file in `docs/portfolio/projects/project-name.md`.
2. Add to navigation in `mkdocs.yml` under `Case Studies > Projects`.
3. Include architecture diagrams in SVG format in `docs/assets/`.
4. Reference existing project templates for structure.

### Adding Blog Posts

1. Create markdown file in `docs/blog/posts/post-title.md`.
2. Include frontmatter with author, date, categories.
3. Author must be defined in `docs/blog/.authors.yml`.

### Adding Images/Diagrams

- Store in `docs/assets/`.
- Prefer SVG for diagrams (better scaling, smaller size).
- Use descriptive filenames and relative links.

## Error Handling and Validation

- For build errors, fix the underlying markdown or config issue.
- Avoid suppressing errors or adding placeholder content.
- Validate any new links or assets paths.
- Ensure new nav items are added in `mkdocs.yml`.

## Deployment Notes

- Python 3.11+ required (see `pyproject.toml`).
- Social card generation needs imaging dependencies (Cairo, etc.).
- Docker provides a consistent environment across platforms.
- Site URL is configured as `https://miguelmirandadias.com`.
- Social links point to LinkedIn, GitHub, YouTube, and Datavengers.

## Formatting Guidance

- Keep Markdown readable with consistent spacing between sections.
- Use blank lines before headings and code blocks.
- Keep unordered lists consistent (`-` style preferred).
- Use consistent indentation in nested lists.

## Content Quality

- Maintain a professional, concise tone consistent with existing pages.
- Avoid placeholders; add real content or leave unchanged.
- Preserve existing branding and theme colors unless instructed.

## Cursor / Copilot Rules

- No Cursor rules found in `.cursor/rules/` or `.cursorrules`.
- No Copilot instructions found in `.github/copilot-instructions.md`.

## Notes for Agents

- Do not add new tooling or dependencies without request.
- Do not commit or push unless explicitly asked.
- Keep changes focused on the user’s request.
