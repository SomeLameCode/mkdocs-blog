# Blog Improvement Ideas

> Generated: 2026-04-17
> Reference: `C:\GitFolder\00_templates` (small-project, medium-project, BAU templates)

---

## Context

This site is a live MkDocs/Material blog deployed to GitHub Pages. Content quality is good — detailed how-tos and project write-ups — but there is no governance layer. Each session starts cold with no memory of what was last done, what is in progress, or what is planned next. The templates in `00_templates` solve exactly these problems for project work; the ideas below adapt those patterns for a blog context.

The blog behaves most like a **BAU (Business-as-Usual) system**: it runs continuously, evolves incrementally, and has no defined end date. The BAU template is the primary reference.

---

## Gap Analysis

| Area | Current State | Gap |
|---|---|---|
| Content pipeline | No tracking | Ideas → draft → published flow is invisible |
| Session continuity | None | No memory file; every session starts cold |
| Content metadata | No frontmatter or tags | No status, date, or tags on any page |
| Site decisions | Undocumented | No record of why nav/theme/plugin choices were made |
| Improvement backlog | None | No place to log enhancement ideas |
| Content quality | No review cycle | No mechanism to flag outdated content |
| CLI tooling | None | No `/blog-update` command to close sessions consistently |
| Content templates | None | Each new article/project page is written from scratch |

---

## Idea 1 — Governance Folder (`_blog/`)

**Template reference:** `2_template_bau/_bau/`

Add a `_blog/` folder to the repo root (not in `docs/`, so it never appears on the published site). Adapted from the BAU `_bau/` pattern:

| File | Purpose | Template analogue |
|---|---|---|
| `session-state.md` | Compact snapshot: site status, in-progress drafts, next actions — **read first each session** | `memory/bau-state.md` |
| `content-backlog.md` | Planned articles/projects with ID, priority, effort, status | `_bau/improvements.md` |
| `change-log.md` | Timestamped log of every publish, update, or structural change | `_bau/change-log.md` |
| `issue-log.md` | Broken links, outdated content, missing sections | `_bau/issue-log.md` |
| `decisions.md` | Site decisions with rationale (nav structure, plugins, theme, deploy choices) | `_bau/decisions.md` |

**Content backlog ID scheme:**
- Articles: `ART-NNN`
- Projects: `PRJ-NNN`
- Site improvements: `SI-NNN`

**Status markers (same as template convention):**
```
- [ ] Planned
- [x] Published / Done
- [-] Deferred
- [/] Descoped
```

---

## Idea 2 — Content Metadata (Frontmatter + Tags)

**Template reference:** Metadata header pattern from all templates

Add YAML frontmatter to every page in `docs/`. Example:

```yaml
---
title: "MkDocs Initial Setup on Windows"
description: "Step-by-step guide to setting up MkDocs with the Material theme on Windows."
date: 2026-01-15
tags:
  - mkdocs
  - windows
  - github-pages
status: published
---
```

Status values: `draft` | `review` | `published` | `outdated`

**Required change to `mkdocs.yml`:** Enable the Material tags plugin:

```yaml
plugins:
  - tags
```

This adds a tag index page and lets readers filter content by topic — improves discoverability as the content library grows.

---

## Idea 3 — Content Templates (in `00_templates\3_template_blog`)

**Template reference:** Delivery templates from `1_template-medium-project/_project-delivery/`

Create reusable starter files so new content has consistent structure and frontmatter from the first line:

### `3_template_blog/docs/articles/_template-article.md`
For technical how-to guides and tutorials:

```markdown
---
title: "[Article Title]"
description: "[One-sentence summary]"
date: YYYY-MM-DD
tags: []
status: draft
---

# [Article Title]

[Brief intro — what this covers and why it matters]

---

## Overview

## Prerequisites

## Steps

### Step 1 — [Name]

### Step 2 — [Name]

## Verification

## References
```

### `3_template_blog/docs/projects/_template-project.md`
For project documentation and case studies:

```markdown
---
title: "[Project Name]"
description: "[One-sentence summary]"
date: YYYY-MM-DD
tags: []
status: draft
---

# [Project Name]

[Brief description — what the project is and what problem it solves]

---

## 1. What Is It?

## 2. Architecture

## 3. Setup & Configuration

## 4. Outcomes & Observations

## 5. References
```

---

## Idea 4 — Claude Code CLI Command (`/blog-update`)

**Template reference:** `.claude/commands/bau-update.md` from `2_template_bau`

Create `3_template_blog/.claude/commands/blog-update.md` — a slash command that bookends each session:

**At session start:** Read `_blog/session-state.md` to load current context (what is live, what is in progress, next actions).

**At session end:**
1. Update `_blog/session-state.md` with current status and next actions
2. Log any published or changed content to `_blog/change-log.md`
3. Update status of any backlog items worked on in `_blog/content-backlog.md`
4. Log any issues found to `_blog/issue-log.md`

This mirrors the `/bau-update` command and ensures sessions always end with a clean handoff to the next session.

---

## Idea 5 — CLAUDE.md Updates

Extend `CLAUDE.md` in the notes repo to document the new governance conventions:

- **Session workflow:** Start → read `_blog/session-state.md`; End → run `/blog-update`
- **Frontmatter convention:** Required fields and status values
- **Content backlog IDs:** ART-NNN, PRJ-NNN, SI-NNN schemes
- **Governance folder:** What lives in `_blog/` and why

---

## Idea 6 — Populate Placeholder Sections

The `code/` and `notebooks/` sections show "Coming Soon" notices. These reduce the perceived completeness of the site. Recommended first entries:

- **Code:** A reusable Python utility or shell script (already produced during other projects)
- **Notebooks:** A simple data exploration notebook (pandas/matplotlib) from any home lab dataset

Tracking these in `_blog/content-backlog.md` as `ART-001` and `ART-002` would be the starting point.

---

## Idea 7 — Review Cycle for Existing Content

Once frontmatter with `date` and `status` fields is in place, a lightweight review process becomes possible:

- Filter pages with `status: published` and `date` older than 6 months
- Flag them as `status: review` in frontmatter
- Add to `_blog/issue-log.md` as review candidates

No tooling required initially — this is a manual check run occasionally (e.g., quarterly).

---

## Implementation Order (Suggested)

| Step | What | Effort |
|---|---|---|
| 1 | Create `_blog/` folder + governance files | Low |
| 2 | Create `3_template_blog` content templates | Low |
| 3 | Create `/blog-update` Claude command | Medium |
| 4 | Add frontmatter to existing pages | Low |
| 5 | Enable tags plugin in `mkdocs.yml` | Low |
| 6 | Update `CLAUDE.md` with new conventions | Low |
| 7 | Populate Code + Notebooks placeholders | Medium |
