---
layout: post
title: 06-05 Real-Time, Event-Driven, and Serving-Adjacent Spark Apps
chapter: '06'
order: 5
owner: Nguyen Le Linh
lang: en
categories:
- chapter06
lesson_type: optional
---

The required stack (Spark SQL, Structured Streaming, optional MLlib) is enough to *compute*. This optional lesson shows how 2022–2026 products wire that stack into **event-driven SOA**: Kafka/CloudEvents in, lakehouse tables out, and a separate **online** path (feature store + model server) so Spark is not on the user request path.

## Learning objectives

- Separate *stream processing* (Spark/Flink) from *request serving* (gRPC/HTTP, KServe, vLLM).
- Describe a Kappa-style path that still uses batch Spark for backfills.
- Sketch Structured Streaming output as an Iceberg/Delta table consumed by many engines.

## 1. Two clocks in one product

| Clock | Typical engine | SLA |
| --- | --- | --- |
| Event time (fraud, click, IoT) | Spark Structured Streaming, Flink | seconds |
| Request time (score *this* user) | Feature store + model HTTP/gRPC | milliseconds |
| Business time (month-end, GDPR) | Spark SQL batch | hours |

A common failure is to expose a Streaming query as a synchronous API. Spark micro-batches are **not** a 50 ms p99 model server. The application pattern is: Streaming **updates state**; Serving **reads** a cache or feature store.

```python
# Conceptual: stream writes features; it does not answer HTTP
from pyspark.sql import functions as F

features = (
    kafka_df
    .select(F.col("user_id"), F.col("event_ts").cast("timestamp").alias("ts"), "amount")
    .withWatermark("ts", "10 minutes")
    .groupBy("user_id", F.window("ts", "1 hour"))
    .agg(F.sum("amount").alias("amt_1h"))
)

(
    features.writeStream
    .format("iceberg")
    .outputMode("append")
    .option("checkpointLocation", "s3://lab/chk/feat")
    .toTable("prod.features_user_1h")
)
```

Online path (not Spark): a service loads recent features and calls a model.

```python
def score(user_id: str, model_client) -> float:
    feat = feature_store.get(user_id, columns=["amt_1h"])
    return model_client.predict(feat)  # gRPC / OpenAI-compatible HTTP
```

## 2. Event-driven ML applications (2023–2026)

- **Nearline recommendations**: session events → Streaming aggregates → Redis/feature store → ranker service.
- **LLM evaluation offline**: Spark SQL over traces (prompts, tokens, latency) stored as Parquet/Iceberg; the LLM itself is served by vLLM/KServe.
- **MLlib in 2026**: still useful for large linear/tree jobs on Spark Connect (Spark 4.0 added ML-on-Connect). Many teams now train with Ray/PyTorch and only use Spark for **data**.

CNCF’s 2024 survey notes serverless and eventing remain in the toolkit but teams drop them when cost/complexity dominate. Use events for **facts**; use RPC for **questions**.

## 3. Unified engine, split deployment

The required “unified stack” is a *development* win. Production usually **splits** the jar:

1. Streaming job (always on, checkpointed).
2. Batch job (Airflow/dagster, same DataFrame code).
3. Serving replica (no Spark driver in the pod).

That split is how you keep Structured Streaming’s exactly-once *sink* semantics from colliding with an autoscaling HTTP fleet.

<div class="content-box insight-box">
<p><strong>Watermarks are business rules.</strong> A 10-minute watermark is a statement about late mobile events, not a Spark trivia setting. Write it down next to the SLO.</p>
</div>

## Challenges

- Late data vs. dashboard “freshness” marketing.
- Dual writes (Kafka + table) without a transactional outbox.
- Training/serving skew when Streaming features differ from batch backfill.

## Exercises

1. For a payments app, list three metrics that belong in Streaming vs three that belong in a nightly Spark SQL job.
2. Explain why `outputMode("complete")` to a serving API is usually the wrong SOA boundary.
3. Design a backfill: same feature table, batch Spark, event-time correctness for one week of late Kafka replay.

## References

1. Apache Spark Structured Streaming programming guide (current 4.x docs).
2. Apache Spark 4.0.0 — ML on Spark Connect: [spark-release-4-0-0.html](https://spark.apache.org/releases/spark-release-4-0-0.html).
3. CloudEvents: [cloudevents.io](https://cloudevents.io/).
4. CNCF *Cloud Native 2024 Annual Survey*: [cncf.io/reports/cncf-annual-survey-2024](https://www.cncf.io/reports/cncf-annual-survey-2024/).
5. vLLM / KServe serving docs — the request-time counterpart to this streaming path.
