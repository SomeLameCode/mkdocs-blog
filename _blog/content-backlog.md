# Content Backlog — Florin Neagu Notes

> **Process:** Identify → Draft (temp/ note) → Review → Publish → Record in change-log.md → Close (mark [x] + run /blog-update)
>
> **Effort:** S = < 1 hr · M = 1–4 hrs · L = > 4 hrs / multiple sessions
> **Risk:** Low = new content only · Med = changes existing pages · High = structural/nav change
> **Status:** [ ] = open · [x] = published/done · [-] = deferred · [~] = superseded · [/] = descoped
> Never delete a row — use a marker and note the reason in the Item column.
> When this file exceeds 50 rows, archive completed rows to `_blog/_archive/backlog-YYYY.md`.

## Content — Articles (ART-NNN)

| ID | Title / Topic | Section | Priority | Effort | Risk | Status | Published | Notes |
|---|---|---|---|---|---|---|---|---|
| ART-001 | First code snippet (reusable utility) | Code | High | S | Low | [ ] | | Unblocks Code placeholder |
| ART-002 | First Jupyter notebook | Notebooks | Medium | M | Low | [-] | | Deferred. Use mkdocs-jupyter plugin (not nbconvert) when ready — install via pip, add to mkdocs.yml plugins |

## Content — Projects (PRJ-NNN)

| ID | Title / Topic | Section | Priority | Effort | Risk | Status | Published | Notes |
|---|---|---|---|---|---|---|---|---|
| PRJ-001 | Paperless-ngx page update | Projects | Medium | M | Low | [x] | 2026-04-18 | Update existing `docs/projects/paperless-ngx.md` — implemented improvements + new items added since original publish |
| PRJ-002 | GitHub in a Nutshell project page | Projects | Medium | L | Low | [ ] | | New page from `projects/github-in-a-nutshell`; distill 29-chapter manual into project write-up |
| PRJ-003 | Wako Star Scaffold project page | Projects | Medium | L | Med | [ ] | | New page from `projects/wako-star-scaffold`; keep story, remove client/org specifics |

## Platform Improvements (SI-NNN)

> MkDocs config, plugins, deployment tooling, Claude commands, site structure.

| ID | Item | Priority | Effort | Risk | Status | Completed | Notes |
|---|---|---|---|---|---|---|---|
| SI-001 | Add frontmatter + tags plugin | High | M | Med | [x] | 2026-04-17 | Enables tag navigation; requires editing all existing pages |
| SI-002 | Add /blog-update Claude command | High | S | Low | [x] | 2026-04-17 | Session close ritual |
| SI-005 | Add /start command for session-start briefing | Medium | S | Low | [x] | 2026-04-18 | Mirrors /blog-update; reads _blog/session-state.md and presents briefing |
| SI-006 | Add /publish-mkdocs command | Medium | S | Low | [x] | 2026-04-18 | Push + deploy in one command; guards against uncommitted changes before deploying |

## Content Quality (SI-NNN)

> Standards, templates, governance conventions, editorial process.

| ID | Item | Priority | Effort | Risk | Status | Completed | Notes |
|---|---|---|---|---|---|---|---|
| SI-003 | Content templates — iterative | Medium | S | Low | [ ] | | Draft article + project templates after PRJ-001; refine after each subsequent project. In 00_templates\3_template_blog |
| SI-004 | Update CLAUDE.md with governance conventions | Medium | S | Low | [x] | 2026-04-17 | Document session workflow, ID scheme, frontmatter |
