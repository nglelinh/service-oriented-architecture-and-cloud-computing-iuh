---
layout: post
title: "How to Contribute"
chapter: home
order: 3
owner: Nguyen Le Linh
---

---

## 1. Editing content directly

### (1) Open your local repository directory. If you do not have a local clone yet, see [Initial Settings]({{ site.baseurl }}/contribution/initial_settings/).

### (2) Sync with the remote repository.

```bash
$ git checkout main
$ git pull --all
```

### (3) Create a new branch for your changes. Use the naming pattern `[prefix]/[chapter]-[description]` ([Branch Naming Convention]({{ site.baseurl }}/contribution/conventions/)). For example:

```bash
$ git checkout -b bugfix/chapter01-fix-typo
```

### (4) Edit the files. Follow the [Conventions]({{ site.baseurl }}/contribution/conventions/) when creating or updating content.

### (5) Push your branch to the remote. For example:

```bash
$ git push origin bugfix/chapter01-fix-typo
```

### (6) Open a pull request to `main` on [GitHub](https://github.com/nglelinh/service-oriented-architecture-and-cloud-computing-iuh/pulls). See the GitHub docs for details:

- [Creating a pull request](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request)

---

## 2. Requesting content changes

- You can open an [issue](https://github.com/nglelinh/service-oriented-architecture-and-cloud-computing-iuh/issues) on the GitHub repository.

---