# Lecture_1.2_Distributed System


## Slide 1

### Distributed Systems


## Slide 2

### Outline


1. What is a Distributed System
2. Examples of Distributed Systems
3. Common Characteristics
4. Basic Design Issues
5. Summary


2


Outline


## Slide 3

### Fully
Distributed


Data


Processors


Control


Fully replicated


Not fully replicated
master directory


Local data,
local directory


Master-slave


Autonomous 
transaction based


Autonomous
fully cooperative


Homog. 
special
purpose


Heterog.
special
purpose


Homog.
general
purpose


Heterog.
general
purpose


3


1. Distributed System Types


## Slide 4

### 1. What is a Distributed System?


Definition: A distributed system  is one in which components located at networked computers communicate and coordinate their actions only by passing messages.  

This definition leads to the following characteristics of distributed systems:
 Concurrency of components
 Lack of a global ‘clock’
 Independent failures of components


4


1. What is a Distributed System?


## Slide 5

### A distributed system is. . .


“. . . a system in which the failure of a computer you  didn’t even know existed can render your own computer  unusable.” — Leslie Lamport

. . . multiple computers communicating via a network. . .
. . . trying to achieve some task together

Consists of “nodes” (computer, phone, car, robot, . . . )


A distributed system is. . .


## Slide 6

### What is a distributed system?


Independent components or elements
(software processes or any piece of hardware used to run a  process, store data, etc)


What is a distributed system?


## Slide 7

### What is a distributed system?


Independent components or elements that are connected by  a network.


What is a distributed system?


## Slide 8

### What is a distributed system?


Independent components or elements that are connected by  a network and communicate by passing messages.


What is a distributed system?


## Slide 9

### What is a distributed system?


Independent components or elements that are connected by  a network and communicate by passing messages to achieve a  common goal, appearing as a single coherent system.


What is a distributed system?


## Slide 10

### 1.1 Centralized System Characteristics


One component with non-autonomous parts
Component shared by users all the time
All resources accessible
Software runs in a single process
Single point of control
Single point of failure


10


1.1 Centralized System Characteristics


## Slide 11

### Why make a system distributed?


It’s inherently distributed:
e.g. sending a message from your mobile phone to your  friend’s phone
For better reliability:
even if one node fails, the system as a whole keeps  functioning
For better performance:
get data from a nearby node rather than one halfway  round the world
To solve bigger problems:
e.g. huge amounts of data, can’t fit on one machine


Why make a system distributed?


## Slide 12

### Multiple autonomous components
Components are not shared by all users
Resources may not be accessible
Software runs in concurrent processes on different processors
Multiple points of control
Multiple points of failure


12


1.2 Distributed System Characteristics


## Slide 13

### Local Area Network and Intranet
Database Management System
Automatic Teller Machine Network
Internet/World-Wide Web
Mobile and Ubiquitous Computing


13


2. Examples of Distributed Systems


## Slide 14

### 14


2.1 Local Area Network


## Slide 15

### 2.2 Database Management System


15


2.2 Database Management System


## Slide 16

### 16


2.3 Automatic Teller Machine Network


## Slide 17

### 2.4  Internet


17


2.4  Internet


## Slide 18

### 2.4.1 World-Wide-Web


18


2.4.1 World-Wide-Web


## Slide 19

### 19


2.4.2  Web Servers and Web Browsers


## Slide 20

### 20


2.5  Mobile and Ubiquitous Computing


## Slide 21

### Examples of distributed systems


World Wide Web
A cluster of nodes on the cloud (AWS, Azure, GCP)
Multi-player games
BitTorrent
Online banking
……..


Examples of distributed systems


## Slide 22

### Example: scaling up Facebook


2004: Facebook started on a single server Web server front end to assemble each user’s page. Database to store posts, friend lists, etc. 
2008: 100M users 
2010: 500M users 
2012: 1B users 
2019: 2.5B users
2024: 3B users
How do we scale up?


Example: scaling up Facebook


## Slide 23

### Example: scaling up Facebook


One server running both webserver and DB
Two servers: one for webserver, and one for DB
System is offline 2x as often!
Server pair for each social community
E.g., school or college 
What if server fails? 
What if friends cross servers?


Example: scaling up Facebook


## Slide 24

### Example: scaling up Facebook


Scalable number of front-end web servers.
Stateless: if crash can reconnect user to another server.
Use various policies to map users to front-ends.


Scalable number of back-end database servers.
Run carefully designed distributed systems code.
If crash, system remains available.


Example: scaling up Facebook


## Slide 25

### What are important ?


Distributed system concepts and algorithms
How can failures be detected?
How do we reason about timing and event ordering?
How do concurrent processes share a common resource?
How do they elect a “leader” process to do a special task?
How do they agree on a value? Can we always get them to agree?
How to handle distributed concurrent transactions?
….
Real-world case studies
Distributed key-value stores
Distributed file servers
Blockchains
…


What are important ?


## Slide 26

### Latency and bandwidth


Latency: time until message arrives
In the same building/datacenter: ≈ 1 ms
One continent to another: ≈ 100 ms
Hard drives in a van: ≈ 1 day

Bandwidth: data volume per unit time
3G cellular data: ≈ 1 Mbit/s
Home broadband: ≈ 10 Mbit/s
Hard drives in a van: 50 TB/box ≈ 1 Gbit/s
(Very rough numbers, vary hugely in practice!)


Latency and bandwidth


## Slide 27

### What are we trying to achieve when we construct a distributed system?
Certain common characteristics can be used to assess distributed systems
Heterogeneity - Tính không đồng nhất 
Openness - Tính mở
Security - Bảo mật
Scalability - Khả năng mở rộng
Failure Handling - Xử lý lỗi
Concurrency - Tính đồng thời
Transparency - Tính minh bạch


27


Common Characteristics of Distributed Systems


## Slide 28

### 4. Basic Design Issues


General software engineering principles include rigor and formality, separation of concerns, modularity, abstraction, anticipation of change, …
Specific issues for distributed systems:
Naming
Communication
Software structure
System architecture
Workload allocation
Consistency maintenance


28


Basic Design Issues


## Slide 29

### 4.1 Naming


A name is resolved when translated into an interpretable form for resource/object reference.
Communication identifier (IP address + port number)
Name resolution involves several translation steps
Design considerations
Choice of name space for each resource type
Name service to resolve resource names to comm. id.
Name services include naming context resolution, hierarchical structure, resource protection


29


Basic Design Issues


## Slide 30

### 4.2 Communication


Separated components communicate with sending processes and receiving processes for data transfer and synchronization.
Message passing: send and receive primitives
synchronous or blocking
asynchronous or non-blocking
Abstractions defined: channels, sockets, ports.
Communication patterns: client-server communication (e.g., RPC, function shipping) and group multicast


30


Basic Design Issues


## Slide 31

### 4.3 Software Structure


31


Layers in centralized computer systems:


## Slide 32

### 4.3 Software Structure


32


Layers and dependencies in distributed systems:
