---
title: "Appendix B — Git LFS (Large File Storage)"
description: "Appendix B — Git LFS (Large File Storage)."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
status: published
---

# Appendix B — Git LFS (Large File Storage)

Git is designed for text files: source code, configuration, documentation. It stores every version of every file in the object store, which works efficiently because text files compress well and diffs are small. Binary files — compiled executables, images, audio, video, datasets, design files — do not compress well and cannot be diffed meaningfully. Storing many large binaries in Git causes the repository to grow without bound, slowing every clone and fetch.

**Git LFS** (Large File Storage) solves this by replacing large files in the repository with small text **pointer files**, while the actual binary content is stored on a separate LFS server. Clones and fetches are fast because only the pointers travel with the repository; the binary content is downloaded on demand.

---

## How Git LFS Works

When you stage a tracked file, the Git LFS client intercepts it and:

1. Uploads the file's content to the LFS server (GitHub's LFS storage, a self-hosted server, or another provider).
2. Writes a small **pointer file** in its place — a plain text file containing the file's SHA-256 hash and size.
3. Stores the pointer in the Git object store as a normal blob.

The pointer file looks like this:

```
version https://git-lfs.github.com/spec/v1
oid sha256:4d7a214614ab2935c943f9e0ff69d22eadbb8f32b1258daaa5e2ca24d17e2393
size 12345678
```

When you check out a branch or run `git lfs pull`, the LFS client reads the pointer and downloads the actual binary from the LFS server, replacing the pointer in the working tree with the real file. To the rest of your toolchain the file appears normal — only the storage mechanism changes.

---

## Installing Git LFS

Git LFS is a separate install from Git itself.

**macOS (Homebrew):**

```bash
brew install git-lfs
```

**Linux (Debian/Ubuntu):**

```bash
sudo apt install git-lfs
```

**Windows:**

Download the installer from [git-lfs.com](https://git-lfs.com) or install via `winget`:

```bash
winget install GitHub.GitLFS
```

After installing, run the one-time global setup to add LFS hooks to your Git configuration:

```bash
git lfs install
```

This adds a `filter.lfs` entry to `~/.gitconfig` and hooks that intercept `git add` and `git checkout` for tracked file types.

---

## Tracking File Types

Tell Git LFS which files to manage using `git lfs track`. This writes patterns to a `.gitattributes` file in the repository root:

```bash
git lfs track "*.psd"
git lfs track "*.png"
git lfs track "*.mp4"
git lfs track "*.zip"
git lfs track "*.bin"
```

Each command appends an entry to `.gitattributes`:

```
*.psd filter=lfs diff=lfs merge=lfs -text
*.png filter=lfs diff=lfs merge=lfs -text
*.mp4 filter=lfs diff=lfs merge=lfs -text
```

Commit `.gitattributes` to the repository so that every contributor's LFS client uses the same tracking rules:

```bash
git add .gitattributes
git commit -m "Configure Git LFS tracking for binary assets"
```

**Order matters:** track before adding. Files added to Git before LFS tracking was configured are already stored as regular blobs. To migrate them, see [Migrating Existing Files](#migrating-existing-files) below.

### Checking what is tracked

```bash
git lfs track          # list all currently tracked patterns
git lfs status         # show LFS-managed files in the staging area
git lfs ls-files       # list all LFS-tracked files in the current commit
```

---

## Normal Workflow

Once LFS is installed and `.gitattributes` is configured, the day-to-day workflow is identical to normal Git:

```bash
# Add a large file — LFS intercepts and uploads it
git add assets/hero-video.mp4
git commit -m "Add hero video"
git push

# Clone — gets pointers only; LFS content downloaded on checkout
git clone https://github.com/example/project.git

# Pull new LFS content after a fetch
git lfs pull
```

If you clone a repository that uses LFS but do not have Git LFS installed, you will see the raw pointer files in your working tree instead of the actual binaries.

---

## Migrating Existing Files

If large files were already committed to the repository as regular blobs before LFS was configured, they continue to bloat the history. `git lfs migrate` rewrites history to replace those blobs with LFS pointers:

```bash
# Preview what would be migrated (dry run)
git lfs migrate info --include="*.psd,*.mp4"

# Rewrite history for the current branch
git lfs migrate import --include="*.psd,*.mp4"

# Rewrite history for all branches
git lfs migrate import --include="*.psd,*.mp4" --everything
```

`migrate import` rewrites commits — it changes SHAs, so this is a **destructive, history-rewriting operation**. Coordinate with all collaborators before doing this on a shared repository: everyone must re-clone or reset their local branches after the migration. Force-push will be required.

After migrating, verify:

```bash
git lfs ls-files        # should list the migrated files
git log --oneline       # SHAs will have changed
```

---

## Fetching and Pulling LFS Content

By default, `git clone` and `git fetch` download LFS pointers but not the binary content. Use `git lfs pull` to download the content for the current checkout:

```bash
git lfs pull
```

To fetch LFS content for all refs (not just the current branch):

```bash
git lfs fetch --all
```

To skip LFS content entirely (useful in CI environments that do not need binaries):

```bash
GIT_LFS_SKIP_SMUDGE=1 git clone <url>
```

The `smudge` filter is the LFS hook that replaces pointers with real files on checkout. Setting this environment variable disables it, leaving pointer files in the working tree.

---

## Storage and Bandwidth on GitHub

GitHub provides LFS storage and bandwidth that counts against your account quota, separate from your repository storage:

| Plan | Free LFS storage | Free LFS bandwidth (monthly) |
|---|---|---|
| Free | 1 GB | 1 GB |
| Pro / Team | 2 GB | 2 GB |
| Enterprise | Configurable | Configurable |

Additional storage and bandwidth can be purchased in data packs. Monitor your usage in **Settings → Billing → Git LFS**.

LFS objects are not deleted when you delete commits or run `git gc` on the server — they must be explicitly removed through the provider's interface.

---

## Self-Hosted LFS Servers

If you cannot or do not want to use GitHub's LFS hosting, several open-source LFS server implementations exist:

- **[Gitea](https://gitea.io/)** — includes built-in LFS support.
- **[lfs-test-server](https://github.com/git-lfs/lfs-test-server)** — the reference implementation; suitable for testing, not production.
- **[Minio + LFS proxy](https://min.io/)** — store LFS objects in object storage (S3-compatible).

Configure the LFS endpoint per repository:

```bash
git config lfs.url https://lfs.example.com/myrepo
```

Or globally in `.lfsconfig` (committed to the repo):

```
[lfs]
    url = https://lfs.example.com/myrepo
```

---

## Common Pitfalls

**Adding files before running `git lfs track`**

Files added before a pattern is tracked are stored as regular Git blobs. The LFS filter only applies to files staged after the `.gitattributes` rule exists. Always configure tracking and commit `.gitattributes` before adding large files.

**Not committing `.gitattributes`**

If `.gitattributes` is not committed, other contributors' LFS clients will not track the right file types. This is the most common setup mistake.

**History rewrites after `migrate import`**

`git lfs migrate import` changes commit SHAs. Any open pull requests, bookmarks, or clones based on the old history become invalid. Plan migrations with the team, communicate the rewrite, and require everyone to re-clone.

**LFS quota exhaustion**

LFS bandwidth is consumed on every download of a tracked file — not just the first. A popular repository with large assets can exhaust the monthly bandwidth allowance quickly. Consider a CDN for files that need frequent public access, and reserve LFS for assets accessed only during development.

**Partial clones and LFS**

Git's `--filter=blob:none` (partial clone) and Git LFS are separate mechanisms. They can be used together but require care — LFS smudge runs during checkout, and a partial clone without LFS content will show pointer files for undownloaded blobs.

---

## Quick Reference

```bash
# One-time global setup
git lfs install

# Track a file pattern
git lfs track "*.psd"
git add .gitattributes && git commit -m "Track PSD files with LFS"

# Normal add/commit/push (LFS is transparent)
git add design/mockup.psd
git commit -m "Add mockup"
git push

# Clone with LFS content
git clone --recurse-submodules <url>   # LFS downloads automatically

# Download LFS content after fetch
git lfs pull

# List tracked patterns
git lfs track

# List LFS files in current commit
git lfs ls-files

# Check LFS staging status
git lfs status

# Migrate existing history (destructive — coordinate with team)
git lfs migrate import --include="*.psd" --everything

# Clone without downloading LFS content (CI)
GIT_LFS_SKIP_SMUDGE=1 git clone <url>
```

---

## Summary

- Git LFS replaces large binary files with small pointer files in the Git object store; actual content lives on an LFS server.
- Install with `git lfs install` (once per machine), then declare patterns with `git lfs track` and commit `.gitattributes`.
- Everyday workflow — `git add`, `git commit`, `git push`, `git pull` — is unchanged; LFS is transparent.
- `git lfs migrate import` rewrites history to bring existing large blobs under LFS management; this changes SHAs and requires coordination with collaborators.
- GitHub provides 1–2 GB of free LFS storage and bandwidth per month; additional capacity is available via data packs.
- Skip LFS content in CI with `GIT_LFS_SKIP_SMUDGE=1` to speed up builds that do not need binary assets.

> **Further reading:** [Git LFS documentation](https://git-lfs.com) · [GitHub Docs — Git LFS](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-git-large-file-storage)

---

*Previous: [Appendix A — Git Submodules](appA-submodules.md)* · *Next: [Appendix C — Signed Commits & GPG](appC-signed-commits.md)*

