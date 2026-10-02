---
title: "Hello, world"
date: 2026-09-27
summary: "Why this site exists, and what I plan to put on it."
---

This is the first article on the site. It lives at
`_articles/hello-world.md`, which is why its address is
`/articles/hello-world/` — the filename becomes the URL, so name files
the way you want them to read.

## Writing a new one

Add a Markdown file to `_articles/`:

```markdown
---
title: "The title, in full"
date: 2026-09-27
summary: "One sentence. Optional; used as the page description."
---

Your first paragraph.
```

`title` and `date` are required. The date orders the list on the
[articles page]({{ '/articles/' | relative_url }}) and prints under the
title. Anything Markdown can do works here: lists, quotes, tables, code.

> Drafts stay out of the build if you add `published: false` to the
> front matter.

Delete this file once you've written something real.
