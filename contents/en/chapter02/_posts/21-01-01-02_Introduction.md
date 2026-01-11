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

## Why Distributed Systems?

Single machines have reached their physical limits (vertical scaling). To handle the staggering scale of modern internet applications—serving billions of users and processing petabytes of data—we have no choice but to coordinate thousands of machines working together (horizontal scaling). This chapter explains the theoretical and practical foundations of how we achieve this coordination.
