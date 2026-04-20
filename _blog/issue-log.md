# Issue Log — Florin Neagu Notes

> Lightweight tracker for content issues, broken links, and outdated pages.
> Log issues when found; record resolution when closed.
> If an issue recurs twice, add it to `content-backlog.md` as a SI item.
> When this file exceeds 30 rows, archive resolved rows to `_blog/_archive/issue-log-YYYY.md`.
>
> **Resolved column values:** *(blank)* = open · `YYYY-MM-DD` = resolved · `deferred` = acknowledged, not actioning · `accepted` = known limitation, will not fix

| Date | Title | Description | Resolution | Resolved |
|---|---|---|---|---|
| 2026-04-18 | `/personal:commit` does not stage files | The skill commits only what is already staged; it should run `git add` on modified tracked files before committing, or prompt the user to stage | Update the skill definition to include a staging step | |
| 2026-04-18 | Broken cross-chapter link in ch03 | `docs/projects/github-in-a-nutshell/part1/ch03-install-configure.md` links to `ch07-gitignore.md` — resolves against `part1/` but the file is in `part2/` | Fix link to `../part2/ch07-gitignore.md` | |
