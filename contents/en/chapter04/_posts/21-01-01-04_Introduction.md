---
layout: post
title: 04 Introduction to Hadoop MapReduce
chapter: '04'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter04
---

This chapter delves into MapReduce, the programming paradigm that popularized big data processing on commodity hardware.

## Learning Objectives

- Understand the MapReduce programming model (Map, Shuffle, Reduce)
- Write MapReduce programs to solve parallelizable problems (e.g., Word Count)
- Analyze the flow of data: InputSplit → Mapper → Partitioner → Reducer → Output
- Understand how MapReduce achieves fault tolerance through re-execution

## The Paradigm Shift

MapReduce simplified distributed computing by abstracting the complexities of parallelization, fault tolerance, data distribution, and load balancing. Programmers simply define a `Map` function (to process data) and a `Reduce` function (to aggregate results), and the framework handles the rest.
