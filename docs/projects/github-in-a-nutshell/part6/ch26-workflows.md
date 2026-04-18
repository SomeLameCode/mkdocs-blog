---
title: "Chapter 26 — Git Workflows"
description: "Chapter 26 — Git Workflows."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - workflows
status: published
---

# Chapter 26 — Git Workflows

Git gives you the tools — branches, merges, rebases, tags — but not the rules for how to use them. A **Git workflow** is a convention that a team agrees on: which branches exist, what they mean, how work moves from development to production, and who is allowed to do what. Choosing the right workflow for your team's size, release cadence, and deployment model is one of the most consequential decisions in a project's technical setup.

This chapter covers three widely-used workflows: **Feature Branch**, **GitFlow**, and **Trunk-Based Development**. Each represents a different point in the tradeoff between isolation, integration frequency, and release control.

---

## Feature Branch Workflow

The feature branch workflow is the starting point for most teams on GitHub. Its rule is simple: **all work happens on branches; `main` is always deployable**.

### How it works

1. `main` (or `master`) always contains production-ready code
2. Every new feature, bug fix, or task gets its own branch created from `main`
3. Work is committed to the feature branch; `main` is never committed to directly
4. When complete, the branch is merged into `main` via a pull request
5. The branch is deleted after merging

```
main:     A─────────────────────────M──→
                                   /
feature:         B───C───D───E────
```

### Branch naming

Consistent naming makes branches scannable:

```
feature/add-user-authentication
fix/null-pointer-on-login
chore/upgrade-dependencies
docs/update-api-reference
```

### Strengths

- Simple — one rule covers everything
- Every merge to `main` is reviewed (via PR) and tested (via CI)
- Works well with GitHub's PR model and branch protection rules

### Limitations

- No explicit release management — `main` is always the release candidate
- No mechanism for hotfixes separate from ongoing feature work
- Can become chaotic on large teams with many concurrent branches

**Best suited for:** small to medium teams with continuous deployment, SaaS products that ship frequently, open-source projects.

---

## GitFlow

GitFlow (introduced by Vincent Driessen in 2010) defines a strict branching model built around scheduled releases. It uses two permanent branches and three types of short-lived branches, each with a specific role.

### Branch structure

**Permanent branches:**

| Branch | Role |
|---|---|
| `main` | Production-ready code. Every commit is a tagged release. |
| `develop` | Integration branch. Features are merged here first. |

**Short-lived branches:**

| Branch type | Prefix | Branches from | Merges into |
|---|---|---|---|
| Feature | `feature/` | `develop` | `develop` |
| Release | `release/` | `develop` | `main` + `develop` |
| Hotfix | `hotfix/` | `main` | `main` + `develop` |

### The GitFlow lifecycle

```
main:     1.0────────────────────────────────1.1──→
           \                                  /
hotfix:     \──────────────────── 1.0.1──────/──→ develop
             \                              /
develop:  ────────────F1──F2──rel/1.1──────────────→
                     /   /      \
feature/A:   ───────/   /
feature/B:       ───────/
```

**Feature branches:** Branch from `develop`, merge back to `develop` when done. Never interact with `main` directly.

**Release branches:** When `develop` has enough features for a release, branch off as `release/1.1`. Only bug fixes go into a release branch — no new features. When stable, merge into both `main` (tagged with the version) and `develop` (to pick up the fixes).

**Hotfix branches:** Branch from `main` to fix a critical production bug. When done, merge into both `main` (tagged with a patch version) and `develop`.

```bash
# Start a feature
git switch -c feature/add-oauth develop

# Finish a feature
git switch develop
git merge --no-ff feature/add-oauth
git branch -d feature/add-oauth

# Start a release
git switch -c release/1.1 develop

# Finish a release
git switch main
git merge --no-ff release/1.1
git tag -a v1.1 -m "Release 1.1"
git switch develop
git merge --no-ff release/1.1
git branch -d release/1.1

# Hotfix
git switch -c hotfix/1.1.1 main
# ... fix ...
git switch main
git merge --no-ff hotfix/1.1.1
git tag -a v1.1.1 -m "Hotfix 1.1.1"
git switch develop
git merge --no-ff hotfix/1.1.1
git branch -d hotfix/1.1.1
```

The `git-flow` CLI tool automates these sequences:

```bash
git flow init
git flow feature start add-oauth
git flow feature finish add-oauth
git flow release start 1.1
git flow release finish 1.1
```

### Strengths

- Explicit release management — release branches provide a stabilisation period before shipping
- Hotfix path is well-defined and isolated from ongoing feature work
- Clear history — the branch structure tells the story of what was in each release

### Limitations

- High ceremony — many merges, tags, and branch management operations
- `develop` diverges from `main` over time, creating long-lived merge debt
- Not suited to continuous deployment — designed for discrete release cycles
- Rebasing is discouraged (too many branch relationships to maintain)
- Vincent Driessen himself noted in 2020 that GitFlow is not appropriate for most web applications that deploy continuously

**Best suited for:** versioned software with scheduled releases — mobile apps, desktop software, libraries/packages, firmware — where you maintain multiple supported versions simultaneously.

---

## Trunk-Based Development

Trunk-based development (TBD) takes the opposite approach to GitFlow: **everyone integrates into a single shared branch (the "trunk", usually `main`) continuously — at least once a day**.

### Core principle

Long-lived feature branches are the root cause of most merge pain. The solution is to make branches so short-lived they never have time to diverge significantly.

```
main (trunk):  A──B──C──D──E──F──G──H──→
                    ↑  ↑     ↑
               dev1 dev2    dev3 (all integrate daily)
```

### Two variants

**Pure trunk-based development:** Developers commit directly to `main`. Used by very senior teams with strong discipline, comprehensive automated testing, and feature flags.

**Scaled trunk-based development (more common):** Short-lived feature branches (1–2 days maximum) with PRs. The key constraint is the branch lifespan — never more than a day or two before merging.

### Feature flags

When a feature takes longer than a day to complete, trunk-based development uses **feature flags** (also called feature toggles) to merge incomplete code safely. The code ships to production but is disabled behind a flag:

```js
if (featureFlags.isEnabled('new-checkout-flow')) {
  return <NewCheckout />;
}
return <LegacyCheckout />;
```

The feature flag is enabled in production only when the feature is fully ready. This separates **deployment** (putting code in production) from **release** (enabling it for users).

### Release branches in TBD

Some TBD teams branch from `main` at release time (e.g. `release/2026-Q1`) and only cherry-pick critical fixes onto it. The branch is read-only after branching; all fixes are first committed to `main` and then cherry-picked back ("branch by abstraction"). This gives a stable release artifact without long-lived development branches.

### Strengths

- Continuous integration in the true sense — no big-bang merges
- Forces small, incremental commits that are easier to review, revert, and debug
- Merge conflicts are rare and small when they do occur
- Works naturally with CI/CD — the trunk is always the candidate for deployment
- Scales to very large teams (Google, Facebook, and Microsoft use variants of TBD)

### Limitations

- Requires a mature test suite and fast CI — merging daily to trunk only works if broken builds are caught and fixed within minutes
- Feature flags add complexity to the codebase and must be cleaned up
- Less intuitive for developers accustomed to long-lived branches
- Hotfix procedures need explicit definition (usually: fix on trunk, cherry-pick to release branch)

**Best suited for:** teams with continuous delivery pipelines, mature automated test coverage, and SaaS products that deploy multiple times per day.

---

## Choosing a Workflow

No workflow is universally correct. The right choice depends on your team and product:

| Factor | Feature Branch | GitFlow | Trunk-Based |
|---|---|---|---|
| Team size | Any | Medium–large | Any (with discipline) |
| Deploy frequency | Continuous | Scheduled releases | Continuous |
| Multiple live versions? | No | Yes | Sometimes |
| Release stability window needed? | No | Yes | No (use flags) |
| Merge complexity | Low | High | Very low |
| CI/CD maturity required | Medium | Low | High |
| Open source contributions? | Excellent fit | Workable | Harder for external contributors |

### Practical guidance

- Start with **feature branch** workflow. It is simple, well-supported by GitHub's tooling, and sufficient for most teams.
- Graduate to **GitFlow** only if you genuinely need to support multiple release versions simultaneously or have a structured release cycle with a QA phase.
- Move toward **trunk-based development** as your test coverage and deployment automation matures and you find that long-lived branches are causing friction.

Regardless of workflow, three practices universally improve outcomes:

1. **Short-lived branches** — the longer a branch lives, the more painful the merge
2. **Small, focused commits** — easier to review, easier to revert, easier to bisect
3. **Automated quality gates** — CI must run on every push; broken builds are fixed immediately

---

## Summary

- **Feature Branch**: all work on branches, `main` always deployable, merge via PR. Simple and broadly applicable.
- **GitFlow**: two permanent branches (`main`, `develop`) plus feature, release, and hotfix branch types. Designed for scheduled versioned releases; high ceremony.
- **Trunk-Based Development**: integrate to `main` at least daily; use feature flags for incomplete work. Requires fast CI and test discipline; minimises merge pain.
- Choose based on deploy frequency, whether you support multiple live versions, and your CI/CD maturity.
- Start simple; add structure only when the pain of not having it is clear.

> **Further reading:** [A successful Git branching model (GitFlow — Driessen)](https://nvie.com/posts/a-successful-git-branching-model/) · [Trunk Based Development (trunkbaseddevelopment.com)](https://trunkbaseddevelopment.com/)

---

*Previous: [Chapter 25 — GitHub Pages](../part5/ch25-github-pages.md)* · *Next: [Chapter 27 — Git Aliases & Productivity Tips](ch27-aliases-tips.md)*

