---
title: "Chapter 02 — Shell Commands Primer"
description: "Chapter 02 — Shell Commands Primer."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - getting-started
status: published
---

# Chapter 02 — Shell Commands Primer

Git is a command-line tool at its core. While graphical interfaces and IDE integrations exist, understanding the shell gives you the clearest mental model of what Git is actually doing and the most control over it. This chapter covers the subset of shell commands you need to navigate your filesystem, manage files, and follow along with every example in this manual.

> **Which shell?** On macOS and Linux, the default shell is usually **bash** or **zsh**. On Windows, use **Git Bash** (installed with Git for Windows), which provides a bash-compatible environment. All examples in this chapter work in any of these.

---

## The Shell Prompt

When you open a terminal, you are greeted by a **prompt** — a line indicating that the shell is waiting for your input. It typically shows your username, hostname, and current directory:

```
username@hostname:~/projects$
```

The `$` symbol marks the end of the prompt. Everything you type appears after it. Press **Enter** to execute a command.

---

## Navigating the Filesystem

### `pwd` — Print Working Directory

Before navigating anywhere, it helps to know where you are. `pwd` prints the full path to your current directory:

```bash
pwd
# /home/alice/projects
```

You will reach for `pwd` often when something behaves unexpectedly — confirming your location is the fastest first debugging step.

### `ls` — List Files and Directories

`ls` lists the contents of the current directory:

```bash
ls
# README.md  src  tests  package.json
```

Common flags:

```bash
ls -l       # Long format — shows permissions, owner, size, and modification date
ls -a       # All files — includes hidden files (names starting with .)
ls -la      # Both flags combined
```

Hidden files (`.gitignore`, `.env`, `.git/`) are invisible to plain `ls`. The `-a` flag reveals them.

### `cd` — Change Directory

`cd` moves you into a different directory:

```bash
cd projects          # Move into the 'projects' subdirectory
cd /home/alice       # Move to an absolute path
cd ..                # Move up one level to the parent directory
cd ../sibling        # Move up one level, then into 'sibling'
cd ~                 # Move to your home directory
cd -                 # Return to the previous directory
```

**`.` and `..`** are special directory aliases:

- `.` refers to the **current directory**. You use it when a command requires a path and you mean "here" (e.g. `git init .`).
- `..` refers to the **parent directory**. Chain them to move up multiple levels: `../../` goes up two levels.

---

## Creating Directories and Files

### `mkdir` — Make Directory

`mkdir` creates a new directory:

```bash
mkdir my-project                 # Create a single directory
mkdir -p src/components/ui       # Create nested directories in one step
```

The `-p` flag (parents) is important: without it, `mkdir src/components/ui` fails if `src/` or `src/components/` does not already exist.

### `touch` — Create a New File

`touch` creates an empty file, or updates the modification timestamp of an existing file:

```bash
touch README.md          # Create an empty README.md
touch .gitignore         # Create an empty .gitignore
```

`touch` is the quickest way to create a placeholder file before editing it.

---

## Reading and Writing Files

### `cat` — Display File Contents

`cat` prints the full contents of a file to the terminal:

```bash
cat README.md
cat .gitignore
```

It is most useful for short files. For large files, use a pager like `less` or an editor.

### `echo` — Print to the Terminal

`echo` prints text to the terminal (or, combined with redirection, into a file):

```bash
echo "Hello, world"
# Hello, world

echo $HOME
# /home/alice
```

`echo` is also frequently used in shell scripts and in examples that build configuration files.

### `>` — Write to a File

The `>` operator redirects the output of a command into a file, **overwriting** any existing content:

```bash
echo "# My Project" > README.md
```

After this, `README.md` contains exactly one line: `# My Project`. Any previous content is gone.

### `>>` — Append to a File

The `>>` operator appends output to a file rather than overwriting it:

```bash
echo "node_modules/" >> .gitignore
echo ".env"          >> .gitignore
```

Each `>>` adds a new line at the end of the file without touching existing content.

---

## Editing Files

### `nano` — Command-Line Text Editor

`nano` is a simple, beginner-friendly terminal text editor:

```bash
nano README.md
```

Once inside nano:

| Key | Action |
|---|---|
| `Ctrl + O` | Save (Write Out) |
| `Enter` | Confirm filename when saving |
| `Ctrl + X` | Exit |
| `Ctrl + K` | Cut the current line |
| `Ctrl + U` | Paste a cut line |

The available commands are always displayed at the bottom of the screen, making nano the most approachable choice when you need to make a quick edit without leaving the terminal.

> **Other editors:** Many developers use `vim` or `emacs` in the terminal. Git defaults to `vim` when it opens an editor automatically (e.g. during a rebase). If you prefer nano or VS Code, you can configure Git's default editor — covered in [Chapter 03](ch03-install-configure.md).

---

## Removing Files and Directories

### `rm` — Remove Files

`rm` permanently deletes files. There is no recycle bin:

```bash
rm old-notes.txt            # Delete a single file
rm file1.txt file2.txt      # Delete multiple files
rm -r build/                # Delete a directory and all its contents (recursive)
rm -rf dist/                # Force-delete without prompting (use with caution)
```

> **Warning:** `rm -rf` is irreversible. Double-check the path before running it. This is particularly important inside a Git repository — deleting the `.git/` directory destroys the entire version history.

---

## Getting Help

### `man` — Manual Pages

`man` opens the built-in documentation for any command:

```bash
man ls
man git
```

Press `q` to quit the manual viewer. Manual pages are comprehensive but dense — useful as a reference once you know the basics.

Most commands also accept a `--help` flag, which prints a concise usage summary directly in the terminal without entering the pager:

```bash
ls --help
git --help
git commit --help
```

---

## Productivity Shortcuts

### Tab Autocomplete

Pressing **Tab** while typing a command or path tells the shell to complete it automatically:

```bash
cd pro<Tab>       # Completes to 'cd projects/' if that directory exists
git chec<Tab>     # Completes to 'git checkout'
```

If there are multiple matches, press **Tab** twice to see all options. Learning to reach for Tab instinctively is one of the biggest speed improvements you can make when working in the terminal.

### `clear` — Clear the Terminal

`clear` wipes the visible terminal output, giving you a clean screen:

```bash
clear
```

This does not delete any history — you can still scroll up to see previous output. The keyboard shortcut `Ctrl + L` does the same thing without typing.

---

## Quick Reference

| Command | What it does |
|---|---|
| `pwd` | Print the current working directory |
| `ls` | List files and directories |
| `cd <path>` | Change to `<path>` |
| `.` | Alias for the current directory |
| `..` | Alias for the parent directory |
| `mkdir <name>` | Create a directory |
| `touch <file>` | Create an empty file |
| `cat <file>` | Display file contents |
| `echo <text>` | Print text to the terminal |
| `echo <text> > <file>` | Write text to a file (overwrite) |
| `echo <text> >> <file>` | Append text to a file |
| `nano <file>` | Open a file in the nano editor |
| `rm <file>` | Delete a file |
| `rm -r <dir>` | Delete a directory recursively |
| `man <command>` | Open the manual page for a command |
| `<command> --help` | Show usage summary |
| `clear` | Clear the terminal screen |
| Tab | Autocomplete command or path |

---

*Previous: [Chapter 01 — Introduction to Git and Version Control](ch01-introduction.md)*
*Next: [Chapter 03 — Installing & Configuring Git](ch03-install-configure.md)*
