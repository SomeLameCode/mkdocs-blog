# Blog Visibility & SEO — Execution Plan

> **Purpose of this file:** A comprehensive, prioritized checklist of visibility/SEO
> improvements for the mkdocs-blog site, broken into independent items. This is
> **not** meant to be run start-to-finish in one session — pick an item, hand its
> prompt to Claude Code (or do it manually where noted), confirm it, move to the
> next. Order reflects effort-to-impact, not a hard dependency chain — most items
> are independent of each other.
>
> **Why prompts, not instructions (for Claude Code items):** These state intent and
> constraints, not exact steps. Claude Code should read the live repo state before
> acting and flag anything that looks like a real judgment call.

---

## Verified current state (checked 2026-07-14, via live `mkdocs.yml`)

- ✅ `site_url` **is** set correctly (`https://somelamecode.github.io/mkdocs-blog/`)
  — this means MkDocs' built-in sitemap.xml generation should already be working.
  No action needed here beyond confirming it in the built `site/` output once.
- ❌ No `robots.txt` present in `docs/`
- ❌ No per-page `description:` frontmatter convention confirmed — needs an audit
- ❌ No `social` plugin (Material's auto-generated OpenGraph card images)
- ❌ No `rss` plugin
- ❌ No analytics configured (`extra:` block has nothing for Google
  Analytics / Plausible / Fathom / GoatCounter)
- Current plugins: `search`, `tags`, `mermaid2`, `git-revision-date-localized`
- Current palette: indigo (default) — noted separately as a design/aesthetics item,
  not part of this SEO plan

---

## How to use this plan

Each item below is self-contained. For items marked **[Claude Code]**, the prompt
can be handed to a Claude Code session on `C:\GitFolder\mkdocs-blog` as-is. For
items marked **[Manual]**, these are one-off actions outside the repo (web
consoles, external submissions) that don't route through code changes — do these
yourself, no execution prompt needed. Items marked **[Hybrid]** have a small
Claude Code part and a manual follow-up.

Log every completed item in `_blog/change-log.md` and, where it changes site
config or structure meaningfully, note the rationale in `_blog/decisions.md` —
consistent with existing practice on this site.

---

## Item 1 — robots.txt **[Claude Code]**

**Why:** Explicitly tells crawlers they're allowed in and points them straight at
the sitemap. Low effort, zero risk, standard practice.

**Prompt:**

> Add a `robots.txt` file to `docs/` (so it publishes at the site root) with:
> ```
> User-agent: *
> Allow: /
>
> Sitemap: https://somelamecode.github.io/mkdocs-blog/sitemap.xml
> ```
> Confirm the site builds cleanly and that `robots.txt` appears at the root of the
> built `site/` output (not nested under a subfolder). Log this in
> `_blog/change-log.md` as a low-risk site-improvement item (`SI-NNN` in
> `content-backlog.md`).

---

## Item 2 — Confirm sitemap.xml is actually generating correctly **[Claude Code]**

**Why:** `site_url` is set, so this should already work, but it's worth a direct
confirmation rather than assuming — a missing trailing slash or misconfigured
`docs_dir`/`site_dir` can occasionally produce a sitemap with wrong URLs.

**Prompt:**

> Run a local `mkdocs build` and inspect the generated `site/sitemap.xml`. Confirm
> every URL listed is correct (uses the `https://somelamecode.github.io/mkdocs-blog/`
> base, no `localhost` or relative paths, no missing pages, no orphaned/placeholder
> pages like the Notebooks stub that shouldn't be indexed yet). Flag anything odd
> rather than fixing silently. No content changes — this is a verification pass.

---

## Item 3 — Per-page meta descriptions **[Claude Code, content audit]**

**Why:** Probably the single highest-value item here. Without a `description:` in
frontmatter, Google guesses a snippet from page content — often an awkward
mid-sentence fragment. A deliberate 1–2 sentence description directly controls
what shows up in search results and improves click-through.

**Prompt:**

> Audit every published page under `docs/articles/`, `docs/projects/`,
> `docs/manuals/`, and `docs/code/` for a `description:` field in frontmatter.
> Material for MkDocs uses this for the meta description tag. For any page
> missing one, draft a concise 1–2 sentence description (roughly 120–160
> characters, the range search engines typically display) that accurately
> summarizes the page — not a generic tagline, but specific to that page's actual
> content.
>
> Work through sections in this order: Projects first (highest-value pages),
> then Articles, then Manuals overview/hub pages (individual chapters are lower
> priority — flag whether per-chapter descriptions are worth the effort or
> whether the manual's own hub page description is sufficient), then Code.
>
> Present the list of proposed descriptions for review before committing them
> to files, since this is editorial judgment on many pages at once — don't
> silently write to 30+ files in one pass.

**➡️ Human checkpoint:** Review the proposed descriptions before they're written
to files — this is the item most likely to need editorial tweaks.

---

## Item 4 — Social cards (auto-generated OpenGraph images) **[Claude Code]**

**Why:** When a link to your site is shared (LinkedIn, X, Slack, Discord), Material's
`social` plugin auto-generates a preview card image per page instead of a bare
link. Meaningfully increases click-through on shared links — cheap to enable.

**Prompt:**

> Enable Material's built-in `social` plugin for auto-generated OpenGraph card
> images. This requires `pillow` and `cairosvg` — check `requirements.txt` and add
> them if missing. Add the `social` plugin to `mkdocs.yml` under `plugins:`.
>
> Check whether the site already has a font/logo suitable for the card template,
> or whether Material's defaults are fine as a starting point — don't over-invest
> in card styling in this pass, just get it working with sensible defaults.
>
> Build the site and spot-check a few generated cards (they land in
> `site/assets/images/social/`) for a couple of representative pages (one project,
> one article) to confirm they render correctly before finishing. Log this as a
> config change in `_blog/change-log.md`.

---

## Item 5 — RSS feed **[Claude Code]**

**Why:** Lets people and some aggregators/directories subscribe to new posts
without you doing anything per-post. Low effort, standard on most technical
blogs, and a small ongoing discovery channel.

**Prompt:**

> Add Material's `rss` plugin (check `requirements.txt` for `mkdocs-rss-plugin`
> and add if missing). Configure it to feed from `docs/articles/` and
> `docs/projects/` at minimum — use your judgment on whether `docs/manuals/`
> content (reference material, not date-driven posts) belongs in the feed or not;
> reference manuals arguably shouldn't clutter an RSS feed meant for "what's new."
> Confirm the feed builds and validates (a quick check against the generated
> `feed_rss_created.xml` / `feed_rss_updated.xml` structure is enough). Log this
> as a config change.

---

## Item 6 — Internal linking pass **[Claude Code, content audit]**

**Why:** Search engines weight internal link structure as a relevance signal, and
it's free — no new content needed, just connecting what already exists. It also
directly helps readers discover related material.

**Prompt:**

> Audit the site for obvious internal linking gaps between clearly related
> content that currently doesn't cross-link:
> - The Prisma Access SASE **manual** and the Prisma Access SASE **project** page
>   (if both exist and don't already reference each other)
> - The GitHub in a Nutshell **manual** and any **articles** that touch Git/GitHub
>   topics (check `docs/articles/` for overlap)
> - The NICU AI Strategy Workshop and NICU IRRBB PoC project pages (confirm the
>   cross-link noted in their respective execution plans is actually present and
>   bidirectional, not just Part 2 → Part 1)
> - Any other pairs of pages covering related tools/topics that don't reference
>   each other
>
> For each gap found, add a natural in-context link (not just a "see also" bolt-on
> unless that reads better for the specific case). Present the list of proposed
> links before making changes, since this touches many files. Log as a content
> quality pass in `_blog/change-log.md`.

**➡️ Human checkpoint:** Quick skim of proposed links — this is low-risk but
touches many files at once.

---

## Item 7 — Analytics **[Claude Code + Manual signup]**

**Why:** Without analytics, none of the above can be measured — you won't know if
traffic increases, where it comes from, or which pages people actually land on
from search. Worth having before investing more effort in the items above, so you
can tell what's working.

**Note on choice:** Google Analytics is free and ubiquitous but collects a lot and
needs a cookie-consent conversation. Privacy-friendly alternatives (Plausible,
Fathom, GoatCounter) are cookieless, lighter-weight, and need no consent banner —
better fit for a personal technical blog, though most aren't free. Your call.

**Manual step first:** Sign up for whichever analytics provider you choose and
get a site ID/tracking snippet.

**Prompt (once you have a provider and ID):**

> Wire up [chosen analytics provider] for the site. If it's Google Analytics,
> Material has built-in support via the `extra.analytics` block in `mkdocs.yml` —
> use that rather than a hand-rolled script tag. If it's a third-party
> cookieless provider (Plausible/Fathom/GoatCounter), add its script tag via
> `extra_javascript` or the appropriate Material hook, checking Material's docs
> for the recommended integration pattern for that specific provider. Confirm it
> loads on a test build without breaking anything. Log this as a config change.

---

## Item 8 — Google Search Console + Bing Webmaster Tools **[Manual]**

**Why:** Submitting the sitemap directly gets pages indexed in days instead of
however long organic crawling takes. Free, one-time setup, highest
return-on-effort item on this whole list.

**Steps (outside the repo, no Claude Code needed):**
1. Go to [Google Search Console](https://search.google.com/search-console),
   add the property (`https://somelamecode.github.io/mkdocs-blog/`), verify
   ownership (usually via a DNS TXT record or an HTML file — GitHub Pages
   supports either).
2. Submit `sitemap.xml` under Search Console's Sitemaps section.
3. Repeat for [Bing Webmaster Tools](https://www.bing.com/webmasters) — Bing
   also powers some other search surfaces, worth the extra five minutes.
4. Come back in a week or two and check the Coverage/Indexing report for any
   pages Google flagged as excluded — that's useful signal for later items.

---

## Item 9 — Off-site distribution **[Manual, judgment-heavy]**

**Why:** For a personal technical blog, this is often where the real traffic
comes from — search indexing takes time to compound, but a well-placed post can
drive traffic immediately.

**Options, roughly in order of effort:**
- **Cross-posting with canonical link** to dev.to or Hashnode: paste the article,
  set the canonical URL field to point back to your site, so you get the reach
  without splitting SEO credit. Low effort per post.
- **GitHub repo backlinks:** if `SomeLameCode/mkdocs-blog` or other repos
  (e.g. the NICU AI Rollout Strategy repo) have READMEs that could naturally
  link to a relevant blog post, that's a free, relevant backlink. Worth a quick
  pass — could hand this to Claude Code as a small follow-up item if you want.
- **Hacker News / relevant subreddits (r/devops, r/sysadmin, r/selfhosted,
  etc.) / lobste.rs:** for posts that genuinely fit a community's interest —
  one well-timed submission can outperform months of organic search for a
  personal blog. Judgment call per post, not something to automate or force.

This item doesn't need an execution prompt — it's an ongoing habit per post
rather than a one-time site change.

---

## Item 10 — Performance sanity check **[Manual, low priority]**

**Why:** Google factors page speed into ranking, though static MkDocs sites are
usually fine by default. Low priority — do this last, mostly as a "confirm
nothing's broken" step rather than expecting to find real problems.

**Steps:**
1. Run the live site through [PageSpeed Insights](https://pagespeed.web.dev/)
   or a local Lighthouse run on a couple of representative pages (homepage, one
   long manual chapter with a Mermaid diagram).
2. Only worth acting on if something looks clearly wrong (e.g. unexpectedly
   large images) — this isn't expected to surface much given the site's already
   static and lightweight.

---

## Suggested order

If you want a straightforward sequence rather than picking freely:

1. **Item 1** (robots.txt) — trivial, do it now
2. **Item 2** (verify sitemap) — five minutes, confirms Item 1 has something to
   point to
3. **Item 8** (Search Console / Bing) — do this early so indexing starts
   compounding while you work through the rest
4. **Item 3** (meta descriptions) — highest content-effort item, biggest
   click-through impact
5. **Item 4** (social cards) — cheap, high visual payoff for shared links
6. **Item 7** (analytics) — so you can start measuring before investing further
7. **Item 6** (internal linking) — cheap, ongoing
8. **Item 5** (RSS) — cheap, low urgency
9. **Item 9** (off-site) — ongoing habit, start whenever a post feels ready
10. **Item 10** (performance) — last, low expected payoff

---

## Notes for whoever runs this

- Verified state above reflects `mkdocs.yml` as of 2026-07-14 — if time has
  passed since, a quick re-check of current plugins/config is worth it before
  starting Item 4 or 5 (dependency check) or Item 7 (avoiding double-adding
  analytics if someone else already did it).
- None of these items depend on the Prisma Cloud, Manuals, or NICU publishing
  work in flight — safe to run independently and in parallel.
- Per the project's standing workflow: Claude Code writes directly to disk
  (`C:\GitFolder\mkdocs-blog`), commits go local-to-remote via normal git push,
  and none of this routes through Google Drive.
