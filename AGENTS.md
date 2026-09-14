# AGENTS.md

## Project

Jekyll 4.3 multilingual course site for **Service-Oriented Architecture and Cloud Computing** at the Industrial University of Ho Chi Minh City (IUH). Built on the Lanyon theme, deployed to GitHub Pages via GitHub Actions. Uses custom Jekyll plugins for language switching, URL redirects, and full-text search.

**Live site:** https://nglelinh.github.io/service-oriented-architecture-and-cloud-computing-iuh/

**Instructor:** Nguyen Le Linh (`nglelinh@gmail.com`)

## Commands

```bash
bundle install                              # Install Ruby dependencies (Jekyll ~> 4.3.0)
bundle exec jekyll serve                    # Local dev at http://127.0.0.1:4000/service-oriented-architecture-and-cloud-computing-iuh/
bundle exec jekyll build                    # Production build (output: _site/)
```

After changing `_config.yml`, restart Jekyll.

## Repository Layout

```
.
├── _config.yml                 # Site config, translations, author, baseurl
├── _includes/                  # head.html (MathJax, CSS), sidebar.html (chapter nav)
├── _layouts/                   # default.html, page.html, post.html
├── _plugins/                   # Custom Jekyll plugins (see below)
├── contents/
│   ├── en/chapterXX/
│   │   ├── index.html          # Chapter landing page (sidebar entry)
│   │   └── _posts/             # Lecture markdown files
│   └── vi/chapterXX/           # Vietnamese mirror (not yet populated)
├── home/_posts/                # Home page sections (order-driven)
├── contribution/_posts/        # Contributor docs (legacy convex-optimization text)
├── reference/                  # Reference materials (chapter 26)
├── extracted_slides/             # Source lecture slide markdown (conversion reference)
├── public/                     # CSS, JS, logos (served at /public/)
├── img/chapter_img/            # Lecture images (mix of cloud + legacy optimization assets)
├── index.html                  # Home page (renders home/_posts by order)
├── Gemfile                     # jekyll, jekyll-feed, jekyll-paginate, webrick
├── AUTHORS.md                  # Instructor bio
├── DEPLOYMENT.md               # GitHub Pages deployment notes
└── .github/workflows/jekyll.yml
```

## Configuration

Key settings in `_config.yml`:

| Setting | Value |
|---------|-------|
| `title` | Service-Oriented Architecture and Cloud Computing |
| `url` | https://nglelinh.github.io |
| `baseurl` | `/service-oriented-architecture-and-cloud-computing-iuh` |
| `imgurl` | https://nglelinh.github.io/service-oriented-architecture-and-cloud-computing-iuh/img |
| `languages` | `["en", "vi"]` |
| `default_lang` | `"en"` |
| `markdown` | `kramdown` |
| `version` | `0.0.1` |

UI strings live under `t.en.*` and `t.vi.*` (`home`, `chapters`, `required`, `optional`, `switch_language`, etc.).

GitHub repo link in `_layouts/default.html` points to `nglelinh/service-oriented-architecture-and-cloud-computing-iuh`.

## Course Content (14 Chapters)

English lectures live under `contents/en/chapter01/` through `chapter14/`. Vietnamese (`contents/vi/`) has Chapter 01 populated; later chapters are not translated yet.

| Ch | Sidebar title (`index.html`) | Actual lecture topics (`_posts/`) |
|----|------------------------------|-------------------------------------|
| 01 | Cloud Computing Fundamentals | NIST definition, service/deployment models, benefits, data centers |
| 02 | MapReduce Fundamentals | Distributed systems fundamentals |
| 03 | MapReduce Programming | Computing models, Hadoop YARN |
| 04 | Hadoop Ecosystem and YARN | Hadoop MapReduce |
| 05 | Apache Spark Fundamentals | Spark architecture, RDDs, transformations |
| 06 | Spark Streaming | Spark Streaming, Spark SQL, MLlib |
| 07 | Spark SQL | NoSQL and distributed databases |
| 08 | Spark MLlib | Virtualization, containerization, Docker, serverless |
| 09 | NoSQL Key-Value Stores | Kubernetes fundamentals, pods, services/deployments |
| 10 | Data Sourcing and Cleaning | Big data platforms and processing |
| 11 | Advanced Cloud Computing | Data sourcing, cleaning, preparation |
| 12 | Containerization with Docker | AWS, Azure, GCP cloud providers |
| 13 | Infrastructure as Code (IaC) | Deployment, security, compliance |
| 14 | Kubernetes Container Orchestration | Infrastructure as Code with Terraform |

**Known inconsistency:** Several `index.html` sidebar titles do not match the lecture post content (legacy from template migration). When editing, align `index.html` titles with actual `_posts/` topics.

`extracted_slides/` contains source slide decks (e.g., `Lecture_1.1_Clouds Intro.md`, `Lecture_6.2_Docker_kukenetes.md`) useful as reference when writing or expanding lectures.

## Content Structure

### Lecture posts

Path: `contents/{lang}/chapterXX/_posts/YYYY-MM-DD-title.md`

Required front matter:

```yaml
---
layout: post
title: "Lesson Title"
chapter: 'XX'           # Two-digit chapter number as string
order: N                # Integer ordering within chapter (drives prev/next nav)
owner: Nguyen Le Linh
lang: en                # 'en' or 'vi'
categories:
- chapterXX             # Must match chapter directory name
lesson_type: required   # Optional: 'required' or 'optional' (shows badge in sidebar)
---
```

Language switching matches posts by `chapter` + `order` across `en`/`vi`. Keep these aligned when adding bilingual content.

Post filenames follow `YY-MM-DD-chapter_lesson_name.md` (e.g., `21-01-01-01_01_Cloud_Computing_Characteristics.md`).

### Chapter landing pages

Each chapter needs `contents/{lang}/chapterXX/index.html`:

```yaml
---
layout: page
lang: en
title: "Chapter Title"
chapter: "XX"
owner: "Nguyen Le Linh"
---
```

Sidebar lists pages with `layout: page` and a `chapter` field, sorted by `chapter`.

### Home page

`index.html` renders posts from `home/_posts/` where `categories` includes `home`, sorted by `order`:

| File | Purpose |
|------|---------|
| `21-01-20-contents.md` | Course outline (still contains legacy convex-optimization text — needs update) |
| `21-01-20-introduction.md` | Course intro (stub) |
| `21-02-03-makers.md` | Instructor info (stub, hidden) |
| `21-05-20-author-details.md` | Author details |
| `21-01-27-link_to_how_to_contribute.md` | Contribution link |

Home posts use `chapter: home` and an `order` field.

### Adding a new chapter

1. Create `contents/en/chapterXX/` and `contents/vi/chapterXX/` with `_posts/` subdirs
2. Add `index.html` in each language directory
3. Add lecture posts with matching `chapter`, `order`, `lang`, and `categories`
4. Posts auto-appear in sidebar via chapter index and in prev/next navigation via `order`

### Adding Vietnamese translations

1. Mirror the English directory structure under `contents/vi/chapterXX/`
2. Use the same `chapter` and `order` values as the English counterpart
3. Set `lang: vi` in front matter
4. Add matching `index.html` with `lang: vi`

## Math and LaTeX

MathJax 3 is loaded in `_includes/head.html`. Use `$$...$$` for both inline and display math (not single `$`):

```markdown
Inline: $$f(x) = x^2$$

Display block:
$$
\nabla f(x) = 0
$$
```

## Images

Place images in `img/chapter_img/` and reference in markdown:

```markdown
![Alt text]({{ site.imgurl }}/chapter_img/chapter01/image.png)
```

For centered figures with captions, use the HTML figure convention (see `contribution/_posts/21-02-03-conventions.md`):

```html
<figure class="image" style="align: center;">
<p align="center">
  <img src="{{ site.imgurl }}/chapter_img/image.png" alt="description" width="80%">
  <figcaption style="text-align: center;">Caption text</figcaption>
</p>
</figure>
```

Note: `img/chapter_img/` still contains legacy optimization-course images (chapters 05, 09, 12, 13, 15, 18, 25). Add cloud-specific images under appropriate `chapterXX/` subdirectories.

## Internal Links

Use the `multilang_post_url` Liquid tag for cross-post links:

```markdown
[See Spark Fundamentals]({% multilang_post_url contents/chapter05/21-01-01-05_01_Spark_Fundamentals %})
```

## Custom Plugins

Located in `_plugins/`:

| Plugin | Tags / behavior |
|--------|-----------------|
| `multilang.rb` | `{% t key %}` — translate UI string; `{% language_switch %}` — lang toggle link; `{% lang en %}` — active class |
| `multilang_post_url.rb` | `{% multilang_post_url contents/chapterXX/post-name %}` — resolve post URL in current language |
| `redirect_generator.rb` | Generates redirect pages from legacy `/contents/chapterXX/` URLs to `/contents/en/chapterXX/` |
| `search_generator.rb` | Builds `search-index.json` and `search-index-vi.json` at build time for Lunr.js full-text search |

Search index files are gitignored (generated on build). Lunr.js is loaded in `_includes/head.html`; search UI is in `public/js/search.js`.

## URL Structure

- English chapters: `/contents/en/chapter01/`
- Vietnamese chapters: `/contents/vi/chapter01/` (when populated)
- Legacy redirects: `/contents/chapter01/` → `/contents/en/chapter01/`
- All URLs are prefixed with `baseurl` on GitHub Pages

## Styling and Assets

- CSS: `public/css/` (`lanyon.css`, `poole.css`, `syntax.css`, `github-markdown.css`, `multilang.css`, `content-boxes.css`, `search.css`)
- JS: `public/js/script.js`, `public/js/multilang.js`, `public/js/search.js`
- Logos: `public/logo.png`, `public/convex-logo-144x144.png` (legacy favicon name)

## Deployment

Push to `main`. GitHub Actions (`.github/workflows/jekyll.yml`) builds with Ruby 3.2 and deploys via `actions/deploy-pages`.

Repository settings: **Settings > Pages > Source: GitHub Actions**.

Custom plugins require Actions-based deploy (not the default GitHub Pages Jekyll build). `.nojekyll` is present to skip the default build.

## Lecture Writing Guidelines

See `.cursor/rules/` when authoring course content:

- `lecture_notes_rule.md` — Course is "Introduction to Cloud Computing Technologies" (14 lectures). Structure: objectives, prerequisites, introduction, key concepts, algorithms/methods, examples (Python), applications, challenges, exercises, references. Target 1500–3000 words per lecture. Content lives in `contents/en` (English) and `contents/vi` (Vietnamese).
- `math_formula_rules.md` — LaTeX conventions; always use `$$` delimiters.

## Contribution Conventions

From `contribution/_posts/` (originally from a convex-optimization template; adapt for this repo):

- Branch naming: `feature/chapterXX-description` or `bugfix/chapterXX-description`
- Post front matter must include `layout`, `title`, `chapter`, `order`, `owner`, `lang`, `categories`
- Do not edit Jekyll config/plugins without discussion (PR merge may be blocked)

Update stale references in `contribution/_posts/` and `home/_posts/` that still point to `convex-optimization-for-all` when working in those areas.

## Legacy / Stale Content to Be Aware Of

Agents should treat the following as known technical debt, not current truth:

- `home/_posts/21-01-20-contents.md` — still describes a convex optimization course
- `contribution/_posts/21-01-27-initial_settings.md` — still references `convex-optimization-for-all` GitHub URLs
- `DEPLOYMENT.md` — local URL example references `optimization-for-data-science-iuh-2025`
- `reference/index.html` — owner still listed as "Kyeongmin Woo"
- Chapter `index.html` titles vs. `_posts/` content mismatches (see table above)
- `img/chapter_img/` — contains optimization-course images unrelated to cloud topics
- `contents/vi/chapter01/` is populated (six lessons + chapter landing); Chapters 02–14 and `contents/en/chapter00/` are not yet present

When creating or editing content, use the cloud computing chapter topics from `_posts/` as the source of truth.

## Related Docs

- `AUTHORS.md` — Instructor bio
- `DEPLOYMENT.md` — GitHub Pages setup
- `LICENSE.md` — License terms
- `README.md` — Repository overview and setup guide