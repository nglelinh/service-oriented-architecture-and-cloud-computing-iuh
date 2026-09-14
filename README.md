# Service-Oriented Architecture and Cloud Computing

Open course materials for **Service-Oriented Architecture and Cloud Computing** at the Industrial University of Ho Chi Minh City (IUH). Lecture notes are published as a multilingual Jekyll site with full-text search, chapter navigation, and English/Vietnamese support.

**Live site:** https://nglelinh.github.io/service-oriented-architecture-and-cloud-computing-iuh/

**Instructor:** Nguyen Le Linh ([nglelinh@gmail.com](mailto:nglelinh@gmail.com))

## About the Course

This course introduces cloud computing technologies and distributed systems used to build scalable data-intensive applications. Students learn foundational concepts (IaaS/PaaS/SaaS, virtualization, containers), big-data processing frameworks (Hadoop, Spark), modern deployment practices (Docker, Kubernetes), and cloud platform operations (AWS, Azure, GCP), with an emphasis on practical skills for software engineering and data science workflows.

### Course Objectives

- Understand core cloud computing concepts, service models, and deployment models.
- Work with distributed computing frameworks such as Hadoop MapReduce and Apache Spark.
- Apply containerization and orchestration tools (Docker, Kubernetes) to deploy applications.
- Explore NoSQL databases, data pipelines, and big-data platform architectures.
- Gain exposure to major cloud providers, security practices, and Infrastructure as Code (Terraform).

## Course Outline

The site contains 14 chapters. Lecture topics below reflect the current `_posts/` content.

| Ch | Topic |
|----|-------|
| 01 | Cloud computing fundamentals — NIST characteristics, service/deployment models, benefits, data centers |
| 02 | Distributed systems fundamentals |
| 03 | Computing models and Hadoop YARN |
| 04 | Hadoop MapReduce |
| 05 | Apache Spark fundamentals — architecture, RDDs, transformations |
| 06 | Spark Streaming, Spark SQL, MLlib |
| 07 | NoSQL and distributed databases |
| 08 | Virtualization, containerization, Docker, serverless |
| 09 | Kubernetes fundamentals — pods, services, deployments |
| 10 | Big data platforms and processing |
| 11 | Data sourcing, cleaning, and preparation |
| 12 | Cloud providers (AWS, Azure, GCP) |
| 13 | Deployment, security, and compliance |
| 14 | Infrastructure as Code with Terraform |

Vietnamese translations (`contents/vi/`) have started: Chapter 01 is available in Vietnamese. Chapters 02–14 remain English-only for now.

## Local Development

### Prerequisites

- Ruby 3.x
- Bundler

### Setup and serve

```bash
git clone https://github.com/nglelinh/service-oriented-architecture-and-cloud-computing-iuh.git
cd service-oriented-architecture-and-cloud-computing-iuh
bundle install
bundle exec jekyll serve
```

Open http://127.0.0.1:4000/service-oriented-architecture-and-cloud-computing-iuh/

After changing `_config.yml`, restart the Jekyll server.

### Production build

```bash
bundle exec jekyll build
```

Output is written to `_site/`.

## Project Structure

```
.
├── _config.yml              # Site configuration and translations
├── _includes/               # Head, sidebar, and shared partials
├── _layouts/                # Page layouts
├── _plugins/                # Custom Jekyll plugins (multilang, search, redirects)
├── contents/
│   ├── en/chapterXX/        # English lectures and chapter index pages
│   └── vi/chapterXX/        # Vietnamese mirror (in progress)
├── home/_posts/             # Home page sections
├── contribution/_posts/     # Contributor documentation
├── public/                  # CSS, JavaScript, logos
├── img/chapter_img/         # Lecture images
└── extracted_slides/        # Source slide decks (reference)
```

## Features

- **Multilingual support** — English and Vietnamese UI strings; language switching matches posts by `chapter` + `order`.
- **Full-text search** — Lunr.js index generated at build time.
- **Chapter navigation** — Sidebar and prev/next links driven by post `order`.
- **Math rendering** — MathJax 3 for LaTeX formulas (`$$...$$`).

## Deployment

The site deploys to GitHub Pages via GitHub Actions on pushes to `main`. Custom Jekyll plugins require Actions-based deployment (not the default GitHub Pages Jekyll build).

See [DEPLOYMENT.md](DEPLOYMENT.md) for workflow and repository settings.

## Contributing

Contributions are welcome. Please read:

- [How to Contribute](https://nglelinh.github.io/service-oriented-architecture-and-cloud-computing-iuh/contribution/how_to_contribute/)
- [Conventions](https://nglelinh.github.io/service-oriented-architecture-and-cloud-computing-iuh/contribution/conventions/)
- [Initial Settings](https://nglelinh.github.io/service-oriented-architecture-and-cloud-computing-iuh/contribution/initial_settings/)

Use branch names like `feature/chapter01-add-lecture` or `bugfix/chapter05-fix-typo`.

## License

Released under the [MIT License](LICENSE.md).

## Acknowledgments

Course site built on the [Lanyon](https://github.com/poole/lanyon) Jekyll theme. Originally adapted from open optimization-course templates; substantially reworked for cloud computing topics at IUH.