---
title: "Appendix A — Git Submodules"
description: "Appendix A — Git Submodules."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
status: published
---

# Appendix A — Git Submodules

A **submodule** is a Git repository embedded inside another Git repository. The outer repository (the **superproject**) records a specific commit SHA from the inner repository (the **submodule**) rather than copying the submodule's files. This makes submodules useful when a project depends on another project's source code at a fixed, controlled version — for example, a shared library, a vendored dependency, or a documentation theme.

---

## When to Use Submodules

Submodules are appropriate when:

- You need to include another repository's source code at a pinned version that you control explicitly.
- The dependency is developed separately (perhaps by a different team or organisation) and has its own release cycle.
- You want contributors to be able to update the dependency deliberately, with the update recorded as a commit in the superproject.

Submodules are **not** a good fit when:

- You just need a package: use a language package manager (`npm`, `pip`, `cargo`, `go mod`) instead.
- The dependency changes frequently and you always want the latest version: the overhead of updating a submodule pointer exceeds the benefit.
- Your team is not familiar with submodule workflows: the extra conceptual overhead causes errors (forgetting to initialise, working in a detached HEAD in the submodule, pushing the superproject without pushing the submodule).

---

## Adding a Submodule

```bash
git submodule add <url> [<path>]
```

For example, to add a shared library at `lib/mylib`:

```bash
git submodule add https://github.com/example/mylib.git lib/mylib
```

This does three things:

1. Clones the submodule repository into `lib/mylib/`.
2. Creates or updates `.gitmodules` at the superproject root — a configuration file that records the submodule name, path, and URL.
3. Stages both `.gitmodules` and a special **gitlink** entry for `lib/mylib` (file mode `160000`).

Commit to record the addition:

```bash
git commit -m "Add mylib as submodule at lib/mylib"
```

### `.gitmodules`

```ini
[submodule "lib/mylib"]
    path = lib/mylib
    url = https://github.com/example/mylib.git
```

The gitlink entry in the superproject's tree stores only the submodule's commit SHA — not its files. Run `git cat-file -p HEAD^{tree}` on the superproject and you will see a `160000` mode entry for the submodule path:

```
160000 commit a3f8c21d4e5b...    lib/mylib
```

---

## Cloning a Repository with Submodules

When you clone a superproject, submodule directories exist but are empty by default:

```bash
git clone https://github.com/example/superproject.git
cd superproject
ls lib/mylib   # empty directory
```

To populate them, initialise and fetch the submodule content:

```bash
git submodule init      # register submodules from .gitmodules into .git/config
git submodule update    # clone each submodule and check out the pinned commit
```

Or do both in one step:

```bash
git submodule update --init
```

To recursively initialise nested submodules (submodules within submodules):

```bash
git submodule update --init --recursive
```

The most convenient approach is to clone with submodules fully populated from the start:

```bash
git clone --recurse-submodules <url>
```

---

## The Detached HEAD in Submodules

After `git submodule update`, each submodule is in a **detached HEAD** state — it is checked out at the specific commit SHA recorded by the superproject, not on any branch. This is intentional: the superproject owns the precise version; the submodule has no branch to accidentally advance.

```bash
cd lib/mylib
git status
# HEAD detached at a3f8c21
```

If you need to make changes inside the submodule, explicitly check out a branch first:

```bash
git switch main        # or whatever branch you want to work on
# ... make changes, commit ...
git push
```

Then return to the superproject and update the pinned SHA to point to the new commit.

---

## Updating a Submodule to a Newer Commit

### Fetch and move the pointer manually

```bash
cd lib/mylib
git fetch
git switch main
git pull
cd ../..
git add lib/mylib
git commit -m "Update mylib to latest main"
```

### Use `git submodule update --remote`

`--remote` fetches from the submodule's upstream and updates the working tree to the latest commit on the tracking branch:

```bash
git submodule update --remote lib/mylib
git add lib/mylib
git commit -m "Update mylib submodule"
```

By default `--remote` tracks the `main` branch. You can configure a different branch per submodule in `.gitmodules`:

```ini
[submodule "lib/mylib"]
    path = lib/mylib
    url = https://github.com/example/mylib.git
    branch = stable
```

---

## Checking Submodule Status

```bash
git submodule status
```

Output format: a prefix character, the current commit SHA, the path, and the tag if any:

| Prefix | Meaning |
|---|---|
| ` ` (space) | Submodule is at the commit the superproject expects |
| `-` | Submodule has not been initialised (empty directory) |
| `+` | Submodule is at a different commit than the superproject expects |
| `U` | Submodule has merge conflicts |

```bash
git submodule status
#  a3f8c21d4e5b (lib/mylib)       # clean
# +b1c2d3e4f5a6 (lib/mylib)       # ahead or behind
# -0000000000000000000000000000000000000000 lib/mylib   # not initialised
```

To see a summary of which commits changed across a `git pull` of the superproject:

```bash
git submodule summary
```

---

## Pushing Superproject and Submodule Changes Together

A common mistake is pushing superproject commits that reference submodule commits that have not yet been pushed. Anyone who clones or pulls the superproject will get a gitlink pointing to a SHA that does not exist on the remote.

Guard against this with `--recurse-submodules=check`:

```bash
git push --recurse-submodules=check
```

This aborts the push if any submodule commit referenced by the superproject has not been pushed. Use `on-demand` to push submodules automatically before pushing the superproject:

```bash
git push --recurse-submodules=on-demand
```

Set this as the default:

```bash
git config --global push.recurseSubmodules on-demand
```

---

## Running Commands Across All Submodules

`git submodule foreach` executes a shell command in each submodule directory:

```bash
git submodule foreach git status
git submodule foreach git pull origin main
git submodule foreach 'echo "Submodule: $name at $(git rev-parse --short HEAD)"'
```

The `$name` variable expands to the submodule name defined in `.gitmodules`.

---

## Removing a Submodule

Removing a submodule requires cleaning up in several places:

```bash
# 1. Unregister the submodule
git submodule deinit -f lib/mylib

# 2. Remove the submodule from the index and working tree
git rm -f lib/mylib

# 3. Remove the cached submodule metadata
rm -rf .git/modules/lib/mylib

# 4. Commit
git commit -m "Remove mylib submodule"
```

Skipping step 3 leaves stale metadata in `.git/modules/` that can cause errors if you later try to add a submodule at the same path.

---

## Common Pitfalls

**Forgetting `--recurse-submodules` when cloning**

Contributors who clone the repository without `--recurse-submodules` (or forget to run `git submodule update --init`) will have empty submodule directories and likely encounter build errors. Document the setup steps in your project's README.

**Working in detached HEAD**

After `git submodule update`, the submodule is in detached HEAD. Committing there creates commits unreachable from any branch — they will eventually be garbage collected. Always `git switch <branch>` inside the submodule before making changes.

**Forgetting to push the submodule before the superproject**

The superproject commit points to a submodule SHA that only exists locally. Other contributors can clone the superproject but cannot fetch that SHA. Use `push --recurse-submodules=on-demand` as the default (above) to prevent this.

**Nested submodules**

Submodules can themselves contain submodules. Always use `--recursive` with `clone`, `update`, and `foreach` to ensure all levels are populated.

---

## Quick Reference

```bash
# Add a submodule
git submodule add <url> <path>

# Clone including submodules
git clone --recurse-submodules <url>

# Initialise + populate after clone
git submodule update --init --recursive

# Check status of all submodules
git submodule status

# Update submodule to latest remote commit
git submodule update --remote <path>

# Run a command in every submodule
git submodule foreach <command>

# Push, ensuring submodule commits are pushed first
git push --recurse-submodules=on-demand

# Remove a submodule
git submodule deinit -f <path>
git rm -f <path>
rm -rf .git/modules/<path>
git commit -m "Remove <name> submodule"
```

---

## Summary

- A submodule embeds a separate Git repository inside a superproject, pinned to a specific commit SHA.
- Use submodules for vendored dependencies or shared libraries with independent release cycles. Use a package manager for ordinary dependencies.
- Always clone with `--recurse-submodules`, or run `git submodule update --init --recursive` after cloning.
- Submodule working trees are in detached HEAD after update — check out a branch before committing.
- Protect against unpushed submodule references with `push.recurseSubmodules = on-demand`.
- Removing a submodule requires `git submodule deinit`, `git rm`, and deleting `.git/modules/<name>`.

> **Further reading:** [Git Tools — Submodules (Pro Git)](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

---

*Previous: [Chapter 29 — SHA-1, Hashing & Object Storage Internals](../part7/ch29-hashing-internals.md)* · *Next: [Appendix B — Git LFS (Large File Storage)](appB-git-lfs.md)*

