---
layout: post
title: 09-01 Kubernetes Fundamentals and Architecture
chapter: '09'
order: 2
owner: Nguyen Le Linh
lang: en
categories:
- chapter09
lesson_type: required
---

Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. This lecture covers the fundamentals of Kubernetes and its architecture.

## What is Container Orchestration?

### The Challenge

As applications grow, managing containers manually becomes impossible:

- **Hundreds or thousands** of containers across multiple hosts
- **Dynamic scaling** based on load
- **Service discovery** - how do containers find each other?
- **Load balancing** across container instances
- **Health monitoring** and automatic recovery
- **Rolling updates** without downtime
- **Resource allocation** - which host should run which container?

### Orchestration Solution

An **orchestrator** manages and organizes both hosts and containers running on a cluster:

```
┌─────────────────────────────────────────────┐
│  Orchestrator (Kubernetes)                  │
│  ├── Resource allocation                    │
│  ├── Container scheduling                   │
│  ├── Health monitoring                      │
│  ├── Auto-scaling                           │
│  ├── Load balancing                         │
│  ├── Service discovery                      │
│  └── Rolling updates                        │
└─────────────────────────────────────────────┘
```

### Key Orchestrator Tasks

- **Manage networking and access** between containers
- **Track state** of containers and nodes
- **Scale services** up and down based on demand
- **Load balance** traffic across container instances
- **Relocate containers** when hosts become unresponsive
- **Service discovery** - automatic DNS and routing
- **Attribute storage** to containers (volumes, secrets)
- **Rolling updates** and rollbacks

## Orchestrator Options

### Kubernetes
- **Open-source** project from Google (now CNCF)
- Most popular and feature-rich
- Large ecosystem and community
- Runs on any infrastructure

### Docker Swarm
- **Integrated** into Docker platform
- Simpler to set up than Kubernetes
- Good for smaller deployments
- Less features than Kubernetes

### Apache Mesos
- **Cluster management** tool
- Container orchestration via Marathon plugin
- Can handle non-container workloads too
- More complex setup

> **Winner**: Kubernetes has become the industry standard

## What is Kubernetes?

### Origin and Meaning

- **Name**: Greek for "pilot" or "helmsman of a ship" (κυβερνήτης)
- **Abbreviation**: K8s (K + 8 letters + s)
- **Created by**: Google, based on internal Borg system
- **Open-sourced**: 2014
- **Governed by**: Cloud Native Computing Foundation (CNCF)

### Core Philosophy: Self-Healing

Kubernetes **always tries to steer the cluster to its desired state**:

```
You: "I want 3 healthy instances of redis to always be running."

Kubernetes: "Okay, I'll ensure there are always 3 instances up and running."

[One instance dies]

Kubernetes: "Oh look, one has died. I'm going to spin up a new one."
```

This is called the **reconciliation loop**:

1. **Desired state**: What you want (3 redis instances)
2. **Current state**: What actually exists (2 redis instances)
3. **Action**: Kubernetes takes action to match desired state (create 1 instance)
4. **Repeat**: Continuously monitor and adjust

## Kubernetes Architecture

Kubernetes uses a **master-worker architecture**:

```
┌─────────────────────────────────────────────────────────┐
│  Control Plane (Master Node)                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ API Server   │  │  Scheduler   │  │  Controller  │  │
│  │              │  │              │  │   Manager    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │  etcd (Distributed Key-Value Store)              │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
┌───────▼────────┐ ┌──────▼───────┐ ┌──────▼───────┐
│  Worker Node 1 │ │ Worker Node 2│ │ Worker Node 3│
│  ┌──────────┐  │ │  ┌──────────┐│ │  ┌──────────┐│
│  │ kubelet  │  │ │  │ kubelet  ││ │  │ kubelet  ││
│  ├──────────┤  │ │  ├──────────┤│ │  ├──────────┤│
│  │kube-proxy│  │ │  │kube-proxy││ │  │kube-proxy││
│  ├──────────┤  │ │  ├──────────┤│ │  ├──────────┤│
│  │Container │  │ │  │Container ││ │  │Container ││
│  │ Runtime  │  │ │  │ Runtime  ││ │  │ Runtime  ││
│  └──────────┘  │ │  └──────────┘│ │  └──────────┘│
│  [Pods...]     │ │  [Pods...]   │ │  [Pods...]   │
└────────────────┘ └──────────────┘ └──────────────┘
```

### Control Plane Components

#### 1. API Server
- **Central management** point for the cluster
- **RESTful API** for all operations
- **Authentication** and authorization
- **Validates** and processes API requests
- Only component that talks to etcd

#### 2. etcd
- **Distributed key-value store**
- Stores **all cluster state** and configuration
- **Highly available** and consistent
- **Source of truth** for cluster state

#### 3. Scheduler
- **Watches** for newly created pods with no assigned node
- **Selects** a node for the pod to run on
- **Considers** resource requirements, constraints, affinity rules
- **Does not** actually start the pod (kubelet does that)

#### 4. Controller Manager
- Runs **controller processes**
- Each controller watches for changes and takes action
- Examples:
  - **Node Controller**: Monitors node health
  - **Replication Controller**: Maintains correct number of pods
  - **Endpoints Controller**: Populates endpoint objects
  - **Service Account Controller**: Creates default accounts

### Worker Node Components

#### 1. kubelet
- **Agent** running on each node
- **Ensures** containers are running in pods
- **Communicates** with API server
- **Reports** node and pod status
- **Executes** pod specifications (PodSpecs)

#### 2. kube-proxy
- **Network proxy** running on each node
- **Maintains** network rules
- **Enables** communication to pods
- **Implements** Service abstraction
- **Load balances** across pod backends

#### 3. Container Runtime
- **Software** responsible for running containers
- Examples: **Docker**, **containerd**, **CRI-O**
- Implements **Container Runtime Interface (CRI)**

## Core Concepts

### Desired State Management

Kubernetes operates on a **declarative model**:

```yaml
# You declare what you want
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3  # I want 3 instances
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

Kubernetes **continuously works** to achieve and maintain this state:

- If a pod crashes → Start a new one
- If a node fails → Reschedule pods to healthy nodes
- If you update the image → Rolling update to new version
- If load increases → Scale up (with autoscaling)

### Reconciliation Loop

```
┌─────────────────────────────────────────┐
│  1. Observe current state               │
│     (What is actually running?)         │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│  2. Compare with desired state          │
│     (What should be running?)           │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│  3. Take action to reconcile            │
│     (Create, update, or delete)         │
└────────────┬────────────────────────────┘
             │
             ▼
        [Repeat continuously]
```

This creates a **goal-driven, self-healing system**.

## Kubernetes Objects

Everything in Kubernetes is represented as an **object**:

### Object Structure

All Kubernetes objects have:

```yaml
apiVersion: v1  # API version
kind: Pod       # Object type
metadata:       # Object metadata
  name: my-pod
  labels:
    app: web
spec:           # Desired state
  containers:
  - name: nginx
    image: nginx
```

### Common Object Types

| Object | Purpose |
|--------|---------|
| **Pod** | Smallest deployable unit, runs containers |
| **Service** | Stable network endpoint for pods |
| **Deployment** | Manages pod replicas and updates |
| **ReplicaSet** | Ensures desired number of pod replicas |
| **ConfigMap** | Configuration data |
| **Secret** | Sensitive data (passwords, tokens) |
| **Namespace** | Virtual cluster for isolation |
| **PersistentVolume** | Storage resource |

## Working with Kubernetes

### kubectl - The Kubernetes CLI

**kubectl** is the command-line tool for Kubernetes:

```bash
# Get cluster information
kubectl cluster-info

# List nodes
kubectl get nodes

# List all pods
kubectl get pods --all-namespaces

# Create resources from YAML
kubectl apply -f deployment.yaml

# Get detailed information
kubectl describe pod my-pod

# View logs
kubectl logs my-pod

# Execute command in pod
kubectl exec -it my-pod -- /bin/bash

# Delete resources
kubectl delete pod my-pod
```

### Imperative vs Declarative

**Imperative** (tell Kubernetes what to do):
```bash
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=3
kubectl expose deployment nginx --port=80
```

**Declarative** (tell Kubernetes what you want):
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  # ... rest of spec
```

```bash
kubectl apply -f deployment.yaml
```

> **Best Practice**: Use declarative approach for production (version control, reproducibility)

## Kubernetes Distributions

Kubernetes can run in many environments:

### Cloud-Managed Kubernetes

- **Amazon EKS** (Elastic Kubernetes Service)
- **Google GKE** (Google Kubernetes Engine)
- **Azure AKS** (Azure Kubernetes Service)
- **DigitalOcean Kubernetes**

**Advantages**: Managed control plane, automatic updates, integrated with cloud services

### Self-Managed Kubernetes

- **kubeadm**: Official tool for cluster setup
- **kops**: Kubernetes Operations (AWS focused)
- **Kubespray**: Ansible-based deployment

**Advantages**: Full control, can run anywhere

### Local Development

- **minikube**: Single-node cluster for local development
- **kind**: Kubernetes in Docker
- **k3s**: Lightweight Kubernetes
- **Docker Desktop**: Includes Kubernetes

## Benefits of Kubernetes

✓ **Portability**: Run anywhere (cloud, on-prem, hybrid)  
✓ **Scalability**: Handle massive scale (Google runs billions of containers)  
✓ **High Availability**: Automatic failover and recovery  
✓ **Resource Efficiency**: Optimal bin-packing of containers  
✓ **Declarative Configuration**: Infrastructure as code  
✓ **Extensibility**: Plugin architecture, custom resources  
✓ **Ecosystem**: Huge community, tools, and integrations  

## Challenges

⚠ **Complexity**: Steep learning curve  
⚠ **Overhead**: Requires resources for control plane  
⚠ **Overkill**: May be too complex for simple applications  
⚠ **Networking**: Can be complex to configure  
⚠ **Storage**: Stateful applications require careful planning  

## Summary

Kubernetes is a powerful container orchestration platform that:

- **Automates** deployment, scaling, and management of containers
- Uses a **master-worker architecture** with control plane and worker nodes
- Operates on **desired state** and **reconciliation loops**
- Provides **self-healing** and **auto-scaling** capabilities
- Has become the **industry standard** for container orchestration

In the next lecture, we'll dive deep into **Pods** - the fundamental building block of Kubernetes applications.

## Further Reading

- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Kubernetes Architecture](https://kubernetes.io/docs/concepts/architecture/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
