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

As applications scale, traditional relational databases (RDBMS) often become bottlenecks. This lecture explores Distributed Databases and NoSQL systems designed for scale and flexibility.

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
