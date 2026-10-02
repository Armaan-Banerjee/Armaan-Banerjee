# Armaan Banerjee — personal site

A [Jekyll](https://jekyllrb.com/) site with no plugins, so it builds the
same way locally and on GitHub Pages.

## Structure

```
index.md              Home — the opening statement + recent articles
about.md              /about/
contact.md            /contact/
articles.md           /articles/  — the full list, newest first
_articles/            One Markdown file per article
404.html              Shown for unknown addresses
feed.xml              Atom feed, linked from the footer

_layouts/             default, home, page, article, articles
_includes/            head, nav, footer, article-list
assets/css/style.css  All of the styling (no web fonts, no frameworks)
_config.yml           Title, tagline, links, article permalink
```

## Writing an article

Create a file in `_articles/`. **The filename becomes the URL**, so
`_articles/on-small-tools.md` publishes at `/articles/on-small-tools`.

```markdown
---
title: "On small tools"
date: 2026-09-27
summary: "One sentence, shown under the title in the index. Optional."
---

Your first paragraph.
```

`title` and `date` are required — the date orders the index and prints
in its left margin. Add `published: false` to keep a draft out of the
build.

## Adding a page

Any `.md` or `.html` file at the top level becomes a page:

```markdown
---
layout: page
title: Reading
permalink: /reading/
---
```

To put it in the nav, add it to the `links` list in
`_includes/nav.html`.

## Page width

The site fills the screen. Line length on a wide monitor is set by two
variables at the top of `assets/css/style.css`:

```css
--page-max: none;    /* e.g. 60rem to cap the column on large screens */
--page-pad: 1.25rem; /* side margin; grows at 48rem and 80rem */
```

## Local preview

```bash
bundle install
bundle exec jekyll serve
```

Then open <http://localhost:4000>. Edits to content reload
automatically; changes to `_config.yml` need a restart.

## Publishing to GitHub Pages

The repo currently on `origin` is **`Armaan-Banerjee/Armaan-Banerjee`**,
which GitHub treats as a profile-README repo, not a site. Pick one:

- **Site at `https://armaan-banerjee.github.io`** — push this to a repo
  named `Armaan-Banerjee.github.io`. `_config.yml` is already set for
  this case.
- **Site at `https://armaan-banerjee.github.io/<repo>/`** — keep any
  repo name and set `baseurl: "/<repo>"` in `_config.yml`. Every link
  in the templates goes through `relative_url`, so that one line is the
  only change needed.

Then: **Settings → Pages → Source → Deploy from a branch → `main` / `/`
(root)**. The first build takes a minute or two.
