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

The #1 reason students hesitate to practice Infrastructure as Code is completely rational: **fear of accidentally racking up a $200 cloud bill while experimenting!** The beauty of **OpenTofu / Terraform** is that its core engine—dependency graphs, state files, variable validation, `plan` diffs, and idempotent updates—works identically on **zero-cost local providers** (`local_file`, `random_pet`, or LocalStack/Docker providers) with **$0.00 billing risk**!

Lab B lets you master the full `init` $$	o$$ `plan` $$	o$$ `apply` $$	o$$ drift-detection $$	o$$ `destroy` lifecycle safely on your own machine before ever touching a paid cloud account.

---

## Background Knowledge Map: Learning the IaC Lifecycle With Zero Cloud Spend

1. **Provider Abstraction:** OpenTofu’s core graph engine doesn’t care whether a provider manages AWS EC2 instances or local files/Docker containers—the exact same HCL syntax, state diffing, and DAG scheduling apply.
2. **The Four-Command Loop (`init` $$	o$$ `plan` $$	o$$ `apply` $$	o$$ `destroy`):** `init` downloads providers; `plan` computes the dry-run diff; `apply` reconciles reality to code; `destroy` cleanly removes every tracked resource.
3. **Observing Drift Firsthand:** Manually deleting or editing a resource managed by OpenTofu and re-running `tofu plan` shows you *live* how IaC detects external tampering and plans to heal it.
4. **Input Variables & Outputs:** Parameterizing configs with `variables.tf` and surfacing computed values via `outputs.tf` mirrors real production modules.

---

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

---

### Worked Concrete Example: Tracing Drift Healing Across 4 Commands ($0.00 Cost)

Imagine a minimal OpenTofu config creating a local config file `app.conf` via `local_file`:

| Step | Command / Action | Desired State (HCL) | Live Reality | `tofu plan` / `apply` Action |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `tofu apply` | `app.conf` = `"port=8080"` | Does not exist | **`+ create`** `app.conf` |
| 2 | `tofu apply` (again) | `app.conf` = `"port=8080"` | `"port=8080"` | **`No changes`** (idempotent!) |
| 3 | Manually edit `app.conf` to `"port=9999"` | `"port=8080"` | `"port=9999"` (Drift!) | `plan` shows **`-/+ replace`** to heal drift! |
| 4 | `tofu destroy` | Empty | `"port=8080"` | **`- destroy`** removes `app.conf` cleanly |

### Deep-Dive Explanation: Four Safeguards Before Moving From Local Labs to Real Cloud (Read Slowly)

1. **Never skip `tofu plan`:** Reading the diff habit on local files becomes muscle memory that protects real cloud databases later.
2. **Add `.terraform/` and `*.tfstate*` to `.gitignore` from minute one:** Prevent provider binaries and sensitive state files from polluting Git.
3. **Tag every cloud resource with `project` and `owner`:** When you do graduate to AWS/GCP, cost-allocation tags let you spot any stray resource in seconds.
4. **Always run `tofu destroy` at the end of every lab session:** Make cleanup part of your "Done" definition!

**Quick check.** If you run `tofu apply` twice in a row without editing any `.tf` files or touching any live resources, what should OpenTofu report on the second run—and what is this property called?

---

## Video tham khảo

- **xem trước:** [Introduction to HashiCorp Terraform with Armon Dadgar](https://www.youtube.com/watch?v=h970ZBgKINg) — HashiCorp, an IBM Company, English. Watch first for plan, then apply. This lab stops at understanding the plan so a real account is not changed by surprise.
- [What is OpenTofu Project?](https://www.youtube.com/watch?v=J2JKxY2elSo) — Oleksiy Pototskyy, English. What OpenTofu is, the tool named in the lab title beside Terraform.
- [Terraform Basics](https://www.youtube.com/watch?v=_45W3Z8XWL4) — HashiCorp, an IBM Company, English. HashiCorp’s basics of the Terraform workflow the lab asks you to rehearse without a bill.
