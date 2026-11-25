---
layout: post
title: 05-01 Apache Spark Fundamentals
chapter: '05'
order: 2
owner: Nguyen Le Linh
lang: en
categories:
- chapter05
lesson_type: required
---

Once data scientists tried training iterative machine learning models like Logistic Regression or PageRank on Hadoop MapReduce, they hit a painful wall: every single iteration had to write gigabytes of intermediate state out to slow HDFS disks and read it all back in for the next step! **Apache Spark** shattered that bottleneck by keeping working datasets **in distributed RAM** inside **Resilient Distributed Datasets (RDDs)** and compiling multi-step transformations into an optimized **Directed Acyclic Graph (DAG)**—delivering 10× to 100× speedups over classic MapReduce.

## Key ideas: Spark's In-Memory DAG Architecture

1. **Driver, Executors, and Cluster Manager:** The **Driver** hosts `SparkSession`, translates your code into a logical DAG, splits it into stages, and dispatches tasks to **Executor** JVMs allocated by YARN or Kubernetes.
2. **RDDs, Immutability, and Lineage-Based Fault Tolerance:** Rather than replicating every intermediate RAM dataset across nodes (expensive!), Spark records the exact **lineage graph** of transformations that built each partition; if a node crashes, Spark recomputes *only* the lost partitions from parent data!
3. **Lazy Evaluation & Narrow vs. Wide Transformations:** Transformations (`map`, `filter`, `reduceByKey`) build a DAG lazily without running a single byte until an **Action** (`count`, `collect`, `save`) triggers execution—allowing Spark to pipeline *narrow* stages in memory and minimize expensive *wide* network shuffles.

### Worked Example (Step by Step): Quantifying Spark's In-Memory Speedup Over MapReduce

Consider **10 iterations** of Logistic Regression over a **100 GB** training dataset (aggregate HDFS disk bandwidth = **500 MB/s** = $$0.5	ext{ GB/s}$$; aggregate cluster RAM bandwidth = **20 GB/s**):

| Engine | I/O Work Across 10 Iterations | Estimated I/O Time |
|---|---|---|
| **Hadoop MapReduce** | Reads 100 GB + writes 100 GB to HDFS *every* iteration ($$10 	imes 200	ext{ GB} = 2{,}000	ext{ GB}$$ disk I/O) | $$\dfrac{2{,}000	ext{ GB}}{0.5	ext{ GB/s}} = \mathbf{4{,}000	ext{ s}}$$ (~66.7 min) |
| **Apache Spark (`.cache()`)** | 1 initial HDFS read ($$100	ext{ GB}$$) + 10 in-memory scans ($$1{,}000	ext{ GB}$$ RAM) | $$\dfrac{100}{0.5} + \dfrac{1{,}000}{20} = 200 + 50 = \mathbf{250	ext{ s}}$$ (~4.2 min) |

Caching the dataset in distributed RAM delivers a **$$4{,}000 / 250 = \mathbf{16	imes}$$ speedup** on the exact same cluster hardware!

### Explanation: How Spark's DAG Engine Plans and Recovers Jobs

1. **Lazy evaluation enables global optimization:** because Spark waits until an Action (`count()`, `write()`) before running, it can fuse `filter().map().map()` into a single loop over each partition with zero intermediate arrays.
2. **Stage boundaries occur exclusively at Wide transformations:** *narrow* ops (`map`, `filter`) keep each child partition dependent on 1 parent partition (zero network); *wide* ops (`groupByKey`, `join`, `repartition`) require a full network shuffle and start a new DAG stage.
3. **Prefer `reduceByKey` over `groupByKey`:** `groupByKey` ships *every* raw record across the network before aggregating, whereas `reduceByKey` combines locally on each executor first—cutting shuffle traffic dramatically.
4. **Beware `.collect()` on large RDDs:** calling `collect()` pulls every partition from all executors back into the single Driver's RAM, causing instant `OutOfMemoryError` on big datasets!

**Quick check.** In the pipeline `rdd.filter(f).map(g).reduceByKey(add).filter(h).count()`, how many DAG stages does Spark create? Exactly **2 stages**—Stage 1 fuses `filter(f) → map(g)` + local map-side combine, followed by 1 shuffle boundary for `reduceByKey`, then Stage 2 runs final reduce + `filter(h)` + `count()`.

## Why Spark?

**Limitations of MapReduce:**
- **Slow**: Heavy reliance on disk I/O (reads/writes to HDFS for every stage).
- **Inefficient**: Not good for iterative algorithms (ML, graph processing).
- **Complex**: Verbose code.

**Spark Advantages:**
- **Speed**: Run programs up to 100x faster than Hadoop MapReduce in memory, or 10x faster on disk.
- **Ease of Use**: Write applications quickly in Java, Scala, Python, R, and SQL.
- **Generality**: Combine SQL, streaming, and complex analytics.

## Spark Architecture

### Components
1.  **Driver**: The process running the `main()` function of the application and creating the `SparkContext`. It schedules tasks.
2.  **Executor**: A distributed agent responsible for executing tasks. It runs in a JVM on a worker node.
3.  **Cluster Manager**: External service for acquiring resources (Standalone, YARN, Mesos, Kubernetes).

### Execution Flow
1.  Driver converts user code into valid tasks.
2.  Driver connects to Cluster Manager to negotiate resources.
3.  Cluster Manager launches executors on worker nodes.
4.  Driver sends tasks to executors.
5.  Executors execute tasks and return results to the driver.

## RDD (Resilient Distributed Dataset)

The primary data abstraction in Spark.
- **Resilient**: Fault-tolerant (recomputes missing partitions using lineage).
- **Distributed**: Data resides on multiple nodes.
- **Dataset**: Collection of objects.

### Characteristics
- **Immutable**: Once created, cannot be changed.
- **Lazy Evaluation**: Transformations are not executed immediately.
- **Cacheable**: Can be persisted in memory for fast reuse.

### RDD Operations

#### 1. Transformations (Lazy)
Create a new RDD from an existing one.
- `map(func)`
- `filter(func)`
- `flatMap(func)`
- `groupByKey()`
- `reduceByKey(func)`

#### 2. Actions (Eager)
Return a value to the driver program after running a computation on the dataset.
- `count()`
- `collect()`
- `take(n)`
- `saveAsTextFile(path)`

### Lazy Evaluation

Spark records transformations as a **DAG (Directed Acyclic Graph)** but does nothing until an action is called.
- Allows Spark to optimize the execution plan (e.g., pipelining maps and filters).
- Reduces unneeded data transfer.

## Example: Word Count in PySpark

```python
from pyspark import SparkContext

sc = SparkContext("local", "Word Count")

# 1. Load Data (Transformation)
text_file = sc.textFile("input.txt")

# 2. Transformations (Lazy)
counts = text_file.flatMap(lambda line: line.split(" ")) \
             .map(lambda word: (word, 1)) \
             .reduceByKey(lambda a, b: a + b)

# 3. Action (Trigger Execution)
counts.saveAsTextFile("output")
```

## Spark vs. Hadoop MapReduce

| Feature | Hadoop MapReduce | Apache Spark |
|---------|------------------|--------------|
| **Processing** | Disk-based (Iterative writes) | In-memory (Cacheable) |
| **Speed** | Slower | Up to 100x faster |
| **Difficulty** | High (Verbose Java) | Low (High-level APIs) |
| **Use cases** | Batch processing | Batch, Streaming, ML, Interactive |

## Summary

Apache Spark is the successor to MapReduce for most modern big data workloads. Its in-memory capability and rich ecosystem (SQL, Streaming, MLlib) make it a versatile tool for data engineers and data scientists.

## Video tham khảo

- **xem trước:** [What is Apache Spark?](https://www.youtube.com/watch?v=p8FGC49N-zM) — Databricks, English. Watch first for what Spark is, before the lesson’s driver, executor, and RDD sections.
- [02 How Spark Works - Driver & Executors \| How Spark divide Job in Stages \| What is Shuffle in Spark](https://www.youtube.com/watch?v=nKM4f1olwsI) — Ease With Data, English. How a job splits across a driver and executors, including shuffle, matching the execution-flow section.
- [What Are RDDs in Apache Spark? \| RDD Explained in 10 Minutes for Beginners \| PySpark](https://www.youtube.com/watch?v=46WOZ1Kcles) — Data Forge, English. What an RDD is, for the resilient distributed dataset section and the word-count example.
