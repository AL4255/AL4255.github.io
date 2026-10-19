---
title: "Hello, world: how this site is built"
date: 2026-10-19
tags: [programming]
---

This site is built with [Hugo](https://gohugo.io) and hosted for free on GitHub Pages.

Every push to `main` triggers a GitHub Action that runs:

```bash
hugo --minify
```

and publishes the generated `public/` folder. No servers to patch, nothing to log into — just static files.
