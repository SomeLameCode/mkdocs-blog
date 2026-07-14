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
| ART-001 | First code snippet (reusable utility) | Code | High | S | Low | [/] | | Descoped 2026-07-14 — Code section already has 3 live pages (invoice-processing, transcript-action-extractor, tts-audio-generator); "Unblocks Code placeholder" note was stale |
| ART-002 | First Jupyter notebook | Notebooks | Medium | M | Low | [-] | | Deferred. Use mkdocs-jupyter plugin (not nbconvert) when ready — install via pip, add to mkdocs.yml plugins |
| ART-003 | AI Meeting Transcript → Action Items | Code | Medium | S | Low | [x] | 2026-04-24 | Article (not project); workplace experiment; JSON output angle; from `workplace/transcript-action-extractor` |

## Content — Projects (PRJ-NNN)

| ID | Title / Topic | Section | Priority | Effort | Risk | Status | Published | Notes |
|---|---|---|---|---|---|---|---|---|
| PRJ-001 | Paperless-ngx page update | Projects | Medium | M | Low | [x] | 2026-04-18 | Update existing `docs/projects/paperless-ngx.md` — implemented improvements + new items added since original publish |
| PRJ-002 | GitHub in a Nutshell project page | Projects | Medium | L | Low | [x] | 2026-04-18 | New page from `projects/github-in-a-nutshell`; distill 29-chapter manual into project write-up. **2026-07-10: migrated out of Projects into the new Manuals section** (`docs/manuals/github-in-a-nutshell.md`) — no longer lives at this path, see change-log |
| PRJ-003 | Wako Star Scaffold project page | Projects | Medium | L | Med | [x] | 2026-04-20 | New page from `projects/wako-star-scaffold`; keep story, remove client/org specifics |
| PRJ-004 | Lifting Diary: Scaffold Speed with Claude Code | Projects | Medium | L | Low | [x] | 2026-04-20 | Hub + 4 sub-pages; Tier 1/2/3 improvements applied; frontmatter set to `published` 2026-04-21 |
| PRJ-005 | Invoice Processor | Projects | Medium | S | Low | [x] | 2026-04-23 | Single project page; business narrative + Mermaid flow + setup.sh + full code embed; from workplace experiment |
| PRJ-006 | Meeting Audio Notes Pipeline | Projects | Medium | S | Low | [x] | 2026-04-24 | Single project page; 3-stage Azure pipeline (speech → action extraction → summarization); reuses Transcript Action Extractor module; full code on separate GitHub repo |
| PRJ-007 | Infrastructure Consolidation — Six Servers to Three | Projects | High | L | Low | [x] | 2026-05-16 | Single case study page; DC virtualization on Proxmox, SCCM stack consolidation via Site Recovery, W2016 decommission; 6 key decisions, 6 troubleshooting cases; from `02_Projects/02_Infrastructure-consolidation` |
| PRJ-008 | Prisma Access SASE Implementation | Projects | High | L | Med | [x] | 2026-07-10 | Case study page from `02_Projects/03_agd_Prisma_Cloud_Web/workspace/success-story/` — fully anonymized (client name, sector/industry, regulatory framework names, country, real domains/IPs/tenant IDs, site codes, AG/AGD-derived naming all stripped, see D-023); scrubbed working set built in `temp/agd-scrubbed/` (uncommitted) before drafting |
| PRJ-009 | NICU AI Strategy Workshop | Projects | High | L | Low | [x] | 2026-07-13 | Case study page from `02_Projects/01_NICU-AI-Strategy/workspace/` (Day 1/Day 2 workshop story only — Part 1 of 2) — light names-only scrub, not full anonymization (institution name, sector, delivery firm, dollar figures kept; only client-side individual names + one internal strategy codename genericized, see D-025); scrubbed working set built in `temp/nicu-strategy-prep/` (gitignored) before drafting; 2 chart images + 2 original Mermaid diagrams. Companion piece: PRJ-010 |
| PRJ-010 | NICU IRRBB Proof of Concept | Projects | High | L | Low | [x] | 2026-07-13 | Technical/governance deep-dive page from `02_Projects/01_NICU-AI-Strategy/workspace/4_ProjectStory/PoC-IRRBB/` (Part 2 of 2, companion to PRJ-009) — no scrub needed, purely technical source with no client individuals named (see D-026); pipeline architecture + upstream data-flow Mermaid diagrams, 3-model comparison table with real gate results (1 of 3 shown failing honestly), APRA governance requirement mapping, LLM-vs-ML tabbed comparison, GitHub repo link (names-checked first); cross-linked both ways with PRJ-009 |

## Content — Manuals (MAN-NNN)

> New prefix (D-022) — Manuals is now a distinct top-level section from Projects; GitHub in a Nutshell predates this prefix and stays tracked under PRJ-002.

| ID | Title / Topic | Section | Priority | Effort | Risk | Status | Published | Notes |
|---|---|---|---|---|---|---|---|---|
| MAN-001 | Prisma Access / SASE training manual | Manuals | Medium | L | Med | [x] | 2026-07-10 | 54 chapters + 4 appendix from `C:\GitFolder\02_Projects\07_SASE-PrismaAccess\workspace\`; first-time publish (no prior site URL); frontmatter normalized, part-level READMEs folded into new overview page, ch54 broken link fixed |

## Platform Improvements (SI-NNN)

> MkDocs config, plugins, deployment tooling, Claude commands, site structure.

| ID | Item | Priority | Effort | Risk | Status | Completed | Notes |
|---|---|---|---|---|---|---|---|
| SI-001 | Add frontmatter + tags plugin | High | M | Med | [x] | 2026-04-17 | Enables tag navigation; requires editing all existing pages |
| SI-002 | Add /blog-update Claude command | High | S | Low | [x] | 2026-04-17 | Session close ritual |
| SI-005 | Add /start command for session-start briefing | Medium | S | Low | [x] | 2026-04-18 | Mirrors /blog-update; reads _blog/session-state.md and presents briefing |
| SI-006 | Add /publish-mkdocs command | Medium | S | Low | [x] | 2026-04-18 | Push + deploy in one command; guards against uncommitted changes before deploying |
| SI-007 | Articles section overhaul — hub, sub-folders, cluster index pages | High | M | High | [x] | 2026-04-21 | Hub page with grid cards, articles moved to mkdocs-material/ and code/ sub-folders, cluster index pages, nav grouped with navigation.indexes, "How to enable" blocks added to 3 feature-demo articles |
| SI-008 | Add robots.txt | Medium | S | Low | [x] | 2026-07-14 | `docs/robots.txt`, points crawlers at sitemap.xml; from `temp/blog-visibility-seo-execution-plan.md` Item 1 |
| SI-009 | Enable RSS feed (mkdocs-rss-plugin) | Medium | S | Low | [x] | 2026-07-14 | Feeds from `articles/` + `projects/` only (manuals excluded — reference material, not date-driven); `use_material_social_cards: false` set to dodge a Windows path-separator bug in the plugin's card lookup, see D-028; from execution plan Item 5 |
| SI-010 | Enable social cards (Material `social` plugin) | Medium | S | Low | [-] | | Deferred — `cairosvg` needs a native `libcairo-2.dll` not installable via pip on Windows; plugin added to `requirements.txt` (pillow, CairoSVG) but left out of `mkdocs.yml` `plugins:`; see D-028 for what unblocks it (GTK3 runtime install, manual). 2026-07-14: site owner confirmed postponing rather than installing the GTK3 runtime now — revisit if/when wanted; from execution plan Item 4 |
| SI-011 | Wire up GoatCounter analytics | Medium | S | Low | [x] | 2026-07-14 | Site owner signed up and chose GoatCounter (cookieless, no consent banner needed) over Google Analytics/Plausible/Fathom; no native Material `extra.analytics` support for GoatCounter, so wired via `docs/javascripts/goatcounter.js` (creates the tracking `<script>` with `data-goatcounter` attribute) + `extra_javascript` in `mkdocs.yml`; from execution plan Item 7 |

## Content Quality (SI-NNN)

> Standards, templates, governance conventions, editorial process.

| ID | Item | Priority | Effort | Risk | Status | Completed | Notes |
|---|---|---|---|---|---|---|---|
| SI-003 | Content templates — iterative | Medium | S | Low | [ ] | | Project template updated 2026-04-20 with 4 project types, At a Glance box, Scope & Limitations, image conventions, pub checklist, hub guidance. 2026-07-14: added a "Before publishing" checklist block to the article template (previously had none) and added two items to both templates' checklists — meta description quality, checked for cross-links to related pages — prompted by the SEO/visibility execution plan. Article template still lacks project template's other enhancements (multiple article types already existed as guidance, but no At a Glance/Scope & Limitations equivalents assessed as needed for articles). |
| SI-004 | Update CLAUDE.md with governance conventions | Medium | S | Low | [x] | 2026-04-17 | Document session workflow, ID scheme, frontmatter |
