---
layout: post
title: Chapter 10 - Big Data Platforms and Processing
chapter: '10'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter10
lesson_type: required
---

Welcome to Chapter 10: **Big Data Platforms and Processing**.

Every second, ride-hailing apps, payment networks, and connected sensors emit millions of tiny JSON events that individually mean very little—but aggregated across millions of users, reveal fraud rings, traffic bottlenecks, and shifting demand. Chapter 10 introduces **Big Data Platforms and Processing**, showing how distributed storage and parallel compute engines replace single-node databases once data outgrows one machine's RAM, disk, and network interface.

## Background Knowledge Map: The Architecture of Large-Scale Data Systems

### 1. Vertical Scaling Limits vs. Horizontal Partitioning (Sharding)
A single server eventually hits hard physical ceilings on memory channels, disk IOPS, and NIC throughput. Big Data platforms scale **horizontally** by partitioning datasets across clusters of commodity machines—trading simple single-machine transactions for massive aggregate bandwidth and fault-tolerant parallel execution.

### 2. Batch Processing vs. Stream Processing
- **Batch processing** (Hadoop MapReduce, Spark Batch) accumulates bounded slices of data (hourly or daily) and optimizes for maximum throughput per dollar.
- **Stream processing** (Apache Flink, Kafka Streams, Spark Structured Streaming) processes unbounded event streams continuously with sub-second to few-second latency—essential for fraud detection, dynamic pricing, and real-time alerting.

### 3. Separation of Storage and Compute in the Cloud
Early on-premise Hadoop clusters tightly coupled local disks and CPUs on every node. Cloud-native platforms decouple elastic compute clusters from durable object storage (S3/GCS/ADLS) and open table formats (Iceberg/Delta), letting teams spin up 100 workers for 10 minutes and pay zero compute when idle.

### Deep-Dive Explanation: Hand-Calculating Daily Event Volume, Throughput, and Freshness Trade-Offs (Read Slowly)

Suppose a mobile platform ingests **100,000 click events per second**, where each compressed event averages **500 bytes**:

1. **Sustained ingestion bandwidth:**
   $$100{,}000\text{ events/s} \times 500\text{ bytes} = 50{,}000{,}000\text{ bytes/s} = 50\text{ MB/s} \quad (= 400\text{ Mbps}).$$
2. **Daily storage growth (86,400 seconds/day):**
   $$50\text{ MB/s} \times 86{,}400\text{ s/day} = 4{,}320{,}000\text{ MB/day} = 4.32\text{ TB/day!}$$
   In just 30 days, raw logs reach **129.6 TB**—far beyond any single relational database server!
3. **Comparing Batch vs. Streaming data freshness:**
   - **Hourly batch job** (takes 10 minutes to run): an event arriving at 10:01 waits 59 minutes for the batch window plus 10 minutes of compute → worst-case staleness = **69 minutes**.
   - **Streaming pipeline** (5-second tumbling window + 2-second processing): worst-case staleness = **7 seconds**—nearly **600× fresher** for time-critical decisions like blocking a stolen credit card.

## Chapter Overview

Big Data has transformed how organizations collect, store, process, and analyze information. This chapter explores the fundamentals of big data, the platforms that enable big data processing, and the architectural patterns used to build scalable data systems.

## Learning Objectives

By the end of this chapter, you will be able to:

- **Understand "big data"** and its characteristics (Volume, Variety, Velocity, Veracity)
- **Explain big data platforms** and their role in modern data ecosystems
- **Design big data architectures** including data pipelines and processing workflows
- **Differentiate between operational and analytical data**
- **Understand data governance** and quality concerns at scale
- **Work with big data processing models** (batch, streaming, real-time)

## Topics Covered

### 1. What is Big Data?
- Characteristics: Volume, Variety, Velocity, Veracity
- Sources of big data (IoT, social media, scientific instruments)
- Operational vs analytical data
- The value of data and data economy

### 2. Big Data Platforms
- Platform definition and characteristics
- Data as asset vs data as product
- Onion architecture for big data platforms
- Core services: ingestion, storage, processing, querying

### 3. Big Data Architectures
- Data-centric virtualized infrastructures
- Middleware platforms
- Big data services and applications
- Multi-cloud and hybrid deployments

### 4. Data Pipelines
- Data ingestion and ETL
- Data storage and management
- Data analysis and machine learning
- Reporting and visualization

### 5. Data Governance
- Data quality and lineage
- Multi-tenancy and SLAs
- Privacy and security concerns
- Compliance (GDPR, regulations)

### 6. Processing Models
- Batch processing
- Stream processing
- Real-time analytics
- Lambda and Kappa architectures

## Why Big Data Matters

Big Data is fundamental to modern applications:

- **AI and Machine Learning**: "No AI Without Data" - data is the fuel for ML models
- **Business Intelligence**: 360-degree customer analytics
- **IoT and Industry 4.0**: Real-time monitoring and predictive maintenance
- **Scientific Research**: Earth observation, healthcare, genomics
- **Smart Cities**: Sustainability and urban optimization

## Prerequisites

To get the most out of this chapter, you should understand:

- Distributed systems concepts (from previous chapters)
- Cloud computing fundamentals
- Database basics
- Programming concepts (MapReduce, Spark from earlier chapters)

## Real-World Examples

Big data platforms power critical services:

- **Lyft**: 60 PB of queryable event data, 10 PB scanned daily
- **Sentinel (ESA)**: Petabytes of Earth observation data
- **NYC Taxi Data**: 112M+ rows of trip data
- **Network Monitoring**: 5M sensors generating 1.4B events/day

Let's explore the world of big data platforms!

## Optional application lesson

Required theory in this chapter is unchanged. For Iceberg/Delta lakehouse catalogs and multi-engine apps (2022–2026), see [10-02 Open Lakehouse Platforms and Multi-Engine Apps]({{ site.baseurl }}{% multilang_post_url contents/chapter10/21-01-01-10_02_Open_Lakehouse_Applications %}).

## Video tham khảo

- **xem trước:** [Big Data Explained](https://www.youtube.com/watch?v=HRfR4dJoKDc) — IBM Technology, English. Watch first: IBM’s explanation of big data, the definition this chapter overview opens with.
- [What is Apache Hadoop?](https://www.youtube.com/watch?v=JWX5Inb--ig) — IBM Technology, English. Hadoop as one platform that made those volumes processable, for the platform section.
- [Big Data Architecture Patterns \| Lambda vs Kappa Architecture](https://www.youtube.com/watch?v=waDJcSCXz_Y) — Software Architecture Academy, English. Lambda versus Kappa, the processing models the chapter overview lists.
