---
title: "Blog Visibility & SEO for a MkDocs Site"
description: "A practical checklist for making a MkDocs + Material site discoverable — robots.txt, sitemap, meta descriptions, RSS, social cards, analytics, and search console verification — including two Windows-specific gotchas."
date: 2026-07-14
tags:
  - mkdocs
  - seo
  - analytics
  - rss
  - social-cards
status: published
---

# Blog Visibility & SEO for a MkDocs Site

A static site with good content is still invisible until crawlers can find it, links to it look good when shared, and you have some way to measure whether any of that is working. This is the checklist this site ran through — in order of effort vs. payoff — plus two Windows-specific dependency problems that aren't obvious until you hit them.

None of this needs a paid tool. Everything below is either a MkDocs/Material plugin, a static file, or a free third-party account.

---

## 1. robots.txt

The cheapest possible win: tell crawlers explicitly they're welcome, and point them straight at the sitemap MkDocs already generates for you (as long as `site_url` is set in `mkdocs.yml`).

Drop this in `docs/robots.txt` so it publishes at the site root:

```
User-agent: *
Allow: /

Sitemap: https://your-username.github.io/your-repo/sitemap.xml
```

Build and confirm it lands at `site/robots.txt`, not nested under a subfolder.

---

## 2. Verify the sitemap is actually correct

`site_url` being set means MkDocs *should* already be generating a correct `sitemap.xml` — but "should" is worth a five-minute check rather than an assumption. After a `mkdocs build`, open `site/sitemap.xml` and confirm:

- Every URL uses your real `https://...` base — no `localhost`, no relative paths
- No placeholder/stub pages you don't actually want indexed yet
- No orphaned pages that build but aren't in `nav` — these still land in the sitemap as crawlable duplicate content

That last point is worth calling out: a page doesn't need a `nav:` entry to build and get indexed. If you have leftover authoring artifacts sitting in `docs/` (old README-style files, draft scratch pages), they'll quietly show up in the sitemap as if they were real content. Either delete them, exclude them, or get them into `nav` properly.

---

## 3. Per-page meta descriptions

Without a `description:` field in frontmatter, Google guesses a snippet from the page body — often an awkward mid-sentence fragment. Material for MkDocs reads `description:` from frontmatter straight into the `<meta name="description">` tag, so this is direct control over what shows up in search results.

```yaml
---
title: "Page Title"
description: "A specific 1-2 sentence summary of this exact page's content, not a generic tagline."
date: 2026-01-01
status: published
---
```

Aim for roughly 120–160 characters — the range most search engines actually display before truncating. Audit your published pages for gaps, starting with your highest-traffic sections (project/portfolio pages before reference chapters), and don't feel obligated to write one for every single chapter in a large multi-chapter manual — a good hub-page description often carries the section.

---

## 4. RSS feed

`mkdocs-rss-plugin` gives readers and aggregators a subscribe path with zero effort per post:

```yaml
plugins:
  - rss:
      match_path: "(articles|projects)/.*"
      date_from_meta:
        as_creation: date
      categories:
        - tags
```

`match_path` scopes the feed — reference material (a multi-chapter manual, for instance) usually doesn't belong in a feed meant to answer "what's new," so exclude it. Dates come from your frontmatter `date:` field, categories map from `tags:`.

!!! warning "Windows gotcha: broken social-card URLs in the feed"
    The plugin's default `use_material_social_cards: true` builds per-page card-image URLs using the OS's native path separator. On Windows, that means a mix of forward slashes and backslashes — `social/projects\index.png` instead of `social/projects/index.png` — which 404s in any RSS reader that tries to load it. This has nothing to do with whether the `social` plugin itself works (see below); it's purely how the RSS plugin joins the path.

    Set `use_material_social_cards: false` to sidestep it. Readers fall back to your site's default OG image instead of a per-page card — a reasonable trade-off until the plugin fixes its path handling on Windows.

---

## 5. Social cards (auto-generated OpenGraph images)

Material's built-in `social` plugin generates a preview card image per page — the thing you see when a link gets shared on Slack, LinkedIn, X, or iMessage instead of a bare URL. Enable it:

```yaml
plugins:
  - social
```

It needs `pillow` and `cairosvg` in your environment. On Linux and macOS that's usually just a `pip install`. On Windows, it isn't.

!!! failure "Windows gotcha: cairosvg needs a native Cairo build"
    `cairosvg` depends on the native Cairo C library (`libcairo-2.dll`), which has no pip wheel on Windows. `pip install cairosvg` succeeds, but the first `mkdocs build` fails with `cairosvg` crash warnings the moment it tries to render a card.

    The fix that actually worked, without installing a heavier GTK3 runtime:

    1. Install [MSYS2](https://www.msys2.org/) — accept the defaults.
    2. Open the **MSYS2 UCRT64** shell specifically (a separate Start Menu entry from install — not the plain MSYS2 shell).
    3. In that shell, install the Cairo package:
       ```bash
       pacman -S mingw-w64-ucrt-x86_64-cairo
       ```
    4. Add `C:\msys64\ucrt64\bin` to your Windows PATH (System Properties → Environment Variables → Path).
    5. Restart your terminal (and IDE) so the PATH change takes effect.
    6. Verify:
       ```bash
       python -c "import cairosvg; print('ok')"
       ```

    Once that prints `ok`, `social` builds cleanly and generates a PNG per page under `site/assets/images/social/`.

Spot-check a couple of generated cards before calling it done — open one directly and confirm title, description, and brand color render as expected, not just that the build didn't error.

To actually see a card once deployed: view a page's source and look for the `og:image` meta tag, or open that image URL directly. A social-media link-preview debugger tool will render the full "shared post" look, but that sends your URL to a third party, so it's optional.

---

## 6. Internal linking pass

Free, and easy to skip by accident: search engines weight internal link structure as a relevance signal, and it directly helps readers discover related material. Periodically scan for pairs of pages covering related topics that don't reference each other — a project page and the reference manual it draws on, two articles that touch the same underlying tool — and add a natural in-context link, not just a bolted-on "see also" unless that genuinely reads better.

---

## 7. Analytics

You can't tell whether any of the above is working without some way to measure it. Material's `extra.analytics` block has native support for Google Analytics only. If you'd rather use a cookieless, consent-banner-free alternative (GoatCounter, Plausible, Fathom), wire it in by hand via `extra_javascript`:

```js
// docs/javascripts/goatcounter.js
(function () {
  var script = document.createElement("script");
  script.async = true;
  script.src = "//gc.zgo.at/count.js";
  script.setAttribute("data-goatcounter", "https://your-site.goatcounter.com/count");
  document.head.appendChild(script);
})();
```

```yaml
extra_javascript:
  - javascripts/goatcounter.js
```

Confirm the script tag actually renders on both the homepage and a nested page — relative paths in `extra_javascript` resolve differently depending on how deep the current page sits, so it's worth checking both, not just one.

---

## 8. Search Console + Bing Webmaster Tools

The highest return-on-effort item on this whole list, and the only one that isn't a code change:

1. Add your property in [Google Search Console](https://search.google.com/search-console), verify ownership (DNS TXT record or an HTML verification file — GitHub Pages supports either; drop the file straight in `docs/` so it publishes at the site root).
2. Submit `sitemap.xml` under Search Console's Sitemaps section.
3. Repeat for [Bing Webmaster Tools](https://www.bing.com/webmasters) — a few extra minutes, and Bing feeds other search surfaces too.
4. Check back in a week or two for anything flagged as excluded in the coverage report.

---

## Key Takeaways

- robots.txt, sitemap verification, and meta descriptions are nearly free and worth doing first
- `use_material_social_cards` and the `social` plugin are two independent things — a bug in one doesn't mean the other is broken, and it's worth testing them separately when something doesn't render right
- On Windows, `cairosvg` needs a native Cairo build — MSYS2 + `mingw-w64-ucrt-x86_64-cairo` is a lighter path than a full GTK3 runtime install
- `mkdocs-rss-plugin`'s social-card image URLs use OS-native path separators, which breaks on Windows regardless of whether Cairo is working — verify the actual built feed XML, not just that the build exited cleanly
- Analytics should go in early, not last — you can't tell if any of this moved the needle without it

---

## References

- [Material for MkDocs — Social plugin](https://squidfunk.github.io/mkdocs-material/setup/setting-up-social-cards/)
- [mkdocs-rss-plugin](https://guts.github.io/mkdocs-rss-plugin/)
- [MSYS2](https://www.msys2.org/)
- [Google Search Console](https://search.google.com/search-console)
- [Bing Webmaster Tools](https://www.bing.com/webmasters)
