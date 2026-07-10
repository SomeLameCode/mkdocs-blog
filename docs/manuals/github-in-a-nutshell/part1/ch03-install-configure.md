---
title: "Chapter 03 — Installing & Configuring Git"
description: "Chapter 03 — Installing & Configuring Git."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - getting-started
status: published
---

# Chapter 03 — Installing & Configuring Git

Before you can use Git, you need to install it and tell it who you are. This chapter covers installation on all major operating systems and walks through the essential first-time configuration so that your commits are correctly attributed and your environment behaves predictably.

---

## Installing Git

### Windows

The recommended approach on Windows is **Git for Windows**, which installs Git along with Git Bash — a bash-compatible terminal emulator that provides the Unix-style shell environment used in this manual.

1. Download the installer from [https://git-scm.com/download/win](https://git-scm.com/download/win).
2. Run the installer. The defaults are sensible for most users; the only setting worth reviewing is the default editor (covered below).
3. After installation, open **Git Bash** from the Start menu to verify:

```bash
git --version
# git version 2.x.x.windows.x
```

> **Windows Terminal + Git Bash:** If you prefer Windows Terminal, you can add Git Bash as a profile. This gives you a modern terminal interface with tabs and GPU rendering while still using Git Bash as the shell.

### macOS

macOS ships with a system Git (installed as part of Xcode Command Line Tools), but it is often outdated. The cleanest approach is to install Git via **Homebrew**:

```bash
# Install Homebrew if you don't have it (see https://brew.sh)
brew install git
```

After installation, open a new terminal and verify:

```bash
git --version
# git version 2.x.x
```

If you see an older version number, your shell may still be finding the system Git. Check with `which git` — it should return `/usr/local/bin/git` (Intel) or `/opt/homebrew/bin/git` (Apple Silicon), not `/usr/bin/git`.

### Linux

On Debian/Ubuntu-based distributions:

```bash
sudo apt update
sudo apt install git
```

On Fedora/RHEL-based distributions:

```bash
sudo dnf install git
```

On Arch Linux:

```bash
sudo pacman -S git
```

Verify after installation:

```bash
git --version
```

> **Further reading:** The official Git downloads page at [https://git-scm.com/downloads](https://git-scm.com/downloads) covers additional platforms and package manager options.

---

## First-Time Configuration

Git stores configuration in plain text files. Before making your first commit, you must set your **name** and **email address** — Git attaches these to every commit you create.

### Your Identity

```bash
git config --global user.name "Alice Nguyen"
git config --global user.email "alice@example.com"
```

Use the `--global` flag to apply these settings to every repository on your machine. You can override them per-repository if needed (see [Configuration Scopes](#configuration-scopes) below).

> The email you configure here will appear in your commit history. If your repository is public and you want to keep your address private, GitHub provides a no-reply email address (`<username>@users.noreply.github.com`) that you can use instead — find it in your GitHub account settings under *Emails*.

### Default Branch Name

When you run `git init`, Git creates an initial branch. Historically this was named `master`; the current convention (and GitHub's default) is `main`. Set this now to avoid mismatches:

```bash
git config --global init.defaultBranch main
```

### Default Editor

Git opens a text editor for certain operations — most notably when you run `git commit` without a `-m` message flag, or during an interactive rebase. By default Git uses `vim`, which has a steep learning curve if you are not familiar with it.

To use **nano** instead:

```bash
git config --global core.editor nano
```

To use **Visual Studio Code**:

```bash
git config --global core.editor "code --wait"
```

The `--wait` flag tells Git to pause until you close the VS Code tab, rather than treating the editor as immediately finished.

### Line Endings

Windows and Unix-like systems use different line ending characters (`\r\n` vs `\n`). If left unconfigured, this causes noisy diffs in cross-platform teams. The `core.autocrlf` setting controls how Git handles line endings:

| Platform | Recommended setting | Effect |
|---|---|---|
| Windows | `true` | Convert `LF → CRLF` on checkout; `CRLF → LF` on commit |
| macOS / Linux | `input` | Convert `CRLF → LF` on commit; no conversion on checkout |

```bash
# Windows
git config --global core.autocrlf true

# macOS / Linux
git config --global core.autocrlf input
```

> For teams working across platforms, an alternative is to commit a `.gitattributes` file that declares line endings per file type — this is the more robust solution and is covered in [Chapter 07](../part2/ch07-gitignore.md).

---

## Configuration Scopes

Git has three configuration scopes, each stored in a different file:

| Scope | Flag | File location | Applies to |
|---|---|---|---|
| System | `--system` | `/etc/gitconfig` (Linux/macOS) or `C:\Program Files\Git\etc\gitconfig` (Windows) | Every user on the machine |
| Global | `--global` | `~/.gitconfig` | All repositories for the current user |
| Local | `--local` | `.git/config` inside the repository | That repository only |

When Git reads a configuration value, it checks in the order **local → global → system**. Local settings always win.

**Practical use of local config:** If you maintain both personal and work repositories on the same machine, you can set your work email locally in each work repository without changing the global default:

```bash
# Inside a work repository only
git config --local user.email "alice.nguyen@company.com"
```

---

## Viewing Your Configuration

### List all settings

```bash
git config --list
```

This prints every setting Git can see, along with which file each one came from. If a key appears multiple times, the last one listed (lowest scope = most local) wins.

### Show a specific setting

```bash
git config user.name
git config user.email
git config --global core.editor
```

### Show which file a setting comes from

```bash
git config --list --show-origin
```

This is useful for diagnosing unexpected settings — you can see exactly which config file is responsible.

---

## The `.gitconfig` File

All global settings are stored as plain text in `~/.gitconfig` (your home directory). You can view and edit it directly:

```bash
cat ~/.gitconfig
```

A typical `.gitconfig` after following this chapter looks like:

```ini
[user]
    name = Alice Nguyen
    email = alice@example.com
[init]
    defaultBranch = main
[core]
    editor = code --wait
    autocrlf = input
```

You can edit this file manually in any text editor — the format is straightforward. Any mistake will cause Git to report a parse error with the file and line number.

---

## Useful Aliases (Optional)

Git allows you to define short aliases for commands you use frequently. While [Chapter 27](../part6/ch27-aliases-tips.md) covers aliases in depth, a few useful ones are worth setting up now:

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.lg "log --oneline --graph --decorate"
```

After setting these, `git st` runs `git status`, `git lg` shows a compact decorated graph log, and so on.

---

## Quick Reference

| Task | Command |
|---|---|
| Check Git version | `git --version` |
| Set your name | `git config --global user.name "Your Name"` |
| Set your email | `git config --global user.email "you@example.com"` |
| Set default branch to `main` | `git config --global init.defaultBranch main` |
| Set editor to nano | `git config --global core.editor nano` |
| Set editor to VS Code | `git config --global core.editor "code --wait"` |
| List all settings | `git config --list` |
| List settings with file origins | `git config --list --show-origin` |
| Show one setting | `git config <key>` |
| Edit global config file directly | `nano ~/.gitconfig` |

---

*Previous: [Chapter 02 — Shell Commands Primer](ch02-shell-commands.md)* · *Next: [Chapter 04 — GitHub Authentication (SSH, PATs, gh CLI)](ch04-github-authentication.md)*
