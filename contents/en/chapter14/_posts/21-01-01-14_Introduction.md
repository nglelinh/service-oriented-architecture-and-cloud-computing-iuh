---
layout: post
title: 14 Introduction to Infrastructure as Code
chapter: '14'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter14
---

This chapter introduces Infrastructure as Code (IaC), the practice of managing and provisioning computing infrastructure through machine-readable definition files rather than physical hardware configuration or interactive configuration tools.

If you can rebuild your entire cloud architecture—networks, databases, load balancers, and IAM roles—from scratch in 5 minutes with a single command, you truly own your infrastructure. If losing one server means days of frantic console clicking, your infrastructure owns you. Chapter 14 introduces **Infrastructure as Code (IaC)** through Terraform, OpenTofu, and Policy as Code—turning cloud provisioning into reviewable, testable software engineering.

---

## Key ideas: Three Shifts From Manual Cloud Ops to Automated IaC

1. **Declarative Topology:** Describe *what* cloud resources should exist in Git rather than *how* to click them together manually.
2. **Drift Detection & Idempotency:** Continuously compare live cloud state against code so environments (dev, staging, prod) stay consistent.
3. **Automated Governance (Policy as Code):** Enforce security, tagging, and cost guardrails automatically on every pull request.

---

## Learning Objectives

- Define Infrastructure as Code and its benefits (Version Control, Reproducibility)
- Differentiate between Imperative and Declarative approaches
- Learn the basics of Terraform (Providers, Resources, State)
- Understand Ansible for configuration management

## Automating the Cloud

Manual configuration ("ClickOps") is error-prone and unscalable. IaC allows developers to treat infrastructure as software—enabling code review, testing, and automated deployment of the entire data center stack.

## Optional application lesson

Required theory in this chapter is unchanged. For OpenTofu vs Terraform BSL, OPA policies, and platform modules (2023–2026), see [14-02 OpenTofu, Policy as Code, and Platform Engineering]({{ site.baseurl }}{% multilang_post_url contents/chapter14/21-01-01-14_02_OpenTofu_and_Policy_as_Code %}). The Terraform lesson now includes a short **CDK / Pulumi concept compare** (same desired state, different authoring surface) for the [Deploy Track]({{ site.baseurl }}/contents/en/chapter15/).

---

### Worked Concrete Example: Manual ClickOps vs. Declarative IaC Across 3 Environments

| Task | Manual Console (ClickOps) | Declarative IaC (Terraform / OpenTofu) |
| :--- | :--- | :--- |
| Provisioning Dev + Staging + Prod | 3× manual setup; subtle firewall drift | **1 reusable module**, 3 variable files |
| Auditing who opened port 5432 | Hunt through CloudTrail logs | `git blame security_groups.tf` (2 seconds!) |
| Tearing down a lab to stop billing | Easy to forget 1 NAT Gateway ($32/mo!) | **`tofu destroy`** removes 100% of tracked resources! |

### Explanation: Three Habits to Build in Chapter 14

1. **Always run `plan` before `apply`:** Read every single line of the diff like a code review.
2. **Keep environments parameterized, never copy-pasted:** Share modules and pass environment-specific sizes (`dev` vs. `prod`) as variables.
3. **Remember `destroy` is your best friend in learning labs:** Clean teardown prevents surprise cloud bills!

**Quick check.** Why is `tofu destroy` much safer for cleaning up a student cloud lab than manually deleting resources one by one in the AWS Console?

---

## Video tham khảo

- **xem trước:** [Introduction to HashiCorp Terraform with Armon Dadgar](https://www.youtube.com/watch?v=h970ZBgKINg) — HashiCorp, an IBM Company, English. Watch first: Armon Dadgar on what Terraform is for, the automation this introduction promises.
- [Introduction to Terraform](https://www.youtube.com/watch?v=ZFLWA1kQ3ls) — HashiCorp, an IBM Company, English. A later HashiCorp introduction that shows plan and apply as the lifecycle.
