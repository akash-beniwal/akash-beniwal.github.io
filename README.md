# akash-beniwal.github.io

Personal portfolio and blog, built with [Jekyll](https://jekyllrb.com/).

## Running locally

```bash
# one-time setup (uses rbenv-managed Ruby 3.4.10, see .ruby-version)
bundle install

# serve with live reload at http://localhost:4000
bundle exec jekyll serve --livereload
```

## Structure

- `_layouts/`, `_includes/`: page templates and shared partials (header, footer, head)
- `_posts/`: blog posts
- `assets/css/main.scss`: all styling (light/dark via `prefers-color-scheme`)
- `index.html`: home page, doubles as the Work page (all projects)
- `about.md`: About page (bio, quick facts, contact links, resume link)
- `blog.md`: Writing index page
- `.github/workflows/pages.yml`: builds and deploys to GitHub Pages on push to `main`

Nav is intentionally 3 tabs: Work (home), Writing (blog), About.

## Deployment

GitHub Pages is configured to build via the Actions workflow above rather than the legacy
Jekyll-on-Pages pipeline, so any Jekyll version/plugin in the `Gemfile` is supported. In the repo
settings, **Pages → Build and deployment → Source** must be set to **GitHub Actions**.
