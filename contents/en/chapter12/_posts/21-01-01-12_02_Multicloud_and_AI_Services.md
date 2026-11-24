---
layout: post
title: 12-02 Multi-Cloud Architectures and Managed AI Services (2022–2026)
chapter: '12'
order: 3
owner: Nguyen Le Linh
lang: en
categories:
- chapter12
lesson_type: optional
---

The required lesson maps EC2/VM, S3/Blob, and free tiers across AWS, Azure, and GCP. This optional note is how those catalogs are *used* in 2022–2026: **multi-cloud by constraint**, FinOps across invoices, and **managed AI** (Bedrock, Azure OpenAI, Vertex AI) as a new row in the comparison table.

In boardroom slides, "Multi-Cloud" sounds like the ultimate insurance policy: *never get locked into one vendor, and always pick the best service from every cloud!* In production engineering, however, naive multi-cloud often doubles operational complexity, duplicates IAM policies, and triggers eye-watering network egress bills. Between 2022 and 2026, the most pragmatic multi-cloud pattern has become **workload-specialized multi-cloud**—for example, running core transactional microservices on one primary cloud while consuming managed **AI/LLM services** (Bedrock, Vertex AI, Azure OpenAI) or specialized analytics where they offer clear model, latency, or compliance advantages.

## Background Knowledge Map: Multi-Cloud Trade-Offs, Data Gravity, and Managed AI APIs

### 1. Data Gravity and Egress Economics
Code is tiny and portable (a 100 MB Docker image moves anywhere in seconds); **data is heavy and expensive to move**. Cloud providers make data *ingress* free but charge steep **egress fees** (often 0.08–0.12 USD/GB) to pull data out to the Internet or another cloud. Moving compute close to where your primary data lives almost always beats shipping terabytes across clouds.

### 2. Managed AI Services vs. Self-Hosted GPUs
Running open-weights LLMs on dedicated GPU VMs gives full control over weights and data residency, but requires paying for expensive GPUs 24/7 even during quiet hours. **Managed AI APIs** (serverless token-billed endpoints) turn inference into pure pay-per-token OpEx with built-in autoscaling, safety filters, and SLA-backed availability.

### 3. Avoiding Lowest-Common-Denominator Lock-In Traps
Trying to abstract every cloud down to bare VMs throws away the productivity of managed services. Smart teams standardize on **open portable layers** (Kubernetes, OpenTofu/Terraform, OpenTelemetry, Iceberg/Parquet, OpenAI-compatible inference gateways) so workloads remain portable *without* sacrificing managed-cloud speed.

### Deep-Dive Explanation: Hand-Comparing Monthly LLM Token Costs vs. Cross-Cloud Data Egress (Read Slowly)

Suppose a SaaS app hosted on Cloud A calls a managed LLM API on Cloud B for **100,000 customer support tickets/month**, and also syncs **5 TB (5,120 GB)** of training logs every month from Cloud A to Cloud B.

1. **Managed AI Token Cost:**
   Each ticket averages **1,500 input tokens** (prompt + context) and **500 output tokens** (answer):
   - Total input tokens: $$100{,}000 \times 1{,}500 = 150\text{ million tokens}$$.
   - Total output tokens: $$100{,}000 \times 500 = 50\text{ million tokens}$$.
   - At an illustrative mid-tier API price of **0.50 USD / 1M input** and **1.50 USD / 1M output**:
     $$\text{AI bill} = (150 \times 0.50) + (50 \times 1.50) = 75.00 + 75.00 = 150.00\text{ USD/month}.$$

2. **Cross-Cloud Egress Cost for the 5 TB Log Sync:**
   At a typical Internet egress rate of **0.09 USD/GB**:
   $$\text{Egress bill} = 5{,}120\text{ GB} \times 0.09\text{ USD/GB} = 460.80\text{ USD/month!}$$

Notice the punchline: shipping raw logs across clouds costs **over 3× more than the entire AI inference workload itself**! Keeping lightweight JSON API calls cross-cloud is cheap, while bulk data pipelines should stay inside the same cloud region where the data originates.

## Learning objectives

- Separate *portable interfaces* (S3 API, Kubernetes, OTel, Iceberg) from *sticky* services (proprietary AI APIs).
- Compare managed LLM platforms on the same axes you used for compute/storage.
- Design a thin adapter so the rest of the SOA does not import one vendor’s SDK everywhere.

## 1. Multi-cloud in the data, not in the slide

Flexera 2025: hybrid is normal (**70%**), about **2.4** public clouds, spend is the #1 pain (**84%**), FinOps teams **59%**. AWS and Azure remain neck-and-neck; GCP is a strong third, often for data/ML. That is the landscape the required “Big 3” table now sits in.

**Good multi-cloud** (portability where it pays):

- Kubernetes + Gateway API + OTel on any cloud.
- Open table formats on object storage.
- Identity federation (OIDC) instead of long-lived keys.

**Expensive multi-cloud** (duplicating everything):

- Three warehouses, three service meshes, three CI systems “for resilience.”
- Cross-cloud chatty gRPC (egress + RTT).

```python
class CompletionClient:
    """Portable application port — adapters hide vendor SDKs."""

    def complete(self, prompt: str, max_tokens: int) -> str:
        raise NotImplementedError

class BedrockAdapter(CompletionClient):
    def complete(self, prompt: str, max_tokens: int) -> str:
        return _bedrock_invoke(prompt, max_tokens)

class VertexAdapter(CompletionClient):
    def complete(self, prompt: str, max_tokens: int) -> str:
        return _vertex_invoke(prompt, max_tokens)
```

The *application* depends on `CompletionClient`. FinOps and residency decide which adapter is injected.

## 2. Managed AI as a cloud service row

| Axis | AWS | Azure | GCP |
| --- | --- | --- | --- |
| Managed foundation models | Bedrock (multi-model) | Azure OpenAI / AI Foundry | Vertex AI / Gemini |
| GPU IaaS | EC2 / Trainium, Inferentia | N-series / ND | A3/A4, TPU |
| Data plane gravity | S3 + IAM | ADLS + Entra | GCS + IAM |
| Typical sticky bit | Account + IAM graph | Entra ID + M365 | BigQuery + analytics |

Flexera 2025 notes **~72%** of respondents using generative AI and rising AI pressure on cloud bills. Treat a managed model API as **SaaS-like PaaS**: you do not patch the GPU, but you *do* own prompt injection, logging, and data residency.

<div class="content-box warning-box">
<p><strong>Egress is the silent multi-cloud tax.</strong> Training in one cloud and serving in another can cost more than the GPU hours. Draw the bytes before you draw the boxes.</p>
</div>

## 3. Sovereignty and “which region” as architecture

For IUH-style systems, questions that were optional in 2018 are now design inputs:

- May student prompts leave the country?
- Is a EU sovereign offering required for a partner?
- Do you need customer-managed keys on the embedding store?

Sometimes the answer is **one** hyperscaler + an on-prem or national cloud for the regulated slice—not three public clouds.

## Challenges

- Invoice incomparability (RUM, tokens, GPU-seconds).
- Different IAM models (the real lock-in).
- AI feature velocity: today’s adapter is tomorrow’s deprecated model id.

## Video tham khảo

- **xem trước:** [What is Multicloud? How Do You Manage It?](https://www.youtube.com/watch?v=AjtdZ3gFRjU) — IBM Technology, English. Watch first for multi-cloud as something you operate, which is this lesson’s “in the data, not in the slide.”
- [Introducing Amazon Bedrock \| Amazon Web Services](https://www.youtube.com/watch?v=_vdK5PgcNvc) — Amazon Web Services, English. Amazon Bedrock as a managed generative-AI row, one of the three catalogs the lesson adds.
- [What is Vertex AI](https://www.youtube.com/watch?v=Rl9Gz1Yf6wM) — Google Cloud, English. What Vertex AI is, the Google managed-AI row in the same comparison.

## Exercises

1. Extend the required service-mapping table with *one* AI row and *one* observability row (OTel collectors).
2. Estimate egress for 5 TB of embeddings copied monthly between two regions (use a public price page).
3. Write an ADR: “We accept Azure OpenAI lock-in for 18 months because …” (or the opposite).

## References

1. Flexera *2025 State of the Cloud Report*: [press](https://www.flexera.com/about-us/press-center/new-flexera-report-finds-84-percent-of-organizations-struggle-to-manage-cloud-spend), [blog](https://www.flexera.com/blog/finops/the-latest-cloud-computing-trends-flexera-2025-state-of-the-cloud-report/).
2. AWS Bedrock, Azure OpenAI Service, Google Vertex AI product documentation (compare quotas, data-use policies, regional availability).
3. CNCF 2024 survey — Kubernetes as the portable compute layer: [cncf.io/reports/cncf-annual-survey-2024](https://www.cncf.io/reports/cncf-annual-survey-2024/).
4. Required lesson provider mapping — still the baseline this note extends.
