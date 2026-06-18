---
layout: post
title: "Conventions"
chapter: home
order: 2
owner: "Nguyen Le Linh"
---

## 1. Directory Convention

- Main course content lives under `contents/en/chapterXX/` (and `contents/vi/chapterXX/` for Vietnamese). Images are stored under `img/chapter_img/`.
- A typical chapter directory looks like this:

```
contents/en/chapter01
├── _posts
│   ├── 21-01-01-01_Introduction.md
│   ├── 21-01-01-01_01_Cloud_Computing_Characteristics.md
│   └── ...
└── index.html
```

- Jekyll treats Markdown or HTML files inside `_posts` as blog posts. To add a new lecture, create a file in the chapter's `_posts` directory.
- Post filenames must follow this naming convention:
    - `yy-mm-dd-post_name.md`
- Files outside `contents/` and `img/` are mostly site configuration. For stability, please open an issue instead of editing Jekyll config or plugins directly (changes there may block PR merges).

## 2. Posting Convention

### 2.1. Header Fields

- Every post must include front matter like the example below.

```
---
layout: post
title: "Cloud Computing Fundamentals"
chapter: "01"
order: 1
owner: "Nguyen Le Linh"
lang: en
categories:
- chapter01
lesson_type: required
---
```

- **layout** must be `post`.
- **title** can be any string that describes the lecture.
- **chapter** is the two-digit chapter number as a string (e.g., `"01"`).
- **order** controls sorting within the chapter and prev/next navigation.
- **owner** identifies the maintainer of the post.
- **lang** is `en` or `vi` for language switching.
- **categories** must match the chapter directory name (e.g., `chapter01`).
- **lesson_type** is optional: `required` or `optional`.

### 2.2. LaTeX

- Write math using LaTeX syntax.
- Use double dollar signs (`$$`) to delimit formulas.

```
$$\theta x_1 + (1-\theta)x_2 \in C$$
```

The formula above renders as:

$$\theta x_1 + (1-\theta)x_2 \in C$$

### 2.3. Image Convention

- When inserting images in a post, use the following HTML pattern:

```
<figure class="image" style="align: center;">
<p align="center">
  <img src="{{ site.imgurl }}/chapter_img/chapter01/example.png" alt="description of image" width="80%">
  <figcaption style="text-align: center;">Caption text</figcaption>
</p>
</figure>
```

- The figure class must be `image`.
- Replace placeholder values with the correct image path, alt text, and caption.

### 2.4. Hyperlink Convention

- For links to other posts on this site, use the `multilang_post_url` Liquid tag. For example, a link to the first lecture in chapter 01:

```
[Cloud Computing Characteristics]({% multilang_post_url contents/chapter01/21-01-01-01_01_Cloud_Computing_Characteristics %})
```

- For external URLs, use standard Markdown links:

```
[NIST Cloud Computing Definition](<https://csrc.nist.gov/publications/detail/sp/800-145/final>)
```

## 3. GitHub Convention

If you have questions or find something that needs correction, you can:

- Leave a comment on the relevant page
- Open an issue on the repository

To add new content or edit existing material, create a branch, make your changes, and open a `Pull Request`. Anyone may contribute.

### 3.1. Repository Policy

Pull requests merged into `main` require approval from at least one reviewer.

### 3.2. Branch Naming Convention

Name branches using this pattern:

```
[feature|bugfix]/[chapter**|settings]-change-description
```

Use two prefixes:

- **feature**
  - Migration work
  - Changes to text, formulas, or images
  - New content
- **bugfix**
  - Typo fixes
  - Broken LaTeX rendering fixes

Examples:

- `feature/chapter01-migration`: migrate chapter 01 content
- `feature/chapter01-fix-formula`: update a formula in chapter 01
- `feature/settings-update-branch-convention`: update conventions
- `bugfix/chapter01-fix-typo`: fix a typo in chapter 01