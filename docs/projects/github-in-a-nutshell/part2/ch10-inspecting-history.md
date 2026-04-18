---
title: "Chapter 10 — Inspecting History"
description: "Chapter 10 — Inspecting History."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - core-workflow
status: published
---

# Chapter 10 — Inspecting History

One of Git's greatest strengths is that it remembers everything. Every commit is a snapshot you can examine, compare, and navigate. This chapter covers the four main tools for reading repository history: `git log` for browsing commits, `git diff` for comparing snapshots, `git show` for inspecting a single commit, `git blame` for tracing line-level authorship, and `git bisect` for finding exactly which commit introduced a bug.

---

## `git log` — Browsing Commit History

`git log` lists commits reachable from the current HEAD, newest first.

```bash
git log
# commit a3f8c21 (HEAD -> main)
# Author: Ada Lovelace <ada@example.com>
# Date:   Mon Mar 24 14:32:01 2026 +0000
#
#     Add user authentication module
#
# commit 7b4c91e
# ...
```

### Compact output

```bash
git log --oneline
# a3f8c21 (HEAD -> main) Add user authentication module
# 7b4c91e Fix null pointer in user lookup
# 3e8f20a Initial commit
```

### Visual branch graph

```bash
git log --oneline --graph --all --decorate
# * a3f8c21 (HEAD -> main) Add user authentication module
# | * f91c3b2 (feature-x) Experiment: new layout
# |/
# * 7b4c91e Fix null pointer in user lookup
# * 3e8f20a Initial commit
```

`--all` includes commits on all branches (not just the current one); `--decorate` shows branch and tag labels alongside SHAs.

### Files changed per commit

```bash
git log --stat
# commit a3f8c21
# ...
#  src/auth.js  | 42 +++++++++++++++++++++
#  src/user.js  |  8 ++--
#  2 files changed, 46 insertions(+), 4 deletions(-)
```

### Full patch per commit

```bash
git log -p
# equivalent: git log --patch
```

Shows the complete diff for every commit in the log. Combine with `-n` to limit output:

```bash
git log -p -n 3   # patch for last 3 commits only
```

### Limiting results

```bash
git log -n 10              # last 10 commits
git log --since="2026-01-01"
git log --until="2026-03-01"
git log --since="2 weeks ago"
```

### Filtering by author

```bash
git log --author="Ada"        # matches partial name or email
git log --author="ada@example.com"
```

### Searching commit messages

```bash
git log --grep="authentication"    # commits whose message contains the string
git log --grep="fix" -i            # case-insensitive
```

Multiple `--grep` flags are OR'd by default; add `--all-match` to require all patterns.

### Pickaxe — find when a string was added or removed

```bash
git log -S "secretKey"        # commits that added or removed this exact string
git log -G "secret.*Key"      # commits where diff matches this regex
```

The pickaxe (`-S`) is invaluable for finding where a particular piece of code was introduced or deleted — especially useful for security audits.

### Tracking a file's history across renames

```bash
git log --follow -- src/utils.js
```

Without `--follow`, if `utils.js` was previously called `helpers.js`, log stops at the rename. With `--follow`, it traces through the rename.

### Range syntax

```bash
git log main..feature-x        # commits in feature-x not in main
git log A..B                   # commits reachable from B but not from A
git log A...B                  # commits reachable from either but not both (symmetric diff)
```

> **Further reading:** [`git log` documentation](https://git-scm.com/docs/git-log)

---

## `git diff` Across Commits

`git diff` is covered for the working directory and staging area in [Chapter 06](ch06-file-states.md). It also compares commits, branches, and arbitrary tree states.

### Working directory vs a commit

```bash
git diff HEAD             # all changes since last commit
git diff HEAD~3           # changes since 3 commits ago
git diff a3f8c21          # changes since a specific commit
```

### Between two commits

```bash
git diff 7b4c91e a3f8c21          # what changed between these two SHAs
git diff HEAD~5 HEAD              # last 5 commits worth of changes
```

### Between two branches

```bash
git diff main feature-x           # what feature-x has changed relative to main
git diff main..feature-x          # equivalent
```

### Summary formats

```bash
git diff --name-only main feature-x     # just filenames
git diff --name-status main feature-x   # filenames with change type (M/A/D/R)
git diff --stat main feature-x          # file-level summary with line counts
```

> **Further reading:** [`git diff` documentation](https://git-scm.com/docs/git-diff)

---

## `git show` — Inspect a Single Object

`git show` displays the metadata and diff for a single commit, or the content of a specific file at a specific point in history.

### Show a commit

```bash
git show                  # most recent commit (HEAD)
git show a3f8c21          # specific commit by SHA
git show HEAD~2           # relative reference
git show v1.4.0           # a tag
```

Output: commit metadata (author, date, message) followed by the full diff.

### Show a file at a specific commit

```bash
git show HEAD:src/auth.js              # current version in HEAD
git show a3f8c21:src/auth.js           # version at a specific commit
git show HEAD~3:README.md              # version 3 commits ago
```

This is useful for recovering a version of a file without checking out the whole commit — you can redirect the output to a file:

```bash
git show HEAD~1:src/config.js > config-previous.js
```

> **Further reading:** [`git show` documentation](https://git-scm.com/docs/git-show)

---

## `git blame` — Line-by-Line Authorship

`git blame` annotates every line of a file with the commit SHA, author, and date that last modified it. It answers the question: *who wrote this, and when?*

```bash
git blame src/auth.js
# a3f8c21 (Ada Lovelace  2026-03-24 14:32:01 +0000  1) import bcrypt from 'bcrypt';
# 7b4c91e (Bob Smith     2026-02-10 09:15:44 +0000  2)
# a3f8c21 (Ada Lovelace  2026-03-24 14:32:01 +0000  3) export async function hashPassword(pw) {
# ...
```

Each line shows:
- The abbreviated commit SHA
- The author name
- The commit date and time
- The line number
- The line content

### Restrict to a line range

```bash
git blame -L 20,40 src/auth.js    # only lines 20–40
git blame -L 20,+15 src/auth.js   # line 20 plus the next 15 lines
```

### Show the full commit SHA

```bash
git blame -l src/auth.js          # long (full 40-character) SHAs
```

### Follow a commit to its context

After identifying the SHA from `git blame`, use `git show` to read the full commit:

```bash
git show a3f8c21
```

### An important caveat

`git blame` shows who **last modified** each line — not who originally wrote it. If a line was reformatted, moved, or touched by a bulk change, blame will point to that mechanical change rather than the original author. Use `git log -p -- <file>` to trace a line's full history.

> **Further reading:** [`git blame` documentation](https://git-scm.com/docs/git-blame)

---

## `git bisect` — Binary Search for a Regression

You know a bug exists in the current commit. You know an earlier commit was clean. But there are hundreds of commits in between. Testing each one manually would take hours. `git bisect` automates this: it performs a binary search through the commit history, halving the search space with each step.

With *n* commits to search, bisect finds the culprit in at most ⌈log₂(n)⌉ steps — 10 steps for 1,000 commits, 17 for 100,000.

### Manual bisect workflow

```bash
# Step 1 — start the bisect session
git bisect start

# Step 2 — mark the current (broken) commit as bad
git bisect bad

# Step 3 — mark a known-good earlier commit
git bisect good v2.0.0          # a tag
git bisect good 3e8f20a         # or a SHA

# Git checks out the midpoint commit automatically
# Bisecting: 47 revisions left to test after this (roughly 6 steps)

# Step 4 — test the checked-out commit, then mark it
git bisect good    # if this commit is clean
git bisect bad     # if this commit is broken

# Repeat step 4 until Git announces the culprit:
# a3f8c21 is the first bad commit
# commit a3f8c21
# Author: Ada Lovelace ...

# Step 5 — return to your original HEAD
git bisect reset
```

### Skip a commit

If a particular commit cannot be tested (e.g. it does not compile), skip it:

```bash
git bisect skip
```

Git will work around it, though the identified first-bad commit may be slightly imprecise.

### Automated bisect with a test script

If you have a script that returns exit code `0` for a good commit and non-zero for a bad one, bisect can run entirely automatically:

```bash
git bisect start
git bisect bad HEAD
git bisect good v2.0.0
git bisect run npm test              # or any shell script
# Git runs the script at each midpoint and marks good/bad automatically
# When done, announces the first bad commit and stops
git bisect reset
```

The script must be **non-interactive** and must exit `0` for good commits, `1`–`127` (excluding `125`) for bad commits. Exit code `125` means "skip this commit".

> **Further reading:** [`git bisect` documentation](https://git-scm.com/docs/git-bisect)

---

## Quick Reference

### Useful `git log` invocations

| Command | What it shows |
|---|---|
| `git log --oneline` | Compact one-line history |
| `git log --oneline --graph --all --decorate` | Full visual branch graph |
| `git log --stat` | Files and line counts per commit |
| `git log -p -n 5` | Full patch for last 5 commits |
| `git log --author="Name"` | Commits by a specific author |
| `git log --grep="keyword"` | Commits whose message matches |
| `git log -S "string"` | Commits that added/removed a string |
| `git log --follow -- file` | History of a file across renames |
| `git log main..branch` | Commits in branch not yet in main |

### Tool selection guide

| Question | Tool |
|---|---|
| What commits exist? What happened when? | `git log` |
| What changed between two points in time? | `git diff <a> <b>` |
| What does this specific commit look like? | `git show <sha>` |
| Who wrote this line of code? | `git blame <file>` |
| Which commit introduced this bug? | `git bisect` |

---

*Previous: [Chapter 09 — Stashing](ch09-stashing.md)* · *Next: [Chapter 11 — Branches and HEAD](../part3/ch11-branches-head.md)*
