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

## Learning Objectives

- Define Infrastructure as Code and its benefits (Version Control, Reproducibility)
- Differentiate between Imperative and Declarative approaches
- Learn the basics of Terraform (Providers, Resources, State)
- Understand Ansible for configuration management

## Automating the Cloud

Manual configuration ("ClickOps") is error-prone and unscalable. IaC allows developers to treat infrastructure as software—enabling code review, testing, and automated deployment of the entire data center stack.

## Optional application lesson

Required theory in this chapter is unchanged. For OpenTofu vs Terraform BSL, OPA policies, and platform modules (2023–2026), see [14-02 OpenTofu, Policy as Code, and Platform Engineering]({{ site.baseurl }}{% multilang_post_url contents/chapter14/21-01-01-14_02_OpenTofu_and_Policy_as_Code %}). The Terraform lesson now includes a short **CDK / Pulumi concept compare** (same desired state, different authoring surface) for the [Deploy Track]({{ site.baseurl }}/contents/en/chapter15/).

## Video tham khảo

- **xem trước:** [Introduction to HashiCorp Terraform with Armon Dadgar](https://www.youtube.com/watch?v=h970ZBgKINg) — HashiCorp, an IBM Company, English. Watch first: Armon Dadgar on what Terraform is for, the automation this introduction promises.
- [Introduction to Terraform](https://www.youtube.com/watch?v=ZFLWA1kQ3ls) — HashiCorp, an IBM Company, English. A later HashiCorp introduction that shows plan and apply as the lifecycle.
