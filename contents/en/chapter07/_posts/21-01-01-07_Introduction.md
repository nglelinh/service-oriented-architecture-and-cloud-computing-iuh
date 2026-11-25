---
layout: post
title: 07 Introduction to NoSQL and Distributed Databases
chapter: '07'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter07
---

This chapter moves beyond traditional Relational Database Management Systems (RDBMS) to explore the world of NoSQL and Distributed Databases, designed to handle the scale, velocity, and variety of modern big data.

## Learning Objectives

- Explain the limitations of monolithic RDBMS in distributed environments
- Understand the CAP Theorem and consistency trade-offs (BASE vs ACID)
- Explore the four main types of NoSQL databases: Key-Value, Document, Column-family, and Graph
- Analyze distributed database concepts: Sharding, Replication, and Consistent Hashing

Choosing the wrong database is the hardest architectural mistake to undo: rewriting a stateless microservice takes a week, while migrating 50 terabytes of live customer data without downtime takes a year. Chapter 07 gives you a rigorous framework for navigating the modern storage universe—from **CAP and ACID vs. BASE**, across the **four classic NoSQL families**, all the way to **Serverless SQL and Vector Databases**.

## Key ideas: Matching Data Access Patterns to Storage Engines

1. **Consistency vs. Availability Trade-Offs (CAP / Quorums):** Every distributed store chooses how it behaves under network splits and how many replicas must agree before acknowledging a write.
2. **The Four NoSQL Families:** Key-Value, Document, Column-Family, and Graph engines optimize radically different access patterns by changing how data is laid out on disk and sharded across nodes.
3. **Modern Convergence (Serverless, Distributed SQL, Vectors):** Today's systems blend elastic serverless compute, Raft-backed distributed ACID, and HNSW semantic vector indexes for AI workloads.

### Worked Example (Step by Step): Selecting the Right Database for Four E-Commerce Workloads

Match four concrete requirements inside one online marketplace to the optimal storage family:

| Workload | Access Pattern | Best-Fit Family | Why? |
|---|---|---|---|
| **1. User shopping-cart session** | Lookup by `session_id` in < 2 ms | **Key-Value** (Redis/DynamoDB) | Pure $$O(1)$$ hash lookup, simple TTL expiration |
| **2. Diverse product catalog** | Nested attributes vary by category (shoes vs. laptops) | **Document** (MongoDB) | Flexible JSON schema per item, single-document reads |
| **3. Clickstream / IoT telemetry** | 500,000 append-only writes/sec by `(device_id, time)` | **Column-Family** (Cassandra) | LSM-tree sequential writes scale linearly horizontally |
| **4. Fraud ring detection** | *"Find accounts sharing a card within 3 hops"* | **Graph** (Neo4j) | Index-free adjacency traverses multi-hop edges in ms |

### Explanation: How to Navigate Chapter 07 Like a Data Architect

1. **Start from access patterns, not hype:** write down your top 5 queries, read/write ratio, latency SLA, and consistency requirement *before* naming a database product.
2. **Master CAP and quorums (`07-01`) first:** understanding $$R + W > N$$ lets you reason about Cassandra, DynamoDB, and Raft-based SQL from first principles.
3. **Embrace polyglot persistence selectively:** use one primary database as long as possible; add a specialized store (Redis cache, Graph DB, Vector index) only when a concrete bottleneck justifies the extra operational surface.
4. **Stay current with 2022–2026 engines (`07-02`):** know when `pgvector` or Serverless SQL solves the problem cleanly without standing up three separate clusters.

**Quick check.** Why is a relational `JOIN` across 4 hops in a 10-million-node social graph much slower than a native Graph database? Because relational B-tree joins multiply intermediate row lookups at every hop, whereas a Graph DB follows direct memory pointers from each vertex to its neighbors!

## The Data Evolution

As applications scaled to millions of users, the vertical scaling limits of SQL databases became a bottleneck. This led to the emergence of distributed databases that sacrifice some consistency guarantees for partition tolerance and high availability.

## Optional application lesson

Required theory in this chapter is unchanged. For serverless databases, distributed SQL, and vector/RAG stores (2022–2026), see [07-02 Serverless, Distributed SQL, and Vector Databases]({{ site.baseurl }}{% multilang_post_url contents/chapter07/21-01-01-07_02_Serverless_and_Vector_Databases %}).

## Video tham khảo

- **xem trước:** [What is CAP Theorem?](https://www.youtube.com/watch?v=eWMgsk7mpFc) — IBM Technology, English. Watch first for the CAP theorem, the limit this introduction gives for leaving a single-node RDBMS.
- [SQL vs. NoSQL: What's the difference?](https://www.youtube.com/watch?v=Q5aTUc7c4jg) — IBM Technology, English. SQL versus NoSQL, the data-evolution story the introduction sketches.
