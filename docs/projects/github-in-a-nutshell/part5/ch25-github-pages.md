---
title: "Chapter 25 — GitHub Pages"
description: "Chapter 25 — GitHub Pages."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - automation
  - ci-cd
status: published
---

# Chapter 25 — GitHub Pages

**GitHub Pages** is a free static site hosting service built into GitHub. It serves the contents of a repository branch or folder as a website, with no server configuration required. It is the quickest way to publish documentation, a portfolio, a project landing page, or any site that does not require server-side rendering.

---

## How GitHub Pages Works

GitHub Pages reads from a specified **source** in your repository and serves the files at a public URL. The source can be:

- A branch (e.g. `gh-pages`, `main`)
- A folder within a branch (`/` root or `/docs`)

Every push to the configured source branch triggers a rebuild and redeploy. For plain HTML the files are served as-is. For Jekyll sites, GitHub runs a build step before publishing.

### URL format

| Repository type | Default URL |
|---|---|
| User/organisation site (`<username>.github.io`) | `https://<username>.github.io` |
| Project site (any other repo) | `https://<username>.github.io/<repo-name>` |

A user site is served from a repository named exactly `<username>.github.io`. All other repositories produce project sites at a subpath.

### Enabling GitHub Pages

In your repository: **Settings → Pages → Source** — choose the branch and folder, then save. GitHub shows the published URL once the first build completes.

---

## Approach 1: Plain HTML

The simplest possible GitHub Pages site is a single `index.html` at the root of the publishing branch. GitHub serves it directly with no build step.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>My Project</title>
</head>
<body>
  <h1>Hello from GitHub Pages</h1>
  <p>A plain static page — no build tools needed.</p>
</body>
</html>
```

Add CSS and JavaScript files alongside `index.html`. Link them with relative paths. Any file in the branch/folder is served — images, fonts, PDFs, anything.

**When to use:** documentation landing pages, simple portfolios, single-page references where you want zero toolchain overhead.

---

## Approach 2: Jekyll (Markdown to HTML)

GitHub Pages has built-in support for [Jekyll](https://jekyllrb.com/), a static site generator that converts Markdown files and Liquid templates into HTML. You write content in Markdown; Jekyll renders it as a styled website.

### Minimal setup

Two files are enough to create a Jekyll site:

**`_config.yml`** — site metadata and theme selection:

```yaml
title: My Project Site
description: A site showcasing project work
url: "https://yourusername.github.io"
baseurl: ""           # empty for user site; "/repo-name" for project site

theme: jekyll-theme-cayman   # one of GitHub's supported themes
markdown: kramdown
```

**`index.md`** — the home page content:

```markdown
---
layout: default
title: Home
---

# Welcome

This site is built with Jekyll and hosted on GitHub Pages.
```

The YAML block between `---` delimiters is **front matter** — metadata Jekyll uses to determine the layout and title for the page.

### Supported themes

GitHub Pages supports a set of built-in themes you can reference directly in `_config.yml`:

```yaml
theme: jekyll-theme-cayman      # clean, modern
theme: jekyll-theme-minimal     # minimal, text-focused
theme: jekyll-theme-slate       # dark sidebar
theme: minima                   # blog-friendly default
```

Any Jekyll theme on RubyGems can be used with `remote_theme:` instead of `theme:` — this gives access to the full ecosystem.

### Adding pages

Every `.md` file in the repository becomes a page. `about.md` with `layout: default` in its front matter becomes `/about/` on the site. Navigation links between pages use relative URLs.

### Local preview

To preview a Jekyll site locally before pushing:

```bash
gem install bundler jekyll
bundle init
# Add to Gemfile: gem "github-pages", group: :jekyll_plugins
bundle install
bundle exec jekyll serve
# Visit http://localhost:4000
```

**When to use:** project documentation, technical blogs, portfolio sites where you want Markdown authoring with a polished theme and no JavaScript build step.

---

## Approach 3: React (Create React App + gh-pages)

Single-page applications built with Create React App or Vite need a build step before deployment. The standard approach for CRA uses the `gh-pages` npm package to push the built output to a dedicated `gh-pages` branch.

### Setup

**1. Install the `gh-pages` package:**

```bash
npm install --save-dev gh-pages
```

**2. Add the `homepage` field to `package.json`:**

This tells Create React App the subpath where the app will be served:

```json
{
  "homepage": "https://yourusername.github.io/your-repo-name",
  "scripts": {
    "predeploy": "npm run build",
    "deploy": "gh-pages -d build"
  }
}
```

For a user site (served at the root): `"homepage": "https://yourusername.github.io"`.

**3. Deploy:**

```bash
npm run deploy
```

This runs `npm run build` (the `predeploy` script), then pushes the `build/` directory to the `gh-pages` branch of the origin remote. GitHub Pages is then configured to serve from that branch.

### Enabling the Pages source

After the first deploy, go to **Settings → Pages** and set the source to the `gh-pages` branch.

### Client-side routing

React apps that use client-side routing (React Router) have a known issue with GitHub Pages: navigating directly to a subpath like `https://user.github.io/repo/about` returns a 404 because GitHub Pages looks for a real file at that path and finds nothing.

The standard workaround is to add a `404.html` that redirects to `index.html` with the path encoded as a query parameter, then decode it back in `index.html`. The [spa-github-pages](https://github.com/rafgraph/spa-github-pages) project provides ready-made scripts for this.

**When to use:** React, Vue, Angular, or Svelte apps that need a build step and don't require a server — documentation sites with interactive components, personal portfolios, project demos.

---

## Approach 4: GitHub Actions Deployment (Modern)

GitHub Actions (Chapter 24) can deploy to GitHub Pages from any branch, any build output directory, and using any build tool — not just CRA with `gh-pages`. This is the recommended approach for new projects.

Configure the Pages source to **GitHub Actions** instead of a branch (**Settings → Pages → Source → GitHub Actions**), then add a workflow:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm run build

      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist/           # your build output directory

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/deploy-pages@v4
        id: deployment
```

This approach works with Vite, Next.js (static export), Astro, Hugo, MkDocs, or any other static site generator. The `upload-pages-artifact` / `deploy-pages` action pair handles the upload and deployment atomically.

**When to use:** any project where the build tool is not CRA, or where you want full control over the build and deploy process.

---

## Custom Domains

GitHub Pages supports custom domains (e.g. `docs.example.com` instead of `username.github.io/repo`).

**1.** Add a `CNAME` file at the root of the publishing branch containing just the custom domain:

```
docs.example.com
```

Or add it in **Settings → Pages → Custom domain**.

**2.** Configure your DNS provider:

- For an apex domain (`example.com`): add `A` records pointing to GitHub's Pages IP addresses (listed in the [GitHub Docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site))
- For a subdomain (`docs.example.com`): add a `CNAME` record pointing to `<username>.github.io`

**3.** Enable **Enforce HTTPS** in the Pages settings once DNS has propagated.

---

## Limitations

| Limitation | Detail |
|---|---|
| Static only | No server-side code — no PHP, Node.js, Python, databases |
| Storage | Repositories should be under 1 GB; published sites under 1 GB |
| Bandwidth | 100 GB soft limit per month |
| Build time | 10-minute limit for Jekyll builds |
| Private repositories | Pages are public even from private repos (unless on GitHub Pro/Team/Enterprise) |
| No server-side rendering | Use Vercel, Netlify, or Cloudflare Pages for SSR frameworks |

---

## Summary

- GitHub Pages serves static files from a branch or folder at `<username>.github.io` or `<username>.github.io/<repo>`.
- **Plain HTML** — zero setup, commit an `index.html` and done.
- **Jekyll** — write in Markdown, configure a theme in `_config.yml`; GitHub builds automatically.
- **React/CRA** — add `homepage` to `package.json`, use the `gh-pages` package to push the build output to a `gh-pages` branch.
- **GitHub Actions** — the modern approach for any build tool; use `upload-pages-artifact` and `deploy-pages` actions.
- Custom domains require a `CNAME` file plus DNS configuration; HTTPS is available free.
- GitHub Pages is for static sites only — no server-side execution.

> **Further reading:** [GitHub Pages documentation](https://docs.github.com/en/pages)

---

*Previous: [Chapter 24 — GitHub Actions & CI/CD](ch24-github-actions.md)* · *Next: [Chapter 26 — Git Workflows](../part6/ch26-workflows.md)*
