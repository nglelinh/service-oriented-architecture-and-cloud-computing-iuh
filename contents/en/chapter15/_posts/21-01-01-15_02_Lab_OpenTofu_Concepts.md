---
layout: post
title: 15-02 Lab B — OpenTofu / Terraform concepts without a surprise bill
chapter: '15'
order: 3
owner: Nguyen Le Linh
lang: en
categories:
- chapter15
lesson_type: optional
---

**Format:** checklist lab.  
**Timebox:** about 2 hours.  
**Cloud apply:** off by default. The learning outcome is **declare → plan → read the graph**, not “provision a VPC on a personal credit card.”

Original IUH material. Not a CS 40 handout.

## Purpose

Write a small **declarative** stack that *would* describe the Course Board network from [01-07]({{ site.baseurl }}{% multilang_post_url contents/chapter01/21-01-01-01_07_Networking_Crash_Course %}), then prove you can reason about **state** and **plan**. Use **OpenTofu** (`tofu`) *or* Terraform (`terraform`)—the workflow is the same for this lab. Chapter 14-02 explains why OpenTofu exists.

## Outcomes

- HCL that names a VPC/VNet cartoon, two subnets, and three security-group-like resources *or* an equivalent **Docker provider** stack if you stay 100% local.
- A saved `plan` file (or plan text) you can explain line by line.
- A paragraph on what **state** would contain after apply—and why two people must not apply from two laptops to one remote state without a lock.

## Track 1 — Local only (recommended)

Use the **Docker** provider (Terraform/OpenTofu) *or* skip providers and write HCL as a *design document* that `tofu validate` cannot run. Preferred: Docker provider so `init` + `plan` are real.

### Checklist

- [ ] Install OpenTofu **or** Terraform; record the version in `NOTES.md` (`tofu version` / `terraform version`).
- [ ] `tofu init` (or `terraform init`) succeeds in an empty `lab-b/` directory.
- [ ] You declare at least: one Docker network, one named volume, one container for a tiny API image you already built in Lab A *or* `nginx:1.27-alpine`.
- [ ] Labels include `project=iuh-courseboard` and `pair=...`.
- [ ] `tofu plan -out=lab.tfplan` (or Terraform equivalent) is saved. You can say which resources would be created, changed, or destroyed.
- [ ] You read the plan: no unexpected `destroy`.
- [ ] **If** you apply locally (`tofu apply lab.tfplan`), you also `tofu destroy` before you close the laptop, and you paste the destroy summary.
- [ ] `NOTES.md` answers: what is state, where is it sitting (`local` vs object storage), and what goes wrong if both pair members apply at once.

Do **not** configure AWS/Azure/GCP providers on Track 1.

## Track 2 — Cloud plan only (instructor opt-in)

Allowed only if the instructor has issued an account **and** you finished [Lab C]({{ site.baseurl }}{% multilang_post_url contents/chapter15/21-01-01-15_03_Lab_Free_Tier_Notes %}) budgets.

### Checklist (additive)

- [ ] Provider pinned to a **version constraint**.
- [ ] Region/location is the one the instructor named (not a random default).
- [ ] Resources are the *minimum* to express: one network, one public subnet, one private subnet, three security groups (edge / api / db) as in 01-07. **No GPU, no NAT if you can avoid it, no Kubernetes cluster.**
- [ ] `plan` is reviewed by both pair members.
- [ ] Default is **stop after plan**. `apply` only with a written instructor yes.
- [ ] After any apply: tags, budget still green, then **destroy** the same day.

## Concept questions (both tracks)

Write answers in `NOTES.md`—these *are* the lab if the tooling misbehaves:

1. Declarative vs the ClickOps path you would have taken in a console.
2. Why `plan` is not a security boundary by itself (Chapter 14-02 policy as code).
3. One sentence on CDK or Pulumi from the [14-01 compare section]({{ site.baseurl }}{% multilang_post_url contents/chapter14/21-01-01-14_01_Infrastructure_as_Code %}): same desired state, different authoring surface.
4. What you would **never** put in state (passwords, unencrypted student data).

## What not to do

- Do not commit `*.tfstate`, `*.tfplan` with secrets, or `.terraform/` provider binaries if your instructor asked for a small patch only—follow the repo’s `.gitignore`.
- Do not copy a public “entire AWS VPC module” you cannot explain.
- Do not create committed-use reservations (FinOps lesson).

## Done when

Both pair members can walk a third student through the plan output without reading it for the first time. Destroy evidence exists if anything was applied.
