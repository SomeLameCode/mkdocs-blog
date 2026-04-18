---
title: "Chapter 27 — Git Aliases & Productivity Tips"
description: "Chapter 27 — Git Aliases & Productivity Tips."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - workflows
status: published
---

# Chapter 27 — Git Aliases & Productivity Tips

Git's command-line interface is consistent and precise, but many common operations involve typing the same long commands repeatedly. **Aliases** let you define shorthand for any Git command or sequence of commands. Beyond aliases, a handful of configuration settings and lesser-known features can meaningfully reduce the friction in everyday Git use.

---

## Git Aliases

An alias maps a short name to any Git command (with arguments). Aliases are stored in your Git configuration and can be defined at any scope — local (repository), global (your account), or system-wide.

### Creating aliases

```bash
# Global alias (stored in ~/.gitconfig)
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
```

After creating these, `git st` runs `git status`, `git co main` runs `git checkout main`, and so on.

### Editing aliases directly

All aliases live under `[alias]` in `~/.gitconfig`. You can edit the file directly for bulk changes:

```ini
[alias]
    st   = status
    co   = checkout
    br   = branch
    ci   = commit
    sw   = switch
    re   = restore
    lg   = log --oneline --graph --decorate --all
    last = log -1 HEAD --stat
    unstage = restore --staged
    undo    = reset --soft HEAD~1
    aliases = config --get-regexp alias
```

### Useful aliases

#### Compact, visual log

```bash
git config --global alias.lg \
  "log --oneline --graph --decorate --all"
```

`git lg` produces a compact graph of all branches — far more readable than the default `git log` for understanding repository structure.

#### Log with file stats

```bash
git config --global alias.ll \
  "log --oneline --stat"
```

Shows each commit on one line followed by the files changed — useful for a quick scan of recent work.

#### Show the last commit

```bash
git config --global alias.last "log -1 HEAD --stat"
```

#### Unstage a file

```bash
git config --global alias.unstage "restore --staged"
# Usage: git unstage src/auth.js
```

#### Soft undo (keep changes staged)

```bash
git config --global alias.undo "reset --soft HEAD~1"
# Undoes the last commit but leaves the changes staged
```

#### List all aliases

```bash
git config --global alias.aliases "config --get-regexp alias"
# git aliases  →  lists every alias you have defined
```

#### Amend without editing the message

```bash
git config --global alias.amend "commit --amend --no-edit"
# git amend  →  adds staged changes to the last commit
```

### Shell command aliases (with `!`)

Prefixing an alias value with `!` runs it as a shell command rather than a Git subcommand. This unlocks arbitrary scripting:

```bash
# Open the repository root in VS Code
git config --global alias.code '!code $(git rev-parse --show-toplevel)'

# Push and immediately open a PR in the browser
git config --global alias.pr '!git push -u origin HEAD && gh pr create --web'

# Delete all fully merged local branches
git config --global alias.cleanup \
  '!git branch --merged main | grep -v "^\* \|main\|master\|develop" | xargs git branch -d'
```

---

## Useful Configuration Settings

Beyond aliases, these `git config` settings change default behaviour in ways that reduce friction.

### Auto-setup remote tracking

When you push a new branch, automatically set up the upstream tracking without `-u`:

```bash
git config --global push.autoSetupRemote true
# Now: git push  (on a new branch)  just works
```

### Default branch name

Set the default branch name for new repositories:

```bash
git config --global init.defaultBranch main
```

### Pull with rebase by default

Avoid merge commits when pulling:

```bash
git config --global pull.rebase true
```

### Prune stale remote-tracking branches on fetch

```bash
git config --global fetch.prune true
```

### Auto-stash during rebase

Automatically stash and restore uncommitted changes when rebasing:

```bash
git config --global rebase.autoStash true
```

### Better diffs with word-level highlighting

```bash
git config --global diff.colorMoved zebra
# Highlights moved code differently from new code
```

### Rerere — reuse recorded resolutions

`rerere` (Reuse Recorded Resolution) records how you resolve merge conflicts and automatically applies the same resolution if the same conflict appears again:

```bash
git config --global rerere.enabled true
```

Particularly useful on long-running branches with frequent rebases — you resolve each conflict once, and Git handles it automatically thereafter.

### Credential caching

Avoid re-entering credentials on every push (HTTPS users):

```bash
# Cache for 1 hour (3600 seconds)
git config --global credential.helper 'cache --timeout=3600'

# macOS keychain (persists indefinitely)
git config --global credential.helper osxkeychain

# Windows credential manager
git config --global credential.helper manager
```

---

## Productive Command Patterns

### Amend the last commit

Add forgotten files or fix the commit message without creating a new commit:

```bash
git add forgotten-file.js
git commit --amend         # opens editor for message
git commit --amend --no-edit  # keep existing message
```

### Stage interactively

Review each changed hunk before staging — useful when a file contains both related and unrelated changes:

```bash
git add -p               # patch mode: y/n/s/e for each hunk
git add -i               # full interactive mode
```

### Partial stash

Stash only specific files, leaving others in the working directory:

```bash
git stash push src/auth.js -m "Auth changes only"
```

### Cherry-pick a single commit

Apply a specific commit from any branch onto the current branch:

```bash
git cherry-pick a3f8c21
git cherry-pick a3f8c21..f4e5d6c   # range (exclusive start)
git cherry-pick a3f8c21^..f4e5d6c  # range (inclusive start)
```

Useful for pulling a hotfix from a feature branch without merging the whole branch.

### Find when a bug was introduced — `git bisect`

Binary search through history to find the commit that introduced a regression:

```bash
git bisect start
git bisect bad               # current commit is broken
git bisect good v1.0.0       # v1.0.0 was known good
# Git checks out a midpoint commit — test it, then:
git bisect good              # or: git bisect bad
# Repeat until Git identifies the first bad commit
git bisect reset             # return to HEAD when done
```

Automate with a test script (exit 0 = good, non-zero = bad):

```bash
git bisect run npm test
```

### See what changed in a range

```bash
git log main..feature-branch   # commits in feature-branch not in main
git diff main...feature-branch # diff of the changes feature-branch introduces
                               # (triple dot: diff from common ancestor)
```

The three-dot `...` syntax is particularly useful in code review — it shows only the changes the branch introduces, not unrelated commits merged from main.

### Recover a deleted branch

If you accidentally delete a branch, `git reflog` still has the SHA:

```bash
git reflog | grep "branch-name"
git checkout -b branch-name <sha>
```

### Show which branch contains a commit

```bash
git branch --contains a3f8c21      # local branches
git branch -r --contains a3f8c21  # remote-tracking branches
```

### List files changed in a commit

```bash
git show --name-only a3f8c21      # names only
git show --name-status a3f8c21    # with A/M/D status
git diff-tree --no-commit-id -r --name-only a3f8c21  # names only, no header
```

---

## `.gitconfig` Template

A starting-point `~/.gitconfig` combining the above:

```ini
[user]
    name  = Your Name
    email = you@example.com

[init]
    defaultBranch = main

[push]
    autoSetupRemote = true

[pull]
    rebase = true

[fetch]
    prune = true

[rebase]
    autoStash = true

[rerere]
    enabled = true

[diff]
    colorMoved = zebra

[alias]
    st       = status -sb
    sw       = switch
    co       = checkout
    br       = branch -vv
    ci       = commit
    lg       = log --oneline --graph --decorate --all
    ll       = log --oneline --stat
    last     = log -1 HEAD --stat
    unstage  = restore --staged
    undo     = reset --soft HEAD~1
    amend    = commit --amend --no-edit
    aliases  = config --get-regexp alias
    cleanup  = !git branch --merged main | grep -v \"^\\* \\|main\\|master\\|develop\" | xargs git branch -d
```

---

## GitHub CLI Productivity

The `gh` CLI (installed separately — see Chapter 4) rounds out the command-line workflow:

```bash
gh repo clone <owner>/<repo>     # clone without constructing the URL
gh pr create --fill               # open PR with commit title/body pre-filled
gh pr checkout 42                 # check out PR #42 locally
gh pr merge --squash --delete-branch  # merge and delete in one command
gh issue list --assignee @me      # your open issues
gh run list                       # list recent Actions runs
gh run watch                      # watch a running workflow in the terminal
```

```bash
# Create an alias for common gh patterns
gh alias set prc 'pr create --fill --web'
# gh prc  →  opens the PR creation page with title pre-filled
```

---

## Summary

- Aliases are defined with `git config --global alias.<name> <command>` or by editing `~/.gitconfig` directly.
- Essential aliases: `lg` (visual log), `unstage`, `undo`, `amend`, `last`, `aliases`.
- Shell aliases (`!` prefix) enable arbitrary scripting — branch cleanup, push-and-PR, opening the repo in an editor.
- Key config settings: `push.autoSetupRemote`, `pull.rebase`, `fetch.prune`, `rebase.autoStash`, `rerere.enabled`.
- Productive patterns: `git add -p` for selective staging, `git cherry-pick` for single commits, `git bisect run` for automated regression hunting, triple-dot diff for PR review.
- The `gh` CLI complements `git` for GitHub-specific tasks — PR creation, checkout, merge, issue listing, and Actions monitoring.

> **Further reading:** [Git configuration documentation](https://git-scm.com/docs/git-config) · [gh CLI manual](https://cli.github.com/manual/)

---

*Previous: [Chapter 26 — Git Workflows](ch26-workflows.md)* · *Next: [Chapter 28 — Git Object Model: Blobs, Trees & Commits](../part7/ch28-object-model.md)*
