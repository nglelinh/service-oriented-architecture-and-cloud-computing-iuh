---
layout: post
title: 03-02 Event-Driven Models, gRPC, and Cluster Schedulers (2022–2026)
chapter: '03'
order: 3
owner: Nguyen Le Linh
lang: en
categories:
- chapter03
lesson_type: optional
---

The required notes contrast client–server, peer-to-peer, and event-driven models, then introduce YARN as a cluster operating system. This optional lesson shows how those *computing models* look in 2022–2026 service architectures: **gRPC** for synchronous RPC, **CloudEvents** for asynchronous contracts, and **Kubernetes** as the scheduler that largely succeeded YARN outside Hadoop estates.

## Learning objectives

- Place gRPC and CloudEvents on the same model diagram as client–server vs event-driven.
- Explain why Spark Connect (gRPC) is a computing-model change, not only a Spark feature.
- Compare YARN ApplicationMasters with Kubernetes controllers without rewriting YARN internals.

## 1. Synchronous SOA: gRPC as the modern stub

gRPC (CNCF graduated) is HTTP/2 + Protocol Buffers + generated stubs. It is still **client–server**, but with:

- Deadlines and cancellation (first-class timeouts).
- Streaming RPCs (client, server, or bidi) that blur “request/response” and “event stream.”
- Content types that service meshes can route (Kubernetes Gateway API **GRPCRoute** moved to the Standard channel in Gateway API v1.1, May 2024).

```protobuf
syntax = "proto3";
service ResourceBroker {
  rpc Allocate (AllocateRequest) returns (AllocateReply);
  rpc WatchAllocations (WatchRequest) returns (stream AllocationEvent);
}
```

`Allocate` is classic client–server. `WatchAllocations` is an event stream *on top of* RPC. Students should be able to say which model they are in before they pick a retry policy.

## 2. Asynchronous SOA: CloudEvents and brokers

[CloudEvents](https://cloudevents.io/) (CNCF) is a common envelope (`id`, `source`, `type`, `specversion`) so producers and consumers do not invent a new JSON shape per topic. Event-driven platforms (Knative Eventing, Kafka, cloud buses) treat the envelope as the *portable* contract.

```python
event = {
    "specversion": "1.0",
    "id": "8a3c-2026-lab",
    "source": "urn:iuh:orders",
    "type": "com.iuh.order.created.v1",
    "datacontenttype": "application/json",
    "data": {"order_id": "A-19", "items": 3},
}
```

This is the required lesson’s event-driven model with a **standard name** for the message. Idempotency keys (`id`) are how you survive independent retries—the same reliability issue YARN solved for batch tasks, now at the *service* layer.

## 3. Who is the cluster OS now?

| Concern | Hadoop YARN (required lesson) | Kubernetes (2020s default) |
| --- | --- | --- |
| Unit of work | Application / container on NM | Pod + controllers |
| Scheduler | RM + queues (Fair/Capacity) | kube-scheduler + queues/priority |
| Per-job brain | ApplicationMaster | Operator / Job / Spark Driver |
| Multi-framework | MR, Spark, Flink on one HDFS cluster | Any container on one API |

YARN did not disappear: many data platforms still schedule Spark on YARN. But CNCF’s 2024 survey (published 2025) puts **Kubernetes in production at 80%** of respondents. The *idea* you learned—decouple **resource negotiation** from **application logic**—moved to a different control plane.

**Spark Connect** (Apache Spark 3.4+, strengthened in **Spark 4.0.0**, 23 May 2025) makes the decoupling literal: the user process is a thin gRPC client; the cluster holds the Spark session. That is client–server *to a cluster OS*, not a fat driver JVM on your laptop.

## 4. A compact decision guide

1. Need an immediate answer with a deadline? **gRPC / HTTP** (client–server).
2. Need to react to a fact that already happened? **CloudEvent + broker** (event-driven).
3. Need to place CPU/GPU/memory for minutes to hours? **Scheduler** (YARN or Kubernetes)—do not hide this inside a random shell script.

<div class="content-box insight-box">
<p><strong>Anti-pattern.</strong> Using a message bus as a synchronous RPC transport (“wait for the reply event”) without timeouts recreates client–server with worse debugging. If you need RPC, use RPC.</p>
</div>

## Challenges

- Exactly-once *business* effects still need idempotent consumers; the broker only gives at-least-once in most clouds.
- gRPC across browser and public internet often needs a proxy (Connect/gRPC-Web, Gateway API).
- Running both YARN and Kubernetes in one company creates two queues, two identity models, and two sets of limits.

## Exercises

1. Classify five calls in a ride-hailing app (quote price, driver location stream, trip finished, receipt email, fraud score) as RPC vs event.
2. Write a 10-line proto for `WatchAllocations` and list two failure modes of a stream that a unary RPC does not have.
3. In one paragraph, map YARN’s ApplicationMaster onto a Kubernetes `Job` + Spark Connect driver.

## References

1. Kubernetes SIG Network, “Gateway API v1.1: Service mesh, GRPCRoute, and a whole lot more” (9 May 2024): [kubernetes.io/blog/2024/05/09/gateway-api-v1-1](https://kubernetes.io/blog/2024/05/09/gateway-api-v1-1/).
2. Apache Spark, *Spark Release 4.0.0* (23 May 2025): [spark.apache.org/releases/spark-release-4-0-0.html](https://spark.apache.org/releases/spark-release-4-0-0.html).
3. Apache Spark, *Spark Connect Overview*: [spark.apache.org/docs/4.0.0/spark-connect-overview.html](https://spark.apache.org/docs/4.0.0/spark-connect-overview.html).
4. CloudEvents specification: [cloudevents.io](https://cloudevents.io/).
5. CNCF, *Cloud Native 2024 Annual Survey*: [cncf.io/reports/cncf-annual-survey-2024](https://www.cncf.io/reports/cncf-annual-survey-2024/).
6. gRPC documentation: [grpc.io](https://grpc.io/).
