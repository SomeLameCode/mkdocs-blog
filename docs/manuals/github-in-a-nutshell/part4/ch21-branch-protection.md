---
title: "Chapter 21 — Branch Protection & Code Review Workflows"
description: "Chapter 21 — Branch Protection & Code Review Workflows."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - collaboration
status: published
---

# Chapter 21 — Branch Protection & Code Review Workflows

Pull requests (Chapter 20) define the process for proposing changes. **Branch protection rules** define the *requirements* those changes must meet before they can be merged. Together they form the foundation of a controlled, auditable code review workflow — ensuring that no commit lands on `main` without the right level of scrutiny, regardless of who is doing the merging.

---

## Why Protect Branches?

Without protection, any collaborator with push access can:

- Push directly to `main`, bypassing review entirely
- Force-push and rewrite shared history
- Merge a PR with failing tests
- Merge without any approval

Branch protection rules prevent all of these. They are configured per branch (or branch pattern) in a repository's settings and enforced by GitHub on every push and merge — even for repository administrators unless explicitly exempted.

---

## Configuring Branch Protection Rules

Branch protection rules are set in:

**Repository → Settings → Branches → Add branch protection rule**

You specify a **branch name pattern** (e.g. `main`, `release/*`, `v*`) and then enable the protections you need.

### Require a pull request before merging

The most fundamental protection: no direct pushes to the protected branch. All changes must arrive via a pull request.

Sub-options:

- **Required approvals** — the minimum number of approved reviews before a PR can be merged (commonly 1 or 2)
- **Dismiss stale reviews** — if new commits are pushed to a PR, existing approvals are automatically dismissed
- **Require review from code owners** — approvals must include the designated owner of any changed file (configured via a `CODEOWNERS` file)
- **Restrict who can dismiss reviews** — limits who can manually dismiss a pending review

### Require status checks to pass

Prevents merging until specified CI checks succeed. You list the check names (e.g. `build`, `test`, `lint`) and GitHub blocks the merge button until they all pass with a green tick.

- **Require branches to be up to date** — the PR branch must be rebased or merged against the base branch before the checks run; prevents "works on my machine" merges where the CI ran against stale code

### Require conversation resolution

All review comment threads must be marked **Resolved** before the PR can be merged. Ensures feedback is not silently ignored.

### Require signed commits

Every commit pushed to the branch must carry a valid GPG or SSH signature. Covered in Appendix C.

### Require linear history

Prevents merge commits — only squash merges and rebase merges are allowed. Keeps the branch history linear and easy to read with `git log --oneline`.

### Restrict who can push

Limits direct pushes (if any are still allowed) and force-pushes to specific users, teams, or roles.

### Do not allow bypassing the above settings

By default, repository administrators can bypass branch protection. Enabling this option applies the rules to everyone without exception — useful for compliance-regulated environments.

### Allow force pushes

Off by default on protected branches. Can be selectively re-enabled for specific users or teams (e.g. to allow maintainers to clean up history after a release).

### Allow deletions

Protected branches cannot be deleted by default. Enable this only if your workflow requires it (e.g. automated release branch cleanup).

---

## `CODEOWNERS`

The `CODEOWNERS` file maps file paths to GitHub users or teams who are automatically requested as reviewers whenever those paths are changed in a PR. It lives at one of:

```
CODEOWNERS
.github/CODEOWNERS
docs/CODEOWNERS
```

Syntax:

```
# Each line: <pattern> <owner> [<owner> ...]

# Require @alice to review any change to the auth module
src/auth/           @alice

# Require the security team to review workflow files
.github/workflows/  @org/security-team

# Default owner for everything not matched above
*                   @org/core-team
```

Patterns follow `.gitignore` syntax. The last matching rule wins.

When branch protection requires "review from code owners" and a PR touches `src/auth/`, GitHub automatically adds `@alice` as a required reviewer — the PR cannot be merged until she approves.

---

## Rulesets (GitHub's Newer System)

GitHub has introduced **Rulesets** as a more powerful replacement for classic branch protection rules. Rulesets offer:

- **Targeting by ref pattern** — protect branches and tags with the same rule
- **Enforcement levels** — `active`, `evaluate` (log violations without blocking), or `disabled`
- **Organisation-level rules** — apply a ruleset across all repositories in an organisation without per-repo configuration
- **Bypass lists** — specify roles, teams, or apps that can bypass specific rules

Rulesets and classic branch protection rules coexist; both are enforced. For new repositories, rulesets are the recommended approach.

> **Further reading:** [GitHub Docs — Branch protection rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches) · [GitHub Docs — Rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)

---

## Code Review Workflows

Branch protection defines the rules. The **code review workflow** is the human process that satisfies them. Several patterns are in common use:

### Feature branch workflow

The baseline workflow for most teams:

1. Every piece of work lives on its own feature branch (`feature/add-login`, `fix/null-pointer`)
2. When ready, a PR is opened targeting `main` (or a development branch)
3. One or more reviewers approve; required CI checks pass
4. The author (or a maintainer) merges and deletes the branch

Branch protection enforces: minimum approvals + required checks.

### Required reviewers by file area (CODEOWNERS)

For larger codebases, different teams own different areas. CODEOWNERS ensures the right people are always in the review loop:

- Infrastructure changes require the platform team
- Security-sensitive files require the security team
- Public API surfaces require the API owner

This prevents a developer from inadvertently merging changes to code they do not understand.

### Two-pass review

Some teams separate **design review** (does the approach make sense?) from **implementation review** (is the code correct and safe?). A draft PR invites early design feedback before the implementation is finished, reducing wasted work.

1. Open a draft PR as soon as the approach is sketched out
2. Request a design review — discuss the overall approach in PR comments
3. Convert to ready for review once the implementation is complete
4. Request implementation review — line-by-line code quality check

### Protected release branches

For projects that maintain multiple supported versions, each supported release line gets its own long-lived branch (`release/1.x`, `release/2.x`). Branch protection on these branches:

- Requires PRs (no direct commits)
- Requires the release manager's approval
- May require a linear history (rebase only) to keep changelogs clean

Hotfixes are applied to the relevant release branch and then cherry-picked forward to newer branches or `main`.

### Merge strategies and branch protection

Branch protection's **Require linear history** setting forces teams to choose between squash merge and rebase merge, ruling out standard merge commits. Common combinations:

| Team preference | Merge strategy | Branch protection setting |
|---|---|---|
| Preserve all commits, see branches in graph | Merge commit | *(no linear history requirement)* |
| One clean commit per feature on `main` | Squash and merge | Require linear history |
| Each commit individually replayable, no merge commits | Rebase and merge | Require linear history |

---

## Reviewing a PR Effectively

Beyond GitHub's mechanical requirements, a useful code review does the following:

**Understand the context first.** Read the PR description, linked issue, and any design documents before looking at the diff. Code looks very different when you know what problem it is solving.

**Review in layers:**
1. **Architecture** — is the overall approach sound? Are there simpler alternatives?
2. **Correctness** — does the code do what it claims? Are edge cases handled?
3. **Security** — are there injection risks, authentication bypasses, or insecure data handling?
4. **Maintainability** — is the code readable? Will the next developer understand it?
5. **Tests** — do the tests cover the important cases? Would they catch a regression?

**Be specific.** A comment like "this could be better" is not actionable. "This loop is O(n²) — consider using a Map to reduce to O(n)" gives the author something to act on.

**Distinguish blocking from non-blocking feedback.** Use GitHub's suggestion feature for small, non-controversial fixes. Mark optional feedback clearly ("nit:", "optional:") so the author knows which comments must be addressed before merging.

**Approve when satisfied.** Don't leave a PR in limbo. If the last round of changes addressed your concerns, approve it.

---

## Summary

- Branch protection rules are configured per branch pattern in repository settings and enforced by GitHub on all pushes and merges.
- Key protections: require PR before merging, required approvals, required status checks, conversation resolution, linear history, no force pushes.
- `CODEOWNERS` maps file paths to required reviewers, automatically enforcing domain ownership in reviews.
- GitHub Rulesets are the modern, more flexible replacement for classic branch protection rules.
- Common review workflows: feature branch + required checks, CODEOWNERS for large codebases, two-pass (design then implementation), protected release branches.
- Effective reviews go beyond mechanical approval: check architecture, correctness, security, maintainability, and tests — and make feedback specific and actionable.

---

*Previous: [Chapter 20 — Pull Requests](ch20-pull-requests.md)* · *Next: [Chapter 22 — GitHub Issues & Project Management](ch22-issues-projects.md)*

