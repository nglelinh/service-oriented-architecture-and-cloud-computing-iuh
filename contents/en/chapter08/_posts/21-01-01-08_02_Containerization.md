---
layout: post
title: 08-02 Containerization and Linux Primitives
chapter: '08'
order: 3
owner: Nguyen Le Linh
lang: en
categories:
- chapter08
lesson_type: required
---

Here is one of the best-kept secrets in systems engineering: **there is no such thing as a "container" inside the Linux kernel!** What developers call a container is actually an ordinary Linux process wrapped in two clever kernel primitives—**Namespaces** (controlling *what the process can see*: its own PID tree, network interface, and filesystem mount) and **Control Groups / cgroups** (controlling *how much CPU and RAM it can consume*). By sharing the single host kernel instead of booting twenty duplicate guest OSes, containers deliver VM-like portability at near-bare-metal density and sub-second startup.

## Key ideas: Linux Primitives Behind Every Container

1. **Shared Kernel Architecture:** Unlike VMs that virtualize hardware and boot a full guest OS per instance, containers run directly on the host Linux kernel—eliminating gigabytes of duplicate OS memory and slashing boot times from ~30 seconds to < 500 milliseconds.
2. **Linux Namespaces (Isolation of View):** Give each process its own private illusion of the system: `pid` (sees itself as PID 1), `net` (own IP, ports, routing table), `mnt` (own root filesystem `/`), `uts` (hostname), `ipc`, and `user` (root inside container ≠ root on host).
3. **Control Groups — cgroups v2 (Isolation of Resources):** Enforce hard hardware quotas (`memory.max`, `cpu.max`, I/O throttling) so one runaway container with a memory leak gets OOM-killed cleanly without starving neighbors on the same host.

### Worked Example (Step by Step): Comparing VM vs. Container Density on a 64-GB Host

Suppose you need to pack **20 identical microservices**, each requiring **500 MB** of application RAM, onto a single **64 GB** Linux server (where a full OS kernel + systemd consumes **1.5 GB RAM**):

| Packaging Model | OS Overhead | App RAM (20 services) | Total RAM Used | Startup Time |
|---|---|---|---|---|
| **20 Virtual Machines** | $$20 	imes 1.5	ext{ GB} = \mathbf{30.0	ext{ GB}}$$ | $$20 	imes 0.5	ext{ GB} = 10.0	ext{ GB}$$ | **$$40.0	ext{ GB}$$** (62.5% of host) | ~30 s each |
| **20 Linux Containers** | $$1 	imes 1.5	ext{ GB} = \mathbf{1.5	ext{ GB}}$$ (shared!) | $$20 	imes 0.5	ext{ GB} = 10.0	ext{ GB}$$ | **$$11.5	ext{ GB}$$** (18.0% of host) | **< 1 s each** |

Sharing the host kernel saves **28.5 GB of RAM** (**3.5× higher density**)—meaning the same 64 GB host can actually run over **120 containers** instead of capping out around 30 VMs!

### Explanation: Understanding Container Isolation Boundaries Honestly

1. **Remember the kernel-compatibility rule:** because containers share the host kernel, a Linux container cannot run natively on a Windows kernel without a thin hidden Linux VM (which is what Docker Desktop runs under WSL2/HyperKit).
2. **Enforce cgroup memory limits on every production container:** without `memory.max`, a single container leak can trigger the host kernel's OOM killer against critical system daemons.
3. **Know why container security is weaker than VM security:** all containers issue syscalls to the *same* host kernel; a kernel vulnerability can break out of namespaces, which is why production platforms layer `seccomp` syscall filters, non-root `user` namespaces, and sometimes microVMs.
4. **Trace overlay filesystems (OverlayFS):** containers share read-only image layers on disk and mount one tiny copy-on-write top layer—so launching 20 copies of a 200 MB image consumes only 200 MB of disk, not 4 GB!

**Quick check.** If a process inside a container runs `ps aux` and sees itself as `PID 1`, while the host admin running `ps aux` sees that exact same process as `PID 18432`, which Linux primitive creates that dual view? The **`pid` namespace**!

## Motivation for Containerization

### Limitations of VMs

Virtual machines are powerful but have drawbacks:

- **High runtime overhead**: Each VM runs a complete OS
- **Slow startup**: Booting an OS takes time
- **Resource intensive**: Multiple OS copies consume significant memory
- **Large image sizes**: VM images are typically gigabytes in size

### The Container Solution

**What if we could sandbox applications but share the OS kernel?**

Containers provide:
- ✓ **Faster scaling**: Start in seconds instead of minutes
- ✓ **Lower overhead**: Share kernel, only package app dependencies
- ✓ **Smaller footprint**: Container images are megabytes, not gigabytes
- ✓ **Higher density**: Run more containers than VMs on same hardware

This enables:
- Different software architectures (microservices)
- New development practices (DevOps, CI/CD)
- More efficient resource utilization

## Containers vs Virtual Machines

### Architecture Comparison

**Traditional VM-based Infrastructure:**
```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│    App A     │  │    App B     │  │    App C     │
├──────────────┤  ├──────────────┤  ├──────────────┤
│   Bins/Libs  │  │   Bins/Libs  │  │   Bins/Libs  │
├──────────────┤  ├──────────────┤  ├──────────────┤
│   Guest OS   │  │   Guest OS   │  │   Guest OS   │
├──────────────┴──┴──────────────┴──┴──────────────┤
│              Hypervisor                           │
├───────────────────────────────────────────────────┤
│              Host OS (optional)                   │
├───────────────────────────────────────────────────┤
│              Physical Hardware                    │
└───────────────────────────────────────────────────┘
```

**Container Infrastructure:**
```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│    App A     │  │    App B     │  │    App C     │
├──────────────┤  ├──────────────┤  ├──────────────┤
│   Bins/Libs  │  │   Bins/Libs  │  │   Bins/Libs  │
├──────────────┴──┴──────────────┴──┴──────────────┤
│           Container Runtime (Docker)              │
├───────────────────────────────────────────────────┤
│              Host OS / Kernel                     │
├───────────────────────────────────────────────────┤
│              Physical Hardware                    │
└───────────────────────────────────────────────────┘
```

### Key Differences

| Aspect | Virtual Machines | Containers |
|--------|------------------|------------|
| **OS** | Complete OS per VM | Shared kernel |
| **Size** | Gigabytes | Megabytes |
| **Startup** | Minutes | Seconds |
| **Isolation** | Strong (hardware-level) | Process-level |
| **Overhead** | Higher | Lower |
| **Density** | 10s per host | 100s per host |
| **Portability** | Less portable | Highly portable |

### What Containers Share

Containers have a **separate view** of:
- Root filesystem
- Libraries and utilities
- Process tree
- Users and permissions
- Networking stack
- IPC endpoints

But they **share**:
- The same OS kernel
- System calls interface
- Hardware resources (managed by kernel)

## Big Idea: Less OS Overhead

```
┌─────────────────────────────────────────────┐
│  Traditional VMs: 3 VMs on one host         │
│  - 3 complete OS copies                     │
│  - High memory usage                        │
│  - Slower startup                           │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│  Containers: 10+ containers on same host    │
│  - 1 shared kernel                          │
│  - Lower memory usage                       │
│  - Instant startup                          │
└─────────────────────────────────────────────┘
```

## Important Constraint: Kernel Compatibility

> [!IMPORTANT]
> **Container-Kernel Dependency**
> 
> - A **Linux container** needs a **Linux kernel**
> - A **Windows container** needs a **Windows kernel**
> - You **cannot** run a Windows container on a Linux host natively (or vice versa)

### Solutions

**On Windows:**
- Use **Windows Subsystem for Linux (WSL 2)** to run Linux containers
- Use **Hyper-V** to run a Linux VM that hosts containers
- Use **Windows Server** to run native Windows containers

**On macOS:**
- Docker Desktop uses a lightweight Linux VM
- Containers run inside this VM

**On Linux:**
- Native container support
- Best performance and compatibility

## Linux Primitives for Containers

Containers are built on two fundamental Linux kernel mechanisms:

### 1. Namespaces

**Purpose**: Provide isolated view of global resources

- Group of processes see only their "slice" of a resource
- Other processes cannot see or interfere with this slice
- Creates the **isolation** aspect of containers

### 2. Control Groups (Cgroups)

**Purpose**: Control and limit resource usage

- Set limits on CPU, memory, I/O for process groups
- Prevent resource exhaustion
- Enables **resource management** for containers

```
┌─────────────────────────────────────────┐
│  Namespaces + Cgroups = Container       │
│                                         │
│  Namespaces → Isolation                 │
│  Cgroups    → Resource Limits           │
└─────────────────────────────────────────┘
```

## Namespaces in Detail

Linux provides several types of namespaces to isolate different resources:

### Types of Namespaces

#### 1. Mount Namespace
- **Isolates**: Filesystem mount points
- **Effect**: Each namespace has its own view of the filesystem hierarchy
- **Use**: Containers have their own root filesystem

```bash
# Container sees:
/
├── bin/
├── lib/
├── app/
└── ...

# Host sees different filesystem
```

#### 2. PID Namespace
- **Isolates**: Process ID number space
- **Effect**: First process in namespace gets PID 1
- **Use**: Containers have their own process tree

```
Host PID Namespace:
  PID 1234 → Container init process
  
Container PID Namespace:
  PID 1 → Same process (appears as init)
  PID 2 → First child process
```

#### 3. Network Namespace
- **Isolates**: Network resources (IP addresses, routing tables, ports)
- **Effect**: Each namespace has its own network stack
- **Use**: Containers can have their own IP addresses

```bash
# Container can bind to port 80
# Host can also bind to port 80
# No conflict because different network namespaces
```

#### 4. UTS Namespace
- **Isolates**: Hostname and domain name
- **Effect**: Each namespace can have different hostname
- **Use**: Containers have unique hostnames

#### 5. User Namespace
- **Isolates**: User and group ID number space
- **Effect**: Process can be root in container but unprivileged on host
- **Use**: Enhanced security (rootless containers)

```
Container: UID 0 (root)
    ↓ mapped to
Host: UID 1000 (regular user)
```

#### 6. IPC Namespace
- **Isolates**: Inter-Process Communication endpoints
- **Effect**: Separate message queues, semaphores, shared memory
- **Use**: Prevent IPC interference between containers

### Namespace API

Three key system calls:

#### 1. `clone()`
```c
// Create new process in new namespace
clone(child_func, stack, CLONE_NEWPID | CLONE_NEWNET, args);
```
- More general version of `fork()`
- Flags specify what to share vs create new

#### 2. `setns()`
```c
// Join an existing namespace
setns(namespace_fd, CLONE_NEWNET);
```
- Allows process to enter existing namespace
- Useful for debugging containers

#### 3. `unshare()`
```c
// Create new namespace for calling process
unshare(CLONE_NEWPID | CLONE_NEWNET);
```
- Calling process moves to new namespace
- Equivalent to `fork()` + `clone()`

### Viewing Namespaces

Namespaces are represented as files in `/proc`:

```bash
# View namespaces for your current shell
ls -l /proc/$$/ns

# Output:
lrwxrwxrwx 1 user user 0 Jan  5 10:00 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 user user 0 Jan  5 10:00 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 user user 0 Jan  5 10:00 net -> 'net:[4026531992]'
lrwxrwxrwx 1 user user 0 Jan  5 10:00 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 user user 0 Jan  5 10:00 user -> 'user:[4026531837]'
lrwxrwxrwx 1 user user 0 Jan  5 10:00 uts -> 'uts:[4026531838]'
```

The ID (e.g., `[4026531839]`) uniquely identifies the namespace. Processes with the same ID share that namespace.

## Control Groups (Cgroups)

### Purpose

Cgroups allow you to:
- **Limit** resources (CPU, memory, I/O)
- **Prioritize** resource allocation
- **Account** for resource usage
- **Control** which CPUs/memory nodes processes can use

### Cgroup Hierarchies

Resources can be organized hierarchically:

```
All CPU Resources
├── CPU-Faculty (40%)
│   ├── Fac-Web (50%)
│   └── Fac-Non-Web (50%)
└── CPU-Students (60%)
    ├── Student-Web (50%)
    └── Student-Non-Web (50%)
```

### Creating Cgroups

Managed via filesystem (no new system calls):

```bash
# Cgroup filesystem mounted at
/sys/fs/cgroup/

# Create a cgroup for limiting memory
mkdir /sys/fs/cgroup/memory/mycontainer

# Set memory limit to 512MB
echo 536870912 > /sys/fs/cgroup/memory/mycontainer/memory.limit_in_bytes

# Add process to cgroup
echo $PID > /sys/fs/cgroup/memory/mycontainer/tasks
```

### Resource Types

Cgroups can control:

- **CPU**: CPU time, CPU shares, CPU quotas
- **Memory**: Memory limits, swap limits
- **Block I/O**: I/O bandwidth limits
- **Network**: Network priority (with tc)
- **Devices**: Device access control
- **CPUsets**: Which CPUs/memory nodes to use

## How to Create a Container

Putting it all together:

```
1. Create namespaces for isolation
   ├── PID namespace (process isolation)
   ├── Mount namespace (filesystem isolation)
   ├── Network namespace (network isolation)
   └── User namespace (security)

2. Create and configure cgroups for resource limits
   ├── CPU limits
   ├── Memory limits
   └── I/O limits

3. Create root filesystem
   ├── Base OS files (minimal)
   ├── Application binaries
   ├── Required libraries
   └── Configuration files

4. Enter namespaces, mount rootfs, register in cgroups

5. Execute application or shell

→ Your application is now running in a "container"!
```

## Container Frameworks

### Why Use Frameworks?

Creating containers manually is complex. Frameworks automate:
- Namespace and cgroup configuration
- Filesystem management
- Network setup
- Image distribution

### LXC (Linux Containers)

- **General-purpose** container framework
- Provides standard OS shell interface
- Acts like a lightweight VM
- Uses namespaces and cgroups under the hood

### Docker

- **Application-focused** containers
- Optimized to run a single application
- Easy packaging and distribution
- Dockerfile for reproducible builds
- Docker Hub for sharing images

### Comparison

| Feature | LXC | Docker |
|---------|-----|--------|
| **Purpose** | System containers | Application containers |
| **Interface** | Full OS environment | Single application |
| **Use Case** | VM replacement | App deployment |
| **Ecosystem** | Smaller | Large (Docker Hub) |

## What Containers CAN Do

✓ **Run different Linux distributions** on the same host
  - Ubuntu container on Red Hat host
  - Alpine container on Ubuntu host

✓ **Run applications with different dependencies**
  - Python 3.9 in one container
  - Python 3.11 in another
  - Even if host has no Python installed

✓ **Use the host's hardware and system calls**
  - Access network interfaces
  - Use GPUs (with proper drivers)
  - Access storage

✓ **Provide isolation and security**
  - Process isolation
  - Filesystem isolation
  - Network isolation

## Summary

Containers provide **lightweight isolation** with lower overhead than VMs:

- **Share the same kernel** but have different root filesystems
- **Built on Linux primitives**:
  - Namespaces for isolation
  - Cgroups for resource limits
- **Frameworks like Docker and LXC** provide user-friendly interfaces
- **Enable modern architectures**: microservices, serverless, cloud-native

In the next lecture, we'll explore **Docker** in depth, including how to create, manage, and deploy containerized applications.

## Video tham khảo

- **xem trước:** [Containers: cgroups, Linux kernel namespaces, ufs, Docker, and intro to Kubernetes pods](https://www.youtube.com/watch?v=el7768BNUPw) — Sean, English. Watch first: cgroups, namespaces, and how they differ from a virtual machine, which is this lesson’s Linux-primitives core.
- [Cgroups, namespaces, and beyond: what are containers made from?](https://www.youtube.com/watch?v=sK5i-N34im8) — Docker, English. Docker’s own talk on what containers are made from. It is a long conference session; watch it as a whole.
- [What Is Docker - and why it’s EVERYWHERE](https://www.youtube.com/watch?v=W5fK8BmgBow) — Techquickie, English. A short picture of Docker after the kernel primitives, so the lesson’s “container solution” has a product name.

## Further Reading

- [Namespaces in operation (lwn.net)](https://lwn.net/Articles/531114/)
- [Documentation/cgroups/cgroups.txt](https://lwn.net/Articles/524935/)
- [Linux Containers (LXC)](https://linuxcontainers.org/)
