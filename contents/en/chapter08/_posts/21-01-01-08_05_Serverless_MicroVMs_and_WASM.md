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

For a decade, cloud engineers lived with an uncomfortable fork in the road: full virtual machines gave you ironclad hardware isolation but took tens of seconds to boot and burned gigabytes of RAM, while Linux containers booted in milliseconds but shared a single host kernel—meaning one kernel bug could break multi-tenant isolation. Between 2022 and 2026, two new execution layers broke that trade-off wide open: **MicroVMs** (like AWS Firecracker) stripped virtual machines down to a minimal hardware-enforced hypervisor, while **WebAssembly (WASM/WASI)** brought millisecond, language-level sandboxing straight to the network edge.

## Background Knowledge Map: The Modern Isolation Spectrum

### 1. Hardware Virtualization and Minimalist MicroVMs
Traditional hypervisors emulate legacy PC hardware (BIOS, floppy controllers, PCI buses) that cloud workloads never touch. A **MicroVM** uses Linux KVM hardware virtualization (Intel VT-x / AMD-V) with a tiny, purpose-built virtual machine monitor (VMM) exposing only virtio network and block devices. Each tenant still gets its own guest kernel—preserving VM-grade security—while slashing boot times from ~20 seconds down to ~125 milliseconds.

### 2. Shared-Kernel Containers vs. Capability-Based WASM Sandboxes
Standard containers rely on Linux namespaces (visibility isolation) and cgroups (resource quotas) enforced by one shared kernel. **WebAssembly (WASM)** takes a fundamentally different route: code compiles to a portable, memory-safe bytecode with a linear memory array and zero ambient authority. Under **WASI (WebAssembly System Interface)**, a module cannot open a socket or read a file unless the host runtime explicitly hands it that capability.

### 3. Matching Isolation Overhead to Workload Lifetimes
A long-lived database cares little about a 15-second boot time, whereas an edge authentication filter running for 3 milliseconds cannot afford even a 100 ms cold start. Modern cloud platforms therefore stack all three layers: VMs for infrastructure nodes, MicroVMs for multi-tenant serverless functions, and WASM isolates for ultra-low-latency edge compute.

### Deep-Dive Explanation: Hand-Calculating Host Density and Startup Overhead Across Three Runtimes (Read Slowly)

Suppose you operate a bare-metal host with **64 GiB (65,536 MiB)** of usable RAM dedicated to running idle-ready multi-tenant function sandboxes. Compare the theoretical packing density and cold-start penalty of three isolation technologies:

| Runtime Layer | Per-Instance Overhead | Typical Cold Start | Max Empty Sandboxes on 64 GiB Host | Security Boundary |
| :--- | :---: | :---: | :---: | :--- |
| **Traditional VM** | 512 MiB | ~20,000 ms | $$65{,}536 / 512 = 128$$ | Dedicated guest kernel (KVM) |
| **Firecracker MicroVM** | 5 MiB | ~125 ms | $$65{,}536 / 5 \approx 13{,}107$$ | Dedicated guest kernel (KVM) |
| **WASM / WASI Isolate** | 1 MiB | ~1 ms | $$65{,}536 / 1 = 65{,}536$$ | Linear-memory sandbox + capabilities |

Now verify the **latency overhead ratio** for a short **10 ms** edge API handler:
- On a **MicroVM** cold start: total time = $$125 + 10 = 135\text{ ms}$$ (the startup is **12.5×** longer than the actual work!).
- On a **WASM isolate** cold start: total time = $$1 + 10 = 11\text{ ms}$$ (startup adds only **10%** overhead).

These numbers make the architectural rule of thumb obvious: use **MicroVMs** when you need arbitrary Linux binaries with full kernel isolation, and reach for **WASM** when sub-10 ms edge latency and extreme multi-tenant density dominate.

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

## Video tham khảo

- **xem trước:** [How AWS’s Firecracker virtual machines work \| Amazon Science](https://www.youtube.com/watch?v=BIRv2FnHJAg) — Amazon Science, English. Watch first: Amazon Science on how Firecracker microVMs work, the isolation change this lesson tracks around Lambda.
- [Introducing AWS Lambda SnapStart \| Amazon Web Services](https://www.youtube.com/watch?v=AgxvrZLI1mc) — Amazon Web Services, English. AWS’s short introduction of Lambda SnapStart, the snapshot feature the lesson’s first section is about.
- [Introduction to WebAssembly and WASI](https://www.youtube.com/watch?v=Of07IsplHPY) — Root Access, English. WebAssembly and WASI as isolation without a guest kernel, the lesson’s second section.

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
