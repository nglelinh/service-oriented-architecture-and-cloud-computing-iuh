---
layout: post
title: 08-01 Virtualization Fundamentals
chapter: '08'
order: 2
owner: Nguyen Le Linh
lang: en
categories:
- chapter08
lesson_type: required
---

Before virtualization went mainstream in the early 2000s, enterprise data centers suffered from a shocking inefficiency: because teams refused to install two applications on the same OS for fear of library conflicts, thousands of expensive servers sat running at **10–15% CPU utilization**! **Hardware virtualization**—led by the **hypervisor**—broke the 1-to-1 bond between physical silicon and operating systems, creating the foundational isolation layer that made public cloud computing (EC2, Azure VMs, GCE) possible.

## Background Knowledge Map: Hypervisors, Hardware Assist, and Isolation

1. **The Hypervisor (Virtual Machine Monitor — VMM):** Software or firmware that multiplexes physical CPU, RAM, disk, and NICs across multiple independent Guest Operating Systems while enforcing strict hardware isolation.
2. **Type-1 (Bare-Metal) vs. Type-2 (Hosted) Hypervisors:** Type-1 (KVM, VMware ESXi, Xen, Hyper-V) runs directly on physical hardware with minimal overhead for production clouds; Type-2 (VirtualBox, VMware Workstation) runs as an app atop a host OS for local development.
3. **CPU Virtualization & Hardware Assist (Intel VT-x / AMD-V):** Early x86 instructions couldn't trap cleanly, requiring slow binary translation or OS patching (paravirtualization); modern CPUs add dedicated hypervisor privilege rings (root vs. non-root mode) so guest code runs at near-native silicon speed!

### Worked Example (Step by Step): Server Consolidation and Memory Overcommit Math

Suppose a company operates **10 legacy physical servers**, each with **16 GB RAM** and **8 vCPUs**, running at an average **12% utilization** (consuming 2 GB app RAM + 1 GB OS RAM = **3 GB** per server, and **250 W** of electricity each):
- **Before virtualization (10 physical servers):**
  - Total power draw: $$10 	imes 250	ext{ W} = \mathbf{2{,}500	ext{ W}}$$.
  - Used RAM: $$30	ext{ GB}$$ out of $$160	ext{ GB}$$ (**81.25% idle waste!**).
- **After consolidating onto 1 physical Type-1 hypervisor host (128 GB RAM, 32 cores, 400 W):**
  - All 10 VMs consume $$10 	imes 3	ext{ GB} = \mathbf{30	ext{ GB RAM}}$$ (leaving 98 GB headroom for traffic spikes and hypervisor overhead!).
  - Hardware count drops **10×** (10 servers → 1 server), and power drops from **2,500 W → 400 W** (**84% energy reduction**)—while every application still believes it owns its own dedicated OS and kernel!

### Deep-Dive Explanation: Why VMs Remain the Bedrock of Multi-Tenant Cloud Security (Read Slowly)

1. **Strong kernel-level isolation:** because every VM runs its own guest kernel behind hardware-enforced VT-x/EPT memory tables, a kernel panic or root exploit inside Tenant A's VM cannot trivially jump into Tenant B's memory.
2. **Live migration without downtime:** by copying dirty RAM pages iteratively over the network and pausing the VM for milliseconds, Type-1 hypervisors move running workloads off failing hosts seamlessly.
3. **Understand the VM tax:** each VM duplicates a full guest OS kernel (hundreds of MB to 2 GB of RAM) and boots in 15–60 seconds—which is the exact overhead **containers** eliminate in Lecture `08-02`!
4. **Notice how modern clouds combine both:** AWS Firecracker and GCP gVisor run *containers inside ultra-lightweight microVMs*—combining VM-grade security boundaries with container-grade boot speed!

**Quick check.** Why did early x86 servers need *paravirtualization* (modifying the guest OS kernel) before Intel VT-x and AMD-V existed? Because 17 sensitive x86 instructions failed silently instead of trapping to the hypervisor when executed in Ring 1, breaking Popek–Goldberg's trap-and-emulate condition!

## What is Virtualization?

### General Definition

**Virtualization** refers to creating a virtual (rather than physical) version of computing resources. In the context of this course:

- **A machine implemented in software**, rather than hardware
- **A self-contained environment** that acts like a computer
- **An abstract specification** for a computing device (instruction set, memory, etc.)

### Common Distinction

There are two main categories of virtual machines:

#### 1. Language-Based Virtual Machines
- Instruction set usually does not resemble any existing architecture
- Examples: **Java VM**, **.NET CLR**, Python VM
- Designed for platform independence and security

#### 2. Virtual Machine Monitors (VMM) or Hypervisors
- Instruction set fully or partially taken from a real architecture
- Virtualizes complete hardware systems
- Examples: **VMware**, **Xen**, **KVM**, **Hyper-V**

> This course focuses primarily on **hypervisor-based virtualization** used in cloud computing.

## History of Virtualization

The evolution of virtualization technology:

```
1972 → IBM VM/370
       First VM architecture for mainframe machines
       
1997 → Virtual PC for Mac
       Connectix brings virtualization to personal computers
       
1999 → VMware Virtual Platform
       Commercial virtualization for x86 architecture
       
2003 → Xen Hypervisor
       Open-source hypervisor project launched
       
2005 → VMware Player
       Free VM player for end users
       
2007 → VirtualBox
       Cross-platform virtualization software
       
2010s → Cloud Era
       AWS, Azure, GCP built on virtualization
```

## Virtualization Terminology

Understanding the key terms:

### Guest OS vs Host OS

- **Guest OS**: The operating system running inside the virtual machine
- **Host OS**: The operating system running on the physical machine (for Type 2 hypervisors)
- **Physical Machine (PM)**: The actual hardware
- **Virtual Machine (VM)**: The virtualized environment

### Type 1 Hypervisor (Bare Metal)

```
┌─────────────────────────────────────┐
│     Virtual Machine 1  │  VM 2      │
│  ┌──────────────┐  ┌──────────────┐ │
│  │  Guest OS 1  │  │  Guest OS 2  │ │
│  └──────────────┘  └──────────────┘ │
├─────────────────────────────────────┤
│      Type 1 Hypervisor              │
├─────────────────────────────────────┤
│      Hardware (CPU/RAM/Disk)        │
└─────────────────────────────────────┘
```

- **Runs directly on hardware**
- No need for a host operating system
- Better performance and efficiency
- Examples: **VMware ESXi**, **Xen**, **Hyper-V**, **KVM**
- Used in: Data centers, cloud providers

### Type 2 Hypervisor (Hosted)

```
┌─────────────────────────────────────┐
│     Virtual Machine 1  │  VM 2      │
│  ┌──────────────┐  ┌──────────────┐ │
│  │  Guest OS 1  │  │  Guest OS 2  │ │
│  └──────────────┘  └──────────────┘ │
├─────────────────────────────────────┤
│      Type 2 Hypervisor              │
├─────────────────────────────────────┤
│         Host OS                     │
├─────────────────────────────────────┤
│      Hardware (CPU/RAM/Disk)        │
└─────────────────────────────────────┘
```

- **Runs as an application** on top of a host OS
- Easier to set up and use
- Slightly lower performance due to host OS overhead
- Examples: **VMware Workstation**, **VirtualBox**, **Parallels Desktop**
- Used in: Development, testing, desktop virtualization

## Properties of Virtual Machines

Virtual machines provide four key properties that make them valuable:

### 1. Partitioning

- **Run multiple operating systems** on one physical machine
- **Divide system resources** between virtual machines
- Each VM gets allocated CPU, memory, disk, and network resources

**Example:**
```
Physical Server: 64 GB RAM, 16 CPU cores
├── VM1 (Web Server):    16 GB RAM, 4 cores
├── VM2 (Database):      32 GB RAM, 8 cores
└── VM3 (App Server):    16 GB RAM, 4 cores
```

### 2. Isolation

- **Fault isolation**: If one VM crashes, others continue running
- **Security isolation**: VMs are separated at the hardware level
- **Performance isolation**: Resource controls prevent "noisy neighbor" problems

**Benefits:**
- Secure multi-tenancy in cloud environments
- Contain security breaches
- Predictable performance

### 3. Encapsulation

- **Save entire VM state to files**
- **Move and copy VMs** as easily as files
- **Snapshot and restore** VM states

**Use Cases:**
```bash
# VM files typically include:
- .vmdk / .vdi  → Virtual disk files
- .vmx / .vbox  → VM configuration
- .nvram        → BIOS settings
- .vmem         → Memory snapshot
```

This enables:
- Easy backup and disaster recovery
- VM migration between hosts
- Template-based deployment

### 4. Hardware Independence

- **Provision or migrate** any VM to any physical server
- **Abstract hardware** from the operating system
- **Standardized virtual hardware** regardless of physical hardware

**Advantages:**
- Workload mobility across different hardware
- Simplified hardware upgrades
- Cloud provider flexibility

## How Virtualization Works

### The Illusion

The VM gives users an **illusion of running on a physical machine**:

- Applications run normally without modification
- OS believes it has direct hardware access
- Users interact with VM like a real computer

### The Reality

Behind the scenes:

1. **Operating systems normally run in privileged mode**
   - Direct access to hardware
   - Can execute privileged instructions

2. **VM OSs run in user mode**
   - No direct hardware access
   - Privileged instructions are trapped

3. **Most instructions execute directly**
   - Hardware executes them without hypervisor intervention
   - Provides near-native performance

4. **Resource management handled by hypervisor**
   - Memory allocation
   - Peripheral access
   - CPU scheduling

5. **Privileged instructions are "trapped"**
   - Hypervisor intercepts them
   - Emulates the instruction
   - Returns control to VM

### Hardware Assistance

Modern CPUs include virtualization support:

- **Intel VT-x** (Intel Virtualization Technology)
- **AMD-V** (AMD Virtualization)

These provide:
- Hardware-assisted virtualization
- Improved performance
- Better security isolation

## VM Components

A virtual machine virtualizes several key components:

### 1. CPU Virtualization
- Virtual CPUs (vCPUs) mapped to physical CPUs
- CPU scheduling by hypervisor
- Support for multiple CPU architectures

### 2. Memory Virtualization
- Virtual memory presented to guest OS
- Memory management by hypervisor
- Techniques like memory ballooning and page sharing

### 3. Network Virtualization
- Virtual network interfaces
- Virtual switches and routers
- Network isolation and bridging

### 4. Disk Virtualization
- Virtual disks stored as files
- Thin provisioning and snapshots
- Multiple disk formats (VMDK, VHD, QCOW2)

### Storage and Migration

- **VM is stored as a file** → Easy to save and migrate
- **Can be moved** along with apps and configuration
- **Enables** cloud elasticity and disaster recovery

## Paravirtualization

An alternative approach to full virtualization:

### Concept

- **VM OS knows about virtualization**
- Makes specific hypercalls instead of privileged instructions
- Requires modified guest OS

### Advantages

- Better performance than full virtualization
- More efficient resource usage
- Lower overhead

### Example: Xen

Xen hypervisor uses paravirtualization:
- Guest OS modified to make hypercalls
- Direct communication with hypervisor
- Used by early AWS EC2 instances

### Modern Trend

- Hardware-assisted virtualization has reduced the need for paravirtualization
- Most modern systems use full virtualization with hardware support
- Paravirtualization still used for specific optimizations (e.g., virtio drivers)

## Comparison: Type 1 vs Type 2

| Aspect | Type 1 (Bare Metal) | Type 2 (Hosted) |
|--------|---------------------|-----------------|
| **Installation** | Directly on hardware | On top of host OS |
| **Performance** | Higher (direct hardware access) | Lower (host OS overhead) |
| **Use Case** | Production servers, cloud | Development, testing |
| **Examples** | ESXi, Xen, Hyper-V | VirtualBox, VMware Workstation |
| **Management** | More complex | Easier to use |
| **Cost** | Often enterprise licensing | Often free or low-cost |

## Real-World Applications

### Cloud Computing

All major cloud providers use virtualization:

- **AWS**: Xen, KVM (Nitro), bare metal options
- **Azure**: Hyper-V
- **Google Cloud**: KVM

### Enterprise Data Centers

- Server consolidation (reduce physical servers)
- Disaster recovery and business continuity
- Test and development environments

### Desktop Virtualization (VDI)

- Virtual desktops for remote workers
- Centralized management
- Enhanced security

## Summary

Virtualization is a cornerstone technology that:

✓ **Enables multiple VMs** on a single physical machine  
✓ **Provides isolation** for security and fault tolerance  
✓ **Allows easy migration** and backup through encapsulation  
✓ **Abstracts hardware** for flexibility and portability  
✓ **Powers modern cloud computing** platforms  

In the next lecture, we'll dive deeper into **how virtualization works internally**, including binary translation and dynamic translation techniques.

## Video tham khảo

- **xem trước:** [Virtualization Explained](https://www.youtube.com/watch?v=FZR0rG3HKIk) — IBM Technology, English. Watch first: IBM’s virtualization overview, before the lesson splits guest, host, and hypervisor types.
- [What is a Hypervisor?](https://www.youtube.com/watch?v=LMAEbB2a50M) — IBM Technology, English. What a hypervisor is, the term the terminology section depends on.
- [Type 1 vs. Type 2 Hypervisors](https://www.youtube.com/watch?v=UEk0CKoeUnA) — Pluralsight, English. Type 1 versus Type 2 hypervisors, the bare-metal and hosted split in the lesson.
