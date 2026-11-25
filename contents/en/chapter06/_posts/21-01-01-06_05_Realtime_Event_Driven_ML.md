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

A common rookie mistake in cloud AI projects is wiring a customer's checkout HTTP request directly into an Apache Spark job—and watching page load time explode to 4 seconds! Production 2023–2026 ML systems run on **two completely separate clocks**: a fast **online request path** (Feature Store + gRPC Model Server answering in ~25 ms) and an asynchronous **event-driven Spark/Kafka path** that continuously updates rolling features and lakehouse training tables in the background.

## Key ideas: Two-Clock Event-Driven ML Architecture

1. **The Online Serving Clock (Milliseconds):** User requests hit an API gateway → fetch precomputed low-latency features from an **Online Feature Store** (Redis/DynamoDB) → call a dedicated **Model Server** (KServe, Triton, vLLM over gRPC) → return prediction in < 50 ms. Spark is *never* on this hot path!
2. **The Asynchronous Streaming Clock (Seconds):** Every click, payment, or sensor ping publishes a **CloudEvent** to Kafka; **Spark Structured Streaming** (or Flink) computes rolling aggregates (e.g., *"transactions in last 5 minutes"*) and writes them both to the Online Feature Store and to an **Offline Lakehouse** (Iceberg/Delta).
3. **Avoiding Training–Serving Skew:** Using the exact same feature definition for both offline Iceberg backfills and online Redis updates ensures the model sees identical distributions at training and inference time.

### Worked Example (Step by Step): End-to-End Latency Budget for Real-Time Fraud Scoring

A payments API has a strict **50 ms p99 latency SLA** at checkout:

| Hop on the Online Request Path | Component | p99 Latency |
|---|---|---|
| 1. Auth + Request validation | API Gateway | $$5	ext{ ms}$$ |
| 2. Fetch user's 5-min velocity features | Redis Online Feature Store | $$4	ext{ ms}$$ |
| 3. Run fraud model inference | KServe / Triton (gRPC) | $$18	ext{ ms}$$ |
| 4. Apply approve/block business rule + publish `tx.scored` to Kafka | Checkout Service | $$3	ext{ ms}$$ |
| **Total Synchronous Checkout Latency** | **Online Path** | **$$30	ext{ ms}$$** (20 ms safety margin!) |

Meanwhile, on the **asynchronous clock**, Spark Structured Streaming consumes `tx.scored` from Kafka every **2 seconds**, recomputes each user's rolling 5-minute spend, and updates Redis—so the next transaction sees fresh features without ever waiting on Spark!

### Explanation: Wiring Spark Into Event-Driven SOA Cleanly

1. **Keep Spark off the synchronous user path:** Spark's job scheduling overhead (100–500 ms minimum) makes it ideal for high-throughput stream/batch computation, never for single-request HTTP serving.
2. **Use a dual-sink lakehouse pattern:** write curated stream outputs to *both* low-latency Redis (for live inference) and immutable Iceberg tables (for reproducible model retraining and audits).
3. **Design for graceful feature fallback:** if the online feature store times out after 10 ms, serve safe default features or a lightweight rule-based fallback rather than blocking checkout.
4. **Log exact inference inputs for drift monitoring:** emitting `(trace_id, features_used, prediction)` asynchronously to Kafka lets offline Spark jobs detect feature drift and label feedback loops automatically.

**Quick check.** What happens if a team computes `user_avg_spend_7d` in Python for live inference and in a slightly different SQL query for offline training? Subtle rounding or timezone differences create **training–serving skew**, silently degrading production fraud accuracy even though offline test AUC looked great!

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

## Video tham khảo

- **xem trước:** [Event-Driven Architecture(EDA) vs. Request/Response (RR): System Design Explained](https://www.youtube.com/watch?v=7fkS-18KBlw) — Confluent, an IBM Company, English. Watch first: event-driven versus request/response, the lesson’s warning not to put Spark on the user request path.
- [What is a Feature Store for Machine Learning?](https://www.youtube.com/watch?v=DESBDukN9gw) — Hopsworks, English. What a feature store is for, the online path the lesson splits from the streaming job.
- [Introduction to Vertex AI Feature Store](https://www.youtube.com/watch?v=jXD8Sfx4hvQ) — Google Cloud Tech, English. Vertex AI Feature Store as a managed example of that online feature path.

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
