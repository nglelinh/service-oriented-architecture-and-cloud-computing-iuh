---
layout: post
title: 05 Apache Spark Fundamentals
chapter: '05'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter05
---

Apache Spark is a unified analytics engine for large-scale data processing, offering significant performance improvements over traditional MapReduce through in-memory computing and advanced optimization techniques.

## Learning Objectives

- Understand Spark architecture and core concepts
- Learn RDD (Resilient Distributed Dataset) programming
- Explore Spark's execution model and optimization
- Compare Spark with MapReduce
- Set up and configure Spark clusters
- Write Spark applications in Scala, Python, and Java

Apache Spark transformed big data from a slow, disk-bound overnight batch chore into a unified, interactive analytics engine capable of running SQL, streaming, machine learning, and graph processing at in-memory speeds. Chapter 05 unpacks how Spark works from the inside out—starting with **RDDs, DAGs, and lazy evaluation**, then stepping directly into **Spark Connect, Spark 4.0, and Kubernetes** for modern cloud-native deployments.

## Key ideas: The Spark Execution Stack

1. **In-Memory Resilient Computing (RDDs & Lineage):** Keeps hot working sets in distributed RAM while recovering lost partitions deterministically through lineage graphs rather than costly disk replication.
2. **DAG Scheduling & Catalyst Optimization:** Compiles high-level transformations into pipelined stages separated only by necessary wide shuffles, fusing operations to eliminate redundant scans.
3. **Cloud-Native Evolution (Spark Connect & K8s):** Decouples lightweight clients from remote Spark servers over gRPC and schedules elastic executors as Kubernetes Pods.

### Worked Example (Step by Step): Counting Stages and Shuffles in a Spark DAG

Trace two ways of writing Word Count on 100 million log lines across 10 partitions:
1. **`rdd.flatMap(split).map(w => (w, 1)).groupByKey().mapValues(sum)`:**
   - **Stage 1 (Narrow):** `flatMap` + `map` fuse into 1 loop per partition, emitting **100M pairs** into the shuffle buffer.
   - **Shuffle (Wide):** all **100M pairs** cross the network!
   - **Stage 2:** `sum` runs on each key's huge list.
2. **`rdd.flatMap(split).map(w => (w, 1)).reduceByKey(_ + _)`:**
   - **Stage 1 (Narrow + Map-side combine):** each partition locally sums duplicate words (say 10,000 unique words/partition → only $$10 	imes 10{,}000 = \mathbf{100{,}000	ext{ pairs}}$$ enter the network—**1,000× less shuffle traffic!**).
   - **Stage 2:** final merge across 10 partitions finishes in milliseconds!

### Explanation: How to Study Chapter 05 Effectively

1. **Think in partitions, not rows (`05-01`):** every Spark performance puzzle comes down to partition count, skew, and how much data crosses a wide shuffle boundary.
2. **Distinguish Transformations from Actions:** nothing executes until an Action fires—use this mental model to read Spark UI DAG visualizations effortlessly.
3. **Bridge RDD theory to DataFrames and Spark Connect (`05-02`):** learn RDDs to understand *how* Spark works under the hood, then write production pipelines in DataFrames/SQL so Catalyst and Spark Connect can optimize them.
4. **Always inspect the Spark UI:** stage count, shuffle read/write bytes, and straggler task durations tell the real performance story better than guessing.

**Quick check.** If a dataset has **200 partitions** and you run `df.filter("country == 'VN'").select("user_id")`, how many bytes are shuffled across the network? **Zero bytes!** Both `filter` and `select` are narrow transformations executed completely locally inside each partition.

## Key Topics

1. **Spark Core Architecture**
   - Driver and executor processes
   - Cluster managers (Standalone, YARN, Mesos, Kubernetes)
   - Spark Context and Spark Session
   - Memory management and caching

2. **RDD Programming Model**
   - Creating RDDs from data sources
   - Transformations vs Actions
   - Lazy evaluation and lineage
   - Partitioning and data locality

3. **Performance and Optimization**
   - In-memory computing advantages
   - Catalyst optimizer
   - Tungsten execution engine
   - Broadcast variables and accumulators

## Optional application lesson

Required theory in this chapter is unchanged. For Spark Connect, Spark 4.0, and Spark-on-Kubernetes applications, see [05-02 Spark Connect, Spark 4.0, and Cloud-Native Applications]({{ site.baseurl }}{% multilang_post_url contents/chapter05/21-01-01-05_02_Spark_Connect_and_Cloud_Native %}).

## Video tham khảo

- **xem trước:** [What is Apache Spark?](https://www.youtube.com/watch?v=p8FGC49N-zM) — Databricks, English. Watch first: Databricks’ two-minute answer to what Spark is, the engine this chapter introduction names.
- [02 How Spark Works - Driver & Executors \| How Spark divide Job in Stages \| What is Shuffle in Spark](https://www.youtube.com/watch?v=nKM4f1olwsI) — Ease With Data, English. Driver, executors, and shuffle, the architecture terms the introduction lists under Spark Core.
