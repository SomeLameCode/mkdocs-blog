# Blog State — Florin Neagu Notes

> Updated by `/blog-update` at each session close. Read this first at session start.
> If deeper context is needed, read the last 20 rows of `_blog/change-log.md`.

**Updated:** 2026-07-14
**Last session commit:** 45b4cc24d6e8b373a2dc5e6befb73a5614b04014
**Last check:** 2026-07-14 — Closed SI-010 (social cards). Unblocked `cairosvg`'s native Cairo dependency on Windows via MSYS2 (`pacman -S mingw-w64-ucrt-x86_64-cairo` in the UCRT64 shell, `C:\msys64\ucrt64\bin` added to PATH) rather than the GTK3 runtime route D-028 originally pointed to; re-added `social` to `mkdocs.yml`. Re-confirmed the RSS path-separator bug from D-028 is a separate, still-open issue unrelated to Cairo, so `use_material_social_cards` stays `false` (RSS readers fall back to the theme default image; direct page shares get the real per-page card). `mkdocs build --strict` clean, 125 per-page card PNGs generated and spot-checked; verified live via `curl` (`og:image` meta tag present and resolving) and a real link-preview screenshot from the site owner. Committed, pushed, and deployed via `mkdocs gh-deploy`.
**Open content:** None
**Open improvements:** 1 item (SI-003 Content templates — article template pending)
**Open issues:** 1 item — Notebooks stub indexed as a real page (deferred until ART-002 ships)

---

## Live Site Summary

| Section | Status | Pages |
|---|---|---|
| Articles | Live | 8 |
| Projects | Live | 9 |
| Manuals | Live | 92 (GitHub in a Nutshell: 33, Prisma Access / SASE: 59) |
| Code | Live | 3 |
| Notebooks | Placeholder — Coming Soon | 0 |

---

## In Progress

*(none)*

---

## Next Actions

1. Update article template (SI-003) — ~30 min
