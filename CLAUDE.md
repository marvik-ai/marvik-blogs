# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this directory is

It is not a software project: it is a corpus of articles from the Marvik blog (https://www.marvik.ai/blog) extracted from the site and converted to Markdown. There is no build, tests, linter or dependencies. See `README.md` for the layout and the steps to add a post.

## Structure

- `index.csv`: one record per article with columns `date,category,title,authors,url`. It is the reference index of the corpus.
  - `date` uses the site's format (`"June 24th, 2026"`), not ISO; parse it explicitly when sorting or filtering by date.
  - `authors` uses `; ` as the separator between multiple authors.
  - Rows are sorted by date, descending.
  - It has CRLF line endings (pinned in `.gitattributes`): run `tr -d '\r'` before using the fields in the shell.
- `articles/<slug>.md`: one file per article. The file name is the last segment of the `url` in `index.csv` (that's how the two are linked).
- `guide/`: material for writing new posts, not articles. `corpus-analysis.md` gathers the patterns extracted from the corpus and `blog-template.md` is the reusable template (brief, skeleton and checklist). To write a post, start from the template.
- `.claude/skills/`: `stop-slop` and `copy-editing`, taken from github.com/boraoztunc/skills (MIT), for reviewing the writing.

## Format of each article

They all share this header:

```markdown
# <Title>

- Date: <date, same as in index.csv>
- Category: <category>
- Authors: <Author 1; Author 2>
- URL: <url>

---

<article body>
```

When adding a new article, save it in `articles/`, keep that header and add the corresponding row to `index.csv` so both stay in sync.

## Extraction artifacts

The text comes from scraping Webflow HTML and keeps some artifacts; take them into account when processing or quoting the content, but don't "clean" them unless asked:

- Stray zero-width joiner characters (U+200D) on some lines.
- Spaces before punctuation (`code generation .`) where the original had bold text or links.
- Images were not kept; some articles do include code blocks.
