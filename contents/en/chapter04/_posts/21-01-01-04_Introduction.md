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

## Optional application lesson

Required theory in this chapter is unchanged. For lakehouse batch, Beam/Dataflow, and modern shuffle-on-object-storage (2022–2026), see [04-02 Modern Batch Processing beyond Classic MapReduce]({{ site.baseurl }}{% multilang_post_url contents/chapter04/21-01-01-04_02_Modern_Batch_and_MapReduce %}).
