# watsonx Orchestrate Bible

Authoritative technical reference for IBM watsonx Orchestrate — built with [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) and deployed to GitHub Pages.

## Prerequisites

- Python 3.12 or later
- pip

## Install

```bash
pip install -r requirements.txt
```

## Local preview

```bash
mkdocs serve
```

Open [http://127.0.0.1:8000](http://127.0.0.1:8000) in your browser. The server reloads automatically on file changes.

## Build

```bash
mkdocs build
```

The static site is written to `site/`. Check `site/index.html` to verify the output.

## Deploy

Deployment is handled automatically by GitHub Actions. Every push to the `main` branch triggers `.github/workflows/deploy.yml`, which runs `mkdocs gh-deploy --force` and publishes the site to the `gh-pages` branch.

To trigger a manual deploy without a commit, use the **Run workflow** button in the Actions tab.
