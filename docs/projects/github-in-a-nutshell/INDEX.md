---
title: "Git & GitHub Manual"
description: "Git & GitHub Manual."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
status: published
---

# Git & GitHub Manual

> A practical manual compiled from training materials, hands-on labs, and reference knowledge.
>
> **How to use this manual:**
> - Beginners: read Parts 1–3 in order.
> - Intermediate users: jump directly to Parts 3–5.
> - Advanced users: Part 7 covers Git internals in depth.
> - Chapters marked `[GAP]` are authored from knowledge/web sources, not the original training PDFs.

---

## Part 1 — Getting Started

| # | Chapter | Source |
|---|---|---|
| 01 | [Introduction to Version Control & Git](./part1/ch01-introduction.md) | vault |
| 02 | [Shell Commands Primer](./part1/ch02-shell-commands.md) | vault |
| 03 | [Installing & Configuring Git](./part1/ch03-install-configure.md) | GAP |
| 04 | [GitHub Authentication (SSH, PATs, gh CLI)](./part1/ch04-github-authentication.md) | GAP |

---

## Part 2 — Core Git Workflow

| # | Chapter | Source |
|---|---|---|
| 05 | [Creating a Repository & Basic Operations](./part2/ch05-basic-operations.md) | vault |
| 06 | [Tracking Files & File States](./part2/ch06-file-states.md) | vault |
| 07 | [Ignoring Files (.gitignore)](./part2/ch07-gitignore.md) | vault |
| 08 | [Undoing Changes](./part2/ch08-undoing-changes.md) | GAP |
| 09 | [Stashing](./part2/ch09-stashing.md) | GAP |
| 10 | [Inspecting History (log, blame, bisect, diff)](./part2/ch10-inspecting-history.md) | GAP |

---

## Part 3 — Branching & History Management

| # | Chapter | Source |
|---|---|---|
| 11 | [Branches and HEAD](./part3/ch11-branches-head.md) | vault |
| 12 | [Merging Branches](./part3/ch12-merging.md) | vault |
| 13 | [Rebasing](./part3/ch13-rebasing.md) | vault |
| 14 | [Interactive Rebase](./part3/ch14-interactive-rebase.md) | GAP + vault |
| 15 | [Tags & Semantic Versioning](./part3/ch15-tags.md) | vault |
| 16 | [Detached HEAD](./part3/ch16-detached-head.md) | vault + lab |

---

## Part 4 — GitHub & Collaboration

| # | Chapter | Source |
|---|---|---|
| 17 | [GitHub Overview & Remote Repositories](./part4/ch17-github-remotes.md) | vault |
| 18 | [Push, Fetch & Pull](./part4/ch18-push-fetch-pull.md) | vault |
| 19 | [Forks & Contributing to Open Source](./part4/ch19-forks.md) | vault |
| 20 | [Pull Requests](./part4/ch20-pull-requests.md) | vault |
| 21 | [Branch Protection & Code Review Workflows](./part4/ch21-branch-protection.md) | GAP |
| 22 | [GitHub Issues & Project Management](./part4/ch22-issues-projects.md) | GAP |

---

## Part 5 — Automation & Deployment

| # | Chapter | Source |
|---|---|---|
| 23 | [Git Hooks](./part5/ch23-git-hooks.md) | vault + labs |
| 24 | [GitHub Actions & CI/CD](./part5/ch24-github-actions.md) | GAP |
| 25 | [GitHub Pages (Static, Jekyll, React)](./part5/ch25-github-pages.md) | labs |

---

## Part 6 — Workflows & Best Practices

| # | Chapter | Source |
|---|---|---|
| 26 | [Git Workflows (GitFlow, Trunk-based, Feature Branch)](./part6/ch26-workflows.md) | GAP |
| 27 | [Git Aliases & Productivity Tips](./part6/ch27-aliases-tips.md) | GAP |

---

## Part 7 — Under the Hood (Advanced)

| # | Chapter | Source |
|---|---|---|
| 28 | [Git Object Model: Blobs, Trees & Commits](./part7/ch28-object-model.md) | vault |
| 29 | [SHA1, Hashing & Object Storage Internals](./part7/ch29-hashing-internals.md) | vault |

---

## Appendix

| # | Topic |
|---|---|
| A | [Git Submodules](./appendix/appA-submodules.md) |
| B | [Git LFS (Large File Storage)](./appendix/appB-git-lfs.md) |
| C | [Signed Commits & GPG](./appendix/appC-signed-commits.md) |

---

## Source Material Reference

### Vault → Chapter Mapping

| Vault Module | Manual Chapter(s) |
|---|---|
| `01-introduction/` | Ch 01 |
| `02-basic-shell-commands/` | Ch 02 |
| `03-how-git-works-under-the-hood/` | Ch 28, Ch 29 |
| `04-basic-git-operations/` | Ch 05, Ch 06 |
| `05-git-branches-and-head/` | Ch 11 |
| `06-merging-branches/` | Ch 12 |
| `07-github-and-remote-repositories/` | Ch 17 |
| `08-git-push-fetch-and-pull/` | Ch 18 |
| `09-pull-requests/` | Ch 20 |
| `10-forks-and-contribution/` | Ch 19 |
| `11-git-tags/` | Ch 15, Ch 24 (CI/CD slide) |
| `12-rebasing/` | Ch 13, Ch 14 |
| `13-ignoring-files-in-git/` | Ch 07 |
| `14-detached-head/` | Ch 16 |
| `15-git-hooks/` | Ch 23 |

### Lab → Chapter Mapping

| Lab | Manual Chapter |
|---|---|
| `labs/detached-head/` | Ch 16 |
| `labs/gh-pages-static/` | Ch 25 |
| `labs/gh-pages-markdown/` | Ch 25 |
| `labs/gh-pages-react/` | Ch 25 |
| `labs/git-hooks-local/` | Ch 23 |
| `labs/git-hooks-nodejs/` | Ch 23 |
| `labs/GitIgnore/` | Ch 07 |
| `labs/rebase/` | Ch 13, Ch 14 |
| `labs/tags/` | Ch 15 |
| `labs/Git-Training-Udm/` | Ch 05, Ch 06 |
| `labs/my-training-github-repository/` | Ch 17, Ch 18 |
| `labs/vue/` | Reference only |

### Gap Chapters (authored from knowledge/web)

Chapters marked `GAP` have no training PDF source. Content will be authored using:
- Official Git documentation (<https://git-scm.com/docs>)
- GitHub Docs (<https://docs.github.com>)
- The *Pro Git* book (open source, <https://git-scm.com/book>)

Gap chapters: 03, 04, 08, 09, 10, 14 (partial), 21, 22, 24, 26, 27, App A, App B, App C
