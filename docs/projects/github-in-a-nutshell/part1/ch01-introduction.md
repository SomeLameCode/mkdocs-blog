---
title: "Chapter 01 — Introduction to Git and Version Control"
description: "Chapter 01 — Introduction to Git and Version Control."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - getting-started
status: published
---

# Chapter 01 — Introduction to Git and Version Control

## What is Version Control?

Every software project changes over time. Files get edited, features get added, bugs get fixed, and sometimes changes need to be undone. Without a system to track those changes, teams quickly run into problems: files overwritten by accident, no way to roll back a broken change, and no record of who changed what or why.

**Version control** is a system that records changes to files over time so that you can recall specific versions later. It solves three fundamental problems:

- **History** — you can see exactly what changed, when, and who made the change.
- **Collaboration** — multiple people can work on the same codebase without overwriting each other's work.
- **Recovery** — you can revert any file (or the entire project) to a previous known-good state.

### Three Generations of Version Control

Version control systems have evolved through three broad generations:

| Generation | Model | Example | Key Limitation |
|---|---|---|---|
| 1st | Local only | RCS | No collaboration |
| 2nd | Centralised | SVN, CVS | Single point of failure; requires network |
| 3rd | Distributed | Git, Mercurial | — |

In a **centralised** system (like Subversion), there is one authoritative server. Every developer checks out files from that server and commits back to it. If the server goes down, work stops.

In a **distributed** system, every developer holds a full copy of the entire repository — history included. There is no single point of failure. You can commit, branch, and inspect history entirely offline.

---

## What is Git?

Git is a **free, open-source, distributed version control system**. It was created by Linus Torvalds in 2005 to manage development of the Linux kernel, after the project's previous VCS licence was revoked. Torvalds designed Git with three explicit goals:

1. **Speed** — operations on local data should be near-instantaneous.
2. **Data integrity** — every object stored by Git is checksummed; corruption is detectable.
3. **Support for non-linear development** — thousands of parallel branches must be cheap and fast.

Git met all three goals and quickly spread beyond the Linux kernel to become the dominant version control system in the software industry.

> **Further reading:** [Git's official documentation and history](https://git-scm.com/book/en/v2/Getting-Started-A-Short-History-of-Git) provides the definitive account of Git's origins.

### How Git Differs from Older Systems

Older centralised systems store history as a series of *deltas* — the difference between one version of a file and the next. Git takes a different approach: it stores **snapshots**. Each commit records the complete state of every tracked file at that point in time (using a space-efficient content-addressed object store). This makes most operations — branching, merging, checking out history — dramatically faster.

---

## What is GitHub?

**GitHub** is a cloud-based hosting platform for Git repositories. It is not part of Git itself — Git is the version control tool; GitHub is a service built on top of it.

GitHub adds a web interface and a set of collaboration features on top of plain Git:

- **Remote repository hosting** — store your repositories in the cloud and share them with others.
- **Pull requests** — a structured workflow for proposing, reviewing, and merging changes.
- **Issues** — lightweight project tracking built into every repository.
- **GitHub Actions** — a CI/CD automation platform integrated directly with your repository.
- **GitHub Pages** — free static site hosting served from a repository branch.

Other platforms offer similar hosting (GitLab, Bitbucket, Forgejo), but GitHub is the largest and most widely used, hosting the majority of the world's open-source software.

> **Further reading:** [GitHub Docs](https://docs.github.com/en/get-started) covers everything from account setup to advanced workflows.

---

## Why Use Git?

Git has become the industry standard for version control for several reasons:

**Works offline.** Because your local clone contains the full history, you can commit, branch, diff, and inspect logs without any network connection. You only need a network when pushing to or pulling from a remote.

**Branching is cheap.** Creating a new branch in Git takes milliseconds and costs almost no disk space. This makes it practical to open a separate branch for every feature, bug fix, or experiment — and merge or discard it when done.

**Integrity.** Git identifies every object (file content, directory tree, commit) by a SHA-1 hash of its content. It is impossible to change a file's history without Git detecting it.

**Ecosystem.** Virtually every modern development tool — editors, CI systems, deployment pipelines, code review platforms — has first-class Git integration.

---

## Key Concepts at a Glance

The following concepts appear throughout this manual. This table gives a brief orientation; each will be explained in depth in later chapters.

| Concept | What it is |
|---|---|
| **Repository (repo)** | A directory tracked by Git, containing all project files and their full history |
| **Commit** | A snapshot of the repository at a point in time, with a message describing the change |
| **Branch** | A movable pointer to a commit; enables parallel lines of development |
| **Remote** | A version of the repository hosted elsewhere (e.g. on GitHub); used to share work |
| **Working tree** | The files you currently see and edit on disk |
| **Staging area (index)** | A preparation zone where you assemble the next commit before recording it |
| **HEAD** | A pointer to the commit (or branch) you are currently working from |

---

*Next: [Chapter 02 — Shell Commands Primer](ch02-shell-commands.md)*
