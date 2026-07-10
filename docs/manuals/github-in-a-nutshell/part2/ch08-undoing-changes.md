---
title: "Chapter 08 — Undoing Changes"
description: "Chapter 08 — Undoing Changes."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - core-workflow
status: published
---

# Chapter 08 — Undoing Changes

Mistakes happen. You commit with a typo in the message, stage a file you did not mean to, or realise a whole commit introduced a bug. Git provides several distinct mechanisms for undoing work, each suited to a different scenario. This chapter covers them all, explains what each command actually does to your repository, and flags clearly which operations are irreversible.

---

## Four Undo Scenarios

| Scenario | Right tool |
|---|---|
| Fix the message or content of the last commit (not yet pushed) | `git commit --amend` |
| Unstage a file you accidentally staged | `git restore --staged` |
| Discard unsaved edits in the working directory | `git restore` |
| Undo one or more commits on a shared branch | `git revert` |
| Undo one or more commits on a local (unpublished) branch | `git reset` |
| Recover a commit lost to a hard reset | `git reflog` |

---

## Amending the Last Commit

`git commit --amend` replaces the most recent commit with a new one. It is useful for two situations: correcting a commit message, or including a file you forgot to stage.

### Fix a commit message

```bash
git commit --amend -m "Correct commit message here"
```

### Add a forgotten file

```bash
# Stage the file you forgot
git add forgotten-file.js

# Amend — opens your editor with the previous message pre-filled
git commit --amend

# Or amend without changing the message
git commit --amend --no-edit
```

> **Warning:** `--amend` rewrites history — it creates a new commit object with a different SHA. If you have already pushed the commit to a shared branch, amending and force-pushing will cause problems for anyone who has fetched that commit. Only amend commits that exist solely on your local machine.

---

## Undoing Staged and Working Directory Changes

These two scenarios are covered in detail in [Chapter 06 — Tracking Files & File States](ch06-file-states.md). In brief:

```bash
# Unstage a file (keep working directory changes)
git restore --staged <file>

# Discard working directory changes (irreversible)
git restore <file>
```

The rest of this chapter focuses on undoing *committed* work.

---

## `git reset` — Rewinding the Branch Pointer

`git reset` moves the current branch pointer (and HEAD) backwards to an earlier commit. It is a history-rewriting operation and should only be used on commits that have not been shared with others.

### Understanding the three areas

To understand reset's modes, recall the three areas from Chapter 05: the working directory, the staging area (index), and the repository. Each mode of reset affects a different combination of these areas.

### The three modes

```bash
git reset --soft  <target>   # moves HEAD only
git reset --mixed <target>   # moves HEAD + resets index  (default)
git reset --hard  <target>   # moves HEAD + resets index + resets working directory
```

| Mode | HEAD moved | Index reset | Working dir reset | Changes end up… |
|---|---|---|---|---|
| `--soft` | Yes | No | No | Still staged |
| `--mixed` | Yes | Yes | No | Unstaged (in working dir) |
| `--hard` | Yes | Yes | Yes | **Gone — unrecoverable** |

### Specifying the target

```bash
git reset HEAD~1    # one commit back (default: --mixed)
git reset HEAD~3    # three commits back
git reset abc1234   # to a specific commit SHA
```

`HEAD~1` means "the parent of the current commit". `HEAD~2` means the grandparent, and so on.

### When to use each mode

**`--soft`** — undo the commit but keep all changes staged, ready to be re-committed. Useful for combining several small commits into one before pushing.

```bash
# Undo the last commit; changes remain staged
git reset --soft HEAD~1
# Now you can adjust and re-commit
git commit -m "Better commit message with all changes"
```

**`--mixed`** (default) — undo the commit and unstage the changes. The files are intact in the working directory; you can review, re-stage selectively, and recommit.

```bash
git reset HEAD~1
# Changes are unstaged but present — inspect with git diff
git add specific-file.js
git commit -m "Re-commit with correct scope"
```

**`--hard`** — undo the commit and throw away all changes entirely. The working directory is restored to the state of the target commit.

```bash
# Permanently discard the last commit and all its changes
git reset --hard HEAD~1
```

> **Warning:** `git reset --hard` is **irreversible** through normal Git commands. Any uncommitted changes — staged or unstaged — are permanently lost. The only recovery path is `git reflog`, which only works if the lost commits are still in Git's internal reference log (see below).

> **Never reset commits that have been pushed to a shared branch.** Doing so rewrites public history and forces other contributors to reconcile their local copies. Use `git revert` instead.

> **Further reading:** [`git reset` documentation](https://git-scm.com/docs/git-reset)

---

## `git revert` — Safe Undo for Shared History

`git revert` creates a new commit that *applies the inverse* of a previous commit. The original commit remains in the history — it is never deleted or moved. This makes `git revert` safe to use on branches that other people have fetched.

```bash
# Revert the most recent commit
git revert HEAD

# Revert a specific commit by SHA
git revert a3f8c21

# Revert without immediately committing (stage the inverse changes only)
git revert --no-commit HEAD
```

When you run `git revert`, Git opens your editor with a pre-filled commit message describing what is being reverted. Save and close to complete the revert.

### Reverting a range of commits

```bash
# Revert commits from (exclusive) to (inclusive)
git revert abc123..def456
```

This creates one revert commit per reverted commit, in reverse chronological order. Add `--no-commit` to stage all the inverse changes first and then make a single revert commit manually.

### `reset` vs `revert` — when to use which

| Situation | Use |
|---|---|
| Commit not yet pushed; want to remove it cleanly | `git reset` |
| Commit already pushed to a shared branch | `git revert` |
| Need to preserve full audit trail | `git revert` |
| Working alone on a local feature branch | Either — `reset` is cleaner |

> **Further reading:** [`git revert` documentation](https://git-scm.com/docs/git-revert)

---

## `git reflog` — The Safety Net

The **reflog** (reference log) is a local diary of every position HEAD has pointed to. It records commits, merges, rebases, resets — everything. Even after a `git reset --hard`, the reflog still knows where HEAD was before.

```bash
git reflog
# d3a1f02 (HEAD -> main) HEAD@{0}: reset: moving to HEAD~2
# 7b4c91e HEAD@{1}: commit: Add feature X
# 3e8f20a HEAD@{2}: commit: Fix typo in README
# ...
```

Each line shows the commit SHA, a relative reference (`HEAD@{N}`), and what action moved HEAD there.

### Recovering a lost commit

```bash
# Find the SHA of the commit you want to recover
git reflog

# Option 1 — create a new branch pointing to that commit
git checkout -b recovery-branch 7b4c91e

# Option 2 — hard reset back to that commit (on the current branch)
git reset --hard 7b4c91e
```

> **Reflog entries expire** after 90 days by default (`gc.reflogExpire`). Recovery is only possible within that window, and only on your local machine — reflog is not pushed to remotes.

> **Further reading:** [`git reflog` documentation](https://git-scm.com/docs/git-reflog)

---

## Decision Guide

Use this table when you need to undo something and are not sure which tool to reach for.

| What do you want to undo? | Commit pushed? | Use |
|---|---|---|
| Typo in the last commit message | No | `git commit --amend` |
| Forgot to include a file in the last commit | No | `git add <file>` + `git commit --amend --no-edit` |
| Last commit — keep changes staged | No | `git reset --soft HEAD~1` |
| Last commit — keep changes unstaged | No | `git reset HEAD~1` (mixed) |
| Last commit — discard changes entirely | No | `git reset --hard HEAD~1` ⚠️ |
| Several local commits — squash into one | No | `git reset --soft HEAD~N` + recommit |
| A commit on a shared branch | Yes | `git revert <sha>` |
| Accidentally staged a file | Either | `git restore --staged <file>` |
| Unsaved edits in working directory | Either | `git restore <file>` ⚠️ |
| Hard-reset too aggressively | Local only | `git reflog` → `git reset --hard <sha>` |

---

## Safety Summary

| Operation | Reversible? | How |
|---|---|---|
| `git commit --amend` (before push) | Yes | `git reflog` to find original SHA |
| `git reset --soft` | Yes | `git reflog` |
| `git reset --mixed` | Yes | `git reflog` |
| `git reset --hard` | **Rarely** | Only via `git reflog`, within 90 days |
| `git restore <file>` | **No** | Changes were never committed |
| `git revert` | Yes | Revert the revert |

The golden rule: if a commit exists in Git's history, `git reflog` can usually get it back — within the expiry window. If a change was never committed, it is gone forever.

---

*Previous: [Chapter 07 — Ignoring Files (.gitignore)](ch07-gitignore.md)* · *Next: [Chapter 09 — Stashing](ch09-stashing.md)*
