---
title: "Chapter 14 — Interactive Rebase"
description: "Chapter 14 — Interactive Rebase."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - branching
status: published
---

# Chapter 14 — Interactive Rebase

Standard `git rebase` (Chapter 13) moves a branch onto a new base automatically. **Interactive rebase** — `git rebase -i` — goes further: before replaying any commits, it opens an editor and lets you decide what to do with each one. You can reorder, combine, split, rename, or drop commits entirely. It is the primary tool for polishing local history before sharing.

---

## Launching Interactive Rebase

The most common invocation uses `HEAD~N` to select the last *N* commits on the current branch:

```bash
git rebase -i HEAD~4    # open the last 4 commits for editing
```

You can also specify any commit SHA as the base — Git will present all commits *after* that SHA up to HEAD:

```bash
git rebase -i a3f8c21   # edit everything after commit a3f8c21
```

To rewrite the entire branch history from the root commit:

```bash
git rebase -i --root
```

> **Note:** `HEAD~N` refers to the *parent* of the range you want to edit. To edit the last 4 commits, you pass `HEAD~4` (the 5th-most-recent commit), and Git presents commits 4 through 1 in the editor.

---

## The To-Do List

When you run `git rebase -i`, Git opens your configured editor with a list like this:

```
pick a3f8c21 Add login form
pick 7b9d344 Fix typo in login form
pick c1e0f52 Add logout button
pick 88edcd9 Fix logout button styling

# Rebase f7a1b30..88edcd9 onto f7a1b30 (4 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# d, drop <commit> = remove commit
# x, exec <command> = run command (the rest of the line) using shell
```

Commits are listed oldest-first (top = oldest, bottom = newest). Each line begins with a **command** followed by the commit SHA and message.

To change the plan, edit the file — change commands, reorder lines, or delete lines — then save and close. Git executes the plan top to bottom.

---

## Commands Reference

| Command | Shorthand | Effect |
|---|---|---|
| `pick` | `p` | Keep the commit as-is |
| `reword` | `r` | Keep the commit but open the editor to change its message |
| `edit` | `e` | Pause after applying the commit; lets you amend it or add extra commits |
| `squash` | `s` | Combine with the commit above; opens editor to merge both messages |
| `fixup` | `f` | Combine with the commit above; silently discard this commit's message |
| `drop` | `d` | Remove the commit entirely |
| `exec` | `x` | Run a shell command at that point in the replay |

---

## Rewriting Commit Messages — `reword`

Change `pick` to `reword` on any commit whose message needs fixing:

```
pick   a3f8c21 Add login form
reword 7b9d344 Fix typo in login form     ← will pause to edit message
pick   c1e0f52 Add logout button
```

Git replays all commits silently until it reaches the `reword` line, then opens the editor with that commit's message. Edit it, save, and the rebase continues.

---

## Combining Commits — `squash` and `fixup`

To collapse multiple commits into one, move the commits to be merged below the commit they should merge into, and mark them `squash` or `fixup`:

```
pick   a3f8c21 Add login form
squash 7b9d344 Fix typo in login form     ← merged into a3f8c21
pick   c1e0f52 Add logout button
fixup  88edcd9 Fix logout button styling  ← merged into c1e0f52, message discarded
```

**`squash`** opens the editor with both commit messages concatenated, letting you write a combined message.

**`fixup`** silently discards the second commit's message — useful for cleanup commits that add nothing to the log.

The result is two commits instead of four, with a clean message for each.

---

## Reordering Commits

Reorder lines in the to-do file to change commit order:

```
# Before
pick a3f8c21 Add login form
pick c1e0f52 Add logout button
pick 7b9d344 Fix login form typo

# After (move the typo fix up)
pick a3f8c21 Add login form
pick 7b9d344 Fix login form typo
pick c1e0f52 Add logout button
```

Git will replay the commits in the new order. If the reordering creates a conflict (e.g. the typo fix depends on code that hasn't been applied yet), Git pauses and asks you to resolve it, then `git rebase --continue`.

---

## Dropping Commits — `drop`

Delete a line, or change `pick` to `drop`, to remove a commit entirely:

```
pick   a3f8c21 Add login form
drop   7b9d344 Debugging log — do not ship   ← removed from history
pick   c1e0f52 Add logout button
```

> **Warning:** Dropped commits are removed from history. Any code introduced only in those commits will not appear in subsequent commits. Make sure you are not dropping code you need.

---

## Splitting a Commit — `edit`

`edit` pauses the rebase after applying a commit, leaving the changes staged. You can then un-stage some of them, commit what remains, stage the rest, and commit again — effectively splitting one commit into two.

```
pick   a3f8c21 Add login form
edit   c1e0f52 Add auth and logout button   ← pause here to split
pick   88edcd9 Polish styles
```

When Git pauses at the `edit` commit:

```bash
# Un-stage the logout button changes
git restore --staged src/logout.js

# Commit the auth changes alone
git commit -m "Add authentication module"

# Now stage and commit the logout button
git add src/logout.js
git commit -m "Add logout button"

# Resume the rebase
git rebase --continue
```

The original single commit is replaced by two separate commits.

---

## Amending During `edit`

`edit` also lets you simply amend the commit — add more files, change the message, or adjust the diff — before continuing:

```bash
# After Git pauses at an edit commit:
git add forgotten-file.js
git commit --amend --no-edit   # add the file to the existing commit
git rebase --continue
```

---

## `--autosquash` — Automating Squash Targets

If you name a commit with the prefix `fixup!` or `squash!` followed by a message that matches an earlier commit, `git rebase -i --autosquash` will automatically position and mark that commit for fixup/squash:

```bash
# Create a fixup commit targeting an earlier commit
git commit -m "fixup! Add login form"

# Later, when rebasing:
git rebase -i --autosquash HEAD~5
```

Git pre-populates the to-do list with the fixup commit already moved below its target and marked `fixup`. You only need to save and close the editor.

This is commonly combined with `git commit --fixup=<sha>`, which creates the `fixup!`-prefixed commit automatically:

```bash
git add src/login.js
git commit --fixup=a3f8c21    # creates "fixup! Add login form"
```

---

## `exec` — Run Commands During Replay

The `exec` command runs a shell command at that point in the replay. Useful for running tests after each commit to find which commit broke the build:

```
pick a3f8c21 Add login form
exec npm test
pick 7b9d344 Fix typo in login form
exec npm test
pick c1e0f52 Add logout button
exec npm test
```

If any `exec` command exits with a non-zero status, the rebase pauses so you can investigate.

---

## Handling Conflicts

Interactive rebase replays commits one at a time, so conflicts are resolved the same way as standard rebase:

```bash
# Git pauses on a conflict
# 1. Edit the conflicted file(s)
git add <resolved-file>
git rebase --continue

# Skip a commit that becomes empty after resolution:
git rebase --skip

# Abandon the entire rebase:
git rebase --abort
```

---

## The Golden Rule Still Applies

Interactive rebase rewrites history just like standard rebase — every affected commit gets a new SHA. **Never interactive-rebase commits that have already been pushed to a shared branch.** It rewrites history that your teammates have already fetched, causing divergence and requiring force-pushes.

Safe uses:
- Polishing commits on a local branch before opening a pull request
- Cleaning up a branch that only exists on your machine

---

## Summary

- `git rebase -i HEAD~N` opens the last N commits in an editor to rewrite, reorder, combine, or drop them.
- **`reword`** — change a commit message without altering the diff.
- **`squash` / `fixup`** — combine commits; squash merges messages, fixup discards the secondary message.
- **`edit`** — pause at a commit to amend it or split it into multiple commits.
- **`drop`** — remove a commit from history entirely.
- **`exec`** — run a shell command during the replay (e.g. run tests after each commit).
- `--autosquash` with `fixup!`/`squash!` commit prefixes (or `git commit --fixup=<sha>`) automates squash targeting.
- The golden rule applies: never interactive-rebase commits already pushed to a shared branch.

---

*Previous: [Chapter 13 — Rebasing](ch13-rebasing.md)* · *Next: [Chapter 15 — Tags & Semantic Versioning](ch15-tags.md)*

