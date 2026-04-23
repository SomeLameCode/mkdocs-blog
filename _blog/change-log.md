# Change Log — Florin Neagu Notes

> One entry per publish, structural change, or significant update to the live site.
> Log after deploying. Reference ART/PRJ/SI IDs for tracked backlog items.
> When this file exceeds 50 data rows, archive the oldest rows to `_blog/_archive/change-log-YYYY.md`.

| Date | What changed | Why | Ref |
|---|---|---|---|
| 2026-04-23 | Published Invoice Processor project page — business narrative, Mermaid flow diagram, setup.sh embed, full Python code, CSV output table, "Going Further" and "Scope & Limitations" sections; nav entry and projects index entry added | PRJ-005 publish | PRJ-005 |
| 2026-04-21 | Renamed GitHub repo "notes" → "mkdocs-blog" — updated `site_url`, `repo_url`, `repo_name` in mkdocs.yml; updated path refs in mkdocs-setup.md; redeployed to GitHub Pages at new URL | Repo name was ambiguous; URL now reflects site purpose | — |
| 2026-04-21 | Fixed broken image path in mkdocs-integrate-with-obsidian.md: `../assets/` → `../../assets/` | File moved one level deeper during sub-folder restructure | — |
| 2026-04-21 | Added collapsed "How to enable" setup block to admonitions.md, content-tabs.md, diagram-examples.md — shows required mkdocs.yml config snippet | Articles showed output but not how to replicate it | SI-007 |
| 2026-04-21 | Restructured articles section — added hub page (articles/index.md), moved 7 articles into mkdocs-material/ and code/ sub-folders, added cluster index pages, updated nav with grouped sections and clickable headers, updated homepage browse link | Visual clustering and scalable structure for future topic growth | SI-007 |
| 2026-04-21 | Updated PRJ-004 frontmatter: status `draft` → `published` on hub + 4 sub-pages | Mark Lifting Diary as officially published | PRJ-004 |
| 2026-04-21 | Fixed broken cross-chapter link in ch03-install-configure.md: `ch07-gitignore.md` → `../part2/ch07-gitignore.md` | Link resolved against wrong folder | — |
| 2026-04-21 | Fixed dead anchor in m365-tenant-separation.md ToC: `#2-business-context--the-forced-cutover` → `#2-business-context-the-forced-cutover` | MkDocs slugify collapses multiple spaces to single dash | — |
| 2026-04-20 | Removed `navigation.sections` from mkdocs.yml — sidebar sub-pages now collapse when navigating away from their section | Fix always-expanded Lifting Diary sidebar | — |
| 2026-04-20 | Updated PRJ-004 pages — Tier 1/2/3 improvements: thesis statement, closing paragraphs, cross-links, screenshot captions, constraint doc excerpt, CLAUDE.md sample, failure modes section, auth/data-ownership paragraph, Scope & Limitations section | Content quality pass before publish | PRJ-004 |
| 2026-04-20 | Published PRJ-004 Lifting Diary — hub page + 4 sub-pages (the-application, building-with-claude-code, architecture-as-input, prompt-log) + 9 screenshots | PRJ-004 publish | PRJ-004 |
| 2026-04-20 | Published M365 Tenant Separation case study — new project page with Mermaid diagrams (Gantt, dependency flowchart, go-live sequence, RAID pie), Material admonition callouts, practitioner tip blocks; nav entry and projects index entry added | PRJ-003 publish | PRJ-003 |
| 2026-04-18 | Published GitHub in a Nutshell project page — overview page + 29 chapter files with frontmatter, nav entry, and projects index entry | PRJ-002 publish | PRJ-002 |
| 2026-04-18 | Updated Paperless-ngx project page — added multi-user/RBAC setup, day-to-day ops, concrete security hardening, three-layer backup with export gap warning, restore scenarios, email ingestion config, updated repo contents | PRJ-001 page refresh | PRJ-001 |
| 2026-04-17 | Added YAML frontmatter and tags to all 8 content pages (7 articles + Paperless-ngx project) | Enables tag navigation and consistent metadata | SI-001 |
| 2026-04-17 | Enabled tags plugin in `mkdocs.yml`; added `docs/tags.md` tag index page | Activates tag browsing on live site | SI-001 |
| 2026-04-17 | Added `_blog/` governance folder (session-state, backlog, change-log, issue-log, decisions) | Blog governance layer from improvement-ideas.md | — |
| 2026-03-XX | Added FN monogram favicon | Branding | — |
| 2026-03-XX | Slim footer, removed prev/next nav | Cleaner UX | — |
| 2026-03-XX | Added Paperless-ngx project page | First project write-up | — |
| 2026-01-XX | Initial site launch (MkDocs + Material theme, GitHub Pages) | Site creation | — |
