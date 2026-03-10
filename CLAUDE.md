# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **MkDocs** static site using the **Material theme**, deployed to GitHub Pages at `https://somelamecode.github.io/notes/`. Content lives in `docs/` as Markdown files; `site/` is generated output and never committed.

## Common Commands

All commands assume the virtual environment is activated and run from the repo root (`C:\GitFolder\notes`).

**Activate the virtual environment (Git Bash):**
```bash
source venv/Scripts/activate
```

**Local dev server** (use PowerShell/CMD on Windows for reliable live reload, not Git Bash):
```bash
mkdocs serve
# Open: http://127.0.0.1:8000/notes/
```

**Deploy to GitHub Pages:**
```bash
mkdocs gh-deploy
```

**Install dependencies:**
```bash
pip install mkdocs mkdocs-material mkdocs-mermaid2-plugin mkdocs-git-revision-date-localized-plugin
```

## Architecture

- `mkdocs.yml` — site config: theme, plugins, nav, markdown extensions
- `docs/` — all Markdown content (source of truth)
  - `index.md` — homepage
  - `articles/` — published articles and how-to guides
- `venv/` — Python virtual environment (not committed)
- `site/` — build output (not committed, managed by `mkdocs gh-deploy`)

## Navigation

The `nav:` section in `mkdocs.yml` controls the sidebar. Nav paths are **relative to `docs/`**. Pages not listed in `nav` still build but won't appear in the sidebar. Always verify nav entries match actual file paths before deploying.

## Plugins in Use

- `mermaid2` — Mermaid diagram support
- `git-revision-date-localized` — shows last-modified dates on pages
- `pymdownx.superfences` with custom `mermaid` fence — enables fenced Mermaid blocks
- `pymdownx.tabbed` — content tabs
- `admonition` + `pymdownx.details` — callout boxes

## Windows-Specific Note

MkDocs auto-reload does **not** reliably work from Git Bash on Windows. Run `mkdocs serve` from PowerShell or CMD for live reload to work correctly.

## Deployment Workflow

```bash
git add .
git commit -m "Update notes"
git push origin main
mkdocs gh-deploy
```

`mkdocs gh-deploy` builds the site and pushes HTML to the `gh-pages` branch automatically.
