# bitbetter.se

Personal blog built with [Astro](https://astro.build), deployed to GitHub Pages.

## Develop

```bash
npm install
npm run dev
```

Open http://localhost:4321.

## Build

```bash
npm run build
npm run preview
```

## Deploy

Pushes to `master` deploy automatically via `.github/workflows/deploy.yml` (GitHub Pages → custom domain `bitbetter.se`).

**One-time setup** (already done if the site is live):

1. Repo Settings → Pages → Source: **GitHub Actions**.
2. Add an `A` record for `bitbetter.se` pointing at GitHub Pages IPs, plus a `CNAME` for `www` → `<user>.github.io`. See [GitHub's docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site).
3. The `public/CNAME` file makes the deploy preserve the custom domain.

## Writing a post

Add a markdown file in `src/content/blog/`. Frontmatter:

```yaml
---
title: My post
subtitle: Optional subtitle / summary
date: 2026-06-29
cover: optional-image.jpg   # file lives in public/images/
tags: [Astro, Notes]
draft: false                # true hides from build
---
```

The filename slug becomes the URL: `src/content/blog/my-post.md` → `https://bitbetter.se/my-post/`.
