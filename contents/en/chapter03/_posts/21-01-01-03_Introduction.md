---
layout: post
title: 03 Introduction to Computing Models and YARN
chapter: '03'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter03
---

This chapter explores various Distributed Computing Models and introduces Apache Hadoop's resource management layer, YARN.

## Learning Objectives

- Distinguish between different computing models: Client-Server, P2P, Event-Driven
- Understand the core components of the Hadoop Ecosystem
- Explain the role of HDFS (Storage) and YARN (Resource Management)
- Describe how YARN decouples resource scheduling from data processing applications

How do you coordinate a thousand computers so they behave like one coherent platform rather than a chaotic pile of laptops? Chapter 03 bridges the gap between abstract distributed-systems theory and real cluster infrastructure—contrasting **Client–Server**, **Peer-to-Peer**, and **Event-Driven** computing models before unpacking **HDFS** and **YARN** (and their modern successors gRPC and Kubernetes).

## Background Knowledge Map: Communication Topologies and Cluster Operating Systems

1. **Interaction Topologies (Client–Server, P2P, Event-Driven):** Define *who talks to whom*—centralized request/response, symmetric peer sharing, or broker-mediated publish/subscribe.
2. **Distributed Storage + Compute Co-Design (Hadoop HDFS & YARN):** Shows how commodity clusters achieve massive throughput by moving computation close to replicated data blocks while cleanly separating resource scheduling from job logic.
3. **Evolution Toward Cloud-Native SOA:** Modern architectures retain the exact same conceptual split—gRPC for synchronous RPC, CloudEvents for asynchronous messaging, and Kubernetes as the universal cluster scheduler.

### Worked Example (Step by Step): Client–Server Bottleneck vs. Peer-to-Peer Scaling

Suppose you must distribute a **10 MB** ($$80	ext{ Mb}$$) software patch to **$$N = 1{,}000$$** machines from an origin server with a **1 Gbps** ($$1{,}000	ext{ Mbps}$$) uplink:
- **Pure Client–Server:** the single server must upload $$1{,}000 	imes 80	ext{ Mb} = 80{,}000	ext{ Mb}$$. Minimum download time for the last client is:
  $$
  T_{	ext{CS}} = rac{80{,}000	ext{ Mb}}{1{,}000	ext{ Mbps}} = \mathbf{80	ext{ seconds}} \quad (	ext{scales linearly as } O(N)).
  $$
- **Peer-to-Peer (each client also uploads once it receives chunks):** total upload capacity doubles every round! Reaching 1,000 peers takes only $$\lceil \log_2(1{,}000) ceil = \mathbf{10	ext{ rounds}}$$—dramatically collapsing distribution time as $$N$$ grows!

### Deep-Dive Explanation: Reading Chapter 03 as a Blueprint for Cluster Design (Read Slowly)

1. **Match the computing model to the traffic pattern:** Client–Server excels at authoritative state; P2P excels at bulk content distribution; Event-Driven excels at decoupled fan-out.
2. **Appreciate YARN's architectural breakthrough (`03-01`):** decoupling the `ResourceManager` from job-specific `ApplicationMasters` unlocked multi-tenant big data clusters.
3. **Connect classic Hadoop directly to 2026 systems (`03-02`):** every concept in YARN maps cleanly onto Kubernetes pods, controllers, and gRPC/CloudEvents microservices.
4. **Watch where state lives:** moving stateless compute is easy; coordinating replicated storage (HDFS, object stores) is always the harder half.

**Quick check.** If $$N$$ increases 10× to **10,000 clients**, Client–Server delivery time jumps 10× (to **800 seconds**), whereas P2P rounds grow from $$\log_2(1{,}000) pprox 10$$ to $$\log_2(10{,}000) pprox \mathbf{14}$$ rounds—only 4 extra rounds!

## Why YARN?

In the early days of Hadoop (v1), MapReduce was the only processing engine. YARN (Yet Another Resource Negotiator) was introduced in Hadoop v2 to create a general-purpose cluster operating system, allowing multiple applications (MapReduce, Spark, Flink) to run on the same shared cluster resources.

## Optional application lesson

Required theory in this chapter is unchanged. For gRPC, CloudEvents, and Kubernetes-as-cluster-OS (2022–2026), see [03-02 Event-Driven Models, gRPC, and Cluster Schedulers]({{ site.baseurl }}{% multilang_post_url contents/chapter03/21-01-01-03_02_Event_Driven_and_gRPC %}).

## Video tham khảo

- **xem trước:** [What is Apache Hadoop?](https://www.youtube.com/watch?v=JWX5Inb--ig) — IBM Technology, English. Watch first for what Hadoop is, since this chapter introduction is about YARN as the cluster layer on that stack.
- [How Apache Hadoop YARN Works?](https://www.youtube.com/watch?v=GCFCohlmWgs) — HandsonERP, English. A short explanation of how YARN schedules work, which is the “why YARN” section of the introduction.
