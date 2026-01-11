---
layout: post
title: 12-01 Cloud Providers - AWS, Azure, and GCP
chapter: '12'
order: 2
owner: Nguyen Le Linh
lang: en
categories:
- chapter12
lesson_type: required
---

Cloud providers offer on-demand computing resources and services that power modern applications. This lecture compares the major cloud providers and their offerings.

## Major Cloud Providers

### The Big Three

The cloud computing market is dominated by three major providers:

1. **Amazon Web Services (AWS)** - Market leader
2. **Microsoft Azure** - Strong enterprise presence
3. **Google Cloud Platform (GCP)** - Innovation leader

### Amazon Web Services (AWS)

**Market Position**: The largest public cloud by far

**Usage**: Powers millions of applications
- **Netflix**: Video streaming infrastructure
- **Reddit**: Social platform backend
- **Spotify**: Music streaming services
- Millions of other applications

> [!WARNING]
> **Critical Infrastructure**
> 
> When AWS goes down, half of the internet goes down. Example: The infamous S3 outage in February 2017 affected thousands of websites and services.

#### AWS Key Services

**Compute**:
- **EC2** (Elastic Compute Cloud): Virtual machines
- **Lambda**: Serverless functions
- **ECS/EKS**: Container orchestration

**Storage**:
- **S3** (Simple Storage Service): Object storage (powers much of the internet)
- **EBS**: Block storage for EC2
- **Glacier**: Archival storage

**Database**:
- **RDS**: Relational databases (MySQL, PostgreSQL, etc.)
- **DynamoDB**: NoSQL database
- **Aurora**: High-performance relational database

**Big Data**:
- **EMR** (Elastic MapReduce): Managed Hadoop/Spark
- **Redshift**: Data warehouse
- **Kinesis**: Real-time data streaming

### Microsoft Azure

**Market Position**: Second largest, strong in enterprise

**Key Strengths**:
- Integration with Microsoft products (Office 365, Active Directory)
- Hybrid cloud capabilities
- Enterprise support

#### Azure Key Services

**Compute**:
- **Virtual Machines**: Similar to AWS EC2
- **App Service**: PaaS for web apps
- **Azure Functions**: Serverless

**Storage**:
- **Azure Storage**: Blob, file, queue, table storage
- **Azure Data Lake**: Big data storage

**Database**:
- **Azure SQL Database**: Managed SQL Server
- **Cosmos DB**: Globally distributed NoSQL
- **Azure Database for PostgreSQL/MySQL**

**Big Data**:
- **HDInsight**: Managed Hadoop/Spark/Kafka
- **Azure Synapse Analytics**: Data warehouse
- **Azure Databricks**: Apache Spark platform

### Google Cloud Platform (GCP)

**Market Position**: Third largest, innovation leader

**Key Strengths**:
- Advanced data analytics and ML
- Kubernetes (originated from Google)
- Global network infrastructure

#### GCP Key Services

**Compute**:
- **Compute Engine**: Virtual machines
- **Cloud Functions**: Serverless
- **GKE** (Google Kubernetes Engine): Managed Kubernetes

**Storage**:
- **Cloud Storage**: Object storage
- **Persistent Disk**: Block storage

**Database**:
- **Cloud SQL**: Managed MySQL/PostgreSQL
- **Cloud Spanner**: Globally distributed database
- **BigTable**: NoSQL for big data
- **BigQuery**: Serverless data warehouse

**Big Data**:
- **Dataproc**: Managed Hadoop/Spark
- **Dataflow**: Stream and batch processing
- **Pub/Sub**: Messaging service

## Feature Parity

> **Key Insight**: All clouds try to compete on features, so they all end up having extremely similar feature sets.

| Service Type | AWS | Azure | GCP |
|--------------|-----|-------|-----|
| **VMs** | EC2 | Virtual Machines | Compute Engine |
| **Serverless** | Lambda | Functions | Cloud Functions |
| **Containers** | ECS/EKS | AKS | GKE |
| **Object Storage** | S3 | Blob Storage | Cloud Storage |
| **NoSQL** | DynamoDB | Cosmos DB | BigTable |
| **Data Warehouse** | Redshift | Synapse | BigQuery |
| **Big Data** | EMR | HDInsight | Dataproc |

## Virtual Machines Comparison

### AWS EC2

**Range**: From tiny to gigantic
- **T2.Nano**: 1 vCPU, 512 MB RAM
- **X1.32xlarge**: 128 vCPU, 2000 GB RAM

**Features**:
- ✓ GPUs available (useful for deep learning)
- ✓ Priced per-second
- ✓ **On-Demand**: Pay for what you use
- ✓ **Spot Instances**: Auction for unused capacity (much cheaper)
  - Caveat: VM may be shut down with short notice

### Azure Virtual Machines

**Similar to AWS**:
- ✓ GPUs available
- ⚠ Max 32 vCPUs (currently)
- ⚠ Max 800 GB RAM (currently)
- Note: Most applications won't hit these limits

### Google Compute Engine

**Features**:
- ✓ Largest: 96 vCPU, 624 GB RAM
- ✓ **Custom-sized machines** (unique feature)
- ✓ Per-second billing
- ✓ Sustained use discounts

## Storage Services

### Object Storage

All three providers offer massive-scale object storage:

**AWS S3**:
- Most widely used
- Powers much of the internet (e.g., Imgur)
- 11 9's of durability (99.999999999%)

**Azure Blob Storage**:
- Similar to S3
- Integrated with Azure ecosystem

**Google Cloud Storage**:
- Similar to S3
- Multiple storage classes

### Use Cases

```
Web Assets → S3/Blob/GCS
Backups → Glacier/Archive/Coldline
Big Data → Data Lake on S3/ADLS/GCS
```

## Hosted Data Processing

All providers offer managed big data services:

### Amazon EMR
- Managed Hadoop, Spark, HBase, Presto, Hive
- Automatic cluster scaling and provisioning

### Microsoft HDInsight
- Managed Hadoop, Spark, Kafka, HBase
- Integration with Azure services

### Google Dataproc
- Managed Hadoop and Spark
- Fast cluster creation (90 seconds)
- Per-second billing

## Database Services

### Managed Relational Databases

**AWS RDS**:
- MySQL, PostgreSQL, MariaDB, Oracle, SQL Server
- Aurora (AWS proprietary, MySQL/PostgreSQL compatible)

**Azure**:
- Azure SQL Database (SQL Server)
- Azure Database for MySQL/PostgreSQL

**GCP**:
- Cloud SQL (MySQL, PostgreSQL, SQL Server)

### NoSQL Databases

**AWS DynamoDB**:
- Key-value and document database
- Serverless, auto-scaling

**Azure Cosmos DB**:
- Multi-model database
- Global distribution
- Multiple consistency levels

**GCP BigTable**:
- Wide-column NoSQL
- Powers Google Search, Maps, Gmail

**GCP BigQuery**:
- Serverless data warehouse
- Petabyte-scale analytics
- SQL interface

## Cloud Regions and Availability

### Azure Regions

- **60+ Azure Regions** worldwide
- Most extensive global presence

**Explore**: [Azure Datacenter Map](https://datacenters.microsoft.com/globe/explore/)

### Reasons to Select a Region

#### 1. Cost
- **Egress fees**: Moving data between regions costs money
- **Regional pricing**: Some services cheaper in certain regions
- **Tip**: Use pricing calculators to estimate costs

#### 2. Available Resources
- Not all services available in all regions
- Check "Products by Region" documentation

#### 3. Security and Compliance
- **Government clouds**: Special regions for government data
- **Data sovereignty**: Keep data in specific countries
- **Compliance**: GDPR, HIPAA, etc.

#### 4. Speed (Latency)
- **Pick closest region** to your users
- Lower latency = better user experience

### Availability Zones

**Definition**: Physically separate locations within each region

**Purpose**: Tolerance to local failures
- Earthquakes
- Floods
- Fires
- Power outages

**Characteristics**:
- Redundancy and logical isolation
- High-performance network between zones
- Round-trip latency < 2ms (Azure)

```
┌─────────────────────────────────────┐
│  Region: East US                    │
│  ┌───────────┐  ┌───────────┐      │
│  │  Zone 1   │  │  Zone 2   │      │
│  │ (DC 1,2)  │  │ (DC 3,4)  │      │
│  └───────────┘  └───────────┘      │
│  ┌───────────┐                      │
│  │  Zone 3   │                      │
│  │ (DC 5,6)  │                      │
│  └───────────┘                      │
└─────────────────────────────────────┘
```

## Unique Features

### Google Cloud Platform

**Cloud Spanner**:
- Planet-scale distributed database
- Strong consistency (CP system)
- Horizontal scalability

**Tensor Processing Unit (TPU)**:
- Custom hardware for deep learning
- Faster than GPUs for specific workloads

**BigQuery**:
- Serverless data warehouse
- Analyze petabytes in seconds

### AWS

**Absurdly large feature set**:
- 200+ services
- Most mature ecosystem

**FPGAs** (Field-Programmable Gate Arrays):
- Custom hardware acceleration
- F1 instances

**Global Infrastructure**:
- Most regions and availability zones

### Azure

**Hybrid Cloud**:
- Azure Stack (on-premises Azure)
- Azure Arc (manage resources anywhere)

**Enterprise Integration**:
- Active Directory integration
- Office 365 integration
- Strong Windows support

## Cloud Security

### Key Security Features

#### Data Storage
- **Encryption at rest**: All providers encrypt stored data
- **Encryption in transit**: HTTPS/TLS for data transfer
- **Regulatory standards**: Compliance with GDPR, HIPAA, SOC 2, etc.

#### Compliance
- **Certifications**: ISO 27001, PCI DSS, FedRAMP
- **Audit logs**: Track all access and changes
- **Data residency**: Control where data is stored

#### Data Migration
- **Secure transfer**: How to move sensitive data?
- **Physical transfer**: AWS Snowball, Azure Data Box
- **Network transfer**: VPN, Direct Connect, ExpressRoute

#### Cloud Permissions
- **IAM** (Identity and Access Management)
- **Role-based access control** (RBAC)
- **Principle of least privilege**
- **Example**: Students don't get sudo access!

#### DDoS Mitigation
- **AWS Shield**: DDoS protection
- **Azure DDoS Protection**: Network security
- **GCP Cloud Armor**: Web application firewall

#### High Scalability
- **Auto-scaling** with security settings
- **Load balancing** across zones
- **Redundancy** for high availability

## Top Benefits of Cloud Computing

1. **Cost Efficiency**
   - Pay only for what you use
   - No upfront capital expenditure
   - Economies of scale

2. **Scalability**
   - Scale up/down on demand
   - Handle traffic spikes
   - Global reach

3. **Reliability**
   - High availability (99.9%+ uptime)
   - Disaster recovery
   - Automatic backups

4. **Performance**
   - Latest hardware
   - Global CDN
   - Low latency

5. **Security**
   - Professional security teams
   - Compliance certifications
   - Regular updates

6. **Innovation**
   - Access to latest technologies
   - AI/ML services
   - Managed services

## Choosing a Cloud Provider

### Decision Factors

**Existing Infrastructure**:
- Already using Microsoft? → Azure
- Already using Google Workspace? → GCP
- Need broadest service selection? → AWS

**Specific Requirements**:
- Best data analytics? → GCP (BigQuery)
- Best enterprise integration? → Azure
- Most mature ecosystem? → AWS

**Cost**:
- Use pricing calculators
- Consider egress fees
- Look for committed use discounts

**Skills**:
- Team expertise
- Training availability
- Community support

### Multi-Cloud Strategy

Many organizations use multiple clouds:

**Benefits**:
- ✓ Avoid vendor lock-in
- ✓ Use best service from each provider
- ✓ Geographic coverage

**Challenges**:
- ⚠ Increased complexity
- ⚠ Higher management overhead
- ⚠ Data transfer costs

## Summary

Cloud providers offer similar core services with unique strengths:

- **AWS**: Largest, most mature, broadest service selection
- **Azure**: Best for enterprises, hybrid cloud, Microsoft integration
- **GCP**: Innovation leader, best data analytics, Kubernetes expertise

**Key Considerations**:
- Feature parity across providers
- Regional availability and compliance
- Security and compliance features
- Cost optimization strategies

Choose based on your specific requirements, existing infrastructure, and team expertise.

## Further Reading

- [AWS Services Overview](https://aws.amazon.com/products/)
- [Azure Services](https://azure.microsoft.com/en-us/services/)
- [GCP Products](https://cloud.google.com/products)
- [Cloud Comparison](https://cloud.google.com/docs/compare/aws)
