# hivion.ai

Static site for **Hivion Labs**, a data-first AI studio.

- **Live:** https://hivion.ai
- **Hosting:** GitHub Pages from `main`. A merge to `main` is a deploy — there is no build step and no staging environment.

## Layout

| Path | Role |
|---|---|
| `index.html` | The site. One self-contained file. |
| `blog/` | Post list, one folder per post, RSS feed, and a `noindex` post template. |
| `privacy.html`, `terms.html` | Policy pages. |
| `assets/` | Paintings and self-hosted font files. |
| `robots.txt`, `sitemap.xml` | Hand-maintained. |
| `.nojekyll` | **Required.** Without it GitHub Pages runs Jekyll, which ignores underscore-prefixed paths and would refuse to publish `blog/_template.html`. |

## Editing

Every page is self-contained — styles inline in `<style>`, no framework, no generator, no npm, no build step.

Two constraints that are easy to break by accident:

1. **Fonts are self-hosted** from `/assets/fonts/` via `@font-face`. There is no Google Fonts link and there cannot be one: every page carries a Content-Security-Policy with `font-src 'self'` and `connect-src 'none'`. Keep that policy identical across new pages.
2. **`.gitignore` is an allowlist.** Line 2 is `/*`, so everything is ignored by default and only explicit `!/path` entries are tracked. A new top-level file is silently dropped from commits and never deploys. Verify with:

   ```
   git check-ignore -v <path>     # must print nothing
   ```
