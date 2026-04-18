---
title: "Chapter 22 — GitHub Issues & Project Management"
description: "Chapter 22 — GitHub Issues & Project Management."
date: 2026-04-18
tags:
  - git
  - github
  - version-control
  - collaboration
status: published
---

# Chapter 22 — GitHub Issues & Project Management

Git tracks code history; GitHub Issues tracks everything else — bugs, feature requests, questions, and tasks. Combined with GitHub Projects, Issues form a lightweight but capable project management layer that lives alongside the code, linked directly to the branches and pull requests that address each item.

---

## GitHub Issues

### What an issue is

An **issue** is a discussion thread attached to a repository. It has a title, a body, and a comment thread, plus a set of metadata fields that make it searchable and organisable:

| Field | Purpose |
|---|---|
| **Title** | Short description of the bug, request, or task |
| **Body** | Full context — steps to reproduce, desired behaviour, screenshots, links |
| **Labels** | Categories (bug, enhancement, documentation, help wanted, etc.) |
| **Assignees** | Who is responsible for resolving the issue |
| **Milestone** | Group issues into a version or sprint goal |
| **Projects** | Link to one or more GitHub Projects boards |
| **Linked pull requests** | PRs that will close this issue when merged |

### Creating an issue

In the GitHub UI: repository → **Issues** tab → **New issue**.

Using the GitHub CLI:

```bash
gh issue create --title "Login fails with special characters in password" \
                --body "Steps to reproduce: ..." \
                --label "bug" \
                --assignee "@me"
```

### Issue templates

Repositories can define issue templates (stored in `.github/ISSUE_TEMPLATE/`) to prompt contributors for the right information. A bug report template might require reproduction steps, expected vs actual behaviour, and environment details. A feature request template might ask for a use-case description and proposed solution.

Templates are Markdown files with YAML front matter:

```yaml
---
name: Bug report
about: Report something that isn't working
labels: bug
---

**Describe the bug**

**Steps to reproduce**

**Expected behaviour**

**Actual behaviour**

**Environment** (OS, browser, version)
```

For more structured intake, **issue forms** (`.github/ISSUE_TEMPLATE/*.yml`) provide validated form fields in the GitHub UI.

### Labels

Labels are coloured tags applied to issues and PRs. GitHub creates a default set when a repository is initialised:

| Default label | Meaning |
|---|---|
| `bug` | Something isn't working |
| `documentation` | Improvements or additions to docs |
| `duplicate` | This issue already exists |
| `enhancement` | New feature or request |
| `good first issue` | Good for newcomers |
| `help wanted` | Extra attention is needed |
| `invalid` | This doesn't seem right |
| `question` | Further information is requested |
| `wontfix` | This will not be worked on |

You can create, rename, and delete labels freely. Many teams add labels for priority (`P1`, `P2`), area (`auth`, `infra`, `frontend`), or status (`needs-triage`, `blocked`).

Filter issues by label:

```bash
gh issue list --label "bug"
gh issue list --label "good first issue"
```

### Milestones

A **milestone** groups related issues and PRs under a named goal with an optional due date. GitHub calculates completion percentage based on how many items in the milestone are closed.

```bash
gh api repos/{owner}/{repo}/milestones --method POST \
  -f title="v2.0.0" \
  -f due_on="2026-06-01T00:00:00Z"
```

Milestones work well for version-based planning: assign all issues targeted for `v2.0.0` to that milestone, and GitHub shows you progress toward the release.

### Closing issues

Issues are closed manually (via the **Close issue** button) or automatically when a linked PR is merged. Use closing keywords in a PR's title, body, or a commit message:

```
Closes #42
Fixes #42
Resolves #42
```

Multiple issues can be closed in one PR:

```
Closes #42, closes #43, resolves #51
```

GitHub parses these keywords and closes the referenced issues when the PR lands on the default branch.

### Searching and filtering issues

```bash
gh issue list                           # open issues in current repo
gh issue list --state closed            # closed issues
gh issue list --assignee "@me"          # assigned to you
gh issue list --label "bug" --label "P1"  # multiple labels (AND)
gh issue list --search "login timeout"  # full-text search
```

In the GitHub UI, the **Filters** bar supports the same queries using the `is:`, `label:`, `assignee:`, `milestone:`, and `author:` qualifiers.

---

## GitHub Projects

GitHub Projects is a flexible project management tool built into GitHub. It provides board-style and spreadsheet-style views over issues and PRs, with custom fields, filters, and automation.

### Projects v2 concepts

| Concept | Description |
|---|---|
| **Project** | A board/table that aggregates items from one or more repositories |
| **Item** | An issue, PR, or draft item added to the project |
| **Field** | Metadata column: built-in (Title, Status, Assignees) or custom (Priority, Sprint, Estimate) |
| **View** | A saved configuration of filters, grouping, and layout (Board, Table, Roadmap) |
| **Workflow** | Automated rules that update item fields when events occur |

### Creating a project

In the GitHub UI: your profile or organisation → **Projects** → **New project**. Choose a template (Basic Kanban, Bug tracker, Feature, Sprint, Roadmap) or start blank.

Using the GitHub CLI:

```bash
gh project create --owner "@me" --title "Q2 Roadmap"
```

### Views

**Board view** — columns based on a Status field (e.g. Todo, In Progress, Done). Drag items between columns to update status.

**Table view** — spreadsheet layout showing all items and their fields. Good for bulk editing, sorting, and filtering.

**Roadmap view** — timeline layout with date fields. Shows when work is scheduled and how items relate in time.

Switch views with the tabs at the top of the project, or add new views for specific filters (e.g. a view that shows only your assigned items, or only high-priority bugs).

### Custom fields

Beyond built-in fields, projects support custom fields:

| Field type | Examples |
|---|---|
| Text | Notes, context |
| Number | Story points, estimate |
| Date | Due date, target date |
| Single select | Priority (P1/P2/P3), Size (S/M/L/XL) |
| Iteration | Sprint 1, Sprint 2 (with start/end dates) |

Custom fields appear as columns in table view and can be used to group and filter items.

### Automation

Projects support automated workflows that update item fields based on events:

- When an issue is opened → set Status to **Todo**
- When a PR is merged → set Status to **Done**
- When an item is added to the project → set Status to **Todo**

Automations are configured in the project's **Workflows** settings. More complex automations (e.g. set Priority based on label, or move items between sprints) can be built with GitHub Actions using the Projects GraphQL API.

### Adding items to a project

From the project's **+ Add item** field:

```bash
# Add an issue to a project by number
gh project item-add <project-number> --owner "@me" --url <issue-url>
```

Items can come from multiple repositories within the same project, making cross-repo planning possible.

---

## Connecting Issues, PRs, and Projects

The real power of GitHub's system comes from the connections between these tools:

```
Issue #42  ←──linked──→  PR #87  ←──closes──→  Issue #42 auto-closed on merge
    │                        │
    └─── milestone: v2.0 ────┘
    │                        │
    └─── project: Q2 Board ──┘
```

A typical workflow:

1. **File an issue** describing the bug or feature
2. **Add it to the project** and set its Status to *Todo*
3. **Create a branch** for the fix: `git switch -c fix/42-login-timeout`
4. **Open a PR** that references the issue: `Closes #42` in the body
5. **Review and merge** the PR — GitHub closes issue #42 automatically and the project automation moves it to *Done*

The issue number appears in the PR, the branch name carries the issue number, and `git log` shows the PR merge commit — the full chain is traceable.

---

## Summary

- GitHub Issues are discussion threads for bugs, features, and tasks. Use labels, milestones, and assignees to organise them.
- Issue templates (`.github/ISSUE_TEMPLATE/`) standardise the information contributors provide.
- Closing keywords (`Closes #N`, `Fixes #N`) in PR descriptions automatically close issues on merge.
- GitHub Projects aggregates issues and PRs into Board, Table, or Roadmap views with custom fields and automation.
- Connect issues → PRs → projects for a fully traceable workflow: file → plan → branch → PR → merge → auto-close.

> **Further reading:** [GitHub Docs — Issues](https://docs.github.com/en/issues) · [GitHub Docs — Projects](https://docs.github.com/en/issues/planning-and-tracking-with-projects)

---

*Previous: [Chapter 21 — Branch Protection & Code Review Workflows](ch21-branch-protection.md)* · *Next: [Chapter 23 — Git Hooks](../part5/ch23-git-hooks.md)*
