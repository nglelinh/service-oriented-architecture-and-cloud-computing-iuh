---
layout: post
title: 04 Introduction to Hadoop MapReduce
chapter: '04'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter04
---

This chapter delves into MapReduce, the programming paradigm that popularized big data processing on commodity hardware.

## Learning Objectives

- Understand the MapReduce programming model (Map, Shuffle, Reduce)
- Write MapReduce programs to solve parallelizable problems (e.g., Word Count)
- Analyze the flow of data: InputSplit → Mapper → Partitioner → Reducer → Output
- Understand how MapReduce achieves fault tolerance through re-execution

Why can't you just download a 10-terabyte log file onto one powerful workstation and write a Python `for` loop to count unique visitors? Do the arithmetic, and physics stops you cold: streaming 10 TB over a 1 Gbps network link takes **over 22 hours** before your script even finishes reading the first pass! Chapter 04 introduces the paradigm shift that flipped big data on its head: **stop moving terabytes of data to one CPU—ship tiny kilobytes of code directly to the disks holding the data**.

## Key ideas: The Big-Data Batch Processing Paradigm

1. **Data Locality (Moving Compute to Data):** Pushing lightweight Map tasks directly onto the cluster nodes storing HDFS blocks turns a single-link network bottleneck into aggregate parallel disk bandwidth.
2. **The Map–Shuffle–Reduce Pipeline:** Breaks arbitrary large-scale aggregations into parallel local transforms (`Map`), network key-grouping (`Shuffle`), and parallel group summaries (`Reduce`).
3. **Commodity Fault Tolerance & Modern Descendants:** Deterministic task re-execution makes 1,000-node commodity clusters reliable—a mental model that still drives Spark, BigQuery, and Iceberg lakehouses today.

### Worked Example (Step by Step): Moving Data vs. Moving Compute on 10 TB

Compare scanning a **10 TB** ($$10{,}000	ext{ GB}$$) dataset two ways:
- **Centralized download (1 server, 1 Gbps = 0.125 GB/s network):**
  $$
  T_{	ext{network}} = rac{10{,}000	ext{ GB}}{0.125	ext{ GB/s}} = \mathbf{80{,}000	ext{ seconds}} pprox \mathbf{22.2	ext{ hours}}!
  $$
- **Data-local MapReduce (100 nodes, 100 GB local disk each at 200 MB/s = 0.2 GB/s):**
  $$
  T_{	ext{local}} = rac{100	ext{ GB}}{0.2	ext{ GB/s}} = \mathbf{500	ext{ seconds}} pprox \mathbf{8.3	ext{ minutes}} \quad (\mathbf{160	imes	ext{ faster!}}).
  $$

### Explanation: Mastering Chapter 04 in Four Steps

1. **Internalize the physical bottleneck:** big data is usually *I/O and network bound*, not CPU bound—data locality is the first lever.
2. **Trace every stage by hand (`04-01`):** follow `InputSplit → Map → Combiner → Partitioner → Shuffle → Reduce` on small word-count examples until data movement is crystal clear.
3. **Spot shuffle bottlenecks early:** whenever keys are heavily skewed (one "hot key" holds 80% of records), a single Reducer stalls the entire job.
4. **Connect classic MapReduce to 2026 lakehouses (`04-02`):** recognize how Spark, Iceberg compaction, and BigQuery inherit the exact same execution principles.

**Quick check.** In our 100-node local-disk calculation, if we double the cluster to **200 nodes** (50 GB per node), what happens to the local read time? It halves to **250 seconds (~4.2 minutes)**—demonstrating linear horizontal scaling!

## The Paradigm Shift

MapReduce simplified distributed computing by abstracting the complexities of parallelization, fault tolerance, data distribution, and load balancing. Programmers simply define a `Map` function (to process data) and a `Reduce` function (to aggregate results), and the framework handles the rest.

## Optional application lesson

Required theory in this chapter is unchanged. For lakehouse batch, Beam/Dataflow, and modern shuffle-on-object-storage (2022–2026), see [04-02 Modern Batch Processing beyond Classic MapReduce]({{ site.baseurl }}{% multilang_post_url contents/chapter04/21-01-01-04_02_Modern_Batch_and_MapReduce %}).

## Video tham khảo

- **xem trước:** [MapReduce - Computerphile](https://www.youtube.com/watch?v=cvhKoniK5Uo) — Computerphile, English. Watch first: Computerphile’s MapReduce explanation, the paradigm shift this introduction names.
- [What is MapReduce?](https://www.youtube.com/watch?v=43fqzaSH0CQ) — internet-class, English. A second, shorter definition of MapReduce before the word-count lesson.
- [What is Apache Spark?](https://www.youtube.com/watch?v=p8FGC49N-zM) — Databricks, English. The optional lesson is batch after classic MapReduce; this is Databricks’ short answer to what Spark is.
