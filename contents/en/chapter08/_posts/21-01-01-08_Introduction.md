---
layout: post
title: Chapter 08 - Virtualization and Containerization
chapter: '08'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter08
lesson_type: required
---

Welcome to Chapter 08, where we explore **Virtualization and Containerization** - two fundamental technologies that have revolutionized modern computing and cloud infrastructure.

Walk into an enterprise data center from the early 2000s and you would see racks of humming physical servers where each machine ran a single application—an email server here, a billing database there—sitting at 10% CPU utilization most of the day while burning electricity and cooling nonstop. **Virtualization** and **containerization** are the two waves of abstraction that turned those rigid physical boxes into fluid, programmable compute pools, laying the foundation for every modern cloud platform.

## Background Knowledge Map: How Abstraction Layers Share Physical Silicon

### 1. Type-1 vs. Type-2 Hypervisors and Hardware Virtualization
A **hypervisor** (Virtual Machine Monitor) sits between physical hardware and guest operating systems, intercepting privileged CPU instructions and multiplexing RAM, storage, and NICs. **Type-1 (bare-metal)** hypervisors like KVM, VMware ESXi, and Xen run directly on the hardware for cloud-grade efficiency, whereas **Type-2** hypervisors (VirtualBox, VMware Workstation) run on top of a host OS for desktop development.

### 2. OS-Level Virtualization: Namespaces, Cgroups, and Layered Filesystems
Containers skip emulating hardware altogether. Every container on a host shares the **same Linux kernel**; isolation comes from kernel **namespaces** (giving each container its own PID tree, network stack, and mount table) while **cgroups** cap CPU and memory usage. Copy-on-write **union filesystems** (OverlayFS) let dozens of containers share identical read-only base image layers, storing only their tiny writable diffs.

### 3. The Operational Shift from "Pets" to "Cattle"
Before container images, deploying software meant hand-configuring servers ("pets") that drifted over time until nobody dared reboot them. Immutable container images package code, runtime, and system libraries into a single content-addressed artifact so any instance can be destroyed and replaced identically in seconds.

### Deep-Dive Explanation: Hand-Calculating Server Consolidation and Memory Footprint Savings (Read Slowly)

Consider a university department running **8 physical servers**, each rated at 100% CPU capacity, but averaging only **12% CPU utilization** per server:
1. **Total aggregate compute demand:**
   $$8 \times 12\% = 96\%\text{ of a single physical server's CPU capacity!}$$
2. **Consolidating with N+1 High Availability:**
   Putting all 8 workloads onto 1 server leaves zero room for failure or spikes. Consolidating onto **2 Type-1 hypervisor hosts** puts each host at $$96\% / 2 = 48\%$$ average utilization—cutting hardware, power, and rack space by **75%** (from 8 servers down to 2) while surviving a full single-host outage!
3. **Comparing VM vs. Container RAM overhead for 10 microservices:**
   Suppose each microservice needs **100 MiB** of application RAM, and a full guest Linux OS in a VM consumes **300 MiB** of base RAM:
   - **10 separate VMs:** $$10 \times (300 + 100) = 4{,}000\text{ MiB}$$ (~3.9 GiB).
   - **10 containers on 1 host OS:** $$300 + 10 \times 100 = 1{,}300\text{ MiB}$$ (~1.27 GiB) — saving **2,700 MiB (67.5%)** of RAM simply by sharing one kernel!

## Chapter Overview

This chapter covers the evolution from traditional bare-metal servers to virtual machines and lightweight containers. You'll learn how these technologies enable efficient resource utilization, application isolation, and scalable cloud deployments.

## Learning Objectives

By the end of this chapter, you will be able to:

- **Understand virtualization concepts** including hypervisors, virtual machines, and their architecture
- **Differentiate between Type 1 and Type 2 hypervisors** and their use cases
- **Explain containerization** and how it differs from traditional virtualization
- **Work with Linux namespaces and cgroups** - the building blocks of containers
- **Use Docker** to create, manage, and deploy containerized applications
- **Understand serverless computing** and AWS Lambda fundamentals

## Topics Covered

### 1. Virtualization Fundamentals
- History and evolution of virtualization
- Virtual Machine Monitors (VMM) and Hypervisors
- Type 1 vs Type 2 virtualization
- Properties of virtual machines: partitioning, isolation, encapsulation, hardware independence

### 2. Virtualization Internals
- How virtualization works: binary translation and dynamic translation
- VM components and architecture
- AWS VM implementations (Xen, Nitro, bare metal)
- Security considerations in virtualized environments

### 3. Containerization
- Motivation for containers and lightweight virtualization
- Containers vs Virtual Machines
- Linux namespaces: PID, mount, network, UTS, user, IPC
- Control groups (cgroups) for resource management

### 4. Container Technologies
- Docker architecture and concepts
- Docker images, containers, and volumes
- Dockerfile and container creation
- Container orchestration with Kubernetes and Docker Swarm

### 5. Serverless Computing
- Introduction to serverless architectures
- AWS Lambda and function-as-a-service (FaaS)
- API Gateway integration
- Serverless application patterns

## Why This Matters

Virtualization and containerization are the backbone of modern cloud computing:

- **Cost Efficiency**: Share physical resources among multiple workloads
- **Agility**: Deploy and scale applications in seconds instead of weeks
- **Portability**: Run applications consistently across different environments
- **Isolation**: Secure separation between applications and users
- **DevOps**: Enable continuous integration and deployment pipelines

## Prerequisites

To get the most out of this chapter, you should be familiar with:

- Basic Linux/Unix command line operations
- Operating system concepts (processes, memory, networking)
- Cloud computing fundamentals from previous chapters

## Practical Applications

Throughout this chapter, you'll see real-world examples including:

- Setting up virtual machines on cloud platforms
- Creating Docker containers for applications
- Understanding how AWS and other cloud providers use these technologies
- Building serverless functions with AWS Lambda

Let's begin our journey into the world of virtualization and containerization!

## Optional application lesson

Required theory in this chapter is unchanged. For Lambda SnapStart, Firecracker snapshots, and WebAssembly isolates (2022–2026), see [08-05 Serverless, MicroVMs, and WebAssembly Applications]({{ site.baseurl }}{% multilang_post_url contents/chapter08/21-01-01-08_05_Serverless_MicroVMs_and_WASM %}). The Docker lesson now also sketches the **manual → managed** ladder (Compose on a VM versus a container platform) used by the [Deploy Track]({{ site.baseurl }}/contents/en/chapter15/).

## Video tham khảo

- **xem trước:** [What is a Hypervisor?](https://www.youtube.com/watch?v=LMAEbB2a50M) — IBM Technology, English. Watch first: what a hypervisor is, the starting point of this chapter’s virtualization half.
- [What Is Docker - and why it’s EVERYWHERE](https://www.youtube.com/watch?v=W5fK8BmgBow) — Techquickie, English. What Docker is, for the container half the chapter overview promises.
- [Introduction to AWS Lambda - Serverless Compute on Amazon Web Services](https://www.youtube.com/watch?v=eOBq__h4OJ4) — Amazon Web Services, English. AWS Lambda as the serverless step the chapter lists after containers.
