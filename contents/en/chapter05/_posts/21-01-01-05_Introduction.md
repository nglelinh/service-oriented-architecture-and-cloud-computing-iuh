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
