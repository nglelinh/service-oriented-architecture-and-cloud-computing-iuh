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

Apache Spark is a unified analytics engine for large-scale data processing. It provides high-level APIs in Java, Scala, Python, and R, and an optimized engine that supports general execution graphs.

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
