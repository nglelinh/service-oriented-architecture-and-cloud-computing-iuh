---
layout: post
title: 12 Introduction to Cloud Providers
chapter: '12'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter12
---

This chapter compares the major public cloud providers—AWS, Azure, and Google Cloud Platform (GCP)—and explores their core service offerings.

Choosing a cloud provider is rarely just a technical checklist—it is a long-term architectural and financial commitment that shapes how fast your team ships, where your users' data legally resides, and how your operating bill scales from ten users to ten million. Chapter 12 demystifies the cloud landscape across **AWS, Azure, and GCP**, exploring multi-cloud realities, managed AI services, and the FinOps literacy every modern engineer needs to read a cloud invoice with confidence.

## Background Knowledge Map: Cloud Economics and Strategic Architecture

### 1. CapEx vs. OpEx and Elasticity Economics
On-premise infrastructure requires large upfront **Capital Expenditure (CapEx)** sized for your *peak* day of the year—meaning servers sit mostly idle the other 360 days. Cloud converts compute into variable **Operational Expenditure (OpEx)** that tracks actual demand.

### 2. Service Breadth vs. Portability
Managed cloud services accelerate time-to-market dramatically, while open standards (containers, SQL/Parquet, OpenTofu) protect your long-term negotiating leverage and exit options.

### 3. Cost Awareness as an Architectural Constraint
Every architectural diagram has an implicit dollar sign attached to each arrow (compute, storage, cross-AZ bandwidth, managed NAT, API calls). Reading pricing calculators early prevents painful re-architectures later.

### Deep-Dive Explanation: Hand-Calculating CapEx vs. OpEx Break-Even for Steady vs. Bursty Workloads (Read Slowly)

Compare buying an on-premise server (**12,000 USD upfront** + **200 USD/month** for power, cooling, and rack space) against renting an equivalent cloud VM at **500 USD/month** On-Demand:

1. **Steady 24/7 Workload:**
   - Cloud costs $$500 - 200 = 300\text{ USD/month}$$ more in recurring spend than the on-prem server's power/cooling.
   - Break-even point:
     $$\frac{12{,}000\text{ USD}}{300\text{ USD/month}} = 40\text{ months} \approx 3.33\text{ years}.$$
   - For a steady 24/7 workload kept for 5 years, on-prem (or discounted Reserved cloud instances) wins on raw hardware cost.

2. **Bursty Workload (e.g., university grading or nightly analytics running 4 hours/day):**
   - Cloud VM runs only $$4 / 24 = 1/6$$ of the day:
     $$\text{Cloud monthly bill} = 500 \times \frac{4}{24} \approx 83.33\text{ USD/month}.$$
   - Even **ignoring** the 12,000 USD upfront purchase, the cloud VM costs less per month than the on-premise server's 200 USD electricity bill alone—**on-premise never breaks even**!

## Learning Objectives

- Compare the "Big 3" cloud providers in terms of global infrastructure and market share
- Map core services across providers (e.g., EC2 = VM, S3 = Blob Storage)
- Understand the pricing models and "Free Tier" offerings
- Analyze the strengths and use cases for each provider

## The Cloud Landscape

While the underlying concepts (virtualization, storage, networking) are the same, each provider wraps them in their own terminology, APIs, and management consoles. A multi-cloud architect must understand these nuances to select the right platform for the job.

## Optional application lesson

Required theory in this chapter is unchanged. For multi-cloud constraints, FinOps, and managed AI APIs (2022–2026), see [12-02 Multi-Cloud Architectures and Managed AI Services]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_02_Multicloud_and_AI_Services %}). The providers lesson now ends with a **scale + cost checklist**. For billing literacy (meters, budgets, free-tier shapes) see [12-03]({{ site.baseurl }}{% multilang_post_url contents/chapter12/21-01-01-12_03_FinOps_Billing_Literacy %}) and the [Deploy Track]({{ site.baseurl }}/contents/en/chapter15/).

## Video tham khảo

- **xem trước:** [What is AWS? \| Amazon Web Services](https://www.youtube.com/watch?v=a9__D53WsUs) — Amazon Web Services, English. Watch first: what AWS is, the first of the three providers this introduction compares.
- [How does Microsoft Azure work?](https://www.youtube.com/watch?v=KXkBZCe699A) — Microsoft Azure, English. How Azure presents the same cloud idea with Microsoft’s vocabulary.
- [What is Google Cloud?](https://www.youtube.com/watch?v=kzKFuHk8ovk) — Google Cloud Tech, English. What Google Cloud is, so the three-way comparison in the chapter has a third primary source.
