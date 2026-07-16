# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Ben Keil's personal website (benkeil.com) — hand-written static HTML + CSS. No build step, no framework, no dependencies, no tests. Deployed by GitHub Pages from the repo root; the `CNAME` file pins the domain.

To preview locally, serve the root over HTTP rather than opening files directly, since pages use absolute paths (`/blog`, `/style.css`):

```
python3 -m http.server 8000
```

## Deployment: Jekyll is on

There is no `_config.yml` and no `.nojekyll`, so GitHub Pages builds with stock Jekyll. The practical consequence: **files and directories prefixed with `_` are not published**. `blog/_template.html` relies on this to stay out of the live site. Adding a `.nojekyll` file would publish it.

## Architecture

Every page is a standalone, complete HTML document — nav, footer, and hamburger script are copy-pasted into each one. There is no include or templating mechanism. A nav change means editing every page.

`style.css` is the shared stylesheet for the whole site. `resumeStyle.css` is loaded *in addition* to it, only by `resume.html`.

### Two generations of pages

The site has been half-rewritten, and the two styles are not compatible:

- **Current** — `index.html`, `blog/index.html`, `blog/day-one.html`. Use `<header><div class="container">` with `.nav-links` and a hamburger button, and the `.section` system in `style.css`.
- **Legacy** — `projects.html`, `connect.html`, `music.html`, `resume.html`. Use a bare `<nav>` of `<a>` tags, load Montserrat from Google Fonts, and title themselves "Benjamin B. Keil". They are orphaned: the current nav points at `/#projects` and `/#contact` (anchors on the home page), not at these files. Their own navs link to `/hobbies`, which does not exist.

When editing, match the generation of the file you are in. Don't assume a class exists site-wide.

## CSS gotchas

These have caused real bugs — read before touching layout.

**`.section` is a full-screen panel, not a container.** It carries `min-height: 100vh`, flex centering, and `text-align: center`, and `.section p` forces `max-width: 600px; margin: 0 auto`. Applying it to ordinary content produces a full-viewport, center-aligned block. It's right for the home page's scroll-snap sections and wrong for everything else. Blog pages deliberately avoid it and use `.blog-list` / `.post` instead.

**Section backgrounds come from DOM position.** `.section:nth-child(even|odd)` alternates `#2a2a2a` / `#1e1e1e`, so inserting or reordering a section on the home page reshuffles every background below it.

**The site nav is `body > header`, scoped deliberately.** It is sticky with `z-index: 100`. The selector is scoped to direct children of `body` so that nested `<header>` elements (like `.post-header` in a blog post) don't inherit a sticky gray nav bar. Do not loosen it back to `header`.

## Blog

`blog/index.html` lists posts; each post is its own file at `blog/<slug>.html`. `blog/_template.html` is the canonical starting point and documents the conventions inline.

**Adding a post takes two edits** — create `blog/<slug>.html` from the template, then add an `<article class="blog-preview">` row to `blog/index.html`. Nothing generates the listing; a post not added there is unreachable.

Post conventions, all encoded in the template:

- **Hashtags are a `<ul class="post-tags">`** after the body, rendered as pills below a divider — never a trailing line of prose.
- **`.post-lead` (centered opener) is only for a first paragraph that is exactly one sentence.** Two or more sentences: drop the class and let it start in `.post-body`, left-aligned. Sentence count is the entire test.
- Body prose is left-aligned in `.post-body`, one `<p>` per paragraph. No inline styles, no `<br>` for spacing.
- `.blog-excerpt` on the index is clamped to two lines by CSS, so excerpts need no manual "…".
