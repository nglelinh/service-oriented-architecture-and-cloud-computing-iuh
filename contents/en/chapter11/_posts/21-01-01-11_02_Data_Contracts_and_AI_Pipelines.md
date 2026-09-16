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
