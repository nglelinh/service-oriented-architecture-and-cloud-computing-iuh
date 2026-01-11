# Lecture_5.2_Distributed_database


## Slide 1

### Distributed Databases


## Slide 2

### Learning objectives


Understand differences between centralized and distributed databases
Understand different architectures, distribution strategies
Understand concurrency control & transactions


## Slide 3

### Distributed Database (DDB)


A collection of multiple, logically interconnected databases that are physically distributed over a computer network on different sites

Data are physically stored across multiple sites, managed by a DBMS that is independent of the other site

Data at any site available to users at other sites

Sites may be far apart, linked by some forms of telecommunication lines (secure lines or Internet)

Sites that are close together may be linked by a local area network (LAN)


Distributed databases – focus on database storage and location transparency


## Slide 4

### 4


Why do we need Distributed Databases?


Example: IBM has offices in London, New York, and Hong Kong.
Employee data:
EMP(ENO, NAME, TITLE, SALARY, …)
Where should the employee data table reside?


## Slide 5

### 5


Data Access Pattern


Mostly, employee data is managed at the office where the employee works
E.g., payroll, benefits, hire and fire
Periodically, we needs consolidated access to employee data
E.g., changes benefit plans and that affects all employees.
E.g., Annual bonus depends on global net profit.


## Slide 6

### 6


EMP


London
Payroll app


London


New York
Payroll app


New York


Hong Kong
Payroll app


Hong Kong


Problem:
NY and HK payroll
apps run very slowly!


## Slide 7

### 7


London
Emp


London
Payroll app


London


New York
Payroll app


New York


Hong Kong
Payroll app


Hong Kong


HK
Emp


NY
Emp


Much better!!


## Slide 8

### 8


London
Payroll app


Annual 
Bonus app


London


New York
Payroll app


New York


Hong Kong
Payroll app


Hong Kong


London
Emp


NY
Emp


HK
Emp


Distribution provides
opportunities for
parallel execution


## Slide 9

### 9


London
Payroll app


Annual 
Bonus app


London


New York
Payroll app


New York


Hong Kong
Payroll app


Hong Kong


London
Emp


NY
Emp


HK
Emp


## Slide 10

### 10


London
Payroll app


Annual 
Bonus app


London


New York
Payroll app


New York


Hong Kong
Payroll app


Hong Kong


Lon, NY
Emp


NY, HK
Emp


HK, Lon
Emp


Replication improves
availability


## Slide 11

### Distributed Databases


Slide 16-11


Figure: Data distribution and replication among distributed databases


## Slide 12

### Distributed Database Management


System (DDBMS)


A centralized software system that manages the DDB

Synchronizes the databases periodically

Provides an access mechanism that makes the distribution transparent to the users (as if it were all stored in a single location)

Ensures that the data modified at any remote site is universally updated

Supports a huge number of users simultaneously

Maintains data integrity of the databases


## Slide 13

### Challenges


Security: due to the Internet usage

Consistency issues: databases must be synchronized periodically to ensure data integrity

Increased storage requirements: due to replication of databases

Multiple location access: transactions may access data at one or more sites

How does the application find data? 

Where does the application send queries?

How to execute queries on distributed data?
	→ Push query to data.
	→ Pull data to query.


## Slide 14

### Distributed Strategies


Based on the organizational needs and information split and exchange requirements, the distributed database environment can be designed in two ways:

Homogeneous
Use the same DBMS for all database nodes that take part in the distribution

Heterogeneous
May use a diverse DBMS for some of the nodes that take part in the distribution


## Slide 15

### Homogeneous Distributed DB


Information is distributed between all the nodes

	The same DBMS and schema are used across all the databases

	The distributed DBMS controls all information

	Every global user must access the information from the same global schema controlled by the distributed DBMS

	A combination of all the individual DB schemas makes the global schema


15


## Slide 16

### Heterogeneous Distributed DB


Information is distributed between all the nodes

	Different DBMS and schemas may be used across the databases

	Local users (interacting with one of the individual database) can access the corresponding DBMS and schema

	Users who want to access the global information can communicate with the distributed DBMS, which has a global schema (a combination of all the individual DB schemas)


16


## Slide 17

### Heterogeneous	DATABASE	EXAMPLE


Middleware


Query Requests


Application Server


Back-end DBMSs


Connectors


## Slide 18

### Distributed DB Setup Method


Ongoing and future information maintenance must be determined
Synchronous: information across all nodes should be kept in sync all the time
Asynchronous: information is replicated at multiple nodes to make it available for other nodes

The setup can be performed in one of the following ways:
Replication
Fragmentation/partitioning (horizontal or vertical)
Hybrid setup


## Slide 19

### Replication


Maintain multiple copies of the database instances, stored in different sites
Easy and minimum risk process as the information is copied from one instance to another without a logical separation
Each individual node has the complete information
Efficient in accessing the information without having network traversals and reduces the risk of network security
Fast retrieval
Increase fault tolerance
Require more storage space
Take longer to synchronize all the nodes when the information across all the nodes needs to be updated


## Slide 20

### Fragmentation (or Partition)


One copy of each data item, distributed across nodes
Split a database into disjoint fragments (or parts)
Fragments can be
Vertical table subsets (formed by RA projection)
Horizontal subsets (formed by RA selection)


## Slide 21

### Horizontal Fragmentation


PK | A | B
 |  | 
 |  | 
 |  | 
 |  | 
 |  | 
 |  | 
 |  | 
… | … | …


PK | A | B
 |  | 
 |  | 


PK | A | B
 |  | 
 |  | 
 |  | 


PK | A | B
 |  | 
 |  | 


Splitting the rows of a table (or a relation between two or more nodes, containing databases) to form a distributed database – “split by region”
Each individual database has a set of rows that belong to the table or relation that belongs to the specific database

N nodes

R1, -inf < PK <= v1

R2, v1 < PK <= v2

…

RN, vN < PK < inf


## Slide 22

### Example: Horizontal Fragmentation


stuId | lastName | firstName | major | credits
S1001 | Smith | Tom | History | 90
S1002 | Chin | Ann | Math | 36
S1005 | Lee | Perry | History | 3
S1010 | Burns | Edward | Art | 63
S1013 | McCarthy | Owen | Math | 0
S1015 | Jones | Mary | Math | 42
S1020 | Rivera | Jane | CSC | 15
… | … | … | … | …


stuId | lastName | firstName | major | credits
S1002 | Chin | Ann | Math | 36
S1013 | McCarthy | Owen | Math | 0
S1015 | Jones | Mary | Math | 42
… | … | … | … | …


stuId | lastName | firstName | major | credits
S1001 | Smith | Tom | History | 90
S1005 | Lee | Perry | History | 3
… | … | … | … | …


stuId | lastName | firstName | major | credits
S1010 | Burns | Edward | Art | 63
… | … | … | … | …


stuId | lastName | firstName | major | credits
S1020 | Rivera | Jane | CSC | 15
… | … | … | … | …


σmajor=“Math” (students)


22


σmajor=“History” (students)


σmajor=“Art” (students)


σmajor=“CSC” (students)


## Slide 23

### Horizontal Fragmentation


© Praphamontripong


23


The information access is efficient
Best if partitions are uniform
Optimal performance as the local data are only stored in a specific database
Allow parallel processing on fragments
More secure as the information belonging to the other location is not stored in the database
If a user wants to access some of the other nodes or a combination of node information, the access latency varies.
If there is a problem with a node or a network, the information related to that node becomes inaccessible to the users


## Slide 24

### Vertical Fragmentation


PK | A | B | C | D | E | F | G | … | X
 |  |  |  |  |  |  |  |  | 
 |  |  |  |  |  |  |  |  | 
… | … | … | … | … | … | … | … | … | …


PK | A | B
 |  | 
 |  | 
… | … | …


PK | C | D | E | F | G
 |  |  |  |  | 
 |  |  |  |  | 
… | … | … | … | … | …


PK | X
 | 
 | 
… | …


…


(aka normalization process in distributed database setup)
Splitting the columns of a table (or a relation between two or more nodes, containing databases) to form a distributed database while keeping a copy of the base column (primary key) to uniquely identifying each record – “split by purpose”
N nodes
Each node contains all rows of a table


Projection must be lossless


~Normalization for distributed databases


© Praphamontripong


24


## Slide 25

### Example: Vertical Fragmentation


stuId | lastName | firstName | major | credits | …
S1001 | Smith | Tom | History | 90 | …
S1002 | Chin | Ann | Math | 36 | …
S1005 | Lee | Perry | History | 3 | …
S1010 | Burns | Edward | Art | 63 | …
S1013 | McCarthy | Owen | Math | 0 | …
S1015 | Jones | Mary | Math | 42 | …
S1020 | Rivera | Jane | CSC | 15 | …
… | … | … | … | … | …


…


stuId | lastName | firstName
S1001 | Smith | Tom
S1002 | Chin | Ann
S1005 | Lee | Perry
S1010 | Burns | Edward
S1013 | McCarthy | Owen
S1015 | Jones | Mary
S1020 | Rivera | Jane
… | … | …


stuId | major
S1001 | History
S1002 | Math
S1005 | History
S1010 | Art
S1013 | Math
S1015 | Math
S1020 | CSC
… | …


stuId | credits
S1001 | 90
S1002 | 36
S1005 | 3
S1010 | 63
S1013 | 0
S1015 | 42
S1020 | 15
… | …


πstuId, lastName, firstName(students)


πstuId, credits (students)


πstuId, major(students)


Notice stuId in all fragments


© Praphamontripong


25


## Slide 26

### Vertical Fragmentation


Appropriate if each of the organizational units located in different geographies have separate operations
Partition based on behavior and function that each node performs
Best if partitions are uniform
Part of the tuple is stored where it is most frequently accessed
Allow parallel processing on fragments
Poorly chosen columns to split can lead to node bottleneck
The aggregation of the data involves complex queries with joins across the location database, as no replication is made for non- primary keys


## Slide 27

### Hybrid Setup


Involve a combination of replication and fragmentation

Relation is partitioned into several fragments

Some information is replicated across the database nodes

Data administrators play a crucial role to choose the right combination to ensure data integrity and security


## Slide 28

### BASE Consistency Model


With the enormous growth in data, achieving ACID or CAP becomes very difficult.
A more relaxed set of properties is BASE

Basically Available, Soft state, Eventually consistent


Key idea:
Databases may not all be in the same state at the same time (“soft state”)
After synchronization is complete, the state will be consistent


Most failures do not cause a complete system outage


System is not always write-consistent


Data will eventually converge to agreed values


28


## Slide 29

### Wrap-Up


Distributed Database Systems  database scaling


Replication
Multiple copies of each database partition
Improves fault tolerance
Read performance ok
Write performance suffers


Fragmentation


Multiple machines to distribute data
Write performance ok
Read performance suffers


## Slide 30

### CONSISTENT	HASHING


1 0

P1

P3


P2


45


0.5


Consistent Hashing is a technique used in distributed systems to distribute data efficiently across multiple servers while minimizing disruption when servers are added or removed. 

It is widely used in caching (Memcached, Redis Cluster), distributed databases (DynamoDB, Cassandra), and load balancing.


## Slide 31

### CONSISTENT	HASHING


1 0


hash(key1)


P1


P3


P2


46


0.5


## Slide 32

### CONSISTENT	HASHING


1 0


hash(key2)


hash(key1)


P1


P3


P2


47


0.5


## Slide 33

### CONSISTENT	HASHING


1 0

P1

P3


P4


P2


48


0.5


New Partition


## Slide 34

### CONSISTENT	HASHING


If hash(key)=P4


1 0

P1


P3


P4


P2


49


0.5


New Partition


## Slide 35

### CONSISTENT	HASHING


1 0


P5


P1


P3


P4


P2


50


0.5


New Partition


## Slide 36

### CONSISTENT	HASHING


1 0
P5
P1

P3


P4


P2


P6


51


New Partition


0.5


## Slide 37

### CONSISTENT	HASHING


1 0


Replication Factor = 3


P5


P1


P3


P4


P2


P6


52


0.5


15-445/645 (Fall 2023)


## Slide 38

### CONSISTENT	HASHING


1 0


hash(key1)
Replication Factor = 3


P5


P1


P3


P4


P2


P6


53


0.5


15-445/645 (Fall 2023)


## Slide 39

### CONSISTENT	HASHING


1 0


hash(key1)
Replication Factor = 3


P5


P1


P3


P4


P2


P6


54


0.5


## Slide 40

### Distributed ID generator


Functional Requirements

Each ID generated by the system must be unique across the entire distributed environment.
IDs should consist solely of numerical values
Each ID should fit into a 64-bit integer, which can store values up to approximately 9.22 × 10^18.
IDs should reflect the order of creation based on time, meaning newer IDs are larger than older ones.
The system must generate a large number of unique IDs efficiently, specifically at least 10,000 IDs per second.


## Slide 41

### Distributed ID generator


Nonfunctional Requirements

The ID generation system should scale effectively as the number of servers or data centers increases.
ID generation should be fast, with minimal delay to avoid becoming a bottleneck in the system.
The system should be reliable, with minimal risk of failure, even in the case of hardware or network issues.
The system should not rely on a central authority or server to generate IDs, reducing the risk of a single point of failure.
The design should be straightforward to implement and maintain, with clear procedures for handling system upgrades or scaling.


## Slide 43

### Distributed ID generator


Solution:

Ticket Server
Multi-master
UUID
ULID
Twitter snowflake


## Slide 44

### Distributed ID generator


## Slide 45

### Distributed ID generator


## Slide 46

### Distributed ID generator


## Slide 47

### Distributed ID generator


## Slide 48

### Snowflake ID


Sign Bit (1 bit): luôn là 0
Timestamp (41 bits): số milisecond kể từ Twitter epoch (mốc 2010-11-04).
Datacenter ID (5 bits): mã định danh cho datacenter. Giá trị được khởi tạo khi bắt đầu chạy trình Snowflake generator.
Machine ID (5 bits): mã định danh cho máy trong datacenter. Giá trị được khởi tạo khi bắt đầu chạy trình Snowflake generator.
Sequence Number (12 bits): Số đếm tăng dần cho các ID được sinh trong một mili giây. Số đếm này sẽ reset về 0 khi sang milisecond khác. Dùng để đảm bảo tính duy nhất trong một mili giây. Như vậy, 1 máy có sinh 2 ^ 12 (=4096) IDs trong vòng 1 mili giây.


## Slide 49

### Distributed ID generator
