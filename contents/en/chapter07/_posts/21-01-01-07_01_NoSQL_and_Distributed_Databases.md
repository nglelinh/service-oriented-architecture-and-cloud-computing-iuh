---
layout: post
title: 07-01 NoSQL and Distributed Databases
chapter: '07'
order: 2
owner: Nguyen Le Linh
lang: en
categories:
- chapter07
lesson_type: required
---

For thirty years, every web application started the exact same way: spin up one relational database (MySQL or PostgreSQL), normalize tables, and rely on ACID transactions. Then the internet hit a billion concurrent users—and a single primary SQL server simply could not absorb millions of writes per second or stay online across continent-wide network splits! **Distributed Databases and NoSQL** emerged to trade rigid single-node schemas for horizontal sharding, high write throughput, and explicit **CAP / BASE** consistency choices.

## Key ideas: CAP, Quorums, and the Four NoSQL Data Models

1. **CAP Theorem & BASE vs. ACID:** During a **Network Partition (P)**—which *will* happen in any real distributed network—a replicated database must choose between **Consistency (C)** (rejecting writes/reads that might be stale) and **Availability (A)** (returning a possibly stale response). BASE embraces *Basic Availability, Soft state, and Eventual consistency* to keep latency low at scale.
2. **Quorum Replication ($$R + W > N$$):** On $$N$$ replicas, requiring each write to acknowledge $$W$$ replicas and each read to consult $$R$$ replicas guarantees strong overlap whenever $$R + W > N$$—turning consistency into a tunable per-query dial!
3. **The Four NoSQL Families:** *Key-Value* (Redis, DynamoDB) for $$O(1)$$ lookups; *Document* (MongoDB) for nested JSON aggregates; *Column-Family* (Cassandra, HBase) for massive write-heavy time-series; *Graph* (Neo4j) for multi-hop relationship traversals.

### Worked Example (Step by Step): Tuning Quorum Consistency ($$R + W > N$$) on a 3-Replica Cluster

Suppose a distributed Key-Value store replicates every key across **$$N = 3$$** nodes (`Node A`, `Node B`, `Node C`):

| Quorum Setting | Overlap Check ($$R + W > N$$) | Fault Tolerance | Consistency Guarantee |
|---|---|---|---|
| **$$W = 2,\ R = 2$$** (Balanced) | $$2 + 2 = 4 > 3$$ (**Overlap $$\ge 1$$**) | Survives **1** dead node for reads & writes | **Strong** (at least 1 node in $$R$$ saw the latest write!) |
| **$$W = 3,\ R = 1$$** (Read-fast) | $$1 + 3 = 4 > 3$$ (**Overlap $$\ge 1$$**) | Reads survive 2 dead nodes; **writes fail if 1 dies!** | **Strong**, optimized for ultra-fast reads |
| **$$W = 1,\ R = 1$$** (Eventual) | $$1 + 1 = 2 \le 3$$ (**No overlap!**) | Survives **2** dead nodes | **Eventual** (fastest latency, yet can read stale data!) |

Trace a concrete failure with $$W = 2, R = 2$$: a client writes `balance = $500` (version 2) to `Node A` and `Node B` (`Node C` is lagging with `$100` v1). Any subsequent read hitting *any* 2 nodes—whether `{A, B}`, `{A, C}`, or `{B, C}`—is mathematically guaranteed to see version 2 (`$500`), and picks the higher version!

### Explanation: Picking Between ACID RDBMS and NoSQL Without Dogma

1. **Never pick NoSQL just because "SQL doesn't scale":** a modern single PostgreSQL node easily handles tens of thousands of reads/sec; adopt distributed NoSQL when your *write volume*, *multi-region availability*, or *flexible access pattern* genuinely demands horizontal sharding.
2. **Design NoSQL schemas around *queries*, not normalization:** in Cassandra or DynamoDB, you avoid distributed joins by denormalizing data into partition keys that serve each query in a single hop.
3. **Watch out for hot partitions:** hashing `order_id` spreads writes evenly across shards, whereas partitioning by `country = 'US'` dumps 80% of traffic onto one overloaded node!
4. **Remember CAP applies *only* during partitions:** when the network is healthy, PACELC reminds us the everyday trade-off is actually between **Latency (L)** and **Consistency (C)**.

**Quick check.** On a **5-replica** cluster ($$N = 5$$), if you set $$W = 3$$ and $$R = 3$$, how many simultaneous node failures can the cluster tolerate while maintaining strong quorum overlap? Since $$3 + 3 = 6 > 5$$ and $$5 - 3 = \mathbf{2}$$, it survives **2 simultaneous node failures** with strong consistency!

## Distributed Databases (DDB)

A collection of multiple, logically interconnected databases that are physically distributed over a computer network.

### Architectures
1. **Homogeneous**: Same DBMS (e.g., all Oracle) and schema at all sites. Easier to manage.
2. **Heterogeneous**: Different DBMSs or schemas. Requires middleware/integration.

### Distribution Strategies

#### 1. Replication
Stroing multiple copies of data (instances) at different sites.
- **Pros**: High availability (fault tolerance), fast local reads.
- **Cons**: Write performance suffers (must update all copies), consistency challenges.

#### 2. Fragmentation (Partitioning)
Splitting the database into fragments.
- **Horizontal Fragmentation (Sharding)**: Splitting a table by rows. (e.g., Customers ID 1-1000 in NY, 1001-2000 in London).
- **Vertical Fragmentation**: Splitting a table by columns. (e.g., Employee Name/Address in table A, Salary/Benefits in table B).

### Consistent Hashing
A technique to distribute data across nodes in a way that minimizes reorganization when nodes are added/removed. Used in DynamoDB, Cassandra, etc.

## NoSQL Databases

"NoSQL" stands for "Not Only SQL". These systems emerged to handle:
- **Big Data**: Petabytes of data.
- **Velocity**: High read/write throughput.
- **Variety**: Unstructured or semi-structured data (JSON, XML).

### CAP Theorem
In a distributed computer system, you can only provide two of the following three guarantees:
1.  **Consistency**: Every read receives the most recent write or an error.
2.  **Availability**: Every request receives a (non-error) response, without the guarantee that it contains the most recent write.
3.  **Partition Tolerance**: The system continues to operate despite an arbitrary number of messages being dropped or delayed by the network.

**Trade-offs:**
- **CA (RDBMS)**: Consistent and Available, but can't handle network partitions (distributed scale).
- **CP (MongoDB, HBase)**: Consistent and Partition Tolerant. If a partition occurs, some nodes may reject writes to ensure consistency.
- **AP (Cassandra, DynamoDB)**: Available and Partition Tolerant. Reads always succeed but might return stale data (Eventual Consistency).

### BASE Model
A softer consistency model for NoSQL systems:
- **Basically Available**: System guarantees availability.
- **Soft state**: State may change over time, even without input.
- **Eventually consistent**: System will become consistent over time.

### Types of NoSQL Databases

1.  **Key-Value Stores**
    -   **Model**: Simple Map (Key -> Value).
    -   **Use cases**: Caching, Session storage.
    -   **Examples**: Redis, Amazon DynamoDB, Riak.

2.  **Document Stores**
    -   **Model**: Store data as documents (JSON/BSON).
    -   **Use cases**: Content management, catalogs.
    -   **Examples**: MongoDB, CouchDB.

3.  **Column-Family Stores**
    -   **Model**: Store data by columns rather than rows. Optimized for writes and analytics.
    -   **Use cases**: Time-series data, Big Data analytics.
    -   **Examples**: Apache Cassandra, HBase.

4.  **Graph Databases**
    -   **Model**: Nodes and Edges.
    -   **Use cases**: Social networks, Recommendation engines.
    -   **Examples**: Neo4j.

## Summary

Distributed databases and NoSQL systems sacrifice strict ACID properties (specifically consistency) to achieve high availability and partition tolerance (scalability). Understanding the CAP theorem and the specific data model (Key-Value, Document, etc.) is crucial for choosing the right tool for the job.

## Video tham khảo

- **xem trước:** [What is CAP Theorem?](https://www.youtube.com/watch?v=eWMgsk7mpFc) — IBM Technology, English. Watch first for CAP, the theorem the lesson uses to separate consistency, availability, and partition tolerance.
- [How do NoSQL databases work? Simply Explained!](https://www.youtube.com/watch?v=0buKQHokLK8) — Simply Explained, English. How NoSQL databases differ in shape, for the key-value, document, column, and graph section.
- [Consistent Hashing \| Algorithms You Should Know #1](https://www.youtube.com/watch?v=UF9Iqmg94tk) — ByteByteGo, English. Consistent hashing, the distribution strategy the lesson names for spreading keys.
