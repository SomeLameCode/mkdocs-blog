---
title: MkDocs Setup — Windows + VS Code + Material Theme
description: Step-by-step guide to setting up MkDocs with the Material theme on Windows, with GitHub Pages deployment.
date: 2026-01-19
tags:
  - mkdocs
  - material-theme
  - windows
  - github-pages
  - setup
status: published
---

# MkDocs Setup (Windows + VS Code + Material Theme)

This document describes the exact steps used to set up **MkDocs with the Material theme** on **Windows**, using **VS Code**, a **Python virtual environment**, and deployment to **GitHub Pages**.

It also documents common Windows-specific issues and their solutions.

---

## 1️⃣ Project Structure

We use a standard MkDocs layout:

```
notes/
├── mkdocs.yml
├── .gitignore
├── requirements.txt
├── venv/
└── docs/
    ├── index.md
    ├── articles/
    ├── projects/
    ├── code/
    ├── notebooks/
    └── assets/
```

- `notes/` → Git repository root  
- `docs/` → All public Markdown content  
- `site/` → Generated HTML (ignored by Git)

---

## 2️⃣ Create and Activate a Virtual Environment

From the `notes/` folder:

```bash
python -m venv venv
```

Activate it:

### PowerShell / CMD
```powershell
venv\Scriptsctivate
```

### Git Bash
```bash
source venv/Scripts/activate
```

---

## 3️⃣ Install MkDocs and Plugins

Install MkDocs and required plugins:

```bash
pip install mkdocs mkdocs-material mkdocs-mermaid2-plugin mkdocs-git-revision-date-localized-plugin
```

Verify installation:

```bash
mkdocs --version
```

---

## 4️⃣ Create `mkdocs.yml`

Place `mkdocs.yml` **in the root (`notes/`)**, not inside `docs/`.

Minimal working configuration:

```yaml
site_name: Florin Neagu Projects & Notes
site_url: https://SomeLameCode.github.io/notes/
site_description: Technical notes, projects, and articles
site_author: Florin Neagu
repo_url: https://github.com/SomeLameCode/notes

theme:
  name: material
  palette:
    primary: indigo
    accent: indigo
  features:
    - navigation.tabs
    - navigation.sections
    - content.code.copy

plugins:
  - search
  - mermaid2
  - git-revision-date-localized:
      fallback_to_build_date: true

docs_dir: docs
site_dir: site
```

---

## 5️⃣ Navigation Strategy (Flexible)

Do **not** reference files that don’t exist yet.

Start minimal:

```yaml
nav:
  - Home: index.md
```

Add sections later as content matures.

MkDocs will still build pages **not listed in `nav`**.

---

## 6️⃣ Run the Local Development Server

From `notes/`:

```bash
mkdocs serve
```

Open in browser:

```
http://127.0.0.1:8000/notes/
```

---

## 7️⃣ Windows Auto-Reload Caveat (Important)

On Windows, **MkDocs auto-reload may NOT work** when run from **Git Bash**.

### Recommended solution
Run MkDocs from **PowerShell or CMD** instead of Git Bash:

```powershell
cd C:\GitFolder\Notes
venv\Scripts\activate
mkdocs serve
```

This reliably enables live reload when saving `.md` files.

> Even with `watchdog` installed, Git Bash may not emit filesystem events correctly.

---

## 8️⃣ VS Code Recommendations

Install:
- **YAML Language Support by Red Hat**

Optional VS Code settings (JSON):

```json
"files.atomicSave": false,
"files.useExperimentalFileWatcher": false
```

This improves compatibility with file watchers.

---

## 9️⃣ `.gitignore`

Create `.gitignore` in `notes/`:

```gitignore
# MkDocs build output
site/

# Virtual environment
venv/

# Python cache
__pycache__/
*.pyc

# VS Code
.vscode/

# OS files
Thumbs.db
.DS_Store
```

---

## 10️⃣ Initialize Git and Push Source Content

From `notes/`:

```bash
git init
git remote add origin https://github.com/SomeLameCode/notes.git
git branch -M main
git add .
git commit -m "Initial MkDocs setup"
git push -u origin main
```

This pushes:
- Markdown content  
- `mkdocs.yml`  
- configuration files  

It does **NOT** push the generated site.

---

## 11️⃣ Deploy to GitHub Pages

MkDocs handles deployment automatically:

```bash
mkdocs gh-deploy
```

This:
1. Builds the site  
2. Pushes it to the `gh-pages` branch  
3. Publishes it via GitHub Pages  

Your site becomes available at:

```
https://SomeLameCode.github.io/notes/
```

---

## 12️⃣ Ongoing Workflow

When you make changes:

```bash
git add .
git commit -m "Update notes"
git push origin main
mkdocs gh-deploy
```

> You don’t need to commit the `site/` folder; `gh-deploy` handles it automatically.

---

## Key Takeaways

- `docs/` = content  
- `mkdocs.yml` = configuration  
- `site/` = generated output (never commit)  
- Use **PowerShell/CMD** for reliable live reload on Windows  
- Keep `nav` minimal early, expand later  

This setup is stable, boring, and production-ready.
