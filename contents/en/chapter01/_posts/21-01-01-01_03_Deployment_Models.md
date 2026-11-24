---
layout: post
title: 01-03 Cloud Deployment Models
chapter: '01'
order: 4
owner: Nguyen Le Linh
lang: en
categories:
- chapter01
lesson_type: required
---

A hospital storing patient MRIs under national privacy law and an e-commerce site bracing for a 10× holiday traffic spike cannot share the exact same hosting blueprint. **Deployment models**—Public, Private, Hybrid, and Community—answer a deceptively simple architectural question: *who owns the physical hardware, who shares it, and where are your data legally allowed to live?*

## Background Knowledge Map: Isolation, Compliance, and Network Topology Across Deployment Models

1. **Multi-Tenancy vs. Single-Tenancy (Public vs. Private):** Public clouds pool hardware across millions of customers using logical isolation (hypervisors, VPCs, KMS encryption); Private clouds dedicate physical racks to a single organization for deterministic performance and strict regulatory audits.
2. **Hybrid Cloud & Cloud Bursting:** Connects an on-premises private cluster to a public cloud via encrypted VPN or dedicated fiber (Direct Connect / ExpressRoute), keeping sensitive state local while bursting stateless compute to the public cloud during spikes.
3. **Community Cloud & Sovereign Boundaries:** Pools infrastructure across organizations with identical compliance mandates (such as healthcare consortia or regional banking networks), sharing audit costs without opening tenancy to the general public.

### Worked Example (Step by Step): Designing a Hybrid Cloud Burst Architecture

A regional bank runs a core ledger database (**200 GB** of regulated PII) that must stay inside its on-premises Private Cloud, while its customer web portal sees **500 req/s** normally and **5,000 req/s** during tax-filing week:

1. **Baseline steady state (500 req/s):** 4 stateless web containers on-premises (125 req/s each) query the local ledger over a $$1	ext{ ms}$$ LAN.
2. **Tax-week surge (5,000 req/s):** capacity needed is $$5{,}000 / 125 = \mathbf{40}$$ containers. Instead of buying 36 extra physical servers for 1 week/year, the load balancer bursts **36 stateless containers** into the Public Cloud over a dedicated $$10	ext{ Gbps}$$ private link ($$8	ext{ ms}$$ RTT).
3. **Latency & compliance budget check:** each burst request pays $$8	ext{ ms}$$ extra network RTT to query tokenized records from the on-premises ledger—well under a $$200	ext{ ms}$$ SLA—while **0 bytes of raw PII** are stored persistently in the public cloud!

### Deep-Dive Explanation: Matching Workload Constraints to Deployment Topology (Read Slowly)

1. **Classify data before picking a cloud:** split your system into *stateful regulated data* and *stateless compute*—most "we must stay 100% private" mandates actually apply only to the regulated data tier.
2. **Account for the hidden cost of hybrid:** connecting two environments introduces cross-cloud latency, split identity systems, and data egress fees that can erase naive compute savings.
3. **Avoid "private cloud" in name only:** a room of manually ticketed VMware hosts without self-service APIs and elastic quotas is legacy virtualization, not a true private cloud.
4. **Plan exit and residency up front:** modern data-sovereignty laws (GDPR, national banking rules) dictate *which region* keys and backups reside in—making hybrid and sovereign regions first-class design constraints.

**Quick check.** In the bank's hybrid burst, why did we burst the *stateless web tier* rather than replicating the *200 GB ledger database* into the public cloud? Because bursting stateless containers violates zero data-residency rules and avoids cross-cloud database write-consistency conflicts!

## Overview of Deployment Models

The four primary cloud deployment models each offer different levels of control, security, and cost considerations:

```
┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐
│   Public Cloud  │  Private Cloud  │  Hybrid Cloud   │ Community Cloud │
23: │ Shared          │ Dedicated       │ Mixed           │ Shared by Group │
24: │ Multi-tenant    │ Single-tenant   │ Best of Both    │ Common Interests│
25: │ Cost-effective  │ High Control    │ Flexible        │ Cost Sharing    │
26: │ Scalable        │ Secure          │ Complex         │ Specialized     │
└─────────────────┴─────────────────┴─────────────────┴─────────────────┘
```

## Public Cloud

Public cloud creates a shared environment where computing resources are accessible to the general public over the internet.

### Definition and Characteristics
In a public cloud, third-party providers like AWS, Microsoft Azure, and Google Cloud Platform own and operate the infrastructure. They deliver computing resources—servers, storage, and applications—over the internet. Multiple organizations, or "tenants," share the same physical hardware, though their data remains logically isolated. This **shared infrastructure** model allows for massive economies of scale, making public clouds highly cost-effective.

### Advantages and Disadvantages
The primary appeal of the public cloud is **cost efficiency**. With no upfront capital investment required for hardware, businesses can treat IT as an operational expense, paying only for what they use. It offers virtually unlimited **scalability**, allowing you to spin up thousands of servers in minutes to handle traffic spikes. However, this model comes with trade-offs. You have **limited control** over where your data physically resides and how the underlying infrastructure is configured, which can be a concern for highly regulated industries. Additionally, since resources are shared, there is a theoretical risk of "noisy neighbors" affecting performance, although modern hypervisors have largely mitigated this.

### Use Cases
Public cloud is the default choice for most modern applications, including web servers, development environments, and data analytics platforms. It is ideal for startups needing to launch quickly and enterprises looking to offload variable workloads.

## Private Cloud

Private cloud offers a dedicated environment where computing resources are used exclusively by a single business or organization.

### Definition and Characteristics
A private cloud can be physically located at your organization's on-site data center or hosted by a third-party service provider. Regardless of location, the key distinction is that the services and infrastructure are maintained on a private network dedicated solely to your organization. This model provides the highest level of security and control, as resources are not shared with other tenants.

### Types of Private Cloud
Private clouds can take several forms. An **On-Premises Private Cloud** is hosted within your own data center, giving you complete control but requiring significant internal expertise to manage the virtualization stack (e.g., VMware, OpenStack). A **Hosted Private Cloud** involves renting dedicated servers from a provider who manages the hardware for you. A **Virtual Private Cloud (VPC)** is a hybrid concept where a public cloud provider creates a logically isolated section of their public cloud for your exclusive use, bridging the gap between public and private models.

### Advantages and Disadvantages
The main advantage of a private cloud is **security and control**. You can customize the environment to meet specific regulatory requirements (like HIPAA or GDPR) and ensure predictable performance. However, this comes at a steep price: **high cost**. Building an on-premises private cloud requires substantial capital investment in hardware and ongoing operational costs for power, cooling, and IT staff. It also lacks the massive elasticity of the public cloud; if you run out of capacity, you must physically buy and install more servers.

### Use Cases
Private clouds are often necessary for highly regulated industries such as **finance, healthcare, and government**, where data privacy laws strictly control where and how data is stored. They are also used for mission-critical legacy applications that require specific hardware configurations not available in the public cloud.

## Hybrid Cloud

Hybrid cloud combines public and private clouds, bound together by technology that allows data and applications to be shared between them.

### Definition and Characteristics
A hybrid cloud gives you the "best of both worlds" by creating a unified environment. You can keep sensitive data and critical applications in your secure private cloud while leveraging the public cloud's computational power for less sensitive tasks. For this to work, there must be seamless connectivity and orchestration between the two environments, often achieved through VPNs, Direct Connect links, or container orchestration platforms like Kubernetes.

### Architecture Patterns
One common pattern is **Cloud Bursting**. An application runs in a private cloud during normal operations but "bursts" into the public cloud during peak demand to handle the overflow traffic. Another pattern is **Data Tiering**, where sensitive customer data is stored on-premises for compliance, while anonymized data is sent to the public cloud for machine learning analysis.

### Advantages and Disadvantages
The hybrid model offers unparalleled **flexibility**. You can optimize costs by using public cloud resources for temporary workloads while maintaining compliance for sensitive data on-premises. It allows for a gradual migration strategy, moving workloads to the cloud at your own pace. However, it is the most **complex** model to manage. It requires sophisticated networking, consistent security policies across different environments, and a high level of technical expertise to ensure interoperability.

## Community Cloud

Community cloud is a collaborative effort where infrastructure is shared between several organizations from a specific community with common concerns.

### Definition and Characteristics
In a community cloud, the infrastructure is shared by several organizations that have shared concerns (e.g., mission, security requirements, policy, and compliance considerations). It can be managed by the organizations themselves or a third party. This model sits somewhere between public and private: it is not open to everyone, but it is not restricted to just one organization.

### Advantages and Disadvantages
The key benefit is **cost sharing**. Organizations with similar needs can pool their resources to build high-quality infrastructure that would be too expensive individually. It fosters **collaboration** and ensures that all members meet the same industry-specific standards. The downside is **shared governance**, which can lead to conflicts over resource allocation and policy updates.

### Use Cases
Community clouds are common in **government**, where different agencies share resources on a secure network. They are also found in **healthcare** (sharing patient records among hospitals) and **academic research** (universities sharing high-performance computing clusters).

## Choosing the Right Deployment Model

Choosing the right deployment model is a strategic decision that balances cost, control, and compliance.

### Decision Framework
- **Public Cloud**: Choose this for general-purpose workloads, web applications, and when cost and speed are primary drivers.
- **Private Cloud**: Choose this if you have strict regulatory requirements, need absolute control over data sovereignty, or have predictable, consistent workloads.
- **Hybrid Cloud**: Choose this if you need to keep some data on-premises for compliance but want the scalability of the public cloud for other parts of your application.
- **Community Cloud**: Choose this if you are part of a consortium or industry group with shared compliance and infrastructure needs.

## Future Trends in Deployment Models

The landscape is evolving toward **Multi-Cloud**, where organizations use services from multiple public cloud providers (e.g., using AWS for compute and Google Cloud for AI) to avoid vendor lock-in. **Edge Computing** is also rising, pushing cloud capabilities closer to the data source (like IoT devices) to reduce latency.

## Conclusion

Understanding cloud deployment models is essential for making informed decisions about cloud strategy. Each model offers different trade-offs in terms of cost, control, security, and complexity. The choice depends on your organization's specific requirements, including data sensitivity, budget, and scalability needs.

In the next lesson, we'll explore the benefits and challenges of cloud computing, helping you understand the full impact of cloud adoption on your organization.

## Video tham khảo

- **xem trước:** [AZ-900 Episode 6 \| Public, Private & Hybrid cloud deployment models \| Azure Fundamentals Course](https://www.youtube.com/watch?v=XlNR7myQydI) — Adam Marczak - Azure for Everyone, English. Watch first for public, private, and hybrid deployment, including who owns the infrastructure.
- [Cloud Deployment Models - Public Cloud, Private Cloud, Hybrid Cloud & Community Cloud](https://www.youtube.com/watch?v=GZsUFbLTH6s) — Software Architecture Academy, English. Adds community cloud next to public, private, and hybrid, which this lesson treats as the fourth model.
- [Public Cloud vs Private Cloud vs Hybrid Cloud](https://www.youtube.com/watch?v=3WIJ4axzFlU) — IntelCanada, English. A short contrast of public, private, and hybrid so the decision framework at the end of the lesson has a picture.
