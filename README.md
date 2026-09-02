# adityasphere.com

Personal blog of Aditya Chauhan. Built with Jekyll, served via GitHub Pages.

## Adding a post

Add a file to `_posts/` named `YYYY-MM-DD-slug.md` (or `.html`):

```
---
layout: post
title: "Your Title"
---
Your content here — Markdown in a .md file, raw HTML in a .html file.
```

It appears automatically on the homepage and at `/blog/slug/`. No build step to run locally — GitHub Pages builds it on push.

## Local preview (optional)

```
bundle exec jekyll serve
```
