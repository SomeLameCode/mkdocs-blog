---
title: MkDocs — Publish to GitHub Pages
description: Cheatsheet for the local update and GitHub Pages deployment workflow for an MkDocs site.
date: 2026-01-19
tags:
  - mkdocs
  - github-pages
  - deployment
  - cheatsheet
status: published
---

## MkDocks - Local Update

### 1️⃣ Move the files physically
From your repo root (`notes/`), using Git Bash or PowerShell:
```bash
# Git Bash
mv docs/code/code-example.md docs/articles/
mv docs/code/content-tabs.md docs/articles/
mv docs/code/admonitions.md docs/articles/
mv docs/code/diagram-examples.md docs/articles/
```

### 2️⃣Update your `mkdocs.yml`
```yml
nav:
  - Home: index.md
  - Articles:
      - Code Simple Example: articles/code-example.md
      - Content Tabs Example: articles/content-tabs.md
      - Admonitions Example: articles/admonitions.md
      - Diagram Example: articles/diagram-examples.md
```
**Important:** the path is **relative to `docs/`**, so `articles/filename.md` is correct.

### 3️⃣Commit the changes with Git
```bash
git add docs/articles/*.md
git rm docs/code/*.md
git commit -m "Move code examples from 'code' to 'articles'"
```
`git rm` removes them from the old location in Git tracking, not your filesystem (since you already moved them).

### 4️⃣Test locally
```bash
mkdocs serve
```
Open the browser at `http://127.0.0.1:8000/notes/` and check that all four pages appear under **Articles**.

## MkDocs GitHub Deployment Cheatsheet

This file consolidates all the commands needed to push your MkDocs site from your local `notes/` folder to **GitHub Pages**.

---
### 1️⃣ Check Status and Commit Local Changes
```bash
# Check which files have changed
git status

# Stage all changes
git add .

# Commit changes with a message
git commit -m "Update articles and navigation"
```

---
### 2️⃣ Push Source Content to GitHub
```bash
# Push your Markdown content and mkdocs.yml to the main branch
git push origin main
```

> Note: Do not push the `site/` folder manually; it will be handled by MkDocs deployment.

---
### 3️⃣ Deploy Site to GitHub Pages
```bash
# Build and deploy your MkDocs site to the gh-pages branch
mkdocs gh-deploy
```

What this does:
1. Builds the site into the `site/` folder  
2. Creates/updates the `gh-pages` branch in your GitHub repo  
3. Pushes the generated HTML content to GitHub Pages  
4. Publishes it at: `https://YourGitHubUsername.github.io/notes/`

---
### 4️⃣ Updating the Site Later
When you make changes to content or configuration:
```bash
git add .
git commit -m "Update notes/articles"
git push origin main
mkdocs gh-deploy
```

> This workflow avoids manual handling of the `site/` folder.

---
### 5️⃣ Notes / Tips

- Always run `mkdocs serve` locally first to verify changes before deploying.  
- Ensure `mkdocs.yml` has correct `nav` paths and matches folder/file names exactly.  
- Keep `site/` in `.gitignore` to avoid accidentally committing it.
