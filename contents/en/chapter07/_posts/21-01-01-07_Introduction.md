---
layout: post
title: 07 Introduction to NoSQL and Distributed Databases
chapter: '07'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter07
---

This chapter moves beyond traditional Relational Database Management Systems (RDBMS) to explore the world of NoSQL and Distributed Databases, designed to handle the scale, velocity, and variety of modern big data.

## Learning Objectives

- Explain the limitations of monolithic RDBMS in distributed environments
- Understand the CAP Theorem and consistency trade-offs (BASE vs ACID)
- Explore the four main types of NoSQL databases: Key-Value, Document, Column-family, and Graph
- Analyze distributed database concepts: Sharding, Replication, and Consistent Hashing

## The Data Evolution

As applications scaled to millions of users, the vertical scaling limits of SQL databases became a bottleneck. This led to the emergence of distributed databases that sacrifice some consistency guarantees for partition tolerance and high availability.
