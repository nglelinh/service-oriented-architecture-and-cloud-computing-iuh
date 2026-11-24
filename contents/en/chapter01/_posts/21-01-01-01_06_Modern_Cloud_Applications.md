---
layout: post
title: 01-06 Modern Cloud Applications (2022–2026)
chapter: '01'
order: 7
owner: Nguyen Le Linh
lang: en
categories:
- chapter01
lesson_type: optional
---

This optional lesson does **not** replace the NIST characteristics, service models, or deployment models in the required notes. It shows how those ideas show up in production systems between about 2022 and 2026: multi-cloud estates, FinOps as measured service, sovereign-cloud constraints, and AI serving as a new first-class cloud workload.

## Learning objectives

- Map NIST *rapid elasticity* and *measured service* onto FinOps practice and AI capacity planning.
- Distinguish *multi-cloud by accident* from *multi-cloud by design* (data residency, best-of-breed services, exit options).
- Sketch an AI-serving path that still respects IaaS/PaaS/SaaS boundaries.

Between 2022 and 2026, the center of gravity in cloud engineering shifted dramatically. Cloud teams stopped asking *"should we migrate?"* and started wrestling with three much harder questions: **how do we stop 27% of our cloud bill from evaporating into waste (FinOps)?**, **how do we serve GPU-hungry AI models at sane unit costs?**, and **how do we operate across 2+ clouds under strict data-sovereignty laws?**

## Background Knowledge Map: Multi-Cloud Reality, FinOps Loops, and AI Serving

1. **Multi-Cloud by Design vs. by Accident:** Running on 2+ providers intentionally (for sovereign compliance, best-of-breed AI accelerators, or exit leverage) requires portable abstractions (containers, Terraform, OpenTelemetry); stumbling into multi-cloud through acquisitions creates duplicate bills and operational drag.
2. **FinOps as Closed-Loop Measured Service:** NIST's *measured service* only saves money when closed into an engineering loop—tagging every workload, computing **unit cost** (e.g., cost per 1,000 API calls or per million LLM tokens), and right-sizing continuously.
3. **AI Serving as a First-Class Cloud Pattern:** Unlike stateless web APIs where CPU scales linearly, LLM and vision inference is **GPU-memory and batching bound**: dynamic batching, KV-cache memory management, and spot/reserved GPU capacity dictate whether an AI product has positive gross margins.

### Worked Example (Step by Step): FinOps Unit-Cost Math for GPU AI Serving

Suppose an AI team rents one cloud GPU node at **$$\$3.50/	ext{hour}$$** to serve an inference endpoint:
- **Unbatched baseline:** the node processes 1 request at a time at **120 requests/min** ($$7{,}200	ext{ req/hr}$$):
  $$
  	ext{Cost per 1,000 requests} = rac{\$3.50}{7.2} pprox \mathbf{\$0.486}.
  $$
- **With dynamic batching (batch size 8, adding only 25 ms latency):** GPU throughput jumps to **300 requests/min** ($$18{,}000	ext{ req/hr}$$):
  $$
  	ext{Cost per 1,000 requests} = rac{\$3.50}{18.0} pprox \mathbf{\$0.194} \quad (	ext{a }\mathbf{60\%}	ext{ unit-cost reduction!}).
  $$
- At **10 million requests/month**, that single FinOps engineering optimization cuts the monthly GPU bill from **$$\$4{,}860$$** down to **$$\$1{,}940$$**—saving **$$\$35{,}040/	ext{year}$$** on the exact same model!

### Deep-Dive Explanation: Connecting 2022–2026 Production Cloud Patterns Back to NIST (Read Slowly)

1. **Measure unit economics, not just total spend:** a rising cloud bill is healthy if revenue per 1,000 requests rises faster; FinOps tracks *cost per unit of business value*.
2. **Treat AI inference as a specialized PaaS/IaaS boundary:** decide whether to call a hosted model API (SaaS-like, fast launch, per-token billing) or self-host weights on GPU clusters (IaaS-like, lower unit cost at sustained volume).
3. **Design for sovereign compliance via region & key isolation:** customer-managed encryption keys (CMEK) and regional data residency keep sensitive workloads legal without rebuilding the entire app.
4. **Fight accidental complexity in multi-cloud:** standardize on portable data formats and observability (OpenTelemetry) rather than trying to abstract away every provider feature into a lowest-common-denominator wrapper.

**Quick check.** If traffic drops to 500,000 requests/month (only ~12 req/min average), should the team still rent a 24/7 $$\$3.50/	ext{hr}$$ dedicated GPU ($$\$2{,}520/	ext{mo}$$)? No! At 500,000 requests/month, a serverless/API model priced at $$\$0.50$$ per 1,000 requests costs just **$$\$250/	ext{mo}$$**—10× cheaper than an idle dedicated GPU!

## 1. Multi-cloud is the steady state, not a slogan

Flexera’s *2025 State of the Cloud Report* (14th edition; 750+ practitioners) finds that **70%** of organizations run a hybrid mix of at least one public and one private cloud, and that estates typically span about **2.4 public providers**. Managing spend is the top challenge for **84%** of respondents; dedicated FinOps teams that own some or all cost optimization rose from **51% to 59%** year over year. Expected spend growth (~28%) and estimated IaaS/PaaS waste (~27%) sit beside a modest repatriation rate (~21% of workloads), so the net cloud footprint is still growing.

That is the NIST model under load: **resource pooling** across providers, **on-demand self-service** via APIs, and **measured service** that finance teams can no longer treat as a rounding error.

```mermaid
graph LR
  A[Product team] -->|API / IaC| B[Public cloud A]
  A -->|API / IaC| C[Public cloud B]
  A -->|private / sovereign| D[On-prem or EU region]
  B --> E[FinOps + policy]
  C --> E
  D --> E
```

**Design question.** If a service can run on only one provider’s proprietary API, you have a *deployment* model (public cloud) but not a *portable* architecture. Portability is a product requirement, not a default.

## 2. FinOps is measured service with a feedback loop

NIST *measured service* stops at metering. FinOps adds allocation, forecasting, and unit economics (cost per checkout, per inference, per tenant). A minimal loop:

1. **Tag and allocate** every billable resource (team, environment, product).
2. **Show unit cost** next to SLOs (p99 latency, error rate), not only next to the invoice.
3. **Act**: rightsizing, reserved/spot mix, scale-to-zero, or *repatriation* when the economics invert.

Python-shaped sketch of a unit-cost check you could hang off a monthly bill export:

```python
def unit_cost(invoice_usd: float, successful_inferences: int) -> float:
    if successful_inferences <= 0:
        raise ValueError("no successful work to allocate cost against")
    return invoice_usd / successful_inferences

# Example: GPU serving cluster vs. managed AI API
gpu_unit = unit_cost(18_400.0, successful_inferences=2_100_000)
api_unit = unit_cost(9_250.0, successful_inferences=2_100_000)
print(f"GPU ${gpu_unit:.4f}  vs  API ${api_unit:.4f} per inference")
```

The number is not the strategy. The strategy is making **elasticity visible** so teams stop treating the cloud as an infinite CapEx substitute.

## 3. AI serving as a cloud application pattern

Generative AI did not invent new NIST characteristics; it stressed them. A typical 2024–2026 serving stack:

| Layer | Role | Cloud mapping |
| --- | --- | --- |
| Client / BFF | Auth, quotas, prompt policy | SaaS or custom PaaS |
| Inference router | Model version, A/B, cache | PaaS / service mesh |
| Engine (e.g. vLLM) | Batched GPU decode | IaaS + GPU SKUs |
| Orchestrator (KServe, Ray Serve) | Scale, canary, protocols | Kubernetes / PaaS |

KServe (CNCF incubating as of late 2025) treats an `InferenceService` as a Kubernetes resource with scale-to-zero and canary traffic. Ray Serve composes Python graphs (retrieve → rerank → generate) on KubeRay. Both commonly wrap **vLLM** for OpenAI-compatible HTTP. Students should notice the SOA move: the *model* is a versioned service with an SLA, not a notebook artifact.

<div class="content-box warning-box">
<p><strong>Cold start vs. GPU cost.</strong> Scale-to-zero is elegant on paper and expensive in tail latency for LLMs. Production teams often set <code>minReplicas: 1</code> and buy elasticity on the <em>replica</em> axis, not the zero axis.</p>
</div>

## 4. Sovereignty and compliance reshape deployment models

Public / private / hybrid / community still apply, but *where the bytes live* now drives architecture. EU data-protection and sector rules, plus customer contracts, push **sovereign regions**, customer-managed keys, and “data never leaves this VPC” designs. Hybrid is no longer only “legacy in the basement”: it is often the *compliance* plane sitting beside a *commodity* public plane.

## Challenges

- **Accidental multi-cloud**: two providers because two teams picked defaults, with no identity, network, or data contract between them.
- **AI bill shock**: token and GPU hours are harder to forecast than VM hours.
- **Skills split**: FinOps, platform, and ML serving teams speak different SLOs.

## Video tham khảo

- **xem trước:** [What is Multicloud? How Do You Manage It?](https://www.youtube.com/watch?v=AjtdZ3gFRjU) — IBM Technology, English. Watch first for multi-cloud as an operating problem, which is the opening claim of this optional lesson.
- [What is FinOps? The Operating Model & Cultural Practice to Maximize the Value of Technology](https://www.youtube.com/watch?v=Y-c_xw9bHFw) — FinOps Foundation, English. FinOps as a practice around measured spend, matching the lesson’s “measured service with a feedback loop.”
- [Introducing Amazon Bedrock \| Amazon Web Services](https://www.youtube.com/watch?v=_vdK5PgcNvc) — Amazon Web Services, English. A short official introduction to a managed generative-AI service, the “AI serving as a cloud workload” pattern in the lesson.

## Exercises

1. Take one required-lesson example (e.g. an e-commerce burst). Add a second provider *only* for object storage. List the identity, latency, and egress costs you just accepted.
2. Using a public price page, estimate monthly cost for 10M short LLM calls on a managed API versus one reserved GPU. State your assumptions.
3. Write a six-line data-residency policy: which logs, embeddings, and prompts may leave Vietnam / the EU, and which must not.

## References

1. Flexera, *2025 State of the Cloud Report* (March 2025). [Press summary](https://www.flexera.com/about-us/press-center/new-flexera-report-finds-84-percent-of-organizations-struggle-to-manage-cloud-spend); [trends recap](https://www.flexera.com/blog/finops/the-latest-cloud-computing-trends-flexera-2025-state-of-the-cloud-report/).
2. Flexera, full report PDF: [Flexera-State-of-the-Cloud-Report-2025.pdf](https://resources.flexera.com/web/pdf/Flexera-State-of-the-Cloud-Report-2025.pdf).
3. CNCF / Linux Foundation Research, *Cloud Native 2024 Annual Survey* (published 1 April 2025): [report page](https://www.cncf.io/reports/cncf-annual-survey-2024/).
4. NIST SP 800-145, *The NIST Definition of Cloud Computing* (the required-lesson baseline these applications still implement).
5. vLLM project documentation and KServe *InferenceService* docs (CNCF; incubating announced November 2025).
