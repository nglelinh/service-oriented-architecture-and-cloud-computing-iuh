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

## Why YARN?

In the early days of Hadoop (v1), MapReduce was the only processing engine. YARN (Yet Another Resource Negotiator) was introduced in Hadoop v2 to create a general-purpose cluster operating system, allowing multiple applications (MapReduce, Spark, Flink) to run on the same shared cluster resources.

## Optional application lesson

Required theory in this chapter is unchanged. For gRPC, CloudEvents, and Kubernetes-as-cluster-OS (2022–2026), see [03-02 Event-Driven Models, gRPC, and Cluster Schedulers]({{ site.baseurl }}{% multilang_post_url contents/chapter03/21-01-01-03_02_Event_Driven_and_gRPC %}).

## Video tham khảo

- **xem trước:** [What is Apache Hadoop?](https://www.youtube.com/watch?v=JWX5Inb--ig) — IBM Technology, English. Watch first for what Hadoop is, since this chapter introduction is about YARN as the cluster layer on that stack.
- [How Apache Hadoop YARN Works?](https://www.youtube.com/watch?v=GCFCohlmWgs) — HandsonERP, English. A short explanation of how YARN schedules work, which is the “why YARN” section of the introduction.
