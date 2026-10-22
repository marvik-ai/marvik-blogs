# Marvik blog corpus

Articles from the [Marvik blog](https://www.marvik.ai/blog), extracted from the site and converted to Markdown, plus a guide for writing new posts based on what works in the existing ones.

There is no code to build or run: everything is plain Markdown and CSV.

## Layout

```
.
├── index.csv          # One row per article: date, category, title, authors, url
├── articles/          # One <slug>.md per article
├── guide/
│   ├── corpus-analysis.md   # Patterns extracted from the corpus
│   └── blog-template.md     # Brief, skeleton and checklist for a new post
└── .claude/skills/    # Claude Code skills for reviewing the writing
```

## Conventions

- **File name = URL slug.** Each `articles/<slug>.md` matches the last segment of its `url` in `index.csv`.
- **Article header.** Every article starts with the same metadata block:

  ```markdown
  # <Title>

  - Date: June 24th, 2026
  - Category: Technical
  - Authors: Author 1; Author 2
  - URL: https://www.marvik.ai/blog/<slug>

  ---
  ```

- **`index.csv`** is sorted by date, descending. Dates use the site's format (`"June 24th, 2026"`), not ISO. Multiple authors are separated by `; `. The file uses CRLF line endings (enforced in `.gitattributes`).
- **Scraping artifacts** (stray U+200D characters, spaces before punctuation where there were links or bold text, missing images) are kept on purpose. Don't clean them up as part of unrelated changes.

## Writing a new post

1. Copy the skeleton from `guide/blog-template.md` and fill in the brief first.
2. Save it as `articles/<slug>.md` with the header above.
3. Add its row to `index.csv`, keeping the date order.
4. Review it with the checklist in the template. If you use [Claude Code](https://claude.com/claude-code), the `/stop-slop` and `/copy-editing` skills in `.claude/skills/` are available automatically in this directory.

## Credits

- Article content © Marvik and the respective authors.
- `.claude/skills/stop-slop` (MIT, Hardik Pandya) and `.claude/skills/copy-editing`, taken from [github.com/boraoztunc/skills](https://github.com/boraoztunc/skills).
