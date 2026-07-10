---
title: "Chapter 09 — Stashing"
description: "Chapter 09 — Stashing."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - core-workflow
status: published
---

# Chapter 09 — Stashing

You are mid-way through a feature when an urgent bug report lands. Your working directory has unsaved edits, some files are staged, and nothing is ready to commit. You need to switch branches immediately — but you do not want to commit half-finished work or throw it away.

**Git stash** is the solution. It shelves your current dirty state onto a stack, leaving the working directory clean, so you can switch context freely and restore your work later.

---

## How Stashing Works

When you run `git stash`, Git takes everything in your working directory and staging area that differs from the last commit, saves it as a stash entry, and resets both areas to match `HEAD`. The stash is stored locally in `.git/refs/stash` — it is not pushed to remotes.

The stash behaves like a stack: the most recent entry is `stash@{0}`, the one before it is `stash@{1}`, and so on.

---

## Basic Stash Operations

### Save dirty state

```bash
git stash
# Saved working directory and index state WIP on main: a3f8c21 Last commit message
```

`git stash` is shorthand for `git stash push`. Both do the same thing.

### List all stash entries

```bash
git stash list
# stash@{0}: WIP on main: a3f8c21 Last commit message
# stash@{1}: WIP on feature-x: 7b4c91e Previous work
```

### Apply the most recent stash and remove it

```bash
git stash pop
```

Applies `stash@{0}` to the working directory and removes it from the stash stack. If there is a conflict between the stash and the current state of the branch, Git pauses and asks you to resolve it — the stash entry is **not** removed until the conflict is resolved.

### Apply without removing from the stack

```bash
git stash apply           # applies stash@{0}, keeps it in the list
git stash apply stash@{2} # applies a specific entry
```

Useful when you want to apply the same stash to multiple branches, or when you are unsure whether the apply will succeed cleanly.

### Delete a specific stash entry

```bash
git stash drop            # drops stash@{0}
git stash drop stash@{1}  # drops a specific entry
```

### Delete all stash entries

```bash
git stash clear
```

> **Warning:** `git stash clear` permanently deletes all stash entries. There is no recovery path.

> **Further reading:** [`git stash` documentation](https://git-scm.com/docs/git-stash)

---

## Stashing with a Message

By default, Git generates an automatic stash message based on the branch and last commit. When you have multiple stashes, these auto-messages become hard to distinguish. Use `-m` to label them clearly:

```bash
git stash push -m "WIP: refactor user auth middleware"
git stash push -m "experiment: alternate layout approach"

git stash list
# stash@{0}: On main: experiment: alternate layout approach
# stash@{1}: On main: WIP: refactor user auth middleware
```

---

## Stashing Untracked and Ignored Files

By default, `git stash` only saves changes to **tracked** files — both staged and unstaged modifications. New files that have never been committed are left behind in the working directory.

### Include untracked files

```bash
git stash push -u
# equivalent: git stash push --include-untracked
```

This stashes tracked changes *and* new untracked files. Untracked files are removed from the working directory as part of the stash.

### Include ignored files too

```bash
git stash push -a
# equivalent: git stash push --all
```

Stashes everything — tracked changes, untracked files, and files matched by `.gitignore`. Rarely needed; use with care since it can sweep up build artefacts and generated files.

---

## Stashing Only the Unstaged Changes

Sometimes you have staged changes you want to commit immediately and unstaged changes you want to set aside. `--keep-index` stashes the unstaged portion while leaving the staging area intact:

```bash
git stash push --keep-index
```

After this:
- Your **staged changes** remain in the index, ready to commit.
- Your **unstaged changes** are stashed and removed from the working directory.

Commit the staged work, then `git stash pop` to bring back the rest.

---

## Applying a Stash to a Different Branch

The stash is branch-agnostic — you can pop or apply it on any branch, not just the one where it was created:

```bash
# Stash on feature-branch
git stash push -m "half-done feature"

# Switch to a different branch
git checkout main

# Apply the stash here
git stash pop
```

If the stash applies cleanly, Git merges the changes into the working directory automatically. If there are conflicts (because the target branch has diverged), Git marks the conflicted files and leaves the stash entry in place until you resolve them.

---

## Creating a Branch from a Stash

If your working branch has diverged significantly since you stashed, `git stash pop` may produce many conflicts. The cleaner approach is to create a new branch at the exact commit where the stash was made, apply the stash there, and merge or rebase afterwards:

```bash
git stash branch <new-branch-name>
# equivalent: git stash branch <new-branch-name> stash@{0}
```

This command:
1. Creates `<new-branch-name>` at the commit that was HEAD when the stash was saved.
2. Checks out that branch.
3. Applies the stash.
4. Drops the stash entry (if the apply succeeds).

Because the branch starts at the same commit, the stash always applies without conflicts.

---

## Practical Workflow Example

**Scenario:** You are implementing a new feature when an urgent bug is reported on `main`.

```bash
# You are on feature-login with uncommitted work
git status
# Changes not staged for commit: src/auth.js
# Untracked files: src/auth-helpers.js

# Stash everything, including the new untracked file
git stash push -u -m "WIP: login feature"

# Confirm the working directory is clean
git status
# nothing to commit, working tree clean

# Switch to main and create a hotfix branch
git checkout main
git checkout -b hotfix/null-pointer

# Fix the bug, commit it
git add src/user.js
git commit -m "Fix null pointer in user lookup"

# Merge the fix back to main
git checkout main
git merge hotfix/null-pointer

# Return to the feature branch
git checkout feature-login

# Restore the stashed work
git stash pop
# Restored: src/auth.js and src/auth-helpers.js

# Resume where you left off
```

---

## Stash vs WIP Commit

Both approaches let you park unfinished work. The right choice depends on your situation.

| | `git stash` | WIP commit |
|---|---|---|
| Stored in | Local stash stack only | Branch history |
| Pushed to remote | Never | Yes (with `git push`) |
| Visible to teammates | No | Yes |
| Persists after `git clone` | No | Yes |
| Part of commit history | No | Yes (clean up with `--amend` or rebase later) |
| Best for | Short context switches on one machine | Work you want to back up remotely or share |

A common pattern when collaborating is to commit WIP work with a message like `WIP: do not merge`, push it to a backup branch for safety, then `git reset HEAD~1` or `git commit --amend` to clean it up before merging.

---

## Summary

- `git stash` saves dirty working directory and staging area state to a local stack; `git stash pop` restores it.
- Label stashes with `-m` when you have more than one — the auto-generated names quickly become indistinguishable.
- Use `-u` to include untracked files; `--keep-index` to stash only the unstaged portion.
- `git stash branch` is the safest way to apply a stash when the branch has diverged.
- Stash is local and ephemeral — for anything that needs to survive a machine change or be shared, use a WIP commit instead.

---

*Previous: [Chapter 08 — Undoing Changes](ch08-undoing-changes.md)* · *Next: [Chapter 10 — Inspecting History](ch10-inspecting-history.md)*
