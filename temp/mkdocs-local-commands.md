# MkDocs — Local Dev & Deploy Cheatsheet

Working directory: `C:\GitFolder\notes`

---

## 1. Activate the virtual environment

Run this once at the start of each session, before any `mkdocs` command.

**Git Bash:**
```bash
source venv/Scripts/activate
```

**PowerShell / CMD:**
```powershell
venv\Scripts\activate
```

You'll see `(venv)` appear at the start of your prompt when it's active.

---

## 2. Preview locally

> Use **PowerShell or CMD** — live reload does not work reliably from Git Bash on Windows.

```powershell
mkdocs serve
```

Open in browser: [http://127.0.0.1:8000/notes/](http://127.0.0.1:8000/notes/)

The site auto-reloads when you save a file. Press `Ctrl+C` to stop.

---

## 3. Publish to GitHub Pages

```bash
git add .
git commit -m "your message"
git push origin main
mkdocs gh-deploy
```

`mkdocs gh-deploy` builds the site and pushes the HTML to the `gh-pages` branch automatically.
Live site: [https://somelamecode.github.io/notes/](https://somelamecode.github.io/notes/)

---

## 4. Install / reinstall dependencies

```bash
pip install -r requirements.txt
```

Or manually:
```bash
pip install mkdocs mkdocs-material mkdocs-mermaid2-plugin mkdocs-git-revision-date-localized-plugin
```

---

## Quick reference

| Task | Command |
|---|---|
| Activate venv (Git Bash) | `source venv/Scripts/activate` |
| Activate venv (PowerShell) | `venv\Scripts\activate` |
| Local preview | `mkdocs serve` |
| Build only (no deploy) | `mkdocs build` |
| Deploy to GitHub Pages | `mkdocs gh-deploy` |
| Install dependencies | `pip install -r requirements.txt` |
