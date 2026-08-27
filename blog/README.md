# Blog — technical reference

How `hivion.ai/blog/` is built and how to publish a post. Static hand-written HTML like the rest of the site: **no generator, no CMS, no build step, no third-party scripts.**

Editorial direction, positioning, and voice live in the private `document/` folder, not here.

## Files

| Path | Role |
|---|---|
| `blog/index.html` | Post list. Rows styled like the home page's services index. |
| `blog/_template.html` | Copy-to-create template. `noindex, nofollow`; never linked. |
| `blog/<slug>/index.html` | One folder per post. |
| `blog/feed.xml` | Hand-maintained RSS 2.0. |
| `sitemap.xml`, `robots.txt` | Repo root. Hand-maintained. |
| `.nojekyll` | Repo root. **Required** — without it GitHub Pages runs Jekyll, which ignores underscore-prefixed paths and would refuse to publish `_template.html`. |

## Hard rules

1. **URLs are `blog/<slug>/index.html`**, served as `https://hivion.ai/blog/<slug>/`. Slugs are kebab-case, lowercase, and carry **no date**. Never `/blog/<slug>.html`.
2. **Slugs are permanent.** GitHub Pages has no redirect mechanism here, so renaming a published slug breaks every inbound link. Choose carefully at publish time.
3. **Trailing slash everywhere, identically.** The canonical, `og:url`, RSS `<link>` and `<guid>`, the `sitemap.xml` `<loc>`, and the blog-index link must be byte-identical. A mismatch on the slash creates a redirect the sitemap points across — self-inflicted duplicate content.
4. **All asset paths are root-absolute** — `/assets/fonts/…`, `/favicon.svg`. Posts sit two levels deep, so a relative path resolves to `/blog/<slug>/assets/…` and 404s. This fails *silently*: the page still renders, just in system fonts. It also looks fine over `file://`, so it survives local checking. Check the Network tab, not the page.
5. **Each file is self-contained** — styles inline in `<style>`, fonts via the same `@font-face` blocks as `privacy.html`. Duplicated CSS across posts is correct here; a build step is not.
6. **No Google Fonts link.** The Content-Security-Policy forbids it and the site is deliberately tracker-free. Every page carries the same CSP; keep it identical across new pages.
7. **Design tokens** are the live site's: `--cream #f8f6ee`, `--ink #1b2333`, `--muted #6f7382`, `--green #8a6a1a` (gold accent), `--hairline #e6e2d3`. Inter for body, JetBrains Mono for kickers and labels. No new colors.

## `.gitignore` is an allowlist — read before creating any file

Line 2 of `.gitignore` is `/*`, so **everything is ignored by default** and only explicitly re-included paths are tracked. A new top-level file or folder is silently dropped from commits and never deploys until a `!/path` entry exists.

`!/blog/` is already present, so anything under `blog/` is tracked automatically. For anything at the repo root, verify:

```
git check-ignore -v <path>     # must print nothing
```

## `og:image` — season per service area

Each service area uses the season painting from its home-page scene. The home page is authoritative; keep these aligned.

| Service area | Image |
|---|---|
| Data platforms & analytics | `/assets/summer.jpg` |
| AI agents & automation | `/assets/spring.jpg` |
| AI MVP for founders | `/assets/autumn.jpg` |
| AI strategy sprint | `/assets/winter.jpg` |

## Publishing a post

1. `cp blog/_template.html blog/<slug>/index.html`. Replace every `[PLACEHOLDER]` — `grep -c PLACEHOLDER` should reach 0.
2. Set `<title>`, meta description (under 160 chars), canonical, `og:`/`twitter:` tags, `article:published_time`, and the `og:image` from the table above.
3. Fill the `BlogPosting` JSON-LD at the bottom — headline, description, `datePublished`, author.
4. Add a `.row` to `blog/index.html`, **newest first**. Note the copy-paste stub above the list sits inside an HTML comment that closes *after* the first real row — paste new rows into the live `.list` block, not into the comment.
5. Add an `<item>` to `blog/feed.xml`, newest first. `pubDate` is RFC-822 (`Sat, 22 Aug 2026 09:00:00 +0000`) — get the weekday right.
6. Add a `<url>` to `sitemap.xml` with `<loc>` byte-identical to the canonical, and `<lastmod>` the publish date. Bump `<lastmod>` on `/blog/` and `/` too — they changed. **Leave `privacy.html` and `terms.html` alone** unless they actually changed; stamping every URL with today's date teaches crawlers your dates are noise.
7. Commit. `git check-ignore -v blog/<slug>/index.html` must print nothing.
8. **After deploy**, verify on the live site: the page returns 200, fonts load with no 404s in the Network tab, `/sitemap.xml` and `/blog/feed.xml` return 200, and the OG preview renders in a real LinkedIn composer (the first scrape is cached aggressively — check before sharing widely).
9. Search Console → URL Inspection → Request indexing for the new post.

## Editorial gate — run on every draft

- [ ] Every factual claim traces to a source that has been **opened and read**, not merely cited. Anything unsourced is cut, not softened.
- [ ] No client names. No invented metrics, benchmarks, or percentages.
- [ ] Charts and diagrams built on modelled rather than measured data say so **in the caption**.
- [ ] No prices or fixed-price language.
- [ ] Read aloud once end to end. If a paragraph sounds like a press release, rewrite it.
- [ ] Anything asserted about our own work has been personally verified by the author.

## Measurement

**Google Search Console only** — verified by DNS TXT, sitemap submitted. It adds no script, no cookie, and no third-party request, which is why `privacy.html`'s "no analytics or tracking scripts" claim stays true and the `connect-src 'none'` CSP is untouched.

Before adding any on-site analytics, note that [privacy.html:82](../privacy.html#L82) publicly commits to updating the privacy policy *before* such a change takes effect. The policy and the script ship in the same pull request — never the script first.

## Deployment

GitHub Pages from `main`. A merge to `main` is a deploy; there is no build step and no staging environment.
