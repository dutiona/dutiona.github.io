# dutiona.github.io

Personal academic website of **Michaël Roynard** (image processing, modern C++, theoretical computer science), live at <https://dutiona.github.io/>. Built on the **al-folio** Jekyll theme, vendored at **v0.16.3** (Tailwind engine). This repo deliberately **carries al-folio's git history but is NOT a GitHub fork** — it is standalone. Long-term goal: publish my papers as arxiv-style static HTML with citations.

## Build / preview / deploy

**Do not `bundle install` locally** — the system Ruby gem dir (`/var/lib/gems`) is root-owned and bundler fails. Use the al-folio **Docker** image for everything:

- **Preview** (hot-reload at <http://localhost:8080>):
  ```bash
  docker run --rm -p 8080:8080 -v "$PWD":/srv/jekyll -w /srv/jekyll \
    -e JEKYLL_ENV=development amirpourmand/al-folio:v0.16.3 \
    bash -lc 'bundle exec jekyll serve --port=8080 --host=0.0.0.0 --force_polling'
  ```
- **One-off build** (to catch errors before pushing): same image, `bash -lc 'bundle exec jekyll build -d /tmp/_site --trace'`. The container runs as root, so `_site`/`.jekyll-cache` land root-owned — clean them via the same image.

**Deploy is automatic**: push to `main` → `.github/workflows/deploy.yml` builds (Ruby + Node + Tailwind + PurgeCSS) and force-pushes `_site` to **`gh-pages`**, which GitHub Pages serves. There is no manual deploy step.

## Where content lives (everything else is theme)

| Path                                                   | What                                                                                            |
| ------------------------------------------------------ | ----------------------------------------------------------------------------------------------- |
| `_bibliography/papers.bib`                             | publications (jekyll-scholar). `selected={true}` → homepage; `bibtex_show={true}` → Cite button |
| `_bibliography/preprints.bib`                          | **GENERATED** — self-archived preprints (rendered on `/publications/`). Do not hand-edit        |
| `papers/<slug>/`                                       | **GENERATED** — arxiv-style HTML of self-archived papers (static passthrough). Do not hand-edit |
| `_data/cv.yml`                                         | CV (rendered by `_pages/cv.md`). Also `coauthors.yml`, `repositories.yml`, `socials.yml`        |
| `_pages/*.md`                                          | site pages (about, cv, publications, projects, repositories, teaching)                          |
| `_projects/*.md`, `_news/*.md`                         | project cards, news items                                                                       |
| `assets/img/photo_perso.png`, `assets/pdf/shortcv.pdf` | personal assets                                                                                 |

> **`papers/` and `_bibliography/preprints.bib` are produced by the `dutiona/papers`
> publishing pipeline** (`tools/publish`, `make publish`). They are committed output:
> review the diff and push, but never hand-edit — the next `make publish` overwrites them.
> Papers live at top-level `/papers/<slug>/` (not under `assets/`) so PurgeCSS scans them
> and the sitemap includes them; their CSS is self-contained so the global purge can't strip it.

## Load-bearing gotchas — do not "fix" these away

- **`deploy.yml` runs `touch _site/.nojekyll`** — REQUIRED. Without it, GitHub Pages re-runs safe-mode Jekyll on the already-built `gh-pages` (al-folio's plugins are disallowed there) and the live site **404s**. Never remove that line.
- **`_config.yml`: `baseurl: ""`** (empty) — this is a _user_ site served at the domain root, not a project subpath. `url: https://dutiona.github.io`.
- **`_data/socials.yml` keys must be jekyll-socials' exact names** (`email`, `github_username`, `x_username`, `scholar_userid`, `orcid_id`, `stackoverflow_id`, `cv_pdf`, `rss_icon`). An unrecognized key (e.g. `twitter`) **crashes the build** via the plugin's custom-social path.
- Comments and analytics are off by design; OpenGraph + schema.org are on (`serve_og_meta` / `serve_schema_org: true`) for discoverability.

## Updating al-folio (lineage-preserving — never re-fork)

The repo keeps al-folio's history, so updates are a merge/cherry-pick, not a re-fork:

```bash
git remote add upstream https://github.com/alshedivat/al-folio.git   # once, if absent
git fetch upstream
git merge upstream/main        # or: git cherry-pick <sha> for selective updates
```

Rebuild + preview via Docker before pushing. Prefer pinning to a release tag.

## More

al-folio's own docs: `docs/INSTALL.md`, `docs/CUSTOMIZE.md`, `docs/FAQ.md`.
