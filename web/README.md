# zkBook Web Version

This is the online version of the book built with [mdBook](https://rust-lang.github.io/mdBook/).

## Development

### Prerequisites

- [mdBook](https://rust-lang.github.io/mdBook/guide/installation.html)

```bash
cargo install mdbook
```

### Build

```bash
cd web
mdbook build
```

The output will be in `web/book/`.

### Serve locally

```bash
cd web
mdbook serve
```

Then open http://localhost:3000

### Serve with live reload

```bash
cd web
mdbook serve --open
```

## Deployment

The `book/` directory contains static HTML files that can be deployed to any static hosting:

- GitHub Pages
- Netlify
- Vercel
- Cloudflare Pages

### GitHub Pages

Add a GitHub Action in `.github/workflows/deploy.yml`:

```yaml
name: Deploy mdBook

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup mdBook
        uses: peaceiris/actions-mdbook@v2
        with:
          mdbook-version: 'latest'
      - name: Build
        run: mdbook build web
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./web/book
```

## Syncing with Source

The chapter files in `web/src/` are copies from `markdown/chapters/`. To sync:

```bash
# From project root
cp "markdown/chapters/01 - The Trust Problem.md" "web/src/01-the-trust-problem.md"
# ... etc
```

Consider creating a sync script if chapters are updated frequently.
