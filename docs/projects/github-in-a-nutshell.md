---
title: "GitHub in a Nutshell — Git & GitHub Reference Manual"
description: "A self-contained Git and GitHub reference manual compiled from a structured training course — from first commit to CI/CD pipelines and Git internals."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - reference
  - devops
status: published
---

# GitHub in a Nutshell — Git & GitHub Reference Manual

A self-contained reference manual covering Git and GitHub from first principles through advanced internals — compiled from a structured training course, hands-on labs, and official documentation into one progressive resource.

---

## Table of Contents

1. [What Is It?](#1-what-is-it)
2. [Who Is It For?](#2-who-is-it-for)
3. [What's Inside](#3-whats-inside)
4. [How It Was Built](#4-how-it-was-built)
5. [How to Use It](#5-how-to-use-it)
6. [Status & What's Next](#6-status-whats-next)
7. [Manual Contents](#7-manual-contents)

---

## 1. What Is It?

**GitHub in a Nutshell** is a 29-chapter reference manual covering Git and GitHub from installation to CI/CD automation and storage internals. It was compiled from a 19-module training course, 13 hands-on lab projects, and gap chapters authored from official documentation — brought together into one structured, self-contained Markdown resource.

The problem it solves: Git knowledge is scattered. The official Git documentation ([git-scm.com](https://git-scm.com/doc)) covers the tool but not GitHub. GitHub's own documentation ([docs.github.com](https://docs.github.com)) covers the platform but not Git fundamentals. Training courses give you exercises but not a reference you can look things up in later. This manual consolidates all three into a single document with a clear progression from beginner to advanced.

> **Key idea:** Everything from "what is version control?" to "how does Git actually store objects on disk?" — in one place, in reading order.

---

## 2. Who Is It For?

The manual is structured for three entry points — pick the one that matches where you are.

### Beginners

Start at **[Chapter 01 — Introduction to Version Control & Git](github-in-a-nutshell/part1/ch01-introduction.md)**. No prior Git or terminal experience assumed. Chapter 02 covers the shell commands you need before any Git command is introduced, so you can follow every example from the first page.

Recommended path: **Parts 1–4** (Chapters 01–22) — from VCS concepts through collaborating on GitHub with pull requests and branch protection.

### Intermediate users

Already comfortable with basic commits and branches? Start at **[Part 3 — Branching & History Management](github-in-a-nutshell/part3/ch11-branches-head.md)** (Chapter 11) or **[Part 4 — GitHub & Collaboration](github-in-a-nutshell/part4/ch17-github-remotes.md)** (Chapter 17).

Useful standalone chapters at this level: [rebasing](github-in-a-nutshell/part3/ch13-rebasing.md) (Ch 13), [interactive rebase](github-in-a-nutshell/part3/ch14-interactive-rebase.md) (Ch 14), [undoing changes](github-in-a-nutshell/part2/ch08-undoing-changes.md) (Ch 08), [inspecting history](github-in-a-nutshell/part2/ch10-inspecting-history.md) (Ch 10).

### Advanced / curious

**[Part 7 — Under the Hood](github-in-a-nutshell/part7/ch28-object-model.md)** (Chapters 28–29) explains the Git object model — how blobs, trees, and commits are stored as SHA1-hashed objects on disk. Useful for understanding what Git is actually doing when you run everyday commands.

---

## 3. What's Inside

| Part | Chapters | Topics |
|---|---|---|
| **1 — Getting Started** | Ch 01–04 | Version control concepts, shell primer, Git installation and configuration, GitHub authentication (SSH keys, Personal Access Tokens, gh CLI) |
| **2 — Core Workflow** | Ch 05–10 | Creating repositories, tracking files and file states, `.gitignore`, undoing changes, stashing, inspecting history (`log`, `blame`, `bisect`, `diff`) |
| **3 — Branching & History** | Ch 11–16 | Branches and HEAD, merging strategies, rebasing, interactive rebase, tags and semantic versioning, detached HEAD state |
| **4 — GitHub & Collaboration** | Ch 17–22 | GitHub overview and remote repositories, push/fetch/pull, forks and open-source contribution, pull requests, branch protection and code review workflows, GitHub Issues and project management |
| **5 — Automation** | Ch 23–25 | Git hooks (client-side and server-side), GitHub Actions and CI/CD pipelines, GitHub Pages (static sites, Jekyll, React) |
| **6 — Workflows & Best Practices** | Ch 26–27 | GitFlow, trunk-based development, feature branch workflow; Git aliases and productivity tips |
| **7 — Under the Hood** | Ch 28–29 | Git object model (blobs, trees, commit objects), SHA1 hashing and content-addressed storage, how Git actually stores your history |
| **Appendix** | A–C | Git Submodules, Git LFS (Large File Storage), signed commits and GPG |

---

## 4. How It Was Built

| Source | What it contributed |
|---|---|
| **Vault notes** | 107 notes converted from 15 PDF training slide decks — the core narrative content for most chapters |
| **Hands-on labs** | 13 lab projects — practical examples and exercises woven into relevant chapters |
| **Gap chapters** | Chapters authored from [git-scm.com](https://git-scm.com/doc) and [docs.github.com](https://docs.github.com) where training materials didn't cover a topic (authentication, CI/CD, workflows) |

Each chapter notes its source — vault, lab, or gap — so the origin of any section is transparent. The shell primer in Chapter 02 was added specifically to make the manual accessible to readers unfamiliar with the terminal.

---

## 5. How to Use It

**Sequential read:** Parts 1–4 (Chapters 01–22) form a complete beginner-to-collaborator progression. Read in order for the full learning path.

**Reference mode:** Each chapter is self-contained. Jump directly to the topic you need using the [index below](#7-manual-contents) — no need to read preceding chapters.

**Platform notes:** All shell examples work on Git Bash (Windows), bash, and zsh (macOS/Linux). Where platform behaviour differs, the chapter notes it.

---

## 6. Status & What's Next

| | Count | Notes |
|---|---|---|
| **Complete** | 20 of 29 chapters | Parts 1–4 fully drafted and ready to use |
| **In progress** | 9 chapters | Parts 5–7 — automation, workflows, and internals |
| **Planned** | 3 appendix chapters | Submodules, Git LFS, signed commits |

---

## 7. Manual Contents

### Part 1 — Getting Started

| Chapter | Title |
|---|---|
| [Ch 01](github-in-a-nutshell/part1/ch01-introduction.md) | Introduction to Version Control & Git |
| [Ch 02](github-in-a-nutshell/part1/ch02-shell-commands.md) | Shell Commands Primer |
| [Ch 03](github-in-a-nutshell/part1/ch03-install-configure.md) | Installing & Configuring Git |
| [Ch 04](github-in-a-nutshell/part1/ch04-github-authentication.md) | GitHub Authentication (SSH, PATs, gh CLI) |

### Part 2 — Core Git Workflow

| Chapter | Title |
|---|---|
| [Ch 05](github-in-a-nutshell/part2/ch05-basic-operations.md) | Creating a Repository & Basic Operations |
| [Ch 06](github-in-a-nutshell/part2/ch06-file-states.md) | Tracking Files & File States |
| [Ch 07](github-in-a-nutshell/part2/ch07-gitignore.md) | Ignoring Files (.gitignore) |
| [Ch 08](github-in-a-nutshell/part2/ch08-undoing-changes.md) | Undoing Changes |
| [Ch 09](github-in-a-nutshell/part2/ch09-stashing.md) | Stashing |
| [Ch 10](github-in-a-nutshell/part2/ch10-inspecting-history.md) | Inspecting History (log, blame, bisect, diff) |

### Part 3 — Branching & History Management

| Chapter | Title |
|---|---|
| [Ch 11](github-in-a-nutshell/part3/ch11-branches-head.md) | Branches and HEAD |
| [Ch 12](github-in-a-nutshell/part3/ch12-merging.md) | Merging Branches |
| [Ch 13](github-in-a-nutshell/part3/ch13-rebasing.md) | Rebasing |
| [Ch 14](github-in-a-nutshell/part3/ch14-interactive-rebase.md) | Interactive Rebase |
| [Ch 15](github-in-a-nutshell/part3/ch15-tags.md) | Tags & Semantic Versioning |
| [Ch 16](github-in-a-nutshell/part3/ch16-detached-head.md) | Detached HEAD |

### Part 4 — GitHub & Collaboration

| Chapter | Title |
|---|---|
| [Ch 17](github-in-a-nutshell/part4/ch17-github-remotes.md) | GitHub Overview & Remote Repositories |
| [Ch 18](github-in-a-nutshell/part4/ch18-push-fetch-pull.md) | Push, Fetch & Pull |
| [Ch 19](github-in-a-nutshell/part4/ch19-forks.md) | Forks & Contributing to Open Source |
| [Ch 20](github-in-a-nutshell/part4/ch20-pull-requests.md) | Pull Requests |
| [Ch 21](github-in-a-nutshell/part4/ch21-branch-protection.md) | Branch Protection & Code Review Workflows |
| [Ch 22](github-in-a-nutshell/part4/ch22-issues-projects.md) | GitHub Issues & Project Management |

### Part 5 — Automation & Deployment

| Chapter | Title |
|---|---|
| [Ch 23](github-in-a-nutshell/part5/ch23-git-hooks.md) | Git Hooks |
| [Ch 24](github-in-a-nutshell/part5/ch24-github-actions.md) | GitHub Actions & CI/CD |
| [Ch 25](github-in-a-nutshell/part5/ch25-github-pages.md) | GitHub Pages |

### Part 6 — Workflows & Best Practices

| Chapter | Title |
|---|---|
| [Ch 26](github-in-a-nutshell/part6/ch26-workflows.md) | Git Workflows (GitFlow, Trunk-based, Feature Branch) |
| [Ch 27](github-in-a-nutshell/part6/ch27-aliases-tips.md) | Git Aliases & Productivity Tips |

### Part 7 — Under the Hood

| Chapter | Title |
|---|---|
| [Ch 28](github-in-a-nutshell/part7/ch28-object-model.md) | Git Object Model: Blobs, Trees & Commits |
| [Ch 29](github-in-a-nutshell/part7/ch29-hashing-internals.md) | SHA1, Hashing & Object Storage Internals |

### Appendix

| Chapter | Title |
|---|---|
| [App A](github-in-a-nutshell/appendix/appA-submodules.md) | Git Submodules |
| [App B](github-in-a-nutshell/appendix/appB-git-lfs.md) | Git LFS (Large File Storage) |
| [App C](github-in-a-nutshell/appendix/appC-signed-commits.md) | Signed Commits & GPG |
