---
layout: post
title: 04-02 Modern Batch Processing beyond Classic MapReduce (2022–2026)
chapter: '04'
order: 3
owner: Nguyen Le Linh
lang: en
categories:
- chapter04
lesson_type: optional
---

MapReduce’s programming model (map, shuffle, reduce, re-execute on failure) is still the mental model for **batch**. What changed in 2022–2026 is *where* the shuffle lives and *which engine* runs it: Spark, Flink batch, BigQuery/Dataflow, and **open lakehouse** tables that make a batch job just another reader of the same files.

## Learning objectives

- Recognize MapReduce descendants in cloud batch services and lakehouse compaction jobs.
- Contrast “write a Mapper class” with SQL/DataFrame batch that still shuffles.
- Explain why batch did not die when streaming became popular (Lambda/Kappa, Iceberg rewrites).

## 1. The paradigm did not vanish—it got compilers

A 2025 Spark or BigQuery SQL such as `SELECT country, COUNT(*) FROM events GROUP BY country` is still MapReduce: local aggregation, shuffle by `country`, final aggregate. The required lesson’s Word Count is the same DAG with nicer syntax.

```python
# Same shuffle as WordCount; engine may be Spark 4.x or a cloud SQL warehouse
from collections import Counter

def map_line(line: str):
    return [(word.lower(), 1) for word in line.split()]

def reduce_counts(pairs):
    c = Counter()
    for k, v in pairs:
        c[k] += v
    return c
```

What *did* change: **fault tolerance** is more often “recompute a lost partition from lineage or from a table snapshot” than “re-run this Java Mapper against an HDFS split.” Object storage (S3, GCS, ADLS) plus **Apache Iceberg / Delta Lake** give you atomic commits so a failed batch does not leave a half-written Hive directory.

## 2. Cloud-native batch applications

| Pattern | 2010s Hadoop | 2022–2026 example |
| --- | --- | --- |
| Nightly ETL | MR + Oozie | Spark/Flink job on Kubernetes or EMR/Dataproc |
| Huge SQL | Hive on MR | BigQuery, Snowflake, Spark SQL on Iceberg |
| Portable pipelines | Vendor lock-in | Apache Beam runners (Dataflow, Flink, Spark) |
| Compact/sort files | `fsimage` + MR | Iceberg rewrite / Delta OPTIMIZE |

**Apache Beam** keeps the MapReduce *portability* idea: one pipeline, many runners. Google Cloud Dataflow is a managed runner; the programming model is still bounded vs unbounded collections—the same split the required notes imply when they isolate a batch job.

Netflix (Iceberg’s origin), Apple, and many banks still run **multi-hour batch** because finance close, feature backfills, and GDPR deletes are naturally bounded. Streaming did not delete those jobs; it added a second clock.

## 3. Lakehouse batch: MapReduce on open tables

Open table formats turn object storage into something a reducer can trust:

- **Snapshot isolation** — readers see a committed version (Iceberg snapshot, Delta version).
- **Hidden partitioning** — you do not hand-write `dt=2026-09-16` in every Mapper.
- **Compaction** — small files from streaming ingest are *batch-rewritten* (classic reduce-side merge).

A 2025–2026 survey of Iceberg practitioners (Data Lakehouse Hub, results posted 2026) reports Iceberg as the dominant open format among respondents, with Spark and Trino as common engines. Treat that as *community evidence*, not a global market share.

<div class="content-box insight-box">
<p><strong>Shuffle is the tax.</strong> Whether you pay it in YARN containers or Kubernetes executors, wide <code>GROUP BY</code> and joins still dominate cost. The application skill is reducing shuffle keys and file counts—not abandoning MapReduce vocabulary.</p>
</div>

## 4. Mini case: GDPR deletion as a batch reduce

1. **Map**: read user-id tombstones (small) and fact partitions (large).
2. **Shuffle**: co-group by `user_id` (or broadcast the tombstone set if it fits).
3. **Reduce**: write a new Iceberg snapshot without those rows.
4. **Fault tolerance**: if a writer dies, the previous snapshot remains readable.

That is the required lesson’s re-execution story, applied to a 2020s compliance job.

## Challenges

- Small-file explosions when streaming ingest never runs a batch compaction.
- “SQL-only” teams that cannot explain a data skew until the bill arrives.
- Exactly-once sinks that are actually “at-least-once + idempotent merge.”

## Exercises

1. Rewrite Word Count as Spark SQL and mark the shuffle boundary.
2. Estimate whether a 2 TB nightly join should use a broadcast map-side join or a shuffled reduce-side join (state a memory assumption).
3. Design a retry policy for a Beam pipeline that writes Iceberg: what is safe to rerun?

## References

1. Apache Spark 4.0.0 release (23 May 2025): [spark.apache.org/releases/spark-release-4-0-0.html](https://spark.apache.org/releases/spark-release-4-0-0.html).
2. Apache Iceberg documentation (table snapshots, rewrites): [iceberg.apache.org](https://iceberg.apache.org/).
3. Data Lakehouse Hub, “The 2025 State of the Apache Iceberg Ecosystem Results” (Feb 2026): [datalakehousehub.com](https://datalakehousehub.com/blog/2026-02-state-of-the-apache-iceberg-ecosystem).
4. Apache Beam programming guide: [beam.apache.org](https://beam.apache.org/).
5. Dean & Ghemawat, “MapReduce” (OSDI 2004) — still the paper behind every cloud batch service.
