---
layout: post
title: 09-03 Services, Deployments, and Networking
chapter: '09'
order: 4
owner: Nguyen Le Linh
lang: en
categories:
- chapter09
lesson_type: required
---

While Pods are the fundamental unit in Kubernetes, they are ephemeral and can be replaced at any time. Services provide stable network endpoints, and Deployments manage pod replicas and updates. This lecture covers these essential Kubernetes objects.

## The Networking Challenge

### Problem: Pods are Ephemeral

Pods have dynamic, changing IP addresses:

```
Pod nginx-abc (IP: 10.244.1.5) → Crashes
Pod nginx-xyz (IP: 10.244.2.8) → Created as replacement

How do clients find the new pod?
How do we load balance across multiple pods?
```

### Communication Challenges

1. **Between pods**: Using hardcoded IPs fails when pods are rescheduled
2. **From outside**: Need to track all pods providing a service and load balance
3. **Service discovery**: How to find which pods provide which services?

## Kubernetes Services

### What is a Service?

A **Service** is:
- An **abstraction** defining a logical set of pods
- A **stable network endpoint** (IP and DNS name)
- A **load balancer** across pod backends
- **Durable** - survives pod restarts and rescheduling

```
┌─────────────────────────────────────────┐
│  Service: web-service                   │
│  IP: 10.96.28.176 (stable)              │
│  DNS: web-service.default.svc.cluster.local │
└────────────┬────────────────────────────┘
             │ Load balances to:
      ┌──────┼──────┬──────────┐
      ▼      ▼      ▼          ▼
   [Pod 1][Pod 2][Pod 3]  [Pod 4]
   10.244.1.5  10.244.1.6  10.244.2.7  10.244.2.8
```

### Service Characteristics

- **Static cluster IP**: Allocated on creation, doesn't change
- **Static DNS name**: `<service-name>.<namespace>.svc.cluster.local`
- **Label selector**: Selects pods to route traffic to
- **Load balancing**: Distributes traffic across healthy pods
- **Service discovery**: Automatic DNS registration

### Service Specification

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: nginx
    env: prod
  ports:
  - protocol: TCP
    port: 80        # Service port
    targetPort: 8080 # Pod port
  type: ClusterIP
```

## Service Types

Kubernetes provides four types of services:

### 1. ClusterIP (Default)

**Purpose**: Internal cluster communication only

- **Accessible** only within the cluster
- **Use case**: Backend services, databases
- **Load balancing**: Across all selected pods

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080
```

**DNS Resolution:**
```bash
# From any pod in the cluster
curl http://backend-service:8080
# or
curl http://backend-service.default.svc.cluster.local:8080
```

### 2. NodePort

**Purpose**: Expose service on each node's IP at a static port

- **Extends** ClusterIP
- **Exposes** port on every node (30000-32767 range)
- **Use case**: Development, testing, simple external access

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 32410  # Optional, auto-assigned if not specified
```

**Access:**
```bash
# From outside cluster
curl http://<any-node-ip>:32410
```

```
External Client
      │
      ▼
Node IP:32410 ──┐
Node IP:32410 ──┼→ Service → Pods
Node IP:32410 ──┘
```

### 3. LoadBalancer

**Purpose**: Expose service via cloud provider's load balancer

- **Extends** NodePort
- **Provisions** external load balancer (AWS ELB, GCP LB, Azure LB)
- **Use case**: Production external access
- **Requires**: Cloud provider support

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
```

**Result:**
```
External Load Balancer (172.17.18.43)
      │
      ▼
NodePort (32410)
      │
      ▼
Service (10.96.28.176)
      │
      ▼
Pods
```

### 4. ExternalName

**Purpose**: Map service to external DNS name

- **No proxy** or load balancing
- **Returns** CNAME record
- **Use case**: Access external services with Kubernetes DNS

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: database.example.com
```

## Service Discovery

Kubernetes provides automatic service discovery via DNS:

### DNS Names

Every service gets a DNS name:

```
<service-name>.<namespace>.svc.cluster.local
```

**Examples:**
```
web-service.default.svc.cluster.local
database.production.svc.cluster.local
api.staging.svc.cluster.local
```

### Using Service Discovery

```yaml
# Pod can reference service by name
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: myapp
    env:
    - name: BACKEND_URL
      value: "http://backend-service:8080"
```

### kube-proxy

**kube-proxy** implements Services:

- Runs on **every node**
- Maintains **iptables rules** (or IPVS)
- **Routes** traffic to pod backends
- **Load balances** across pods

```
Client Request
      │
      ▼
kube-proxy (iptables rules)
      │
      ├→ Pod 1 (33% traffic)
      ├→ Pod 2 (33% traffic)
      └→ Pod 3 (34% traffic)
```

## Fundamental Networking Rules

Kubernetes networking follows these principles:

1. **All containers within a pod** can communicate unimpeded
2. **All pods can communicate** with all other pods without NAT
3. **All nodes can communicate** with all pods (and vice versa) without NAT
4. **The IP a pod sees itself as** is the same IP others see it as

## Deployments

### The Problem with Bare Pods

Managing pods directly is problematic:

- **No automatic restart** if pod dies
- **No scaling** - must create each pod manually
- **No rolling updates** - must manually replace pods
- **No rollback** capability

### What is a Deployment?

A **Deployment** provides:

- **Declarative updates** for pods
- **Replica management** - ensures desired number of pods
- **Rolling updates** - zero-downtime deployments
- **Rollback** capability
- **Scaling** - easy scale up/down

```
Deployment
    │
    ├→ ReplicaSet (v1)
    │     ├→ Pod 1
    │     ├→ Pod 2
    │     └→ Pod 3
    │
    └→ ReplicaSet (v2) - new version
          ├→ Pod 4
          ├→ Pod 5
          └→ Pod 6
```

### Deployment Specification

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  revisionHistoryLimit: 3
  
  selector:
    matchLabels:
      app: nginx
  
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

### Key Deployment Fields

| Field | Purpose |
|-------|---------|
| `replicas` | Desired number of pod instances |
| `selector` | Label selector for pods |
| `template` | Pod template for creating pods |
| `strategy` | Update strategy (RollingUpdate or Recreate) |
| `revisionHistoryLimit` | Number of old ReplicaSets to retain |

## Update Strategies

### 1. RollingUpdate (Default)

Gradually replaces old pods with new ones:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # Max pods above desired count
    maxUnavailable: 0  # Max pods below desired count
```

**Process:**
```
Initial: [v1] [v1] [v1]
Step 1:  [v1] [v1] [v1] [v2]  (maxSurge: 1)
Step 2:  [v1] [v1] [v2]       (remove old)
Step 3:  [v1] [v1] [v2] [v2]
Step 4:  [v1] [v2] [v2]
Step 5:  [v1] [v2] [v2] [v2]
Final:   [v2] [v2] [v2]
```

### 2. Recreate

Kills all old pods before creating new ones:

```yaml
strategy:
  type: Recreate
```

**Process:**
```
Initial: [v1] [v1] [v1]
Step 1:  [ ]  [ ]  [ ]   (kill all)
Step 2:  [v2] [v2] [v2]  (create new)
```

> **Downtime**: Recreate strategy causes downtime

## Working with Deployments

### Creating Deployments

```bash
# From YAML
kubectl apply -f deployment.yaml

# Imperative
kubectl create deployment nginx --image=nginx --replicas=3

# With labels
kubectl create deployment nginx --image=nginx --replicas=3 \
  --labels=app=web,env=prod
```

### Viewing Deployments

```bash
# List deployments
kubectl get deployments

# Detailed information
kubectl describe deployment nginx-deployment

# View rollout status
kubectl rollout status deployment nginx-deployment

# View rollout history
kubectl rollout history deployment nginx-deployment
```

### Updating Deployments

```bash
# Update image
kubectl set image deployment/nginx-deployment nginx=nginx:1.22 --record

# Edit deployment
kubectl edit deployment nginx-deployment

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5

# Apply updated YAML
kubectl apply -f deployment.yaml
```

### Rolling Back

```bash
# Rollback to previous version
kubectl rollout undo deployment nginx-deployment

# Rollback to specific revision
kubectl rollout undo deployment nginx-deployment --to-revision=2

# Pause rollout
kubectl rollout pause deployment nginx-deployment

# Resume rollout
kubectl rollout resume deployment nginx-deployment
```

## ReplicaSets

### What is a ReplicaSet?

A **ReplicaSet** ensures a specified number of pod replicas are running:

- **Created automatically** by Deployments
- **Maintains** desired number of pods
- **Replaces** failed pods
- **Adopts** existing pods with matching labels

> **Best Practice**: Don't create ReplicaSets directly. Use Deployments instead.

### ReplicaSet Specification

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-example
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
      env: prod
  template:
    metadata:
      labels:
        app: nginx
        env: prod
    spec:
      containers:
      - name: nginx
        image: nginx:stable-alpine
        ports:
        - containerPort: 80
```

### ReplicaSet Behavior

```bash
# If you have 3 pods and delete one
kubectl delete pod nginx-abc

# ReplicaSet immediately creates a new pod
# to maintain replicas: 3
```

**Quarantine pods**: Remove label to exclude from ReplicaSet
```bash
kubectl label pod nginx-abc app-
```

## Labels and Selectors

### Labels

**Labels** are key-value pairs attached to objects:

```yaml
metadata:
  labels:
    app: nginx
    env: prod
    tier: frontend
    version: v1.2
```

**Characteristics:**
- **Not unique** - multiple objects can have same labels
- **Flexible** - add/remove anytime
- **Queryable** - select objects by labels

### Selectors

**Selectors** filter objects by labels:

#### Equality-Based

```yaml
selector:
  matchLabels:
    app: nginx
    env: prod
```

```bash
# kubectl with label selector
kubectl get pods -l app=nginx
kubectl get pods -l app=nginx,env=prod
kubectl get pods -l app!=nginx
```

#### Set-Based

```yaml
selector:
  matchExpressions:
  - key: env
    operator: In
    values: ["prod", "staging"]
  - key: tier
    operator: NotIn
    values: ["cache"]
  - key: app
    operator: Exists
```

**Operators**: `In`, `NotIn`, `Exists`, `DoesNotExist`

## Complete Example: Web Application

```yaml
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
---
# Service
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```

**Deploy:**
```bash
kubectl apply -f web-app.yaml

# Check deployment
kubectl get deployments
kubectl get pods
kubectl get services

# Access application
kubectl get service web-service
# Use EXTERNAL-IP to access
```

## Summary

Kubernetes Services and Deployments provide production-ready application management:

- **Services** provide stable network endpoints and load balancing
- **Four service types**: ClusterIP, NodePort, LoadBalancer, ExternalName
- **Deployments** manage pod replicas and updates
- **Rolling updates** enable zero-downtime deployments
- **ReplicaSets** ensure desired pod count
- **Labels and selectors** enable flexible object grouping

Together, these objects form the foundation of Kubernetes application deployment.

## Further Reading

- [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
