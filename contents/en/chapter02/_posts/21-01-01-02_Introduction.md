---
layout: post
title: 02 Introduction to Distributed Systems
chapter: '02'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter02
---


This chapter introduces the fundamental concepts of Distributed Systems, which form the invisible backbone of modern cloud computing and big data processing. Before we can understand the cloud, we must understand the systems that make it possible.

## Learning Objectives

In this chapter, we will:
- Define what a Distributed System is and, crucially, what it isn't.
- Understand the key characteristics that make them unique: concurrency, the lack of a global clock, and independent failures.
- Compare centralized vs. distributed architectures to see why we moved away from mainframes.
- Explore basic design issues including Naming, Communication, and Reliability.
- Analyze real-world examples from the Local Area Networks (LANs) in your office to the massive Cloud Clusters that power the internet.

Every time you tap "Place Order" on a shopping app, your phone triggers an invisible orchestra of dozens of independent machines across multiple data centers—any one of which can stall, reboot, or drop a packet mid-flight. Chapter 02 strips away the illusion of a single giant computer and introduces the exact laws of **Distributed Systems** and **Modern Observability** that keep global services alive despite constant partial failures.

## Key ideas: Core Pillars of Distributed Systems

1. **Concurrency & Independent Failures:** Unlike a single server where components live or die together, distributed nodes fail independently—making timeouts, retries, idempotency, and circuit breakers mandatory architectural building blocks.
2. **Time, Ordering, and Consistency (CAP / PACELC):** Without a shared hardware clock, systems rely on logical clocks and consensus protocols, continuously balancing *strong consistency* against *low-latency availability* across network partitions.
3. **Observability as a First-Class Requirement:** Once work spans multiple machines, logs alone cannot reconstruct a user's journey; distributed traces (`trace_id`), metrics, and SLO error budgets form the nervous system of production operations.

### Worked Example (Step by Step): The Latency Cost of Cross-Region Consistency

Suppose a global e-commerce database runs two replicas—one in Singapore and one in Frankfurt (round-trip network latency $$120	ext{ ms}$$):
- **Local asynchronous write (eventual consistency):** client writes to Singapore replica → disk flush takes **$$2	ext{ ms}$$** → client receives `200 OK` immediately (replication happens in the background).
- **Cross-region synchronous write (strong consistency):** Singapore must forward the write to Frankfurt and wait for an `ACK` before replying → total write latency jumps to $$2 + 120 + 2 = \mathbf{124	ext{ ms}}$$ (**62× slower**!).
That single 62× gap explains why modern cloud architects reserve strong cross-region consistency for ledgers/inventory while letting product catalogs and session caches use fast eventual consistency.

### Explanation: How Chapter 02 Prepares You for the Rest of the Course

1. **Master the three defining traits (`02-01`):** concurrency, lack of a global clock, and independent failures reappear in YARN, MapReduce, Spark, Kafka, and NoSQL.
2. **Treat network latency as a physical law:** light in fiber travels at ~200,000 km/s, so cross-ocean round trips always cost 100–250 ms regardless of how fast your CPU is.
3. **Pair theory with modern operations (`02-02`):** OpenTelemetry tracing and chaos engineering turn distributed failure modes from textbook definitions into daily debugging skills.
4. **Think in trade-offs, never absolutes:** no distributed system maximizes consistency, availability, and latency simultaneously—great design makes the trade-off explicit.

**Quick check.** If an API must answer within a **50 ms** p99 latency SLA, can it perform a synchronous quorum write across Singapore and Frankfurt (120 ms RTT)? No—physics alone forbids it! You must keep quorum replicas within regional Availability Zones (1–2 ms RTT) or replicate asynchronously across continents.

## Why Distributed Systems?

Single machines have reached their physical limits (vertical scaling). To handle the staggering scale of modern internet applications—serving billions of users and processing petabytes of data—we have no choice but to coordinate thousands of machines working together (horizontal scaling). This chapter explains the theoretical and practical foundations of how we achieve this coordination.

## Optional application lesson

Required theory in this chapter is unchanged. For OpenTelemetry, traces, and chaos-style operations (2022–2026), see [02-02 Observability and Modern Distributed Applications]({{ site.baseurl }}{% multilang_post_url contents/chapter02/21-01-01-02_02_Observability_and_Distributed_Applications %}).

## Video tham khảo

- **xem trước:** [Explaining Distributed Systems Like I'm 5](https://www.youtube.com/watch?v=CESKgdNiKJw) — HashiCorp, an IBM Company, English. Watch first: a short account of why one machine is not the system, before the chapter’s design issues.
- [Distributed Systems in One Lesson by Tim Berglund](https://www.youtube.com/watch?v=Y6Ev8GIlbxc) — Devoxx Poland, English. Tim Berglund’s full lecture on distributed-systems design. It is long; watch the whole talk, there is no separate timestamp.
