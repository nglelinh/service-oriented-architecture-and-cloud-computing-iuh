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
