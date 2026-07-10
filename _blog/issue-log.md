# Issue Log — Florin Neagu Notes

> Lightweight tracker for content issues, broken links, and outdated pages.
> Log issues when found; record resolution when closed.
> If an issue recurs twice, add it to `content-backlog.md` as a SI item.
> When this file exceeds 30 rows, archive resolved rows to `_blog/_archive/issue-log-YYYY.md`.
>
> **Resolved column values:** *(blank)* = open · `YYYY-MM-DD` = resolved · `deferred` = acknowledged, not actioning · `accepted` = known limitation, will not fix

| Date | Title | Description | Resolution | Resolved |
|---|---|---|---|---|
| 2026-04-18 | `/personal:commit` does not stage files | The skill commits only what is already staged; it should run `git add` on modified tracked files before committing, or prompt the user to stage | Updated skill to auto-stage tracked modified files with `git add -u` when nothing is staged (D-009) | 2026-04-20 |
| 2026-04-20 | Dead anchor in m365-tenant-separation.md | `docs/projects/m365-tenant-separation.md` contains a link to `#2-business-context--the-forced-cutover` but no such anchor exists on the page — flagged by MkDocs on every deploy | Changed ToC anchor to `#2-business-context-the-forced-cutover` (single dash — MkDocs collapses spaces around em dash) | 2026-04-21 |
| 2026-04-18 | Broken cross-chapter link in ch03 | `docs/projects/github-in-a-nutshell/part1/ch03-install-configure.md` links to `ch07-gitignore.md` — resolves against `part1/` but the file is in `part2/` | Fixed to `../part2/ch07-gitignore.md` | 2026-04-21 |
| 2026-07-10 | Broken footer link in Prisma Access SASE ch54 | `docs/manuals/prisma-access-sase/part9/ch54-configure-saml-in-azure-ad.md` footer read `*Next: Appendix A*` as plain text instead of a link, inconsistent with every other chapter transition in the manual | Fixed to `[Appendix A — Glossary of Terms](../appendix/appA-glossary.md)`, found during MAN-001 publish | 2026-07-10 |
| 2026-07-10 | Manuals section missing from homepage grid | `docs/index.md`'s "What's Here" card grid didn't include Manuals, despite the section being live since Phase 1 of the rollout — caught while browsing the deployed site locally | Added a Manuals card matching the existing style, linking to `manuals/index.md` | 2026-07-10 |
| 2026-04-21 | Broken image path in mkdocs-integrate-with-obsidian.md | File moved from `articles/` to `articles/mkdocs-material/` — image path `../assets/` now resolves to `articles/assets/` instead of `docs/assets/` | Updated path to `../../assets/` | 2026-04-21 |
| 2026-04-21 | Root folder named "notes" is ambiguous | `C:\GitFolder\notes` is indistinguishable from a scratch or system folder. Preferred rename: `mkdocs-blog`. 19 hardcoded references across 5 files need updating. Decision pending: local rename only vs. also renaming GitHub repo (which changes the Pages URL from `/notes/` to `/mkdocs-blog/`) | Renamed local folder and GitHub repo to `mkdocs-blog`; updated site_url/repo_url/repo_name in mkdocs.yml, 12 refs in settings.local.json, 1 ref in publish-mkdocs.md, 4 refs in CLAUDE.md; redeployed to GitHub Pages (D-014) | 2026-04-21 |
