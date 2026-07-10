---
title: "Chapter 24 — GitHub Actions & CI/CD"
description: "Chapter 24 — GitHub Actions & CI/CD."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - automation
  - ci-cd
status: published
---

# Chapter 24 — GitHub Actions & CI/CD

Every time code is pushed, a pull request is opened, or a release tag is created, there is useful work that could be done automatically: run the tests, check formatting, build a Docker image, deploy to staging. **GitHub Actions** is GitHub's built-in automation platform that makes this possible. It runs arbitrary code in response to GitHub events, without requiring any external CI service.

---

## Core Concepts

### Workflow

A **workflow** is a YAML file stored in `.github/workflows/` that defines when to run and what to do. A repository can have many workflows, each triggered independently.

```
.github/
└── workflows/
    ├── ci.yml          # run tests on every push and PR
    ├── release.yml     # build and publish on tag push
    └── codeql.yml      # security analysis on schedule
```

### Event (trigger)

Every workflow starts with an event — something that happens on GitHub. Common triggers:

```yaml
on:
  push:                         # any push to any branch
  pull_request:                 # PR opened, synchronised, or reopened
  push:
    branches: [main]            # push to main only
  push:
    tags: ['v*']                # push of a version tag
  schedule:
    - cron: '0 2 * * *'         # daily at 02:00 UTC
  workflow_dispatch:            # manual trigger from GitHub UI
  release:
    types: [published]          # when a GitHub Release is published
```

### Job

A **job** is a set of steps that runs on a single runner. Jobs in the same workflow run in parallel by default; use `needs:` to sequence them.

### Step

A **step** is a single task within a job — either a shell command (`run:`) or a call to a pre-built **action** (`uses:`).

### Runner

A **runner** is the virtual machine that executes a job. GitHub provides hosted runners:

| Runner label | OS |
|---|---|
| `ubuntu-latest` | Ubuntu Linux (most common) |
| `windows-latest` | Windows Server |
| `macos-latest` | macOS |

You can also self-host runners on your own infrastructure.

### Action

An **action** is a reusable unit of automation — a packaged step that can be called with `uses:`. Actions are distributed through the GitHub Marketplace or referenced directly from any public repository.

```yaml
uses: actions/checkout@v4        # check out your code
uses: actions/setup-node@v4      # install Node.js
uses: docker/build-push-action@v5  # build and push a Docker image
```

---

## Workflow File Anatomy

```yaml
name: CI                          # display name in GitHub UI

on:                               # trigger(s)
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:                           # job ID (arbitrary)
    name: Run tests               # display name
    runs-on: ubuntu-latest        # runner

    steps:
      - name: Check out code
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test
```

Key points:
- `uses: actions/checkout@v4` clones the repository into the runner's workspace — almost every workflow needs this as the first step.
- `with:` passes input parameters to an action.
- `run:` executes a shell command. Multi-line commands use `|`:

```yaml
- name: Build and verify
  run: |
    npm run build
    npm run type-check
```

---

## Common Workflow Patterns

### CI on push and pull request

The most fundamental workflow: run tests on every push and every PR targeting `main`. A failing check blocks the PR from merging (when branch protection requires it — Chapter 21).

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm test
```

### Matrix builds — test across multiple versions

The `strategy.matrix` key runs the same job multiple times with different parameter values:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: ['18', '20', '22']
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci && npm test
```

This produces three parallel jobs — one per Node.js version — giving you confidence that your code works across your supported range.

### Deploy on tag push

Trigger a deployment workflow only when a version tag is pushed (see Chapter 15 for tag conventions):

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build-and-publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t myapp:${{ github.ref_name }} .

      - name: Log in to registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Push image
        run: docker push ghcr.io/${{ github.repository }}:${{ github.ref_name }}
```

### Sequential jobs with dependencies

Use `needs:` to run jobs in sequence — `deploy` only runs if `test` and `build` both pass:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - run: npm run build

  deploy:
    runs-on: ubuntu-latest
    needs: [test, build]
    steps:
      - name: Deploy to production
        run: ./scripts/deploy.sh
```

---

## Expressions and Contexts

GitHub Actions provides **contexts** — objects containing information about the run, the repository, and the event that triggered it. Access them with `${{ }}` syntax:

| Expression | Value |
|---|---|
| `${{ github.sha }}` | Full commit SHA |
| `${{ github.ref_name }}` | Branch or tag name |
| `${{ github.actor }}` | Username that triggered the workflow |
| `${{ github.repository }}` | `owner/repo` |
| `${{ github.event_name }}` | Event that triggered the workflow |
| `${{ runner.os }}` | OS of the runner |
| `${{ secrets.MY_SECRET }}` | A repository secret (never printed in logs) |
| `${{ env.MY_VAR }}` | An environment variable |

### Conditional steps

Use `if:` to run a step only under certain conditions:

```yaml
- name: Deploy (main branch only)
  if: github.ref == 'refs/heads/main'
  run: ./deploy.sh

- name: Notify on failure
  if: failure()
  run: ./notify-slack.sh "Build failed"
```

Built-in status functions: `success()`, `failure()`, `cancelled()`, `always()`.

---

## Secrets and Environment Variables

### Repository secrets

Store sensitive values (API keys, deploy tokens, passwords) as **repository secrets** in:

**Settings → Secrets and variables → Actions → New repository secret**

Access in a workflow:

```yaml
- name: Deploy
  env:
    DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
  run: ./deploy.sh
```

Secrets are masked in logs — their values are never printed, even if you accidentally `echo` them.

`GITHUB_TOKEN` is a special secret automatically provided in every workflow. It is scoped to the repository and expires when the workflow finishes. Use it for GitHub API calls, pushing to the repository, or publishing packages.

### Environment variables

Set variables at the workflow, job, or step level:

```yaml
env:
  NODE_ENV: production           # workflow-level

jobs:
  build:
    env:
      BUILD_DIR: dist            # job-level
    steps:
      - run: npm run build
        env:
          VITE_API_URL: https://api.example.com  # step-level
```

---

## Caching Dependencies

Downloading and installing dependencies on every run is slow. The `actions/cache` action (or the `cache:` input on language setup actions) stores dependency directories between runs:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'        # caches node_modules based on package-lock.json hash
```

For other languages:

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
```

The cache key changes when `requirements.txt` changes, busting the cache when dependencies update.

---

## Artifacts

Jobs can produce files (build outputs, test reports, coverage data) and pass them between jobs or make them downloadable via the **Actions** tab.

```yaml
- name: Upload build output
  uses: actions/upload-artifact@v4
  with:
    name: build-dist
    path: dist/

# In a later job:
- name: Download build output
  uses: actions/download-artifact@v4
  with:
    name: build-dist
```

---

## Reusable Workflows and Composite Actions

### Reusable workflows

A workflow can call another workflow file using `workflow_call`, avoiding duplication across repositories:

```yaml
# Caller workflow
jobs:
  call-shared-ci:
    uses: org/shared-workflows/.github/workflows/ci.yml@main
    with:
      node-version: '20'
    secrets: inherit
```

### Composite actions

For step-level reuse within a repository, create a composite action in `.github/actions/<name>/action.yml`:

```yaml
name: 'Setup and Test'
runs:
  using: composite
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
    - run: npm ci && npm test
      shell: bash
```

---

## Viewing and Debugging Workflows

- **Actions tab** — lists every workflow run; click a run to see each job and its steps with full logs
- **Re-run jobs** — re-run failed jobs (with or without debug logging) from the UI
- **Enable debug logging** — set repository secret `ACTIONS_STEP_DEBUG` to `true` for verbose runner output
- **`act`** — an open-source tool that runs GitHub Actions workflows locally for faster iteration

---

## Summary

- A **workflow** is a `.github/workflows/*.yml` file triggered by GitHub events (`push`, `pull_request`, `schedule`, `workflow_dispatch`, etc.).
- A workflow contains **jobs**, each running on a **runner**; jobs contain **steps** (shell commands or `uses:` action calls).
- `actions/checkout` and `actions/setup-*` are the standard first steps in almost every job.
- **Matrix builds** run a job across multiple parameter values (e.g. multiple language versions) in parallel.
- **Secrets** are encrypted variables injected at runtime; `GITHUB_TOKEN` is provided automatically in every workflow.
- **Cache** dependency directories to speed up runs; **artifacts** pass files between jobs or preserve build outputs.
- Branch protection (Chapter 21) can require specific workflow checks to pass before a PR is mergeable, making CI enforcement automatic.

> **Further reading:** [GitHub Actions documentation](https://docs.github.com/en/actions) · [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)

---

*Previous: [Chapter 23 — Git Hooks](ch23-git-hooks.md)* · *Next: [Chapter 25 — GitHub Pages](ch25-github-pages.md)*

