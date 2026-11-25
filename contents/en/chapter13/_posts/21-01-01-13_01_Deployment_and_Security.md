---
layout: post
title: 13-01 Deployment, Security, and Compliance
chapter: '13'
order: 2
owner: Nguyen Le Linh
lang: en
categories:
- chapter13
lesson_type: required
---

Deploying applications to the cloud requires not just getting the code running, but doing so securely and in compliance with regulations. This lecture covers deployment strategies, identity management, and compliance frameworks.

Writing clean code is only half the engineering battle; pushing that code safely into production—while keeping attackers out and staying compliant with privacy regulations—is where real systems earn trust. A single botched Friday deployment or one publicly exposed database port can undo months of product work in minutes. **Deployment strategies, cloud security, and compliance** work hand in hand to ensure every release is reversible, every identity carries least privilege, and every piece of sensitive data is encrypted both in transit and at rest.

## Key ideas: Progressive Delivery, Defense in Depth, and Compliance

### 1. Progressive Deployment Strategies: Blue-Green, Canary, and Rolling
Instead of shutting down production for a risky "big-bang" upgrade, cloud teams separate *deployment* (putting code on servers) from *release* (routing live users to it). **Blue-Green** keeps two identical environments for instant traffic flip and rollback; **Canary** shifts a tiny slice (1%–5%) of real traffic to the new version first to measure error rates before expanding; **Rolling** replaces instances gradually in place.

### 2. Defense in Depth: Network, Identity, and Data Layers
Never rely on a single firewall perimeter. **Defense in Depth** stacks independent controls: private subnets and Security Groups at the network layer, **Zero-Trust IAM** (short-lived roles, MFA, least privilege) at the identity layer, and **TLS 1.3** (in transit) plus **KMS-managed AES-256** (at rest) at the data layer.

### 3. Compliance as Continuous Engineering (GDPR, SOC 2, ISO 27001)
Compliance frameworks are not just legal paperwork—they map directly to concrete technical controls: audit trails (immutable logs), data residency (keeping EU/VN user data in approved regions), right-to-erasure pipelines, and automated secret scanning.

### Explanation: Hand-Calculating Blast Radius in an All-at-Once Release vs. a 5% Canary Release

Suppose your payment API handles **100,000 requests/minute**, and a newly released version `v2.4` contains a subtle bug that triggers HTTP 500 errors on **20%** of requests. Your monitoring alert fires and rolls back automatically after **2 minutes**.

| Release Strategy | Traffic Exposed to `v2.4` | Failed Requests per Minute | Total Failed Requests in 2 Minutes | Blast Radius |
| :--- | :---: | :---: | :---: | :--- |
| **Big-Bang (100% cutover)** | $$100{,}000\text{ req/min}$$ | $$100{,}000 \times 0.20 = 20{,}000$$ | $$40{,}000\text{ failed requests}$$ | Severe outage! |
| **5% Canary Release** | $$5{,}000\text{ req/min}$$ | $$5{,}000 \times 0.20 = 1{,}000$$ | $$2{,}000\text{ failed requests}$$ | **20× smaller!** |

With a 5% Canary, **95% of users never see a single error**, automated SLO burn alerts still catch the 20% error spike on the canary slice within seconds, and traffic shifts back to `v2.3` cleanly—proving why progressive delivery is a mathematical risk-reduction tool, not bureaucracy.

## Deployment Strategies

### The Need for Automated Deployment

**Deployment is not just "uploading files"**. It involves:
- Infrastructure provisioning (VMs, databases, networks)
- Configuration management
- Application deployment
- Managing dependencies

**Why automate?**
- **Consistency**: Eliminate "it works on my machine" issues
- **Scale**: Deploy to hundreds of servers as easily as one
- **Speed**: Rapid iteration and recovery
- **Reliability**: Reduce human error

### Infrastructure as Code (IaC) Overview

*Note: We will cover IaC in depth in Chapter 14.*

"Infrastructure as Code" is the ability to deploy all your services in the cloud from code rather than manually via the portal.

**Key Options**:
1. **ARM Templates / Bicep** (Azure native)
2. **CloudFormation** (AWS native)
3. **Terraform** (Multi-cloud, industry standard)

| Tool | Pros | Cons |
|------|------|------|
| **ARM Templates** (JSON) | Native to Azure, comprehensive | Verbose, difficult to read/write |
| **Bicep** | Domain Specific Language (DSL), easy to read | Azure only, newer ecosystem |
| **Terraform** | Multi-cloud, massive community | State management complexity |

### Deployment Pipelines (CI/CD)

**GitHub Actions** / **Azure DevOps** allow you to:
1. **Build** your code
2. **Test** your application
3. **Provision** infrastructure (e.g., using Terraform)
4. **Deploy** artifacts
5. **Verify** health

## Identity and Access Management (IAM)

### Azure Active Directory (Azure AD)

Azure AD (now Microsoft Entra ID) is the cloud identity provider.

**Key Features**:
- **Single Sign-On (SSO)**: One login for all apps
- **App Registrations**: Identity for applications
- **Service Principals**: Machine identities for automation
- **Managed Identities**: Passwordless authentication for Azure resources

### Role-Based Access Control (RBAC)

**Principle**: Grant access based on roles, not individual users.

**Common Roles**:
- **Owner**: Full access + Can assign roles to others
- **Contributor**: Full access + Cannot assign roles
- **Reader**: View only, no changes

**Scope**:
- Access can be scoped at different levels:
  - **Management Group** (Organization)
  - **Subscription** (Billing unit)
  - **Resource Group** (Logical grouping)
  - **Resource** (Individual item)

> [!TIP]
> **Least Privilege Principle**: Always grant the minimum permission necessary to perform the task. Start with `Reader` and escalate only if needed.

### Access Control Lists (ACLs)

While RBAC controls access to the **resource** (e.g., "Can you see this Storage Account?"), ACLs often control access to the **data** within (e.g., "Can you read this specific file?").

- **RBAC**: Broad access (Control Plane)
- **ACLs**: Fine-grained access (Data Plane)

## Security and Compliance

### Why Compliance Matters?

**Compliance** means conformity in fulfilling official requirements.

- **Legal requirement**: Fines for non-compliance can be massive.
- **Trust**: Customers require proof that you protect their data.
- **Standards**: Provides a framework for security best practices.

### Key Compliance Frameworks

#### Healthcare
- **HIPAA** (Health Insurance Portability and Accountability Act): US law protecting medical info.
- **HITECH**: Extends HIPAA for electronic health records.
- **HITRUST**: Certification framework normalizing multiple standards (HIPAA, ISO, NIST).

#### Financial
- **SOX** (Sarbanes-Oxley Act): US federal law mandating financial record keeping. Born from Enron/WorldCom scandals.
- **PCI DSS**: Payment Card Industry Data Security Standard (for handling credit cards).

#### General / International
- **ISO 27001**: International standard for Information Security Management Systems (ISMS).
- **SOC 2** (Service Organization Control): Audit procedure for service providers ensuring security, availability, processing integrity, confidentiality, and privacy.
- **GDPR** (General Data Protection Regulation): EU law on data protection and privacy.

### Azure Blueprints

**Azure Blueprints** enables you to define a repeatable set of Azure resources that implements and adheres to an organization's standards, patterns, and requirements.

It orchestrates:
- **Role Assignments**
- **Policy Assignments**
- **ARM Templates**
- **Resource Groups**

This allows you to "stamp out" compliant environments rapidly.

## Summary

- **Deployment**: Move from manual to automated (CI/CD, IaC).
- **Identity**: Use RBAC and Azure AD for secure access control.
- **Security**: Understand the shared responsibility model.
- **Compliance**: Adhere to standards (HIPAA, SOX, ISO) relevant to your industry.

In the next chapter, we will dive deep into **Infrastructure as Code** with Terraform.

## Video tham khảo

- **xem trước:** [Identity & Access Management (IAM)](https://www.youtube.com/watch?v=aNj36g7fSsU) — IBM Technology, English. Watch first: what IAM is, before the lesson’s RBAC and compliance sections.
- [Role-based access control (RBAC) vs. Attribute-based access control (ABAC)](https://www.youtube.com/watch?v=rvZ35YW4t5k) — IBM Technology, English. RBAC versus ABAC, for the access-control models the lesson names.
- [Top 5 Most-Used Deployment Strategies](https://www.youtube.com/watch?v=AWVTKBUnoIg) — ByteByteGo, English. Common deployment strategies, including the blue-green and canary patterns in the lesson’s pipeline section.
