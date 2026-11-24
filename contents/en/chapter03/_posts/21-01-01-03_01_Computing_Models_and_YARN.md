---
layout: post
title: 03-01 Computing Models and Hadoop YARN
chapter: '03'
order: 2
owner: Nguyen Le Linh
lang: en
categories:
- chapter03
lesson_type: required
---

Imagine ten teams at a bank all wanting to run different jobs—SQL reports, fraud-detection ML, and nightly ETL—on the exact same 200-server Hadoop cluster. In early Hadoop 1.0, MapReduce owned *both* cluster scheduling and job execution, forcing every non-MapReduce workload to sit on separate idle hardware! **YARN** (*Yet Another Resource Negotiator*) solved that crisis by splitting **cluster resource management** from **application execution**—becoming the first true distributed operating system for big data.

## Background Knowledge Map: Computing Models, HDFS Storage, and YARN Scheduling

1. **Distributed Computing Models:** *Client–Server* centralizes state behind request/response APIs; *Peer-to-Peer (P2P)* distributes load symmetrically across equal nodes; *Event-Driven* decouples producers and consumers through asynchronous message brokers.
2. **HDFS (Write-Once, Read-Many Distributed Storage):** Splits multi-terabyte files into large blocks (typically **128 MB**) replicated **3×** across racks—optimizing sequential throughput and tolerating disk/rack loss automatically.
3. **YARN (Decoupled Cluster OS):** A central **ResourceManager** allocates CPU/RAM containers across **NodeManagers**, while each submitted job spawns its own **ApplicationMaster** to negotiate containers and monitor tasks—letting MapReduce, Spark, and Flink share one physical cluster.

### Worked Example (Step by Step): HDFS Block Replication & YARN Container Packing Math

Suppose you store a **1 TB** ($$1{,}024	ext{ GB}$$) clickstream dataset in HDFS with **128 MB** ($$0.128	ext{ GB}$$) blocks and replication factor **3**, then process it on a **4-node YARN cluster** (64 GB RAM per node = **256 GB** total):
1. **HDFS block count:** $$\dfrac{1{,}024	ext{ GB}}{0.128	ext{ GB/block}} = \mathbf{8{,}000	ext{ logical blocks}}$$.
2. **Physical disk footprint (3× replication):** $$8{,}000 	imes 3 = \mathbf{24{,}000	ext{ block replicas}} \implies \mathbf{3	ext{ TB}}$$ of raw disk space across racks.
3. **YARN container allocation:** reserve 1 container of **4 GB** for the job's `ApplicationMaster`, leaving $$256 - 4 = 252	ext{ GB}$$ for worker containers. If each worker container requests **8 GB RAM**, YARN can schedule:
   $$
   \left\lfloor rac{252}{8} ightfloor = \mathbf{31	ext{ parallel worker containers}}
   $$
   (with 4 GB unallocated fragmentation).

### Deep-Dive Explanation: Why Separating Storage, Scheduling, and Compute Changed Big Data (Read Slowly)

1. **Large HDFS blocks minimize NameNode RAM pressure:** storing 1 TB in 4 KB OS blocks would require 268 million metadata entries, crashing the NameNode; 128 MB blocks shrink metadata to just 8,000 entries!
2. **Rack-aware 3× replication balances locality and safety:** HDFS places Replica 1 locally, Replica 2 on a different rack, and Replica 3 on a different node in that second rack—surviving a full rack switch outage while keeping cross-rack traffic low.
3. **YARN's ApplicationMaster prevents scheduler bottlenecks:** instead of one master tracking 100,000 task states, each job's `ApplicationMaster` handles its own task retries and speculation, letting the `ResourceManager` focus purely on fair capacity allocation.
4. **See YARN as Kubernetes' conceptual ancestor:** ResourceManager ↔ kube-scheduler, NodeManager ↔ kubelet, YARN Container ↔ Kubernetes Pod!

**Quick check.** If a cluster has **10 TB** of raw disk space and HDFS replication factor is **3**, what is the maximum *usable* dataset size it can store? $$\dfrac{10	ext{ TB}}{3} pprox \mathbf{3.33	ext{ TB}}$$ (and in practice ~2.8 TB leaving 15% headroom for temporary shuffle files!).

## Distributed Computing Models

### 1. Client-Server Model
- **Structure**: Clients request services, servers provide them.
- **Examples**: Web applications, database servers, file servers.
- **Pros**: Centralized control, easier maintenance.
- **Cons**: Single point of failure (server), scalability bottlenecks.

### 2. Peer-to-Peer (P2P) Systems
- **Structure**: All nodes are equal (peers) and share resources directly.
- **Examples**: BitTorrent, Blockchain (Bitcoin/Ethereum).
- **Pros**: High scalability, no single point of failure.
- **Cons**: Difficult management, security challenges.

### 3. Multi-Tier Architecture
- **Structure**: Split efficient processing into layers (tiers).
  - Presentation Tier (UI)
  - Application Tier (Logic)
  - Data Tier (Database)
- **Examples**: Modern web applications (React + Node.js + MongoDB).

### 4. Thin Clients & Compute Servers
- **Thin Client**: Lightweight device relying on a server (e.g., Chromebooks, VDI).
- **Compute Server**: Powerful backend processing simple requests.

### 5. Event-Driven Architecture
- **Structure**: Asynchronous communication based on events/messages.
- **Examples**: IoT systems, real-time analytics, serverless functions.

## Introduction to Hadoop

**Apache Hadoop** is a framework that allows for the distributed processing of large data sets across clusters of computers using simple programming models.

### Key Characteristics
- **Scalable**: From single servers to thousands of machines.
- **Fault-Tolerant**: Handles hardware failures automatically.
- **Shared Nothing Architecture**: Each node is independent.
- **Data Locality**: Move computation to data, not data to computation.

### Hadoop Core Components

1. **HDFS** (Hadoop Distributed File System): Storage layer.
2. **YARN** (Yet Another Resource Negotiator): Resource management layer.
3. **MapReduce**: Data processing model.
4. **Hadoop Common**: Utilities.

## HDFS Architecture

**HDFS** is designed for storing very large files with streaming data access patterns, running on clusters of commodity hardware.

- **NameNode (Master)**: Manages metadata (file names, permissions, location of blocks).
- **DataNode (Worker)**: Stores actual data blocks.
- **Block Size**: Default 128MB. Large blocks minimize seek time.
- **Replication**: Default 3 replicas (for fault tolerance).

### Fault Tolerance
- **Heartbeats**: DataNodes send signals to NameNode every 3 seconds.
- **Re-replication**: If a DateNode fails, NameNode schedules replication of its blocks to other nodes.
- **Rack Awareness**: Places replicas on different racks to survive rack failures.

## Hadoop YARN

**YARN** decoupled the resource management and job scheduling capabilities from the original MapReduce.

### Architecture

1. **ResourceManager (RM)**: Global master that arbitrates resources among all applications in the system.
2. **NodeManager (NM)**: Per-machine agent responsible for containers, monitoring their resource usage (CPU, memory, disk, network) and reporting to the RM.
3. **ApplicationMaster (AM)**: Per-application library that negotiates resources from the RM and works with the NM to execute and monitor the tasks.
4. **Container**: Abstract notion of resources (memory, cpu, disk, network).

### How YARN Works

1. Client submits an application.
2. ResourceManager allocates a container to start the ApplicationMaster.
3. ApplicationMaster asks ResourceManager for more containers.
4. ApplicationMaster contacts NodeManagers to start tasks in allocated containers.

### Schedulers
- **FIFO Scheduler**: First In, First Out. Simple but not efficient for shared clusters.
- **Capacity Scheduler**: Designed for multi-tenancy. Queues get a guaranteed capacity.
- **Fair Scheduler**: Assigns resources so applications get an equal share of resources over time.

## Summary

- **Computing Models** (Client-Server, P2P) define how distributed components interact.
- **Hadoop** revolutionized big data by combining storage (HDFS) and processing on commodity hardware.
- **YARN** enables Hadoop to support multiple processing engines (MapReduce, Spark, Tez) by separating resource management from execution logic.

## Video tham khảo

- **xem trước:** [Event-Driven Architecture(EDA) vs. Request/Response (RR): System Design Explained](https://www.youtube.com/watch?v=7fkS-18KBlw) — Confluent, an IBM Company, English. Watch first for event-driven versus request/response, the computing-model half of the lesson before Hadoop.
- [What is Apache Hadoop?](https://www.youtube.com/watch?v=JWX5Inb--ig) — IBM Technology, English. What Hadoop is, including the stack this lesson then splits into HDFS and YARN.
- [How Apache Hadoop YARN Works?](https://www.youtube.com/watch?v=GCFCohlmWgs) — HandsonERP, English. How YARN accepts an application and hands out containers, matching the architecture and scheduler sections.
