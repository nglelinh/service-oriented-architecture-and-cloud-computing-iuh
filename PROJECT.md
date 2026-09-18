# IUH capstone — Course Board (groups of 2)

This is the student-facing brief for the optional **Deploy Track** capstone in *Service-Oriented Architecture and Cloud Computing* (Industrial University of Ho Chi Minh City).

- **On the course site:** Chapter 15 [Deploy Track hub](https://nglelinh.github.io/service-oriented-architecture-and-cloud-computing-iuh/contents/en/chapter15/) (capstone post `15-04` in the chapter list).
- **Vietnamese companion:** Chapter 15, same `order` (`lang: vi`).
- **Inspiration:** the *idea* of a final deploy project from [Stanford CS 40 / infracourse.cloud](https://infracourse.cloud/). **Do not copy CS 40 assignment text, rubrics, or wording.** This brief is original IUH material.

---

## Purpose

In a pair, design and operate a small **multilayer** service using Deploy Track vocabulary: network tiers, Infrastructure as Code, an elasticity story, a security baseline, DNS/TLS, short operational evidence, and a cost paragraph. Hadoop/Spark are out of scope unless you deliberately add them as an extra backend.

Cloud spend is **optional**. Local Docker Compose plus an OpenTofu/Terraform **plan** is a complete submission.

## Product — IUH Course Board

A **course help board** for one fictional faculty:

- List and create posts (course code, title, body, timestamp).
- Optional: mark resolved, or a small search field.
- **Do not** use real student names, MSSV, grades, or LMS exports. Synthetic fixtures only.

Any familiar language/framework is fine. The assessed craft is the **operations story**.

## Required architecture

Three runtime roles, drawn on **one diagram**:

1. **Edge** — HTTP(S) entry.
2. **Application** — API (stateless if you can; document if not).
3. **Data** — PostgreSQL, MySQL, or Redis, with a persistence story.

Arrows must match Compose networks or security-group intent (see lesson 01-07 and Lab A).

## Required themes (each gets a report subsection)

1. **IaC** — Checked-in HCL (OpenTofu or Terraform). Default is `plan` only. Cloud `apply` only with instructor permission and Lab C budgets. If you used the console, write the ClickOps debt.
2. **Elasticity** — One honest demo: Compose scale, two Kubernetes replicas on a local cluster, or a managed scale-to-zero service with Lab C hygiene. Describe load = 0 and “we killed one replica.”
3. **Security baseline** — Datastore not on the campus LAN; secrets not in Git; lockfile; two-row threat note (leaked `.env`, public DB port). Cloud IAM only if you used cloud (least privilege).
4. **DNS and TLS** — Local lane (`mkcert` or explicit HTTP-only limitation) **or** a public HTTPS hostname with issuer + DNS TTL.
5. **Observe and deliver** — `/healthz`, one forced-failure log or RED screenshot, CI **or** `make test` on a clean machine, two-sentence rollback (previous digest or tag).
6. **Cost** — Even at $0: unit of work, idle cost, who runs destroy. Cloud users attach a dated billing screenshot. See lesson 12-03.

## Deliverables

One report (Markdown or PDF, about 4–8 pages) and a Git repository both members can open:

1. Pair names, student IDs, date.
2. Architecture diagram.
3. The six subsections.
4. How to run locally.
5. What one more week would change.
6. Integrity line: you did not copy another university’s assignment text.

Optional: ≤ 5 minute demo video. Live demo if scheduled.

## Pairing

Split **vertical slices**, not “coder vs writer”:

- Partner A: edge, DNS/TLS lane, public-path health.
- Partner B: API, datastore, Compose/IaC, destroy.

Either partner may be asked any section.

## Safety

- Budget alert before any cloud `apply`.
- No `0.0.0.0/0` on SSH or database ports.
- No secrets in Git.
- Destroy is part of Definition of Done.

## Course pointers

| Topic | Site location |
| --- | --- |
| Deploy Track hub | `contents/en/chapter15/` |
| Networking | `01-07` |
| Observability | `13-03` |
| CI/CD | `13-04` |
| FinOps | `12-03` |
| Labs A–C | `15-01` … `15-03` |

---

## Đồ án (tóm tắt tiếng Việt)

Nhóm **2 sinh viên**. Xây **bảng tin hỗ trợ học phần** (dữ liệu giả). Ba tầng: edge, API, dữ liệu. Báo cáo ngắn (4–8 trang) + repo: IaC (`plan` là đủ), tính đàn hồi, nền bảo mật, DNS/TLS, quan sát/CI, chi phí (kể cả 0 đồng). Không sao chép đề trường khác. Ưu tiên Docker local; cloud chỉ khi có ngân sách và sự đồng ý của giảng viên. Bản đầy đủ: bài **15-04** trên site (`lang: vi`).
