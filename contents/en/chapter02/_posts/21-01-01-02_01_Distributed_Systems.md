---
layout: post
title: 02-01 Distributed Systems Fundamentals
chapter: '02'
order: 2
owner: Nguyen Le Linh
lang: en
categories:
- chapter02
lesson_type: required
---


A distributed system consists of independent computers that communicate to achieve a common goal. This lecture covers the fundamental concepts, characteristics, and design issues of distributed systems.

## What is a Distributed System?

### Definition and Core Concept
Understanding the precise nature of a distributed system is the first step in mastering cloud computing. A widely accepted definition states that **"a distributed system is one in which components located at networked computers communicate and coordinate their actions only by passing messages."**

This seemingly simple definition has profound implications. First, it implies the **concurrency of components**, meaning that multiple processes are executing simultaneously across the network. Second, it highlights the **lack of a global clock**. In a distributed environment, keeping time synchronized between machines is a notoriously difficult computer science problem, making coordination a challenge. Finally, it introduces the reality of **independent failures**. Unlike a monolithic system where a crash often takes down everything, in a distributed system, one component can fail completely while others continue to function normally.

Leslie Lamport, a Turing Award winner and pioneer in the field, offered a more tongue-in-cheek but equally accurate definition: *". . . a system in which the failure of a computer you didn’t even know existed can render your own computer unusable."*

### Centralized vs. Distributed Architectures
To appreciate the shift to distributed systems, it is helpful to contrast them with traditional centralized systems.

In a **Centralized System**, all calculations are performed by a single computer (like a mainframe). Resources are shared and accessible to users at all times (as long as the system is up). There is a single process control and, crucially, a **single point of failure**. If the main computer goes down, the entire system stops.

In contrast, a **Distributed System** is composed of multiple autonomous components. Resources may not always be accessible if a network link fails. Processing is concurrent, occurring on different processors simultaneously. Control is decentralized, meaning there are multiple points of control and, consequently, **multiple points of failure**. While this adds complexity, it also adds resilience, as the failure of one node does not necessarily doom the entire system.

### Why Go Distributed?
Despite the added complexity, there are compelling reasons to adopt distributed architectures:
1.  **Inherent Distribution**: Some applications are naturally distributed. For example, a messaging app on two phones requires a system that bridges the physical distance between them.
2.  **Reliability**: By eliminating the single point of failure, distributed systems can offer higher availability. If one server crashes, others can take over its workload.
3.  **Performance**: We can optimize performance by accessing data from a nearby node (reducing latency) or by executing tasks in parallel across massive clusters (increasing throughput).
4.  **Scale**: This is the primary driver for modern "Big Data" systems. Many problems are simply too large for any single machine to hold in memory or process in a reasonable amount of time.

## Examples of Distributed Systems

Distributed systems are ubiquitous in modern technology. **Local Area Networks (LANs)** connect computers within an office. **Database Management Systems** often run across multiple servers to handle large datasets. The global network of **Automatic Teller Machines (ATMs)** is a distributed system that coordinates financial transactions.

The most famous example is the **World Wide Web (WWW)** itself, a massive distributed system of clients and servers sharing resources via URLs. **Mobile and Ubiquitous Computing** extends this concept to devices moving through physical space. And, of course, **Cloud Computing Clusters** (like those powering AWS, Azure, and GCP) are the pinnacle of distributed infrastructure, orchestrating millions of servers to provide utility computing. Even **Multi-player Online Games** rely on distributed systems to synchronize the state of the virtual world for thousands of players simultaneously.

### Case Study: Scaling Facebook
The evolution of Facebook provides a clear illustration of why distributed systems are necessary:
-   **2004**: It started as a single server handling both the web traffic and the database.
-   **Separation**: As traffic grew, they separated the web server from the database server. While this increased capacity, the system would still go offline if either server failed, and there was no redundancy.
-   **Partitioning**: They began to deploy pairs of servers for specific communities (e.g., one pair for Harvard, one for Yale). This worked until friends from different schools wanted to connect, revealing the problem of isolated partitions.
-   **Horizontal Scaling**: Today, Facebook uses a massive distributed architecture. The **Front-end** consists of a scalable number of stateless web servers, with load balancers distributing user traffic. The **Back-end** uses sharded databases and sophisticated caching layers, managed by complex distributed systems code to ensure billions of users see a consistent view of their news feed.

## Key Characteristics

When assessing the quality and robustness of a distributed system, architects evaluate several key characteristics:

1.  **Heterogeneity**: A robust system can operate over a variety of different networks, hardware, operating systems, and programming languages.
2.  **Openness**: The system should be extendable. This is often achieved through published interfaces (APIs) that allow new components to be added without rewriting existing ones.
3.  **Security**: The system must guarantee the confidentiality, integrity, and availability of data, which is harder when data is moving across a network.
4.  **Scalability**: The system should be able to handle growing loads by simply adding more resources, rather than redesigning the architecture.
5.  **Failure Handling**: The system must be able to detect failures, mask them (so the user doesn't notice), and recover automatically.
6.  **Concurrency**: The system must handle multiple operations executing simultaneously without corrupting data.
7.  **Transparency**: Ideally, the complexity of the distribution should be hidden from the user and the application programmer. The system should appear as a single coherent entity.

## Basic Design Issues

Designing a distributed system involves solving several fundamental problems.

### 1. Naming
How do we identify resources in a vast network? A **Naming Context** is required to resolve user-friendly names to machine-readable identifiers. For example, the Domain Name System (DNS) translates `google.com` into an IP address. Systems also rely on **Unique Identifiers** like URIs (Uniform Resource Identifiers) or UUIDs (Universally Unique Identifiers) to distinguish resources globally.

### 2. Communication
How do components talk to each other? The fundamental primitive is **Message Passing** (Send/Receive). Communication can be **Synchronous (Blocking)**, where the sender waits for a response before continuing, or **Asynchronous (Non-blocking)**, where the sender basically says "fire and forget," handling the response later. Common patterns built on top of these include Client-Server interactions (like RPC), Group Multicast, and Publish-Subscribe systems.

### 3. Latency and Bandwidth
Two physical constraints dominate distributed system performance:
-   **Latency** is the time delay for a message to arrive. It is limited by the speed of light. Sending a message within the same building might take 1ms, but sending it from continent to continent can take 100ms. "Sneakernet" (driving hard drives in a van) has a latency of about a day.
-   **Bandwidth** is the volume of data that can be transmitted per unit of time. Modern fiber networks offer high bandwidth (Gigabits per second). Interestingly, a "station wagon full of tapes hurtling down the highway" has incredibly high bandwidth (Petabytes per day) despite its terrible latency. This classic tradeoff, famously noted by Andrew Tanenbaum, reminds us that physical transport is still sometimes the fastest way to move massive datasets.

### 4. Software Structure
Managing this complexity requires structure. We use **Layers** of abstraction to separate concerns. **Middleware** is a crucial layer of software that sits between the operating system and the application. It provides standard services—like Remote Procedure Calls (RPC) or Remote Method Invocation (RMI)—that simplify the development of distributed applications by handling the messy details of networking and coordination.

## Summary

Distributed systems are the foundation of modern computing, enabling everything from the web to the cloud. While they offer immense benefits in reliability, performance, and scale, they introduce significant complexity. Mastering the challenges of coordination, failure handling, and consistency is essential for any cloud engineer.
