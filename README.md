# Personal site

Portfolio + blog built with [Hugo](https://gohugo.io), deployed to GitHub Pages by `.github/workflows/hugo.yml`.

```bash
hugo server            # preview at http://localhost:1313 (live reload)
hugo new posts/my-post.md
hugo new projects/my-project.md
```

`hugo new` creates posts with `draft = true`. Use `hugo server -D` to preview drafts, and delete that line to publish.
