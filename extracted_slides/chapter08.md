# Lecture_6_1_Virtualization_Containerization


## Slide 1

### Virtualization - Containerization


## Slide 2

### Reading list


2


Reading list


## Slide 3

### Virtualisation


Generally
A machine that’s implemented in software, rather than hardware
A self-contained environment that acts like a computer
An abstract specification for a computing device (instruction set, etc.)
Common distinction:
(language-based) virtual machines 
Instruction set usually does not resemble any existing architecture
Java VM, .Net CLR, many others
Virtual Machine Monitors (VMM) or Hypervisor
instruction set fully or partially taken from a real architecture


Virtualization


## Slide 4

### History


First VM architected by IBM in 1972 VM/370 to provide full VM of mainframe machine
- 1997 Virtual PC for Mac by Connectix
- 1999 VMware’s VMware Virtual Platform
- 2003 Open Source hypervisor Xen
- 2005 VMware Player – free VM player
- 2007 VirtualBox


History


## Slide 5

### Guest OS runs inside the VM, and host OS runs on the PM
Type 1 hypervisor: runs directly on hardware, no need for host OS


Type 2 (hosted) hypervisor: runs as an application on top of host OS


Virtual Machine


Type 1 Hypervisor


Hardware (CPU/RAM)


Virtual Machine


Type 2 Hypervisor


Host OS


Virtualization terminology


## Slide 6

### Types of Virtualisation


Type 1 Virtualisation – Bared metal


Type 2 Virtualisation - Hosted


Types of Virtualization


## Slide 7

### Properties of Virtual Machines


Partitioning
Run multiple operating systems on one physical machine.
Divide system resources between virtual machines.
Isolation
Provide fault and security isolation at the hardware level.
Preserve performance with advanced resource controls.
Encapsulation
Save the entire state of a virtual machine to files.
Move and copy virtual machines as easily as moving and copying files.
Hardware Independence
Provision or migrate any virtual machine to any physical server.
VM gives user an illusion of running on a physical machine


Properties of Virtual Machines


## Slide 8

### VM Components


1. Virtualise hardware
CPU
Memory
Network interfaces
Disks
2. VM is stored as a file and so can be saved and migrated along with apps and configuration information
3. The aim is to run Operating Systems unchanged however, there is Parvirtualisation
VM OS knows about virtualization and makes specific calls (approach taken by Xen VM)


VM components


## Slide 9

### How it works


Operating systems normally run in privileged mode
VM OSs run in user mode
Most instructions can execute by hardware without hypervisor intervening
Resource management (memory, peripherals) handled by hypervisor
In VM privileged instructions are “trapped” by the hypervisor and emulated
Intel and AMD have added instructions to make this possible (not used by VMware)


How it works


## Slide 10

### Binary Translation

• Emulation: 
– Guest code is traversed and instruction classes are mapped to routines that emulate them on the target architecture.
• Binary translation:
 – The entire program is translated into a binary of another architecture. 
 – Each binary source instruction is emulated by some binary target instructions.

Binary Translation


## Slide 11

### Dynamic Binary Translation


vPC


mov   ebx, eax


cli


and   ebx, ~0xfff


mov   ebx, cr3


sti


ret


mov   ebx, eax


mov   [CPU_IE], 0


and   ebx, ~0xfff


mov   [CO_ARG], ebx


call  HANDLE_CR3


mov   [CPU_IE], 1


test  [CPU_IRQ], 1


jne


call  HANDLE_INTS


jmp   HANDLE_RET


start


Guest Code


Translation Cache


Courtesy Scott Define VMware Inc


Dynamic Binary Translation


## Slide 12

### AWS VM


AWS uses different configurations that use Xen
AWS Nitro – a hypervisor based on KVM
AWS also offers actual ”bare metal” machines – with no virtualization
Different VM types are represented by AMIs that are either
PV paravirtual (Linux only)
HVM hardware virtual machine (Linux and Windows)


AWS VM


## Slide 13

### VMs and Security


Hypervisor security
Hypervisor, applications, drivers and libraries are loaded in randomized memory locations to stop exploites
VMWare uses digital signatures to verify modules drivers and applications – much as iOS and Android do
However – Hypervisor represents an “attack surface” not present in native OSs and so they are subject to vulnerabilities
Virtualized OS
Same vulnerabilities as a non-virtualized OS 
VM to VM
Theoretically possible to access memory in one VM from another but very hard to do
AWS itself introduces security vulnerabilities – e.g. access to keys, usernames and passwords


VM and Security


## Slide 14

### Other types of virtualization


Web Server
Virtual hosts using a different hostname, configuration file
Virtual Environments
Java, Python and Ruby amongst others have the ability to configure separate versions of language and libraries
Containers
Like VMs but run in user space and packages binaries and libraries
Docker, Kubernetes, AWS Containers (Docker)
Serverless Environments
Code is executed in response to an event, including HTTP


Other Type of Virtualization


## Slide 15

### Other topics in virtualization


VM live migration and related ideas
VMs can moved from one physical machine to another
Why? Maintenance of machines in the cloud, fault tolerance etc.
How are VMs migrated without impacting the application in it?
Use similar techniques for other uses like VM checkpointing
Containers: lightweight virtualization technique
Underlying Linux concepts of namespaces, Cgroups
Container frameworks like LXC, Docker, Kubernetes


Other Topics of Virtualization


## Slide 16

### Motivation for Containerization


VMs are great, but have high runtime overhead
Can we scale faster and more easily?
What if we could sandbox VMs, but share the OS kernel?
Enables different software architectures and practices


## Slide 17

### Containers


Containers are cut down VMs used to execute code in a isolated environment 
Docker
Originally based on Linux Containers (LXC) but now on runC
All Docker containers use the same underlying OS but present various parts of the OS as if they were dedicated to the running system


Containers


## Slide 18

### Containers: lightweight virtualization


Containers share base OS, have different set of libraries, utilities, root  filesystem, view of process tree, networking, and so on.
VMs have different copies of OS itself
Containers have lesser overhead than VMs, but also lesser isolation


Base OS / kernel


Containers have
separate view of:
Root file system
Libraries and utilities
Process tree
Users
Networking
IPC endpoints


VMM / host OS


VMs are separate  systems with complete  copies of OS, user  processes.


Guest OS


## Slide 19

### Big Idea: Containers have less OS overhead


Traditional VM-based  Infrastructure


Container  Infrastructure


https://newsroom.netapp.com/blogs/containers-vs-vms/


## Slide 20

### Containers


A Linux container needs a Linux kernel.
A Windows container needs a Windows kernel.
You cannot run a Windows container on a Linux host (or vice versa) natively.

🔹 Solution?
Use Windows Subsystem for Linux (WSL 2) or Hyper-V on Windows to run Linux containers.
Use Windows Server to run native Windows containers.


Run Windows Containers on a Linux Host (or Vice Versa)


## Slide 21

### Two mechanisms in Linux kernel over which containers are built:
Namespaces: a way to provide isolated view of a certain global resource (e.g., root  filesystem) to a set of processes. Processes within a namespace see only their slice of the  global resource
Cgroups: a way to control resource on a group of processes
Together, namespaces and cgroups allow us to isolate a set of processes into a  bubble and set resource limits
Container implementations like LXC, Docker leverage these mechanisms to  build the container abstractions
LXC is general container while Docker optimized for single application
Frameworks like Docker Swarm or Kubernetes help manage multiple  containers across hosts, along with autoscaling, lifecycle management, and so


Namespaces and Cgroups


## Slide 22

### Namespaces


Group of processes that have an isolated/sliced view of a global resource
Default namespace for all processes in Linux, system calls to create new  namespaces and place processes in them
Which resources can be sliced?
Mount namespace: isolates the filesystem mount points seen by a group of processes. The  mount() and umount() system calls only affect the processes in that namespace.
PID namespace: isolates the PID numberspace seen by processes. E.g., first process in a  new PID namespace gets a PID of 1.
Network namespace: isolates network resources like IP addresses, routing tables, port  numbers and so on. E.g., processes in different network namespaces can reuse the same  port numbers.
UTS namespace: isolates the hostname seen by processes.
User namespace: isolates the UID/GID numberspace. E.g., a process can get UID=0 (i.e., act  as root) in one namespace, while being unprivileged in another namespace. Mappings to  be specified between UIDs in parent namespace and UIDs in new namespace.
IPC namespace: isolates IPC endpoints like POSIX message queues.
More powerful than chroot() which only isolates root filesystem


Namespaces


## Slide 23

### Namespaces API


Three system calls related to namespaces:
clone() is used to create a new process and place it into a new namespace. More
general version of fork().


Flags specify what should be shared with parent, and what should be created new for  child (including virtual memory, file descriptors, namespaces etc.)
setns() lets a process join an existing namespace. Arguments specify which
namespace, and which type.
unshare() creates a new namespace and places calling process into it. Flags indicate  which namespace to create. Forking a process and calling unshare() is equivalent to  clone().
Once a process is in a namespace, it can open a shell and do other useful  things in that namespace
Forked children of a process belong to parent namespace by default


Namespaces API


## Slide 24

### Namespace handles: how to refer to a namespace


/proc/PID/ns of a process has information on which namespace a process belongs  to. Symbolic links pointing to the inode of that namespace (“handle”)


Namespace handle can be used in system calls (e.g., argument to setns)
Processes in same namespace will have same handle, new handle created when
new namespace created


Namespaces in operation (lwn.net)	https://lwn.net/Articles/531114/


Namespace handles: how to refer to a namespace


## Slide 25

### For your current shell:
ls -l /proc/$$/ns
Meaning of the ID (e.g., [4026531835])
This ID uniquely identifies a namespace in the system. 
If multiple processes share the same namespace ID, they will share the corresponding resource space.


## Slide 26

### PID namespaces (1)


The first process to be created in a new PID  namespace will have PID=1 and will act as init  process in that namespace
Will reap orphans in this namespace
Processes in PID namespace get separate PID  numberspace (child of init gets PID 2 onwards)
A process can see all other processes in its own  or nested namespaces, but not in its parent  namespace
P2, P3 not aware of P1 (parent PID of P2 = 0)
P1 can see P2 and P3 in its namespace (with different  PIDs)
P2=P2’ (just different PIDs in different namespaces)


P1


P3


P3’


PID=X1


PID=2


PID=1


P2’	P2
PID=X2


PID=X3


PID namespaces (1)


## Slide 27

### PID namespaces (2)


First process in a namespace acts as init and has  special privileges
Other processes in namespace cannot kill it
If init dies, namespace terminated
However, parent process can kill it in parent namespace
Who reaps whom?
Init process reaped by parent in parent namespace
Other child processes reaped by parent in same  namespace
Any orphan process in namespace reaped by init of that  namespace


P1


P3


Q1


Q2


Q3


(init)
P2


Reaped by P2 (init)  if orphaned


PID namespaces (2)


## Slide 28

### PID namespaces (3)


Namespace related system calls have slightly different behavior with  PID namespace alone
clone() creates a new namespace for child as expected
However, setns() and unshare() do not change PID namespace of  calling process. Instead, the child processes will begin in a new PID  namespace
Why this difference? If namespace changes, PID returned by getpid()  will also change. But many programs make assumption that getpid()  returns same value throughout life of process.
getpid() returns the PID in the namespace the process resides in


PID namespaces (3)


## Slide 29

### Mount namespaces


Root filesystem seen by a process is constructed from a set of mount  points (mount() and umount() syscalls)
New mount namespace can have new set of mount points
New view of root filesystem
Mount point can be shared or private
Shared mount points propagated to all namespaces, private is not
If parent makes all its mount points private and clones child in new mount  namespace, child starts with empty root filesystem
Container frameworks use mount namespaces to create a custom  root filesystem for each container using a base rootfs image


Mount namespaces


## Slide 30

### Mount namespaces and ps


How does “ps” work?
Linux has a special procfs, in which kernel populates info on processes
Reading /proc/PID/.. does not read file from disk, but fetches info from OS
procfs mounted on root as a special type of filesystem
P1 clones P2 to be in new PID namespace but uses old mount  namespace. We open shell in new PID namespace and run ps. We will  still see all processes of parent namespace. Why?
ps command is still using procfs of parent’s mount namespace
How to make ps work correctly within a PID namespace?
Place P2 in new mount namespace, mount a new procfs at root
New procfs at new mount point is different from parent’s procfs
ps will not show only processes in this PID+mount namespace


/


proc	bin


Mount namespaces and ps


## Slide 31

### Network namespaces (1)


Network namespace can be created by cloning a process into a new  namespace, or simply via commandline

List of network namespaces can be viewed at /var/run/netns, can use  setns() to join existing namespace
Command “ip netns exec” can be used to execute commands within  network namespace, for example, to view all IP links:


Namespaces in operation, part 7: Network namespaces https://lwn.net/Articles/580893/


Network namespaces (1)


## Slide 32

### Network namespaces (2)


Any new network namespace only has loopback interface. How to  communicate with rest of network?
Create a virtual Ethernet link (veth pair) to connect parent namespace to  new child namespace
Assign endpoints to two different namespaces
Assign IP addresses to both endpoints
Can communicate over this link to parent namespace
Can configure bridging/NAT to connect to wider internet


Namespaces in operation, part 7: Network namespaces https://lwn.net/Articles/580893/


lo


eth0


lo


veth0


veth1


Network namespaces (2)


## Slide 33

### The next building block: Cgroups


Namespaces let us isolate processes into a slice with respect to many  resources: mount points, PID/UID numberspace, network endpoints, etc.
The next topic Cgroups will let us assign resource limits on a set of  processes
Divide processes into groups and subgroups hierarchically
Assign resource limits for processes in each group/subgroup
Which resources can be limited? CPU, memory, I/O, CPU sets (which  process can execute on which CPU core), and so on
Specify what fraction of resource can be used by each group of processes


The next building block: Cgroups


## Slide 34

### Cgroups hierarchies


Can create separate hierarchies for each resource, or a combined  hierarchy for multiple resources together


All CPU


CPU-Faculty


CPU-Students


All NW


NW-Web


NW-Non-Web


40%


60%


50%


50%


Browser process created by faculty


CPU-NW


Fac
Web


Fac
Non-Web


Student
Web


Student
Non-Web


Cgroups hierarchies


## Slide 35

### Creating cgroups


No new system calls, managed via the filesystem
A special cgroup filesystem mounted at /sys/fs/cgroup
Create directories and sub-directories for different resources and  different user classes
Write “founding father” task PID to tasks file
All children of this task will be in same cgroup too

Tasks can be assigned to leaf nodes in hierarchy
Tasks will belong to default cgroup of parent if not explicitly placed into any  hierarchy


Documentation/cgroups/cgroups.txt	https://lwn.net/Articles/524935/


Creating cgroups


## Slide 36

### How to create a container?


Suppose you wish to run an application/shell in a container. How?
Create separate namespaces for isolation
Create and configure cgroups for resource limits
Create root filesystem that is compatible with CPU’s ISA and OS binary
All utilities, binaries, configuration files needed to run the application
A process enters the namespaces, mounts rootfs, registers in cgroups,  execs desired application or shell
Your application or shell is running in a “container”!
Many tutorials online on how to create your own container


How to create a container?


## Slide 37

### Container frameworks


Existing container frameworks like LXC and Docker do the  namespace/cgroup configuration automatically “under the hood”
LXC container is a lightweight VM
Provides standard OS shell interface
Uses namespaces and cgroups under the hood
Docker containers are optimized to run a single application
Docker config file specifies base root filesystem, along with utilities needed to run  a specific application
Runs application in a container environment
Easy way to package an application and all its dependencies and run anywhere


Container frameworks


## Slide 38

### Container orchestration frameworks


Docker Swarm, Kubernetes – frameworks to manage multiple  containers on multiple hosts
Kubernetes – popular container orchestration framework
Runs over multiple physical machines ("nodes") each with multiple "pods“
A pod contains one or more containers within the same network namespace,  with the same IP address
Pod is a tier of a multi-tier application (e.g., "frontend", "backend",  "database", "webserver")
Kubernetes manages multiple nodes and their pods, e.g., instantiating pods  on free nodes, auto-scaling pods when load increases, restarting pods when  they crash, etc.


Container orchestration frameworks


## Slide 39

### Container orchestration frameworks


Run Different Linux Distributions on the Same Host
Example: You can run an Ubuntu container on a Red Hat or Alpine Linux host.
Run Applications with Different Dependencies
Each container can have its own set of libraries and binaries.
Example: One container can run Python 3.9, while another runs Python 3.11, even if the host does not have Python installed at all.
Use the Host’s Hardware & System Calls 
Containers can access network interfaces, storage, and CPUs as if they were directly running on the host. 
Example: A TensorFlow container can access the host’s GPU using NVIDIA drivers.


What Containers CAN Do


## Slide 40

### Summary


Containers provide lightweight isolation with lower overhead
Containers share same kernel binary, have different root filesystems  (utilities, configurations) over the kernel
Implemented using two Linux primitives
Namespaces for isolation
Cgroups for resource limits
Frameworks like Docker, LXC, Kubernetes provide more functionality  by building upon these primitives


Namespaces in operation (lwn.net)	https://lwn.net/Articles/531114/


Documentation/cgroups/cgroups.txt	https://lwn.net/Articles/524935/


Summary


## Slide 42

### Docker Concepts


Image
Frozen description of an environment

Container
Running instantiation of an image

Volume
Persistent data storage


## Slide 43

### Docker Concepts


Dockerfile
Describes everything your container needs:
Dependencies
Source Code / Binaries


## Slide 44

### The Dockerfile


Overview Guide to Running Code in Docker:
Inherit from a parent OS/platform container
Install any packages / libraries you need
Add any source code you need
Attach any volumes you need for data persistence
Set a command to be run at startup


## Slide 45

### The Dockerfile


Essential Commands:
FROM - Inherit from a parent container
i.e. “FROM ubuntu”
RUN - Runs a command during the build process
i.e. “RUN apt-get install python3”
ADD - Copies files from the build directory into the image
i.e. “ADD hello_world.py /usr”
EXPOSE - Register a port that the image will listen on
i.e. “EXPOSE 80”
CMD - Set the default command to be executed on startup
i.e. “CMD python /usr/hello_world.py”


## Slide 46

### The Dockerfile


Each command in a Dockerfile creates an intermediate image
Useful for caching!
Structure your Dockerfiles to take advantage of caching
Install packages first, then add source code
Within reason, “Funnel down” from most general to most specific


## Slide 47

### Where to go from here


Docker Swarm
Pools multiple Docker engines into a combined virtual host
Allows multiple VMs to collaborate to host clustered Docker containers
Docker Compose
Orchestrate multiple-container applications
Declarative format for configuring volumes, container networking, and  scaling


## Slide 48

### Docker Architecture


Server (dockerd) long running daemon process
Manages data, network, processes, communication, etc
REST API used to communicate with server
CLI to execute docker containers
Instructions for creating a container is placed in an image
4.  Services orchestrate docker containers in swarms


Docker Architecture


## Slide 49

### Docker


Docker


## Slide 50

### Basic Docker commands


docker run – Runs a command in a new container.
docker start – Starts one or more stopped containers
docker stop – Stops one or more running containers
docker build – Builds an image form a Docker file
docker pull – Pulls an image or a repository from a registry
docker push – Pushes an image or a repository to a registry
docker export – Exports a container’s filesystem as a tar archive
docker exec – Runs a command in a run-time container
docker search – Searches the Docker Hub for images
docker attach – Attaches to a running container
docker commit – Creates a new image from a container’s changes


Docker CLI commands


## Slide 51

### Serverless Computing


Fargate is an example of serverless computing
Don’t need to worry about operating systems or maintaining VMs
Other environment for this is Kubernetes
Execution is simply a container or in the case of AWS Lambda just a piece of code
Difference is between how applications get executed and the choice of development environment
Fits in with Micro Service architectures


51


Serverless Computing


## Slide 52

### AWS Lambda


AWS Lambda removes the container details and allows “functions” to be written
Supports different programming environments:
Java, Javscript, Python, C#
Functions can be connected to “events” coming from a number of different environments:
HTTP
IoT
Mobile
Other AWS services like Kinesis


52


AWS LAmbda


## Slide 53

### AWS Lambda


53


Credits: https://medium.com/@amiram_26122/the-hidden-costs-of-serverless-6ced7844780b


## Slide 54

### AWS Lambda


54


## Slide 55

### Example


Functions need to be packaged and uploaded as zip files
Can be tested locally using SAM CLI – doing much of the same thing as Docker
functions can also be edited using AWS Lambda Console Editor
Create a function
Choose language
Role
Template
Edit code and test


55


Virtualization


## Slide 56

### Add an API Gateway


Simple way to associate a URL with a specific function
Allows the specification of functions associated with specific HTTP operations (GET, POST, etc)
Supports CORS Cross Origin Resource Sharing
Makes taking parameters easy


56


Add an API Gateway


## Slide 57

### Typical Serverless Architecture in AWS


57


Courtesy AWS


Typical Serverless Architecture in AWS
