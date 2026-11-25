---
layout: post
title: 01-02 Cloud Service Models
chapter: '01'
order: 3
owner: Nguyen Le Linh
lang: en
categories:
- chapter01
lesson_type: required
---

Every engineering team faces the same recurring trade-off: *how much of the stack do we want to control, and how much undifferentiated heavy lifting are we happy to hand off?* Renting raw virtual machines (IaaS) lets you tune Linux kernel parameters at 2:00 AM, while adopting a managed platform (PaaS) or turnkey application (SaaS) frees those hours for shipping product features. Choosing between **IaaS, PaaS, and SaaS** is really choosing where to draw the **Shared Responsibility** line.

## Key ideas: Abstraction Layers and the Shared Responsibility Boundary

Think of the cloud stack as nine concentric layers—from physical racks and networking up through OS, runtime, middleware, and application code:

1. **IaaS (Infrastructure as a Service):** The provider manages physical hardware, virtualization, and network fabric; *you* manage the guest OS, security patches, runtime, scaling rules, and application code (e.g., AWS EC2, GCP Compute Engine).
2. **PaaS (Platform as a Service):** The provider also manages the OS, language runtime, patching, and auto-scaling; *you* bring application code and data (e.g., Cloud Run, App Engine, Heroku).
3. **SaaS (Software as a Service):** The provider operates the entire stack end to end; *you* configure tenants, access roles, and business data (e.g., Google Workspace, Salesforce).

### Worked Example (Step by Step): Total Cost of Ownership (IaaS vs. PaaS) for a 5-Engineer Startup

Compare running a backend API for 1 month on raw VMs (**IaaS**) versus a managed container platform (**PaaS**), assuming an engineer's loaded time costs $$\$60/	ext{hr}$$:

| Cost Component | Raw IaaS (EC2 + Self-Managed DB) | Managed PaaS (Cloud Run + Managed DB) |
|---|---|---|
| Cloud infrastructure bill | $$\$300/	ext{mo}$$ | $$\$520/	ext{mo}$$ |
| OS patching, TLS renewal, failover ops | $$15	ext{ hrs} 	imes \$60 = \$900/	ext{mo}$$ | $$2	ext{ hrs} 	imes \$60 = \$120/	ext{mo}$$ |
| **True Monthly TCO** | **$$\$1{,}200/	ext{mo}$$** | **$$\$640/	ext{mo}$$** |

Even though the PaaS cloud invoice is **73% higher** than raw IaaS ($$\$520$$ vs. $$\$300$$), factoring in engineering operations makes PaaS **46.7% cheaper** overall—and returns 13 hours/month of feature development!

### Explanation: Choosing Your Layer Along the Control–Velocity Curve

1. **Never confuse cloud bill with total cost:** raw IaaS looks cheapest on a pricing calculator because it hides OS patching, backup verification, and on-call incident hours.
2. **Move down the stack only when constraints demand it:** drop from PaaS to IaaS when you need custom kernel modules, specialized GPU drivers, or strict network-level compliance controls.
3. **Remember that data security never disappears:** even in SaaS, the customer *always* owns identity governance, role permissions, and data classification.
4. **Watch the modern blur (CaaS & FaaS):** Kubernetes (CaaS) and serverless functions (FaaS) sit cleanly between IaaS and PaaS, letting teams trade server management for fine-grained request-level billing.

**Quick check.** At what scale does self-managed IaaS start beating PaaS in our table? If the infrastructure bill grows 20× (IaaS $$\$6{,}000$$ vs. PaaS $$\$10{,}400$$) while ops effort stays roughly fixed around $$\$1{,}500$$, IaaS total cost ($$\$7{,}500$$) undercuts PaaS ($$\$10{,}520$$)—explaining why startups love PaaS while hyperscale firms eventually optimize on IaaS!

## The Cloud Service Stack

The cloud service models can be visualized as a stack, where each layer builds upon the previous one:

```
┌─────────────────────────────────────┐
│        Software as a Service        │  ← SaaS
│              (SaaS)                 │
├─────────────────────────────────────┤
│       Platform as a Service         │  ← PaaS
│              (PaaS)                 │
├─────────────────────────────────────┤
│     Infrastructure as a Service     │  ← IaaS
│              (IaaS)                 │
├─────────────────────────────────────┤
│        Physical Infrastructure      │  ← On-Premises
└─────────────────────────────────────┘
```

## Infrastructure as a Service (IaaS)

Infrastructure as a Service (IaaS) is the foundation of cloud computing. It provides virtualized computing resources over the internet, allowing businesses to rent rather than buy IT infrastructure.

### Definition and Core Concept
At its core, IaaS offers the most basic building blocks of cloud computing: virtual machines (VMs), storage, networks, and operating systems. Instead of purchasing physical servers and housing them in an on-premises data center, organizations provision these resources on a pay-as-you-go basis from a cloud provider. This model gives you the highest level of flexibility and management control over your IT resources, effectively mimicking a traditional data center but in a virtualized environment.

### Key Components
A typical IaaS environment consists of several key components. **Compute resources** are the workhorses, ranging from standard virtual machines to bare metal servers and serverless functions. **Storage services** provide scalable options for different needs, such as block storage for databases or object storage for vast amounts of unstructured data like backups and media files. **Networking** capabilities allow you to define your own virtual network topology, including subnets, route tables, and firewalls, just as you would with physical switches and routers.

### Responsibility Model
In the IaaS model, the cloud provider manages the underlying physical infrastructure—the host servers, virtualization layer, storage hardware, and physical networking. However, you, the customer, are responsible for everything above the hypervisor. This includes the operating system, middleware, runtime environment, data, and applications. Crucially, you are also responsible for the security configuration of these components, such as patching the OS and configuring firewalls.

### Use Cases
IaaS is particularly well-suited for scenarios requiring granular control. It is ideal for **development and testing**, as teams can spin up temporary environments in minutes and dismantle them just as quickly. It supports **disaster recovery** strategies by allowing you to replicate critical infrastructure in a different geographic region without the cost of a second physical site. Additionally, **High-Performance Computing (HPC)** workloads, which often require specific hardware configurations for scientific simulations or financial modeling, thrive on the scalable compute power of IaaS.

### Advantages and Disadvantages
The primary advantage of IaaS is **control**. You have complete freedom to configure the environment to your exact specifications. It avoids the large capital expenditure of buying hardware and allows for rapid scaling. However, this freedom comes with a burden: **management overhead**. Your team must possess the technical expertise to manage operating systems, security patches, and network configurations, which can be complex and time-consuming.

## Platform as a Service (PaaS)

Platform as a Service (PaaS) removes the burden of managing the underlying infrastructure, allowing you to focus entirely on improved productivity and application development.

### Definition and Core Concept
PaaS provides a complete development and deployment environment in the cloud. It includes not just the infrastructure (servers, storage, and networking) but also the middleware, development tools, business intelligence services, database management systems, and more. This model is designed to support the complete web application lifecycle: building, testing, deploying, managing, and updating.

### Key Components
PaaS offerings typically include a suite of **development tools** and **runtime environments** that support various programming languages like Java, Python, and Node.js. They often provide managed **database services** (both SQL and NoSQL), **caching layers**, and **message queues**, removing the need to install and configure these complex systems manually. Furthermore, PaaS solutions usually come with built-in **deployment pipelines (CI/CD)** and **auto-scaling** capabilities, ensuring your application can handle traffic spikes without manual intervention.

### Responsibility Model
The responsibility shift in PaaS is significant. The cloud provider takes on the management of the operating system, middleware, and runtime environment, in addition to the physical infrastructure. Your responsibility is streamlined to usually just two things: your **applications** and your **data**. This allows developers to focus on writing code rather than patching servers.

### Use Cases
PaaS is the go-to model for **web and mobile application development**. It allows diverse teams to collaborate on projects regardless of their physical location. It is also excellent for implementing **APIs and microservices**, where small, independent components can be deployed and managed easily.

### Advantages and Disadvantages
The biggest benefit of PaaS is **speed**. It significantly reduces the time to market by handling the "plumbing" of application delivery. It reduces development complexity and offers built-in scalability. On the downside, PaaS can lead to **vendor lock-in**, as applications might be built using proprietary tools or APIs that are difficult to migrate to another platform. You also have **less control** over the underlying environment, which might be a constraint for applications with very specific system-level requirements.

## Software as a Service (SaaS)

Software as a Service (SaaS) is the most familiar model for end-users, delivering fully functional applications over the internet.

### Definition and Core Concept
SaaS allows users to connect to and use cloud-based apps over the Internet. Common examples are email, calendaring, and office tools. In this model, the cloud provider manages the entire technology stack—from the physical servers up to the application code itself. Users typically access the software via a web browser or a lightweight client app, usually on a subscription basis.

### Key Characteristics
SaaS is defined by **multi-tenancy**, where a single instance of the software serves multiple customers (tenants) while keeping their data isolated. It typically operates on a **subscription model** (monthly or annual fees) and features **automatic updates**. Users always have access to the latest version of the software without needing to download patches or perform upgrades.

### Responsibility Model
In the SaaS model, the customer has the least amount of responsibility, primarily limited to **managing their data** and **user access**. The provider handles everything else: application software, security, databases, servers, and network infrastructure.

### Categories of SaaS Applications
SaaS spans a vast array of categories. **Productivity suites** like Microsoft 365 and Google Workspace enable collaboration. **Customer Relationship Management (CRM)** tools like Salesforce help businesses manage client interactions. **Enterprise Resource Planning (ERP)** systems like NetSuite integrate core business processes. Even specialized creative tools like Adobe Creative Cloud are now delivered as SaaS.

### Advantages and Disadvantages
SaaS removes the need for installation, maintenance, and hardware acquisition, making it extremely **accessible** and easy to deploy. It provides predictable costs through subscriptions. However, it offers the **least amount of control** and customization. You are bound by the features provided by the vendor, and **data security** relies heavily on the provider's measures.

## Choosing the Right Service Model

Selecting the appropriate service model is a trade-off between control and convenience.

### Decision Framework
When deciding which model to use, consider the following:
- Choose **IaaS** when you need maximum control, are migrating legacy applications that require specific OS configurations, or have a strong operations team.
- Choose **PaaS** when you are building new applications and want to optimize for development speed and minimize administrative overhead.
- Choose **SaaS** for standard business processes (email, CRM, HR) where building a custom solution would not provide a competitive advantage.

Many modern organizations adopt a **hybrid approach**, utilizing SaaS for productivity, PaaS for new customer-facing apps, and IaaS for specialized workloads that need deep customization.

### Comparison Matrix
| Aspect | IaaS | PaaS | SaaS |
|--------|------|------|------|
| **Control** | High | Medium | Low |
| **Flexibility** | High | Medium | Low |
| **Management Overhead** | High | Medium | Low |
| **Time to Market** | Slow | Fast | Immediate |
| **Customization** | High | Medium | Low |
| **Cost Predictability** | Variable | Predictable | Predictable |
| **Technical Expertise** | High | Medium | Low |

## Future Trends in Service Models

As cloud computing evolves, the lines between these models are blurring, and new models are emerging. **Function as a Service (FaaS)**, or serverless computing, is gaining popularity as it abstracts even more infrastructure management than PaaS, executing code only in response to events. **Container as a Service (CaaS)** sits between IaaS and PaaS, offering a managed environment for deploying containerized applications.

## Conclusion

Understanding the three primary cloud service models—IaaS, PaaS, and SaaS—is fundamental to making informed decisions about cloud adoption. Each model offers different trade-offs between control, flexibility, and management overhead. The choice depends on your organization's technical expertise, business requirements, and strategic objectives.

In the next lesson, we'll explore cloud deployment models and how they complement these service models to provide comprehensive cloud solutions.

## Video tham khảo

- **xem trước:** [AZ-900 Episode 5 \| IaaS vs PaaS vs SaaS cloud service models \| Microsoft Azure Fundamentals Course](https://www.youtube.com/watch?v=9CVBohl6w0Q) — Adam Marczak - Azure for Everyone, English. Watch first for a side-by-side of IaaS, PaaS, and SaaS and who operates each layer of the stack.
- [IaaS Explained](https://www.youtube.com/watch?v=XRdmfo4M_YA) — IBM Technology, English. Zooms in on infrastructure as a service: you keep the OS, runtime, and application; the provider keeps the hardware.
- [Software as a Service (SaaS) Explained in 5 mins](https://www.youtube.com/watch?v=20QUNgFIrK0) — IBM Technology, English. The SaaS end of the same stack, where the consumer uses the application and does not patch the platform.
