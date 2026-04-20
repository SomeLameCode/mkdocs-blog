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
  - `projects/` — project documentation and case studies
  - `code/` — code snippets and utilities
  - `notebooks/` — Jupyter notebooks
  - `tags.md` — auto-generated tag index (do not edit manually)
- `_blog/` — governance folder (never published; see Governance section below)
- `temp/` — scratch space for notes and ideas (never published)
- `venv/` — Python virtual environment (not committed)
- `site/` — build output (not committed, managed by `mkdocs gh-deploy`)

## Navigation

The `nav:` section in `mkdocs.yml` controls the sidebar. Nav paths are **relative to `docs/`**. Pages not listed in `nav` still build but won't appear in the sidebar. Always verify nav entries match actual file paths before deploying.

## Plugins in Use

- `tags` — tag-based navigation; tag index rendered at `docs/tags.md`
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

## Working Directory Convention

This repo (`C:\GitFolder\notes`) is the home base for all work done in Claude Code sessions.

- **Reading** from other folders under `C:\GitFolder\` is fine — source projects, reference material, other repos.
- **Writing new files** always happens here, under `C:\GitFolder\notes\`. Do not create files in other project folders.
- **Scratch and synthesis work** goes in `temp/` (never published). Copy relevant material from other project folders into `temp/` before working on it.

## Session Workflow

**At session start** — run `/start` to present the session briefing (last updated, in progress, next actions). If the user has not run it, suggest it.

**At session end** — run `/blog-update` to update all governance files. Commit all work first; the command uses git history as its primary evidence source.

## Governance

The `_blog/` folder is the governance layer for this site. It is never published.

| File | Purpose |
|---|---|
| `_blog/session-state.md` | Compact snapshot — read first each session |
| `_blog/content-backlog.md` | Planned content and site improvements with status tracking |
| `_blog/change-log.md` | Log of every publish or structural change |
| `_blog/issue-log.md` | Known issues — broken links, outdated content |
| `_blog/decisions.md` | Site decisions with rationale |

**ID scheme:**

| Prefix | Used for |
|---|---|
| `ART-NNN` | Planned articles |
| `PRJ-NNN` | Planned project pages |
| `SI-NNN` | Site improvement tasks |
| `D-NNN` | Decisions |

**Status markers** (used in `content-backlog.md`):

```
[ ] = planned   [x] = done   [-] = deferred   [/] = descoped
```

## Content Conventions

Every content page in `docs/` (except index pages and placeholders) must have YAML frontmatter:

```yaml
---
title: "Page Title"
description: "One-sentence summary."
date: YYYY-MM-DD
tags:
  - tag-one
  - tag-two
status: draft | review | published | outdated
---
```

**Status values:**
- `draft` — work in progress, not ready to publish
- `review` — written but needs a pass before publishing
- `published` — live and current
- `outdated` — live but content needs updating (add to issue log)

**Content templates** live in `C:\GitFolder\00_templates\3_template_blog\`:
- `docs/articles/_template-article.md` — for how-to guides and tutorials
- `docs/projects/_template-project.md` — for project documentation and case studies
