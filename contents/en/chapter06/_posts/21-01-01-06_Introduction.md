---
layout: post
title: 06 Introduction to Spark Ecosystem
chapter: '06'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter06
---

This chapter moves beyond the core Spark engine to explore the powerful libraries built on top of it: Spark SQL for structured data, Spark Streaming for real-time processing, and MLlib for machine learning.

## Learning Objectives

- **Spark SQL**: Query structured data using SQL and DataFrames
- **Spark Streaming**: Process real-time data streams with fault tolerance
- **MLlib**: Build and deploy scalable machine learning models
- **GraphX**: (Overview) Analyze graph-structured data

Once a company adopts Apache Spark, it quickly discovers why Spark is called a **unified analytics engine**: instead of maintaining four separate clusters with four incompatible storage formats—one for SQL reports, one for real-time alerts, one for ML training, and one for graph analytics—all of them run on **one shared DataFrame abstraction** and **one distributed execution engine**. Chapter 06 explores that entire ecosystem: **Spark Streaming**, **Spark SQL**, **MLlib**, and how they plug into modern **event-driven ML serving**.

## Background Knowledge Map: The Unified Spark Ecosystem

1. **Spark SQL & DataFrames:** Provides relational queries, schema enforcement, Catalyst query optimization, and columnar Parquet/Iceberg integration.
2. **Spark Streaming / Structured Streaming:** Extends the exact same DataFrame API to unbounded live event feeds with watermarks, windowing, and fault-tolerant checkpointing.
3. **Spark MLlib & Event-Driven Serving:** Scales tabular feature pipelines and distributed training across terabytes, feeding low-latency online feature stores and model servers in production SOA.

### Worked Example (Step by Step): Tracing One Click Event Across All Three Spark Modules

Follow a single e-commerce click event `{"user_id": 42, "item_id": 99, "price": 120.0, "ts": "10:05:12"}`:
1. **Spark Streaming (Seconds):** reads the event from Kafka, aggregates `user_id = 42`'s 5-minute cart value, and appends the raw record to an Iceberg table `events_silver`.
2. **Spark SQL (Minutes):** an analyst runs `SELECT item_id, SUM(price) FROM events_silver WHERE ts >= '10:00' GROUP BY item_id`; Catalyst pushes the timestamp filter into Iceberg partitions and returns results in seconds.
3. **Spark MLlib (Nightly):** reads 90 days of `events_silver` directly via the same DataFrame schema, trains a recommendation/churn pipeline, and publishes updated scores to the online serving tier.

### Deep-Dive Explanation: Mastering the Spark Ecosystem in Four Steps (Read Slowly)

1. **Learn the shared DataFrame core first:** every library in Chapter 06—SQL, Structured Streaming, and MLlib—speaks DataFrames, so mastering schemas and transformations pays triple dividends.
2. **Inspect Catalyst plans (`06-03`) before optimizing code:** understanding broadcast joins and predicate pushdown often yields 10× speedups with zero extra hardware.
3. **Respect stateful boundaries in streaming (`06-02`):** watermarks and checkpoints are the difference between a demo that works for 5 minutes and a pipeline that runs for 5 years.
4. **Decouple computation from serving (`06-05`):** let Spark do heavy distributed feature engineering and training while lightweight microservices handle millisecond user requests.

**Quick check.** Why does sharing one DataFrame schema across Streaming, SQL, and MLlib reduce production bugs? Because data never has to be converted between incompatible batch and streaming engines—eliminating schema drift and duplicate ETL logic!

## The Unified Engine

The true power of Spark lies in its unified stack. You can load data using Spark SQL, train a model using MLlib, and apply that model to a real-time stream using Spark Streaming—all within the same application.

## Optional application lesson

Required theory in this chapter is unchanged. For event-driven features, serving-adjacent ML, and stream-vs-request clocks (2022–2026), see [06-05 Real-Time, Event-Driven, and Serving-Adjacent Spark Apps]({{ site.baseurl }}{% multilang_post_url contents/chapter06/21-01-01-06_05_Realtime_Event_Driven_ML %}).

## Video tham khảo

- **xem trước:** [What is Apache Spark?](https://www.youtube.com/watch?v=p8FGC49N-zM) — Databricks, English. Watch first: what Spark is, the unified engine this introduction says SQL, streaming, and MLlib share.
- [PySpark Tutorial: Spark SQL & DataFrame Basics](https://www.youtube.com/watch?v=3-pnWVWyH-s) — Greg Hogg, English. DataFrame and Spark SQL basics, one of the three libraries the introduction names.
