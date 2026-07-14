# Decisions — Florin Neagu Notes

> Records key site decisions — what was decided, alternatives considered, and why.
> Prevents re-litigating settled choices. Newest entries at the top.

---

## D-028 — Deferred social cards on Windows; disabled RSS plugin's social-card lookup

**Date:** 2026-07-14
**Decision:** Per `temp/blog-visibility-seo-execution-plan.md` Items 4 and 5, attempted to enable Material's `social` plugin for auto-generated OpenGraph card images and `mkdocs-rss-plugin` for an RSS feed together. The `social` plugin's `cairosvg` dependency needs the native Cairo C library (`libcairo-2.dll`), which has no pip wheel on Windows and wasn't present on this machine — `mkdocs build --strict` failed with repeated `cairosvg` crash warnings. Rather than install a system-level DLL from a third-party source unprompted, left `pillow`/`CairoSVG` pinned in `requirements.txt` (ready for later) but removed `social` from `mkdocs.yml`, deferring it as SI-010. Separately — and independent of the Cairo problem — found that `mkdocs-rss-plugin`'s default `use_material_social_cards: true` builds per-page card-image URLs with Windows path separators (`social\projects\foo.png`), producing 404s and failing strict builds on any Windows machine even once Cairo is fixed. Set `use_material_social_cards: false` in the `rss` plugin config to avoid it, unrelated to whether `social` itself is ever enabled.
**Alternatives considered:** Download and manually place a prebuilt `libcairo-2.dll` (e.g. via the GTK3 runtime installer for Windows) to get `social` working immediately. Rejected for this pass — installing a system-level native library is a bigger, less reversible action than a pip install and belongs to the site owner's call, not something to do silently mid-session.
**Rationale:** Keeps `mkdocs build --strict` clean and RSS shipping now rather than blocking both items on an unrelated native-dependency problem. To unblock social cards later: install a Cairo runtime for Windows (GTK3 runtime installer is the standard route), confirm `python -c "import cairosvg"` succeeds, then re-add `social` to `mkdocs.yml` `plugins:` and re-enable `use_material_social_cards` in the `rss` config if desired.

**Update — 2026-07-14 (SI-010 unblocked):** Cairo was unblocked via MSYS2 instead of the GTK3 runtime this decision originally pointed to: installed MSYS2, ran `pacman -S mingw-w64-ucrt-x86_64-cairo` in the UCRT64 shell, added `C:\msys64\ucrt64\bin` to PATH. `python -c "import cairosvg"` then succeeded, and `social` was re-added to `mkdocs.yml plugins:` — `mkdocs build --strict` is clean and per-page OpenGraph card images (125 PNGs) generate and render correctly. **The RSS path-separator bug is still present and unrelated to Cairo** — with `use_material_social_cards: true`, `feed_rss_created.xml` still emitted mixed-separator paths (`social/projects\index.png`), so that setting stays `false`. Pages get proper social cards when shared directly; RSS readers fall back to the theme default image instead of per-page cards until `mkdocs-rss-plugin` fixes its path handling on Windows.

---

## D-027 — Reversed both Articles sub-lists after confirming no tutorial dependency

**Date:** 2026-07-14
**Decision:** Per the nav-reordering plan (`temp/nav-reverse-order-execution-plan.md`), the "MkDocs & Material" and "Code & Data" Articles sub-lists were flagged as a judgment call — they could read as tutorial sequences rather than pure creation-order lists, similar to manual chapters. Checked both clusters' page content for cross-references ("as covered in," "assumes you've read," etc.) and found none — each article has its own standalone "How to enable this" block with no dependency on sibling articles. Proceeded to reverse both sub-lists to newest-first, matching every other reordered list.
**Alternatives considered:** Leave the two Articles sub-lists in creation order as a precaution, treating them like manual chapters.
**Rationale:** The site owner had already confirmed Articles sub-lists should be reversed; the flag was a courtesy check, not a blocker. Content inspection found no sequencing dependency, so no reason to carve out an exception.

---

## D-026 — No scrub needed for NICU IRRBB PoC page; third-person voice for author attribution

**Date:** 2026-07-13
**Decision:** For `docs/projects/nicu-irrbb-poc.md` (companion technical deep-dive to PRJ-009), no names-only scrub was required — all five source files under `.../workspace/4_ProjectStory/PoC-IRRBB/` are purely technical/architecture documents with no client-side individual named anywhere. The only person named in the source is the delivery team's own AI architect (kept, per the same rule as D-025 — normal author attribution). Chose to narrate the page in third person, matching the site's established voice, with a single attribution near the top rather than repeating the author's name throughout as the first-person source material does. Also verified the linked GitHub repo (`SomeLameCode/NICU-AI-Rollout-Strategy`) has no individual names on its page before linking to it directly.
**Rationale:** The source material for this page is a different register from Part 1 (technical/governance reference documents, not workshop narrative with named client stakeholders), so the Phase 0 check surfaced nothing to redact — confirmed via `temp/nicu-poc-prep/prep-notes.md` before drafting, consistent with the gated three-phase workflow used for both NICU pages.

---

## D-025 — Names-only scrub scope for NICU AI Strategy Workshop page (lighter than D-023)

**Date:** 2026-07-13
**Decision:** For `docs/projects/nicu-ai-strategy-workshop.md`, scrubbed only individual client-side names (CTO, CEO, and three IT/security staff → role descriptions) and one internal corporate-strategy codename ("TrueNorth30" → generic "the credit union's corporate strategy"), from source material at `C:\GitFolder\02_Projects\01_NICU-AI-Strategy\workspace\`. Everything else — the institution name (NICU), sector, delivery firm name (Experteq) and delivery team's own names (Florin Neagu, James Sheron), core banking platform name (Ultracs/Ultradata), regulatory framework (APRA/APS 117/CPS 234), and real dollar figures — was kept as-is, per the site owner's explicit confirmation that this engagement does not need institution-level anonymization (unlike D-023's Ausgrid case study, which required stripping sector, country, and all client-derived naming).
**Judgment call flagged:** the source plan named only two staff members to scrub (a security lead and an IT lead, referenced by first name in the Purview section). A third named individual — a member of NICU's IT team, quoted confirming licensing details and emphasising DLP requirements — was found during the scrub and treated as client-side personal information under the same "remove every named individual" rule, given a distinct generic descriptor ("a member of NICU's IT team") rather than reusing "the IT lead," to avoid implying they're the same person.
**Rationale:** The site owner distinguished this engagement from the Ausgrid case study up front — narrow, names-only scope, confirmed via `temp/nicu-strategy-prep/prep-notes.md` before drafting began (three-phase workflow: scrub → draft → nav-wiring, gated on review at each step, mirroring PRJ-008's process but with a lighter Phase 0 pass matched to the actually-required scope).

---

## D-024 — Design-delta content gets its own section on the Prisma Access SASE project page, not folded into delivery

**Date:** 2026-07-10
**Decision:** On `docs/projects/prisma-access-sase-implementation.md`, gave "what changed vs. the original HLD" (scrubbed design-delta content) its own numbered section (§6, "What Changed From the Original Design") between Delivery Approach and Outcomes, rather than folding it into the delivery narrative.
**Alternatives considered:** Fold design deviations into §5 Delivery Approach as inline asides where each deviation naturally arose (keeps section count matched to `m365-tenant-separation.md`'s 7 sections exactly, but scatters a coherent set of before/after facts across a narrative section and makes them harder to scan).
**Rationale:** The design-delta material is a formal deviation record (corrected / accepted-tradeoff / open-item groupings), not a narrative beat — structurally closer to the M365 page's own decision-log section than to its delivery story. Grouping it lets a reader scan "what shipped differently than planned" in one place, matching how the M365 case study gives its decision log a dedicated section (§5, "Why Nothing Was Forgotten") rather than weaving it through the build narrative.

---

## D-023 — Anonymization scope for Ausgrid Prisma Access case study (Phase 0 scrub)

**Date:** 2026-07-10
**Decision:** Produced a scrubbed working set in `temp/agd-scrubbed/` (not committed) from the four source documents in `C:\GitFolder\02_Projects\03_agd_Prisma_Cloud_Web\workspace\`, stripping the client name (Ausgrid → "the client"), sector/industry (electricity distribution/utility → "a large distributed-infrastructure organization"), regulatory framework names (SOCI Act/CIRMP/ASD Essential Eight → "critical infrastructure compliance obligations" / "a recognized security maturity framework"), country/region signals (Australia, Sydney/Central Coast/Hunter Valley, and even Prisma Access's own AU-SE/AU-S/Japan region names → generic "Primary Region"/"Backup Region"/"APAC Fallback Region"), all real domains/tenant IDs/app IDs/IP ranges/site codes (FUJI/HMDC/AEA1/AEA2/ASE1/ASE2 → DC-1 through DC-4B), and all AG/AGD-derived naming (zone names, profile groups, cert profiles, group names). Full substitution table in `temp/agd-scrubbed/00-glossary.md`.
**Flagged for review (not decided unilaterally):**
1. ~~Even the generic "critical infrastructure compliance obligations" framing may narrow the field too far~~ — **Resolved 2026-07-10:** Florin confirmed, then asked for full consistency. Removed the customer-scale sentence ("serving millions of end customers across a large metropolitan and regional service area") and every remaining "critical infrastructure" qualifier throughout `01-success-story-scrubbed.md` (title, intro note, Challenge section, and the file's own footnote — the footnote itself was removed since it's now moot). Final language is sector-neutral "regulatory compliance obligations" / "the applicable regulatory risk management program" with no "critical infrastructure" tie-in anywhere in the file. Glossary updated to record "critical infrastructure" as dropped entirely, not just genericized — do not reintroduce in Phase 1 drafting.
2. Prisma Access's own compute region names (AU-SE, AU-S, Japan Central/South) are Palo Alto product naming, not client-specific — genericized anyway since combined with everything else they still signal the client's country. Reversible if Florin judges this over-cautious.
3. Project dates (HLD 1 April 2026, closure 2026-06-04) were kept as-is — no evidence they're publicly tied to the real client elsewhere, but not independently verified.
4. The Prisma Access Browser Chrome extension ID (`dbpnkcjkchadgbgihmmolkjcgjkodoaf`) was kept — it's Palo Alto's published product ID, not a client identifier.
**Rationale:** The Phase 0 prompt (`temp/agd-prisma-access-blog-publish-plan.md`) required stripping strictly more than the M365 case study (which anonymizes name but keeps sector) — for this client, sector itself (critical infrastructure / energy distribution) is the sensitive fact, so genericization had to go further than a simple find-and-replace on the client's name. Erred toward stripping ambiguous details per the plan's explicit instruction, flagging the borderline calls above rather than silently deciding them.

---

## D-022 — New `MAN-NNN` backlog prefix for Manuals content

**Date:** 2026-07-10
**Decision:** Track future Manuals content under a new `MAN-NNN` prefix in `content-backlog.md`, starting with `MAN-001` (Prisma Access / SASE). GitHub in a Nutshell is not retroactively renumbered — it stays tracked under `PRJ-002`, with a note added at migration time (see change-log, D-020).
**Alternatives considered:** Keep using `PRJ-NNN` for manuals content (simplest, no new scheme, but conflates two content types the site now deliberately distinguishes — see D-020); retroactively renumber GitHub in a Nutshell to a new prefix (rewrites settled history for no real benefit, `PRJ-002`'s row already carries a migration note).
**Rationale:** Manuals is now a distinct top-level section with its own structure and nav pattern (D-020) — its backlog items are a different shape (chapter counts, part structure) from project write-ups and deserve their own ID space going forward, consistent with how `ART-NNN`/`PRJ-NNN`/`SI-NNN` are already split by content type in `CLAUDE.md`.

---

## D-021 — No redirect plugin for GitHub in a Nutshell URL change; links fixed directly

**Date:** 2026-07-10
**Decision:** Migrating GitHub in a Nutshell from `docs/projects/github-in-a-nutshell/` to `docs/manuals/github-in-a-nutshell/` changes its live URL from `/projects/github-in-a-nutshell/` to `/manuals/github-in-a-nutshell/`. No redirect plugin (e.g. `mkdocs-redirects`) was added — the old URL will simply 404 after deploy. `projects/index.md` was updated with a pointer sentence to the new Manuals section for in-site navigation.
**Alternatives considered:** Add `mkdocs-redirects` and configure a stub for the old path (preserves any external bookmarks/search-engine links, but adds a new plugin dependency for a single one-time move); leave a manual redirect stub page at the old path (extra file to maintain with no clear removal trigger).
**Rationale:** Same reasoning as D-014 (repo rename) — this is a personal portfolio site with no known inbound links or SEO stake to protect, and the chapter-level URLs were never linked from nav in the first place (only the overview page had a real nav entry). Adding a redirect plugin for a single migration is disproportionate; if this pattern recurs (e.g. future section reorganizations), revisit installing `mkdocs-redirects` then rather than pre-emptively.

---

## D-020 — New top-level `Manuals` nav section, separate from `Projects`

**Date:** 2026-07-10
**Decision:** Introduce a dedicated top-level `Manuals` nav section to house multi-chapter reference manuals, starting with GitHub in a Nutshell (currently wedged into a single `Projects` nav entry with 32 un-navved chapter files) and then Prisma Access SASE (currently unpublished, living outside the blog entirely). GitHub in a Nutshell will be migrated first to validate the pattern against live, currently-published content; Prisma Access SASE will be published second into the now-proven structure — reversing the lower-risk, net-new-first order that would normally be preferred, because validating against real content the user already knows well is more useful here than validating on unfamiliar net-new material.
**Alternatives considered:** Leave GitHub in a Nutshell as-is inside `Projects` (status quo — no sidebar chapter visibility, no Material auto prev/next); flatten its chapters directly into the `Projects` nav (works mechanically but misrepresents a reference manual as a project narrative — `Projects` tells a build story read once, not a work readers jump into at a chapter); host manuals on a separate site entirely (avoids nav complexity but fragments the single-site, single-workflow setup this repo is built around).
**Rationale:** `Projects` and `Manuals` serve different reading patterns — projects tell a build narrative meant to be read once start to finish, manuals are reference works readers jump into at a specific chapter and expect full sidebar nav, part-level grouping, and prev/next footer links. Wedging a 32-chapter manual into one `Projects` nav entry loses exactly those reference-navigation features. A dedicated section lets `Manuals` be structured for depth (nested nav, part grouping) without changing how `Projects` works for its own content.

---

## D-019 — TTS generator published as PRJ-006 section + code page, not standalone project

**Date:** 2026-05-07
**Decision:** Add the TTS Audio Generator as a section within the Meeting Audio Notes Pipeline project page and a companion code page in the Code section, rather than a standalone project page (PRJ-007) or article.
**Alternatives considered:** Standalone project page (PRJ-007) with its own narrative; article in the Code & Data cluster; code page only with no prose context.
**Rationale:** The tool was built specifically to support PRJ-006 — it has no independent lifecycle. Embedding it as a section in PRJ-006 tells the story correctly (pipeline needed controllable test data → built this tool). A code page follows the established pattern for project companion scripts (same as `invoice-processing.md` and `transcript-action-extractor.md`). A standalone project page would overstate the scope of a 150-line CLI utility.

---

## D-018 — Articles hub cards: max 5, newest on top

**Date:** 2026-04-24
**Decision:** Each cluster card on the articles hub shows at most 5 articles — the 5 most recently dated ones, listed newest first. Articles beyond the top 5 are accessible via "Browse all" only.
**Alternatives considered:** Show all articles in the card (no cap — degrades as content grows); show only 3 (too few context for readers); replace cards with a flat list by cluster (better long-term, but loses the visual cluster framing).
**Rationale:** Cards with too many links become overwhelming and defeat the point of a hub. The 5-article cap keeps the hub scannable now; the "Browse all" path preserves full discoverability. User acknowledged this is a short-term solution — flat cluster sections will replace cards once article counts grow significantly.

---

## D-017 — Transcript Action Extractor published as article, not project page

**Date:** 2026-04-24
**Decision:** Publish the transcript action extractor as an article in `docs/articles/code/` rather than a project page.
**Alternatives considered:** Project page (same pattern as Invoice Processor); standalone code page only (no narrative).
**Rationale:** The deliverable is a single ~80-line script — not enough scope for a project narrative with At a Glance box, scope & limitations, and full business context. Article format (deep-dive + how-to hybrid) fits the size: explain the problem angle, walk through the code, show setup and output. Establishes an informal sizing rule: single-script utilities → article; multi-component solutions → project page.

---

## D-016 — Move script code to Code section, link from project page

**Date:** 2026-04-23
**Decision:** Move `setup.sh` and `invoice_processor.py` out of the Invoice Processor project page and into a dedicated page in the Code section (`docs/code/invoice-processing.md`). Replace inline blocks with one-line descriptions and links.
**Alternatives considered:** Keep both scripts embedded in the project page (complete in one place but heavy); split into separate Code pages per script (more nav entries than warranted for two small files).
**Rationale:** The user found the inline code walls "spammy" — they interrupted the business narrative. The Code section is the right home for raw scripts; the project page should be a readable story, not a code dump. A single code page with both scripts keeps the reference self-contained.

---

## D-015 — Invoice Processor published as a project page, not article or code

**Date:** 2026-04-23
**Decision:** Publish the invoice processor as a project page in `docs/projects/` rather than an article or standalone code page.
**Alternatives considered:** Article in the Code & Data cluster (how-to format, less narrative room); standalone code page only (no business context); split article + code page (adds nav complexity for a small project).
**Rationale:** The project format leads with narrative ("what problem, what solution") and supports the At a Glance box pattern — both match what the user wanted. The invoice processor is a complete working solution, not a reusable technique or tutorial. Project format is the established home for that category.

---

## D-014 — Rename GitHub repo to `mkdocs-blog` (not local only)

**Date:** 2026-04-21
**Decision:** Rename both the local folder and the GitHub repo from "notes" to "mkdocs-blog", accepting the site URL change from `/notes/` to `/mkdocs-blog/`.
**Alternatives considered:** Local rename only, keeping the GitHub repo named "notes" (preserves the old URL but creates a local/remote name mismatch and leaves a non-descriptive URL).
**Rationale:** `/mkdocs-blog/` is self-descriptive and consistent with the repo's purpose. Local and remote names now match. The old URL had no inbound links to break.

---

## D-013 — Collapsed details block for "How to enable" in feature-demo articles

**Date:** 2026-04-21
**Decision:** Add a collapsed `???` details admonition at the top of each feature-demo article containing the required `mkdocs.yml` config snippet and one non-obvious gotcha.
**Alternatives considered:** Inline setup text before examples (clutters quick-reference flow); separate "Setup" section at the bottom; link to official docs only.
**Rationale:** Collapsed by default keeps examples front and centre — readers who know the setup skip it, new readers expand it. Direct config snippet is more actionable than a link to docs.

---

## D-012 — Sub-folder clustering with navigation.indexes for articles section

**Date:** 2026-04-21
**Decision:** Restructure articles into topic sub-folders (`mkdocs-material/`, `code/`) with cluster index pages and a hub landing page, using `navigation.indexes` (already enabled) so cluster section headers are clickable.
**Alternatives considered:** Keep flat list and use tags for filtering; single `articles/index.md` listing all articles by topic without sub-folders.
**Rationale:** Sub-folders scale to future clusters (business, ML, neural networks) — adding a cluster is a new folder + a card on the hub. Tags alone don't provide visual hierarchy. Flat list degrades as article count grows.

---

## D-011 — Remove navigation.sections for collapsible sidebar

**Date:** 2026-04-20
**Decision:** Remove `navigation.sections` from `mkdocs.yml` features so sidebar sub-pages collapse when navigating away from their section.
**Alternatives considered:** Add `navigation.prune` (reduces HTML size but doesn't fix expansion behaviour); restructure nav so Lifting Diary has no children (loses sidebar sub-page links entirely).
**Rationale:** `navigation.sections` renders any nav group with children as a permanently expanded group. With only Lifting Diary having sub-pages, its links were always visible regardless of active page. Removing the feature gives collapsible sections at no cost to other nav behaviour.

---

## D-010 — Multi-page hub layout for Lifting Diary (PRJ-004)

**Date:** 2026-04-20
**Decision:** Structure PRJ-004 as a hub index page + 4 sub-pages, sitting between the GitHub manual (pure reference/links) and M365 (single long narrative).
**Alternatives considered:** One long page like M365 — too dense for a project with four distinct themes; flat reference page like the GitHub manual — too thin for a narrative case study.
**Rationale:** Four angles (the app, the workflow, the architecture thesis, the prompt log) are each dense enough for their own page but too thin to justify a long scroll together. Hub page stays pure navigation; sub-pages carry the content.

---

## D-009 — Auto-stage tracked modified files in /personal:commit

**Date:** 2026-04-20
**Decision:** When `/personal:commit` finds nothing staged but tracked modified files exist, run `git add -u` automatically and proceed rather than stopping to ask the user to stage manually.
**Alternatives considered:** Keep the "nothing staged, stop and ask" behaviour (confusing after governance-only sessions where only `_blog/` files are modified); require explicit staging every time (friction with no safety benefit for tracked files).
**Rationale:** `git add -u` only touches already-tracked files — it cannot accidentally include untracked secrets or binaries. The common post-`/blog-update` pattern is exactly this scenario: governance files modified, nothing staged. Stopping mid-workflow is unhelpful noise.

---

## D-008 — Publish manual as docs subfolder, chapters out of sidebar nav

**Date:** 2026-04-18
**Decision:** Copy only `git-manual-md/` into `docs/projects/github-in-a-nutshell/` as a subfolder. Chapters are accessible via links from the overview page and via site search, but not listed in the sidebar nav.
**Alternatives considered:** Link to GitHub repo (no in-site search or browsing); add manual as its own top-level nav section (too much nav clutter for 29 chapters); upload the full project folder (unnecessary — only the compiled manual is needed).
**Rationale:** Keeps the manual fully searchable and internally linked within the site. Nav stays clean — the overview page acts as the entry point. Python frontmatter script handles bulk metadata injection in one pass.

---

## D-007 — /publish-mkdocs slash command for push + deploy

**Date:** 2026-04-18
**Decision:** Implement push + GitHub Pages deploy as a single `/publish-mkdocs` slash command, separate from the commit step (which stays with `/personal:commit`).
**Alternatives considered:** Run `git push origin main` and `mkdocs gh-deploy` by hand — simple enough but easy to forget the sequence or skip the push.
**Rationale:** Splitting commit from publish matches the natural workflow boundary (review the commit, then deliberately publish). The command also guards against uncommitted changes before deploying, activates the venv inline, and is self-documenting.

---

## D-006 — Content templates built iteratively from project experience

**Date:** 2026-04-18
**Decision:** SI-003 (content templates) will not be designed upfront. Templates will be drafted after completing PRJ-001, then refined after each subsequent project page (PRJ-002, PRJ-003).
**Alternatives considered:** Design templates upfront before any project pages (risks over-engineering for hypothetical content).
**Rationale:** Three concrete projects provide immediate real examples. Templates should emerge from practice, not prediction.

---

## D-005 — /start command for session-start briefing

**Date:** 2026-04-18
**Decision:** Implement session-start briefing as an explicit `/start` slash command backed by `.claude/commands/start.md`, rather than relying on the voluntary CLAUDE.md instruction.
**Alternatives considered:** Strengthen the CLAUDE.md instruction (unreliable — Claude may still skip it); hook-based automation (no session-start hook exists in Claude Code).
**Rationale:** Mirrors the existing `/blog-update` pattern for session end. Explicit invocation is guaranteed; voluntary instructions are not.

---

## D-004 — Four-state content status field in frontmatter

**Date:** 2026-04-17
**Decision:** Use four status values in page frontmatter: `draft`, `review`, `published`, `outdated` — rather than a simple published/unpublished boolean.
**Alternatives considered:** Boolean `published: true/false`; three-state (draft/published/outdated).
**Rationale:** `review` stage catches content that's written but not yet polished enough to surface. `outdated` lets pages stay live while flagging they need updating — avoiding silent rot without forced removal.

---

## D-003 — Governance layer adapted from BAU template

**Date:** 2026-04-17
**Decision:** Use `_blog/` folder with BAU-style governance files (session-state, backlog, change-log, issue-log, decisions).
**Alternatives considered:** No governance (status quo); full project template (too heavy).
**Rationale:** Blog behaves like a BAU system — continuous, no end date, incremental improvements. BAU template maps directly. Governance lives outside `docs/` so it never appears on the published site.

---

## D-002 — Obsidian as editor

**Date:** 2026-01-XX
**Decision:** Use Obsidian as the primary editor for writing content, with MkDocs as the publishing layer.
**Alternatives considered:** Editing Markdown directly in VS Code.
**Rationale:** Obsidian provides a better writing experience and graph view for linking notes. MkDocs handles publishing independently.

---

## D-001 — MkDocs + Material theme on GitHub Pages

**Date:** 2026-01-XX
**Decision:** Use MkDocs with Material theme, deployed to GitHub Pages via `gh-pages` branch.
**Alternatives considered:** Jekyll, Hugo, Docusaurus.
**Rationale:** MkDocs is Python-based (familiar toolchain), Material theme is polished and well-documented, GitHub Pages is free and integrates with the existing git workflow.

---

> Add decisions above this line, newest first.
