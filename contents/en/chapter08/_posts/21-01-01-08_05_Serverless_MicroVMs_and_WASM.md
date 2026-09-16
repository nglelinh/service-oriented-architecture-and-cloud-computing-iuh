---
layout: post
title: 08-05 Serverless, MicroVMs, and WebAssembly Applications (2022–2026)
chapter: '08'
order: 6
owner: Nguyen Le Linh
lang: en
categories:
- chapter08
lesson_type: optional
---

The required lessons already cover hypervisors, Linux namespaces/cgroups, Docker, and AWS Lambda. This optional note tracks what changed *around* those primitives in 2022–2026: **Firecracker snapshots** (Lambda SnapStart), **container-image functions**, and **WebAssembly** as a finer isolation/speed bet—plus why CNCF surveys show serverless adoption is *split*, not uniformly rising.

## Learning objectives

- Explain SnapStart as a snapshot of an initialized Firecracker microVM, not a new programming model.
- Place WASM/WASI next to containers: smaller isolate, different syscall story.
- Decide when FaaS, Knative, or a always-on Deployment is the right SOA boundary.

## 1. SnapStart: attacking cold start without rewriting the app

AWS introduced Lambda SnapStart for **Java** (28 November 2022), then **Python and .NET** (GA 2024), and later **container-image** functions (AWS What’s New, July 2026). Mechanism (AWS docs): on publish, Lambda initializes the environment, takes an encrypted **Firecracker microVM snapshot** of memory and disk, and resumes from that snapshot on invoke/scale-out.

```python
# Application concern: snapshot uniqueness (IDs, connections, entropy)
import uuid
import socket

# BAD if SnapStart caches this at init:
# NODE_ID = uuid.uuid4()

def handler(event, context):
    node_id = str(uuid.uuid4())  # generate per invoke, not per init
    # Re-open connections after restore; do not reuse a snapped socket.
    return {"id": node_id, "host": socket.gethostname()}
```

This is still **virtualization** (required lesson): a microVM, not a namespace-only container. The application lesson is *when* uniqueness and connections are created.

## 2. WASM: isolation without a guest kernel

Projects such as Fermyon **Spin**, Wasmtime, and WASI aim for millisecond starts and a capability-oriented sandbox. The pitch versus Docker: ship a module, not an OS userspace. The catch: WASI I/O, debugging, and team skills are uneven compared with Linux containers. Treat WASM as a **third isolate** (VM / container / wasm), not as “Docker is dead.”

CNCF’s 2024 survey (published 2025) reports serverless use **split**: some organizations expand, others retreat on cost and complexity. That matches student labs where a Lambda + NAT + VPC + cold start is slower to operate than a small Deployment.

## 3. Application patterns

| Pattern | Mechanism | 2020s example |
| --- | --- | --- |
| Burst API | FaaS + SnapStart / provisioned concurrency | Checkout webhooks |
| Scale-to-zero HTTP | Knative / Cloud Run | Internal tools |
| Sidecar-less isolate | WASM on edge or Spin | Auth at the CDN |
| Always-on GPU | Not FaaS-first | vLLM on Kubernetes |

<div class="content-box insight-box">
<p><strong>SOA reminder.</strong> Serverless is a <em>packaging and billing</em> choice. Your service contract (timeouts, idempotency, auth) stays the same as a containerized microservice.</p>
</div>

## 4. Mini architecture: event-driven + gRPC core

A 2024–2026 shape that students will see:

1. API Gateway / Function for *spiky* north–south HTTP.
2. Always-on gRPC workers for *chatty* east–west calls (mesh or ambient).
3. Object storage + queue for payloads that must not live in the event body.

Do not put a 30-second LLM generate inside an API Gateway timeout without an async job id.

## Challenges

- SnapStart uniqueness bugs (certificates, RNGs, connection pools).
- WASM ecosystem lock-in (host APIs differ).
- Cost: per-ms billing vs. a 1-replica Deployment that is cheaper at steady QPS.

## Exercises

1. List three init-time objects that must be rebuilt after a SnapStart restore.
2. Compare a 128 MB Lambda vs. a 1-replica container for a 20 RPS CPU API (qualitative is fine).
3. Sketch where you would use WASM at the edge vs. Firecracker in the region.

## References

1. AWS, “Improving startup performance with Lambda SnapStart”: [docs.aws.amazon.com/lambda/latest/dg/snapstart.html](https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html).
2. AWS News Blog, SnapStart for Python and .NET GA: [aws.amazon.com/blogs/aws/aws-lambda-snapstart-for-python-and-net-functions-is-now-generally-available](https://aws.amazon.com/blogs/aws/aws-lambda-snapstart-for-python-and-net-functions-is-now-generally-available/).
3. AWS What’s New, SnapStart for container image functions (July 2026): [aws.amazon.com/about-aws/whats-new/2026/07/aws-lambda-snapstart-container](https://aws.amazon.com/about-aws/whats-new/2026/07/aws-lambda-snapstart-container/).
4. Fermyon Spin: [fermyon.com/spin](https://www.fermyon.com/spin).
5. CNCF *Cloud Native 2024 Annual Survey* (serverless split): [cncf.io/reports/cncf-annual-survey-2024](https://www.cncf.io/reports/cncf-annual-survey-2024/).
6. Firecracker: [firecracker-microvm.github.io](https://firecracker-microvm.github.io/).
