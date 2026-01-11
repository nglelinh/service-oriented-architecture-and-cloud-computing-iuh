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
