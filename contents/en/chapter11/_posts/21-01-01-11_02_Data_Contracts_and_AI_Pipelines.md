---
layout: post
title: 11-02 Data Contracts, Quality Gates, and AI Data Pipelines
chapter: '11'
order: 3
owner: Nguyen Le Linh
lang: en
categories:
- chapter11
lesson_type: optional
---

The required lesson is sourcing, cleaning, and validation. This optional note is how 2022–2026 platform teams turned those chores into **SOA contracts**: versioned schemas between producers and consumers, automated quality gates, and *separate* pipelines for **LLM/RAG** data that cannot be treated like another CSV.

Picture an upstream backend team quietly renaming a JSON field from `price_cents` to `price_dollars` on a Friday afternoon. Their unit tests pass, the microservice deploys cleanly—and silently poisons every downstream BI dashboard, recommendation model, and RAG vector index for the next three weeks. **Data Contracts** and **Quality Gates** bring the rigor of API versioning to data engineering: treating every dataset as a formal, machine-verifiable interface between producers and consumers so broken data is caught at the gate before it ever corrupts an AI pipeline.

## Background Knowledge Map: Contracts, Quality Gates, and Trustworthy AI Data

### 1. What Makes a Data Contract Enforceable?
A real **Data Contract** is not a forgotten Wiki page—it is a versioned specification (JSON Schema, Avro, Protobuf, or YAML) checked in CI/CD and at runtime. It locks down four guarantees: **schema** (field names, types, nullability), **semantics** (units, allowed enums, business invariants like `amount >= 0`), **SLAs** (freshness, completeness), and **ownership** (which producer team gets paged when the contract breaks).

### 2. Circuit Breakers and Quarantine Patterns (Write–Audit–Publish)
Instead of writing raw batches straight into production analytics tables, mature pipelines follow **Write–Audit–Publish (WAP)**: stage new data into an isolated branch or snapshot, run automated **quality gates** (Great Expectations, dbt tests, Soda), and publish to consumers *only if* every gate passes. Failing records are routed to a **Dead-Letter / Quarantine** table so pipelines fail loudly and safely.

### 3. Why AI and RAG Pipelines Magnify Silent Data Bugs
In classical SQL reporting, a broken column causes an obvious chart spike. In **machine learning and RAG (Retrieval-Augmented Generation)** pipelines, stale documents, duplicate embeddings, or leaky future labels degrade model accuracy silently without throwing a single HTTP 500 error—making lineage tracking and automated contract gates non-negotiable.

### Deep-Dive Explanation: Hand-Evaluating a Write–Audit–Publish Quality Gate on a Daily Batch (Read Slowly)

Suppose an AI feature store ingests **1,000,000 customer-interaction records** every night under a Data Contract with three hard gates:
- **Gate 1 (Schema):** `0` unparseable or missing required fields (`100%` valid).
- **Gate 2 (Completeness):** `null_rate(user_id) <= 0.10%` (at most $$1{,}000$$ nulls per million).
- **Gate 3 (Freshness):** pipeline completes within **15 minutes** of midnight UTC.

Tonight's staged batch arrives in **11 minutes** with `0` schema errors, **1,500** rows missing `user_id`, and **998,500** clean rows.

1. **Evaluate each gate:**
   - Gate 1: $$0\text{ errors} \implies \textbf{PASS}$$.
   - Gate 2: $$\text{null\_rate} = \frac{1{,}500}{1{,}000{,}000} = 0.15\% > 0.10\% \implies \textbf{FAIL}$$.
   - Gate 3: $$11\text{ min} \le 15\text{ min} \implies \textbf{PASS}$$.

2. **Outcome of the Write–Audit–Publish circuit breaker:**
   Because Gate 2 failed, the snapshot pointer **does not advance**. Production RAG and ML models continue serving yesterday's verified snapshot with zero corruption, the 1,500 bad rows are isolated in the quarantine bucket, and the owning producer team receives an automated alert pointing straight to the offending upstream release.

## Learning objectives

- Define a data contract as an interface (schema + SLOs + owner), not a slide.
- Place Great Expectations / dbt tests / schema registries on the required “validation” list.
- List extra cleaning rules for prompts, embeddings, and training corpora.

## 1. Data contracts: the missing WSDL of analytics

A **data contract** (popularized in industry writing by PayPal, GoCardless, and others around 2022–2024) is a producer-owned spec:

- Schema (types, nullability, enums, PII flags).
- Freshness and completeness SLOs.
- Compatibility policy (additive vs. breaking).
- Owner and on-call.

That is SOA: the warehouse table is a *service*. Breaking `user_id` type is the same class of error as breaking a protobuf field.

```yaml
# Conceptual contract (not a specific vendor DSL)
name: campus.enrollments.v2
owner: registrar-platform
sla:
  freshness_minutes: 60
  completeness: 0.995
schema:
  - { name: student_id, type: string, pii: true }
  - { name: course_id, type: string, pii: false }
  - { name: enrolled_at, type: timestamp }
compat: backward
```

Consumers (Spark jobs, dashboards, feature stores) subscribe to **v2**, not to “whatever landed in the landing zone.”

## 2. Quality gates in the pipeline

```python
import pandas as pd

REQUIRED = ["student_id", "course_id", "enrolled_at"]

def quality_gate(df: pd.DataFrame) -> None:
    missing = [c for c in REQUIRED if c not in df.columns]
    if missing:
        raise ValueError(f"contract break: missing {missing}")
    null_rate = df["student_id"].isna().mean()
    if null_rate > 0.005:
        raise ValueError(f"completeness SLO failed: {null_rate:.3%}")
    if df.duplicated(["student_id", "course_id"]).any():
        raise ValueError("duplicate enrollments")
```

Tools you will see on 2024–2026 résumés: **dbt** tests, Great Expectations/GX Cloud, Soda, Monte Carlo (observability), schema registries (Confluent, Glue). The *idea* is the required validation section, automated in CI so a bad file never becomes a “cleaned” Iceberg snapshot.

## 3. AI/LLM data is not “just unstructured text”

| Stage | Failure if you skip cleaning |
| --- | --- |
| Source (web, tickets, LMS) | License / PII in the corpus |
| Dedup & PII redaction | Model memorizes a student email |
| Chunk + embed | Retrieval returns the wrong tenant |
| Eval set | You cannot tell if RAG got worse |
| Prompt/trace store | You train on leaked secrets next quarter |

Application pattern: **two zones**. Zone A is the system of record (contracts, ACID). Zone B is derived embeddings and prompt logs with shorter retention, stricter ACL, and an explicit *re-embed* job when the embedding model changes (see Chapter 07).

<div class="content-box warning-box">
<p><strong>Scraping is not a source strategy.</strong> Robots.txt, licenses, and Vietnam/EU personal-data law apply before you call <code>BeautifulSoup</code>. The required lesson’s “APIs vs scraping” section is a compliance topic in 2026.</p>
</div>

## Challenges

- Contracts without an enforcement point (a wiki page is not a gate).
- Cleaning that drops the only rows that matter (bias).
- LLM-as-cleaner: cheap to demo, expensive to audit.

## Video tham khảo

- **xem trước:** [What is a Data Contract?](https://www.youtube.com/watch?v=-n3OD-ml_k0) — IBM Technology, English. Watch first: IBM’s definition of a data contract, the “missing WSDL of analytics” in this lesson.
- [What Are Data Contracts? (And How They Actually Work in Production)](https://www.youtube.com/watch?v=6svVYbAcfHc) — The Data and AI Guy, English. How contracts behave between producers and consumers in a pipeline, the quality-gate section.
- [What is a Data Contract?](https://www.youtube.com/watch?v=3DU6vuES5eQ) — Monte Carlo, English. A second, shorter definition of a data contract from a data-reliability vendor.

## Exercises

1. Write a 12-line contract for `wifi_sessions` that a networking team must not break.
2. Add one quality test that would have caught a timezone mix-up (`enrolled_at` as local vs UTC).
3. Design a redaction step for a helpdesk RAG corpus (what you delete, what you hash, what you keep).

## References

1. Industry primers on data contracts (PayPal engineering blog; GoCardless “data contracts” posts, 2022–2024) — read for *practice*, not as a standard.
2. dbt tests documentation: [docs.getdbt.com](https://docs.getdbt.com/).
3. Great Expectations docs: [greatexpectations.io](https://greatexpectations.io/).
4. NIST AI Risk Management Framework (AI RMF 1.0, 2023) — data quality and governance for AI systems.
5. Required lesson techniques (missing data, duplicates, schema checks) — this note only automates them.
