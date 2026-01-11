---
layout: post
title: 14-01 Infrastructure as Code with Terraform
chapter: '14'
order: 2
owner: Nguyen Le Linh
lang: en
categories:
- chapter14
lesson_type: required
---

Infrastructure as Code (IaC) has revolutionized how we manage and deploy cloud resources. Instead of manual configuration, we define our infrastructure in software, bringing the power of version control and automation to operations.

## The Problem with Manual Deployment

### Distributed Applications are Complex
- **Configuration Sensitive**: Database servers need different settings than web servers.
- **Updated Continuously**: Code and patches deployed daily or hourly.
- **Human Factors**: Manual steps lead to "operator error".
- **Scale**: Running on tens, hundreds, or thousands of nodes.

### Approaches to Deployment

1. **Manual Setup**
   - Click functionality in the web portal.
   - **Does this scale?** Clearly no.
   - Error-prone and not reproducible.

2. **Custom Scripts**
   - Use cloud provider APIs (e.g., AWS Boto3) to create machines.
   - Programmatically SSH to run tasks.
   - **Does this scale?** Maybe, but hard to maintain. "Why reinvent the wheel?"

3. **Infrastructure as Code (IaC)**
   - Declare infrastructure in a specific format (code).
   - IaC framework deploys/updates the cloud infrastructure.
   - **Does this scale?** Yes!

## IaC Concepts

### Declarative vs. Imperative

- **Declarative**: Define the **target state** (what you want).
  - *Example*: "I want 3 VMs and a Load Balancer."
  - The system figures out how to get there.

- **Imperative**: Define **how** to change the state (steps to take).
  - *Example*: "Create VM 1. Create VM 2. Create VM 3. Create Load Balancer. Add VMs to Load Balancer."

- **Intelligent**: Define relationships and constraints; the system figures out the updates.

### Push vs. Pull

- **Push**: Central server pushes configuration to child servers (e.g., Ansible, Terraform).
- **Pull**: Child servers periodically check central server for configuration (e.g., Puppet, Chef).

### IaC Tools Comparison

| Tool | Style | Method | Mechanism |
|------|-------|--------|-----------|
| **Ansible** | Declarative/Imperative | Push | SSH (Agentless) |
| **Puppet** | Declarative | Pull | Agent-based |
| **Chef** | Imperative | Pull | Agent-based |
| **Terraform** | Declarative | Push | API (Agentless) |

## Terraform by HashiCorp

**Terraform** is the industry standard for cloud provisioning.

### Key Features
- **Open Source**: Huge community.
- **Platform Agnostic**: Supports AWS, Azure, GCP, Kubernetes, and hundreds more.
- **Stateful**: Maintains a `tfstate` file to know what currently exists.
- **Graph-Based**: Understands dependencies (e.g., create VPC *before* creating Subnet).

### Workflow

1. **Write Code**: Define resources in `.tf` files.
2. **Plan**: `terraform plan` compares code to current state and shows what *will* happen.
3. **Apply**: `terraform apply` executes the API calls to make the changes real.

### Basic Syntax (HCL - HashiCorp Configuration Language)

Terraform uses HCL, which is designed to be both machine-readable and human-readable.

#### Resource Definition

```hcl
# <Resource Type>     <Local Name>
resource "aws_instance" "web_server" {
  # Arguments
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "MyWebServer"
  }
}
```

- **Resource Type**: Defined by the provider (e.g., `aws_instance`).
- **Local Name**: Used to reference this resource elsewhere in code (e.g., `aws_instance.web_server.public_ip`).

#### Variables

```hcl
variable "region" {
  type    = string
  default = "us-east-1"
}

provider "aws" {
  region = var.region
}
```

#### Interpolation

Referencing values from other resources:

```hcl
resource "aws_eip" "lb" {
  # Reference the instance ID from the resource above
  instance = aws_instance.web_server.id
  vpc      = true
}
```

### Module Structure

Standard file organization:

- **main.tf**: Core resource definitions.
- **variables.tf**: Input variable definitions (parameters).
- **outputs.tf**: Output values (e.g., Load Balancer IP).
- **providers.tf**: Provider configuration (AWS, Azure versioning).

### Essential Commands

1. **`terraform init`**
   - Downloads provider plugins (e.g., AWS provider code).
   - Initializes the backend (state storage).

2. **`terraform plan`**
   - **Dry run**: Shows execution plan.
   - Does NOT make changes.
   - Vital for safety ("measure twice, cut once").

3. **`terraform apply`**
   - Executes the plan.
   - Creates/Updates/Deletes resources.
   - Can be destructive!

4. **`terraform destroy`**
   - Removes all resources managed by the configuration.

## Use Cases for Terraform

1. **Multi-Tier Applications**:
   - Define Web, App, and Database layers together.
   - Pass dependencies (DB Connection String) automatically to App layer.

2. **Disposable Environments**:
   - Spin up a full "Staging" environment that mimics "Production" exactly.
   - Destroy it when done to save money.

3. **Multi-Cloud Deployments**:
   - Manage AWS (Computing) and Cloudflare (DNS) in the same workflow.
   - Although code isn't 100% portable (AWS resources != Azure resources), the *workflow* is identical.

## Summary

Infrastructure as Code allows us to treat operations like software development:
- **Version Control** (Git) your infrastructure.
- **Peer Review** changes before applying.
- **Automate** testing and deployment.
- **Terraform** provides a powerful, declarative way to manage massive scale across any cloud provider.
