# Decisions — Florin Neagu Notes

> Records key site decisions — what was decided, alternatives considered, and why.
> Prevents re-litigating settled choices. Newest entries at the top.

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
