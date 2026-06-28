# Xariv Engineering Blog

A Jekyll site published with GitHub Pages. New articles are plain Markdown files —
drop one in `_posts/`, push, and it goes live.

---

## ✍️ Publishing a new article (the everyday workflow)

1. Create a file in `_posts/` named `YYYY-MM-DD-your-title.md`
   (the date and slug come from the filename — this controls the URL and order).
2. Paste the template below, fill it in, write your article in Markdown.
3. `git add`, `git commit`, `git push`. GitHub rebuilds the site in ~1 minute.

### Article template

```markdown
---
title: "Your Article Title"
date: 2026-06-15
author: Your Name
category: "Performance"          # becomes the filter chip on the home page
excerpt_override: "One-line summary shown under the title and on the card."
# image: /assets/img/your-hero.png   # optional hero image
---

Opening paragraph.

## A section heading

Body text, **bold**, `inline code`, [links](https://example.com).

\`\`\`python
print("code blocks are styled")
\`\`\`

> Pull quotes look like this.
```

**Categories** are free text. Reuse the same spelling (e.g. always `"Performance"`)
and a filter chip appears automatically. Add images to `assets/img/`.

### XARIV Lens & Pulse

Interactive tools live at `/lens/` and `/pulse/` — both run entirely in the browser.

```bash
cd ../xariv-platform/frontend
npm run deploy:lens
npm run deploy:pulse
```

Then commit and push the updated `lens/` and `pulse/` folders in this repo.

---

## 🔧 First-time configuration

Open `_config.yml` and set:
- `url` → `https://YOUR_USERNAME.github.io`
- `baseurl` → `/REPO_NAME` (or `""` if the repo is `YOUR_USERNAME.github.io`)

Then replace `YOUR_USERNAME` in `_includes/header.html` and `_includes/footer.html`.

---

## 💻 Run locally (optional)

```bash
bundle install
bundle exec jekyll serve
# open http://localhost:4000
```

You only need this if you want to preview before pushing. You can also just push and
preview on the live URL.
