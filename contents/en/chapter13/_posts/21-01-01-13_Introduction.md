---
layout: post
title: 13 Introduction to Deployment and Security
chapter: '13'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter13
---

This chapter focuses on the operational aspects of cloud computing: how to securely deploy, manage, and monitor applications in a production environment.

## Learning Objectives

- Understand cloud security fundamentals: Shared Responsibility Model, IAM, Encryption
- Explore deployment strategies: Blue/Green, Canary, Rolling Updates
- Configure CI/CD pipelines for automated delivery
- Implement monitoring and logging for observability

## Operations & Security

Building the application is only half the battle. Running it securely and reliably requires a deep understanding of network security groups, identity management, and automated deployment pipelines to reduce human error.

## Optional application lesson

Required theory in this chapter is unchanged. For SLSA/SBOM supply-chain controls and OpenTelemetry on canaries (2022–2026), see [13-02 Supply-Chain Security, SLSA, and Production Observability]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_02_Supply_Chain_Security_and_Observability %}). For a depth pass on logs/metrics/traces and the OTel pipeline, see [13-03]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_03_Observability_Depth %}). For a working IUH CI/CD model (build once, OIDC, rollback), see [13-04]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %}). Both feed the [Deploy Track]({{ site.baseurl }}/contents/en/chapter15/).

## Video tham khảo

- **xem trước:** [Identity & Access Management (IAM)](https://www.youtube.com/watch?v=aNj36g7fSsU) — IBM Technology, English. Watch first for identity and access management, the security half of this operations introduction.
- [Deployment Strategies Explained: Blue-Green vs. Canary vs. Rolling](https://www.youtube.com/watch?v=H5z70EBtEow) — CodeLucky, English. Blue-green, canary, and rolling deployment, the release strategies the chapter then automates.
- [CI/CD Explained in 5 Minutes](https://www.youtube.com/watch?v=ddDJxFnv-qs) — TechWorld with Nana, English. CI/CD in five minutes, the pipeline idea the introduction says reduces human error.
