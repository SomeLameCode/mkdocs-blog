# Decisions — Florin Neagu Notes

> Records key site decisions — what was decided, alternatives considered, and why.
> Prevents re-litigating settled choices. Newest entries at the top.

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
