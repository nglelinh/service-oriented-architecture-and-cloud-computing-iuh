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

This lecture covers fundamental distributed computing models and introduces the Hadoop ecosystem with a focus on YARN for resource management.

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
