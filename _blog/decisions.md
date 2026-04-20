# Decisions — Florin Neagu Notes

> Records key site decisions — what was decided, alternatives considered, and why.
> Prevents re-litigating settled choices. Newest entries at the top.

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
