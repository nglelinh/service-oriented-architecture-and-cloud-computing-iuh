---
layout: post
title: 15 Deploy Track — IUH path from network to capstone
chapter: '15'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter15
lesson_type: optional
---

This optional **Deploy Track** is an IUH path through the skills that sit *beside* the big-data spine of the course (Hadoop, Spark, NoSQL). It is inspired by the deployment-first arc of [Stanford CS 40 / infracourse.cloud](https://infracourse.cloud/) (Winter 2024). **None of the IUH lessons, labs, or the capstone copy CS 40 assignment text, rubrics, or wording.** We reuse a *topic sequence*—network, containers, orchestration, IaC, identity, observability, CI/CD, cost—and write original campus material.

Use this hub when you want to **ship a small multilayer service**, not only explain MapReduce.

## What you will do here

1. Read four optional theory lessons (networking, observability depth, CI/CD, FinOps).
2. Complete **two or three local-first lab checklists** (Docker; OpenTofu/Terraform *concepts*; free-tier hygiene).
3. In a **group of two**, run the [capstone brief]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %}) (also [`PROJECT.md`](https://github.com/nglelinh/service-oriented-architecture-and-cloud-computing-iuh/blob/main/PROJECT.md) in the repository).

Cloud spend is **not** required. A complete track can stay on one laptop. If you touch a public cloud, Lab C and the FinOps lesson come *before* the first paid API call.

## Suggested reading order

| Step | Material | Home chapter |
| --- | --- | --- |
| 1 | Required Chapter 01 (NIST, models) | 01 |
| 2 | [01-07 Networking crash course]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_07_Networking_Crash_Course %}) | 01 |
| 3 | Required Chapters 08–09 (containers, Kubernetes) | 08–09 |
| 4 | [Lab A — local Docker multilayer]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_01_Lab_Docker_Local %}) | 15 |
| 5 | Required Chapter 13–14 (security, IaC) | 13–14 |
| 6 | [13-03 Observability depth]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_03_Observability_Depth %}) | 13 |
| 7 | [13-04 CI/CD for cloud apps]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %}) | 13 |
| 8 | [Lab B — OpenTofu/Terraform concepts]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_02_Lab_OpenTofu_Concepts %}) | 15 |
| 9 | [12-03 FinOps / billing literacy]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %}) | 12 |
| 10 | [Lab C — free-tier notes]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_03_Lab_Free_Tier_Notes %}) | 15 |
| 11 | [Capstone — Course Board]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %}) | 15 |

P1 enrichments live *inside* existing required notes: Chapter 08 (manual → managed), Chapter 09 (rolling and canary), Chapter 12 (scale + cost checklist), Chapter 14 (CDK / Pulumi concepts).

## CS 40 topic map → IUH

| CS 40-style block (infracourse.cloud) | Where IUH puts it |
| --- | --- |
| Foundations / building blocks | Chapter 01 required |
| Networking, DNS, TLS | [01-07]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_07_Networking_Crash_Course %}) + Lab A/C |
| Storage / databases | Chapters 07 and 10 |
| Containers / orchestration | Chapters 08–09 + Lab A |
| Infrastructure as Code | Chapter 14 + Lab B + CDK/Pulumi compare in 14-01 |
| IAM / security | Chapter 13 required + 13-02 |
| Observability | 02-02, 13-02, **[13-03 depth]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_03_Observability_Depth %})** |
| Serverless / ML serving | 08-04, 08-05, 01-06, 12-02 |
| CI/CD | 13-01 survey + **[13-04]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %})** |
| Cost / ethics | 01-06, **[12-03]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %})**, Lab C |
| Final deploy project | **[Capstone]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_04_Capstone %})** (pairs) |

CS 40 is a *deployment* course. IUH remains a *platform + data* course with this track as an optional spine. You can pass the conceptual core without Chapter 15; you cannot claim “we deployed a service” on theory slides alone.

## IUH safety rules (all labs)

- Prefer **localhost**. Cloud is optional and instructor-gated.
- **Budget alert before apply.** Destroy is part of the demo.
- No secrets in Git. No `0.0.0.0/0` on SSH or database ports.
- Do not copy assignment text from other universities into your report. Cite ideas, write your own design.

## Vietnamese companions

New Deploy Track lessons have Vietnamese mirrors (`lang: vi`, same `chapter` + `order`). Required theory for Chapters 02–14 is still English-first; Chapter 01 required theory is bilingual.

## References

1. [infracourse.cloud](https://infracourse.cloud/) — CS 40: Cloud Application Deployment (Winter 2024), Stanford.
2. This site’s [README topic map](https://github.com/nglelinh/service-oriented-architecture-and-cloud-computing-iuh#inspiration-stanford-cs-40--infracoursecloud).
3. Course chapters 01, 08–09, and 12–14 as linked above.
