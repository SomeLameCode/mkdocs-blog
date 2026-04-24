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
| ART-003 | AI Meeting Transcript → Action Items | Code | Medium | S | Low | [x] | 2026-04-24 | Article (not project); workplace experiment; JSON output angle; from `workplace/transcript-action-extractor` |

## Content — Projects (PRJ-NNN)

| ID | Title / Topic | Section | Priority | Effort | Risk | Status | Published | Notes |
|---|---|---|---|---|---|---|---|---|
| PRJ-001 | Paperless-ngx page update | Projects | Medium | M | Low | [x] | 2026-04-18 | Update existing `docs/projects/paperless-ngx.md` — implemented improvements + new items added since original publish |
| PRJ-002 | GitHub in a Nutshell project page | Projects | Medium | L | Low | [x] | 2026-04-18 | New page from `projects/github-in-a-nutshell`; distill 29-chapter manual into project write-up |
| PRJ-003 | Wako Star Scaffold project page | Projects | Medium | L | Med | [x] | 2026-04-20 | New page from `projects/wako-star-scaffold`; keep story, remove client/org specifics |
| PRJ-004 | Lifting Diary: Scaffold Speed with Claude Code | Projects | Medium | L | Low | [x] | 2026-04-20 | Hub + 4 sub-pages; Tier 1/2/3 improvements applied; frontmatter set to `published` 2026-04-21 |
| PRJ-005 | Invoice Processor | Projects | Medium | S | Low | [x] | 2026-04-23 | Single project page; business narrative + Mermaid flow + setup.sh + full code embed; from workplace experiment |
| PRJ-006 | Meeting Audio Notes Pipeline | Projects | Medium | S | Low | [x] | 2026-04-24 | Single project page; 3-stage Azure pipeline (speech → action extraction → summarization); reuses Transcript Action Extractor module; full code on separate GitHub repo |

## Platform Improvements (SI-NNN)

> MkDocs config, plugins, deployment tooling, Claude commands, site structure.

| ID | Item | Priority | Effort | Risk | Status | Completed | Notes |
|---|---|---|---|---|---|---|---|
| SI-001 | Add frontmatter + tags plugin | High | M | Med | [x] | 2026-04-17 | Enables tag navigation; requires editing all existing pages |
| SI-002 | Add /blog-update Claude command | High | S | Low | [x] | 2026-04-17 | Session close ritual |
| SI-005 | Add /start command for session-start briefing | Medium | S | Low | [x] | 2026-04-18 | Mirrors /blog-update; reads _blog/session-state.md and presents briefing |
| SI-006 | Add /publish-mkdocs command | Medium | S | Low | [x] | 2026-04-18 | Push + deploy in one command; guards against uncommitted changes before deploying |
| SI-007 | Articles section overhaul — hub, sub-folders, cluster index pages | High | M | High | [x] | 2026-04-21 | Hub page with grid cards, articles moved to mkdocs-material/ and code/ sub-folders, cluster index pages, nav grouped with navigation.indexes, "How to enable" blocks added to 3 feature-demo articles |

## Content Quality (SI-NNN)

> Standards, templates, governance conventions, editorial process.

| ID | Item | Priority | Effort | Risk | Status | Completed | Notes |
|---|---|---|---|---|---|---|---|
| SI-003 | Content templates — iterative | Medium | S | Low | [ ] | | Project template updated 2026-04-20 with 4 project types, At a Glance box, Scope & Limitations, image conventions, pub checklist, hub guidance. Article template not yet updated. |
| SI-004 | Update CLAUDE.md with governance conventions | Medium | S | Low | [x] | 2026-04-17 | Document session workflow, ID scheme, frontmatter |
