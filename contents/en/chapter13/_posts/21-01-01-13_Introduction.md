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

Writing clean microservice code on `localhost` is only half of cloud engineering—the moment your service faces the public internet, **deployment automation, security boundaries, and production observability** decide whether it survives real traffic. Chapter 13 bridges development and operations: securing credentials and network perimeters, instrumenting OpenTelemetry logs/metrics/traces, and automating safe CI/CD pipelines.

---

## Background Knowledge Map: Three Pillars of Cloud Operations & Security

1. **Defense in Depth & Least Privilege:** Assume any single layer can fail—combine TLS in transit, short-lived IAM roles, network security groups, and secret managers so no credential lives in Git.
2. **Observability Over Blind Monitoring:** Instrument RED metrics, structured logs, and distributed traces so unknown failure modes can be diagnosed in minutes.
3. **Automated Progressive Delivery:** Replace fragile manual SSH deployments with repeatable CI/CD pipelines and automated canary rollbacks.

---

## Learning Objectives

- Understand cloud security fundamentals: Shared Responsibility Model, IAM, Encryption
- Explore deployment strategies: Blue/Green, Canary, Rolling Updates
- Configure CI/CD pipelines for automated delivery
- Implement monitoring and logging for observability

## Operations & Security

Building the application is only half the battle. Running it securely and reliably requires a deep understanding of network security groups, identity management, and automated deployment pipelines to reduce human error.

## Optional application lesson

Required theory in this chapter is unchanged. For SLSA/SBOM supply-chain controls and OpenTelemetry on canaries (2022–2026), see [13-02 Supply-Chain Security, SLSA, and Production Observability]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_02_Supply_Chain_Security_and_Observability %}). For a depth pass on logs/metrics/traces and the OTel pipeline, see [13-03]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_03_Observability_Depth %}). For a working IUH CI/CD model (build once, OIDC, rollback), see [13-04]({{ site.baseurl }}{% multilang_post_url contents/chapter13/21-01-01-13_04_CICD_Cloud_Apps %}). Both feed the [Deploy Track]({{ site.baseurl }}/contents/en/chapter15/).

---

### Worked Concrete Example: Mapping One Production Incident Across Security, CI/CD, and Observability

Suppose an unauthenticated endpoint leak is discovered in `v1.8`:
- **Observability** detects an anomalous 10× spike in `401/200` traffic from one IP range within 60 seconds.
- **Security (Least Privilege)** limits damage because the service’s IAM role has read-only access to one S3 prefix—not the entire AWS account.
- **CI/CD** ships the patched `v1.8.1` image through automated tests and rolls it out across all replicas in under 4 minutes!

### Deep-Dive Explanation: Three Questions to Carry Through Chapter 13 (Read Slowly)

1. **If a secret leaks today, how fast can you rotate it with zero downtime?**
2. **If p99 latency triples at 3 AM, can you pinpoint the exact failing microservice in under 2 minutes?**
3. **If a release breaks checkout, can you roll back in one click without SSH-ing into a server?**

**Quick check.** Why is storing an AWS secret key in a `.env` file committed to a private GitHub repository still a severe security violation?

---

## Video tham khảo

- **xem trước:** [Identity & Access Management (IAM)](https://www.youtube.com/watch?v=aNj36g7fSsU) — IBM Technology, English. Watch first for identity and access management, the security half of this operations introduction.
- [Deployment Strategies Explained: Blue-Green vs. Canary vs. Rolling](https://www.youtube.com/watch?v=H5z70EBtEow) — CodeLucky, English. Blue-green, canary, and rolling deployment, the release strategies the chapter then automates.
- [CI/CD Explained in 5 Minutes](https://www.youtube.com/watch?v=ddDJxFnv-qs) — TechWorld with Nana, English. CI/CD in five minutes, the pipeline idea the introduction says reduces human error.
