# Lecture_3.1_Hadoop_YARN


## Slide 2

### Learning objectives


Understand massive scale data management and  processing with Hadoop
Understand and able to use Hadoop components  for big data platform designs
Able to integrate Hadoop with other frameworks  for data ingestion and analytics systems


## Slide 3

### ●
●
●
●


Hbase and Hive


## Slide 4

### ●
●
●
●


Hbase and Hive


## Slide 7

### Hadoop


The goal is to combine storage and processing in the  same cluster system
Designed for massive scale of data and computing  with shared nothing architecture
commodity hardware, highly scalability, fault tolerance,  easy to extend
Suitable for both on-premise and clouds
There are very rich software ecosystems centered
 	around Hadoop


## Slide 8

### Hadoop history


Doug Cutting and Mike Cafarella in the year 2002 worked Apache Nutch project
In 2003, they came across a paper that described the architecture of Google’s distributed file system, called GFS (Google File System) 
In 2004, Google published one more paper on the technique MapReduce
in 2006, Doug Cutting joined Yahoo along with Nutch project
Doug Cutting had a two-year-old son who had named his toy elephant Hadoop
	In 2007, Yahoo successfully tested Hadoop on a 1000 node cluster and start using it.


## Slide 9

### Hadoop history


In December of 2011, Apache Software Foundation released Apache Hadoop version 1.0.
And later in Aug 2013, Version 2.0.6 was available.
And currently, we have Apache Hadoop version 3.0 which released in December 2017


## Slide 11

### Hadoop: layers


Cluster
 	(of shared nothing nodes)


Com  mon


MapReduce |  | others
YARN |  | 
Hadoop File System
(HDFS) |  | 


others


## Slide 12

### Hadoop key components


HDFS as a distributed file system
for managing	data
YARN as a resource management system
for executing and managing analytics tasks
MapReduce as one programming model
for	MapReduce applications
Hadoop Common: 
Libraries and utilities, which provide tools and infrastructure for the entire Hadoop ecosystem, such as authentication, configuration, and logging.


## Slide 13

### ●
●
●
●


## Slide 15

### Hadoop File System (HDFS)


For handling very big data files
GBs of data within a single file
Assumption of data handling
write-once-read-many
not suitable for random-access update → analytics data
Deal with hardware failures, support data locality,  reliability
“high fault tolerance” with “low-cost/commodity hardware”


## Slide 16

### HDFS Architecture


Namenode
Manages the file system namespace. 
Regulates client’s access to files. 
It also executes file system operations such as renaming, closing, and opening files and directories
Datanode
Datanodes perform read-write operations on the file systems, as per client request. 
They also perform operations such as block creation, deletion, and replication according to the instructions of the namenode.
Block data


## Slide 17

### https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html


## Slide 18

### 1


3


4


6


7


8


2


5


7


3


5


1


8


Datanodes


NAME NODE


## Slide 20

### 1


3


4


6


7


8


2


5


7


3


5


1


8


Datanodes


## Slide 23

### 1


3


4


6


7


8


2


5


7


3


5


1


8


Datanodes


## Slide 25

### 1


3


4


6


7


8


2


5


7


3


5


1


8


Datanodes


1


3


New datanode is created  by the namenode


## Slide 27

### Snapshot from: https://data.cityofnewyork.us/Transportation/2018-Yellow-Taxi-Trip-Data/t29m-gskq


Example


e.g., 112M rows


## Slide 28

### HDFS - data blocks


Files are stored in many nodes
but the application accesses HDFS files just like “typical file  systems”
A file includes many blocks
File blocks are replicated and distributed across nodes
Conventional way of access data
naming resolving: hdfs://
common operations: list, put, get, …


## Slide 29

### File blocks, metadata and data  replication


Block size is 128MB (default)
can be configurable but should not be small size
all blocks of the same file are the same, except the last  one
Data is replicated across the cluster
usually, replication factor is 3
NameNode manages file system metadata


## Slide 30

### HDFS fault tolerance


Data blocks
File blocks are replicated and distributed across nodes
Rack awareness
avoid communication problems between nodes in different racks
Monitoring
DataNode reports to NameNode
Read and write
using NameNode for metadata and for information of DataNodes
NameNode has replication (master-worker)


## Slide 31

### Compatible file systems with HDFS


Many file systems are compatible with HDFS
for integration and analysis purpose: m
Examples:
Amazon S3
Azure Blob Storage
Azure Data Lake Storage
OpenStack Swift


## Slide 32

### ●
○
■

○

○


## Slide 36

### Master Node
Spark		Hadoop  YARN


Worker A


Worker B


## Slide 37

### Capacity Scheduler


Resource Manager


AM 1


NM A


Worker A


AM 2


NM B


Worker B


AM = Application Master  NM = Node Manager


## Slide 38

### Resource Manager


Capacity Scheduler


AM 1


NM A


Worker A


AM 2


NM B


Worker B


AM = Application Master  NM = Node Manager


## Slide 39

### Resource Manager


Capacity Scheduler


AM 1


NM A


Worker A


AM 2


NM B


Worker B


AM = Application Master  NM = Node Manager


## Slide 40

### Resource Manager


Capacity Scheduler


AM 1


NM A


Worker A


AM 2


NM B


Worker B


AM = Application Master  NM = Node Manager


## Slide 41

### Resource Manager


Capacity Scheduler


AM 1


NM A


Worker A


AM 2


NM B


Worker B


AM = Application Master  NM = Node Manager


## Slide 42

### Resource Manager


Capacity Scheduler


AM 1


NM A


Worker A


AM 2


NM B


Worker B


AM = Application Master  NM = Node Manager


Task


## Slide 43

### Resource Manager


Capacity Scheduler


AM 1


NM A


Worker A


AM 2


NM B


Worker B


AM = Application Master  NM = Node Manager


Task 1


Task 2


## Slide 44

### YARN (Yet Another Resource  Negotiator)


Manage resources for processing tasks
each node in the cluster provides resources for executing tasks
Computing resource types:
CPU, Memory and Disks
also support GPU and FPGA Node
Computing resources are abstracted into  “Containers”
don’t be confused: it is not a (Docker) container!
Multi-tenancy support


## Slide 45

### With a scheduler


Without a scheduler


## Slide 47

### RM = Resource Manager  AM = Application Master  NM = Node Manager


## Slide 48

### ResourceManager HighAvailability


## Slide 49

### Task Worker C


Task Worker D


YARN // Resource Manager


AM 1


NM A


AM 2


NM B


HDFS // Name Node


MapReduce


Worker A

Data  Block 1


Worker B

Data  Block 2


AM 3


NM C


AM 4


NM D


## Slide 50

### YARN Scheduler Policies


FIFO Scheduler 
Capacity Scheduler 
Fair Scheduler


## Slide 51

### Programming models


YARN is an execution environment
YARN allows different programming models  for applications
MapReduce
Apache Spark
Workflows
E.g., Apache Tez for DAG (direct acyclic  graph) of data processing tasks


## Slide 52

### Integration models


Using Hadoop for developing large-scale data  analysis
Apache Spark, HBase, Hive, Apache Tez
Using Hadoop HDFS as components in a big data  system
Hadoop HDFS can become data storage → storage layer
emerging data lake models, combined batch and stream  ingestions for incremental data processing
e.g., Apache Hudi


## Slide 53

### Hadoop (and additional frameworks)


ETL and Analytics with Hadoop/HDFS


HDFS


Analysis  Engine


Application


Data


Source1


Data


Source1


Data


Source n


Data Ingestion:


HDFS Client/Hadoop  Streaming


Spark Streaming
Kafka Connect
Apache Nifi


Computing/Data Processing  Framework
Apache Spark


Hadoop MapReduce
Apache Tez


Ingestion  Engine


Analysis


result


Database


HDFS as storage for  databases
Accumulo, Druid, etc.


Analysis


## Slide 54

### ●
●
●
●


## Slide 55

### ●
●
○


$ hadoop jar $STREAMING_JAR \
-files mapper.py,reducer.py
-mapper mapper.py -reducer reducer.py \
-input /shared/my_cool_dataset -output my_job_out


## Slide 56

### ●

$ python my_mr_job.py -r hadoop

--output-dir my_job_out
--no-output

# HDFS directory
# Suppress output via STDOUT


hdfs:///data/my_cool_dataset	# HDFS path for input


## Slide 57

### ●
○

●
○
○
○


## Slide 58

### ●
○

●
○
○
○

●
○
○
○


## Slide 59

### ●
●
●
●


Hbase and Hive


## Slide 62

### Is that Hadoop dead/unattractive?


Hadoop is for shared nothing architecture
Shared nothing architectures are not “modern”?
new, advanced developments w.r.t. memory, networks, …
in-memory processing, move data to computing nodes
Many tools seem not powerful for data science/ML?
Java vs Python development for data science and ML
services in ecosystem
near realtime analytics use cases
So why do we still study it?
foundational designs & still many Hadoop services are important (data  warehouse, deep storage)
well-supported by many cloud providers for big data workloads
e.g.,


## Slide 63

### Why do we still need to learn  Hadoop?


Hadoop file systems as a storage for many big data services
databases: Accumulo, Apache Druid
data warehouses/Data Lakes: Hive, Hudi
Support different models of access/analytics
via SQL styles atop big data platforms: You can do  extract/transform/load (ETL), reporting, and data analysis using  SQL styles
large-scale parallel programming (e.g., Spark)
Still many needs, given shared nothing architectures  and on-premise conditions


## Slide 64

### Why do we still need to learn


Hadoop?


SQL-style


Data parallelism


needs for  internal  processing


enables the  implementation


SQL-on-Hadoop
offers SQL features for  analytics


Figure source: https://docs.dask.org/en/stable/graphs.html


Analytics/  ML


YARN Distributed  executors


## Slide 65

### Enabling analysis big data using SQL  style


Provide command line tools & JDBC and server for  integration
“SQL-on-Hadoop”
Examples
Apache Hive, https://hive.apache.org/
Data warehouse, access data in HDFS or Hbase
Apache Druid (using HDFS as a deep storage)
Spark SQL for various types of files


## Slide 66

### Integration with other cloud storage


services


Cloud
big  datasets


Bucket |  | Bucket | Bucket | 
Node | Node |  |  | Node


On-premise  or cloud


Blob  Storage &  Data Lake in  the Cloud


Dremio | Spark | …
Driver/Connector |  | 


## Slide 68

### HBase


NoSQL database atop Hadoop
use HDFS for storing data
use YARN for running jobs on data
Follow a master-based architecture


Reading – Why HBase?
https://engineering.fb.com/2010/11/15/core-data/the-underlying-technology-of-messages/  https://engineering.fb.com/2014/06/05/core-data/hydrabase-the-evolution-of-hbase-facebook/  https://engineering.fb.com/2018/06/26/core-data/migrating-messenger-storage-to-optimize-performance/


## Slide 69

### Recall: Big data: column-family data  model
Many situations we aggregate and scan few columns of million  rows of data ⇒ store big data in columns enable fast  scan/retrieval/aggregation


Column Family = (Column, Column, …):


for similar type of data &  access patterns


Column Key =Family: qualifier

Data = (Key, Value)	where Key =(Row Key, Column Key, Timestamp)


## Slide 70

### Example of a data model in HBase


Column family  (e.g., birdinfo)


Row key


Column


1


species


birdinfo:  country


birdinfo:eng  lish_cname


duration


Aberti


US


…


3


2


species


birdinfo:c  ountry


birdinfo:englis  h_cname


duration


name


url


latitude


longitude


Text


Cell:  versioning


Row


Sparsely stored: not all rows have the same number of columns


## Slide 71

### Example of a data model


Example with families: birdinfo, songinfo, location


Enable analytics based on column families (as well as data  management)


## Slide 72

### HBase data model – sharding and  storage


Table includes multiple Regions
a Region keeps related row data of a Table (partitioning)
Auto-sharding
Regions are spitted based on policies
Region has data of multiple column families
Different column families will be stored in different files
HFiles are used to store real data (also include index data)


## Slide 73

### HDFS


HBase architecture


HMaster


RegionalServer


RegionalServer	RegionalServer


Data Node


Data Node


Data Node


Zookeeper


Zookeeper:
shared state and failure  notification of servers
master address and  recovery


## Slide 74

### RegionalServer


Region


Store


HBase architecture


HFile


MemStore


Region


WAL (Write


Ahead Log)


MemStore: write  cache for data in  memory before  written into files


BlockCache: for  read cache


WAL is for  durability


HFile


BlockCache


## Slide 75

### ACID


Atomic within a row
Consistency
can be programmed: e.g.,
read: STRONG (read performed by the primary  region) and TIMELINE (primary region first, if not,  then the secondary region)
Durability
can be programmed
 	○	WAL (write ahead log)


## Slide 76

### Example of using HBase


Salesforce Shield: for  supporting data  security/compliance
Field Audit Trail: for track  changes
Event Monitoring: log/event
Argus: time series data and


alerts


Figure source: http://opentsdb.net/overview.html


Open Scalable Open Time  Series Database (OpenTSDB)


Salesforce


Source:
https://engineering.salesforce.com/investing-in-big-data-apache-hbase-4  20edfba2d30/


## Slide 77

### Apache Hive


## Slide 78

### Apache Hive


https://hive.apache.org/, on top of Hadoop
data warehouse at a very large-scale
access data in large-scale storage like HDFS or S3
Support access to data via SQL styles
extract/transform/load (ETL), reporting, and data  analysis using SQL styles
Provide command line tools & JDBC and server for  integration


Reading – Hive in Facebook
https://engineering.fb.com/2009/06/10/web/hive-a-petabyte-scale-data-warehouse-using-hadoop/


## Slide 79

### Hadoop


High-level data flow language &  programs


Hive script  (HiveQL)


SQL-alike


Hive Shell and  Execution  Engine


Schema


Distributed Task  Execution Engine


MegaStore


Hive  Shell


HDFS | Amazon
S3
 | Other
storages


## Slide 80

### Hive building blocks


Figure source: https://cwiki.apache.org/confluence/display/Hive/Design


Distributed  tasks with  MapReduce
,
Tez  (Workflow)  or
Spark


## Slide 81

### Hive data organization


Schema-on-read approach
schema inferred from data, structured and unstructured data
Databases
Table
Managed table versus External tables
External table: real data is stored outside Hive and referenced so  delete only table metadata but not the data (read data only)
Table is mapped to a directory in HDFS
Low-level files: CSV, Apache Parquet,ORC, …
Managed vs External tables concept:
also in BigQuery, Snowflake, Databricks Lakehouse


## Slide 82

### Example


Large tables lead  to performance  issues


## Slide 83

### Hive data organization for  performance optimization


Partitioning: using value of a column as a partitioning key
partition keys determine how data in Table will be divided
E.g. date or countries
each partition is stored as a subdirectory
Avoid many sub directories!
Buckets: using a hash function of a column for grouping  records into the same bucket
avoid large number of small partitions
each bucket is stored in a file
bucket columns should be optimized for join/filter operations


## Slide 84

### Example of partitions


CREATE TABLE taxiinfo1 ( ….)  PARTITIONED BY	(year int, month int)
…;


LOAD DATA LOCAL INPATH …… INTO TABLE taxiinfo1  PARTITION (year=2019, month=11);


Define partition names


Indicate	partition info


## Slide 85

### Example of buckets


CREATE TABLE taxiinfo2 (VendorID int, ….)  CLUSTERED BY (VendorID) INTO 2 BUCKETS
……;
Identify bucket column


Combine partitions with buckets


## Slide 86

### ACID


ACID support
for managed tables
different for CRUD and insert-only tables
Locks are used for data isolation
shared lock: for concurrent read of  tables/partitions
exclusive lock: for modifying table/partition


## Slide 87

### Uber example


Figures source:
https://www.uber.com/en-FI/blog/uber-big-data-  platform/


Hive for analytics: “extremely large  queries”, “ “20,000 Hive queries per


day” (see the source)


CS-E4640 Big Data Platforms, Spring 2024, Hong-Linh Truong
07/02/2024
87


## Slide 88

### Hive in the age of ML


Hive has been around for a while → many goals have  been diminished
But	Hive Megastore continues an important role in  data lakes and distributed data querying
power data management and discovery
support unified data management
enable “feature store”, discovery features used for machine  learning


## Slide 89

### Summary


Hadoop software ecosystem is very powerful
many applications and use cases have been developed
Hadoop File System (HDFS) is a crucial subject
Managed Hadoop ecosystem services by cloud  providers
try to look at Azure HDInsight, Google Dataproc, and Amazon EMR
High-level distributed query engineering using  Hadoop components (HDFS, Hive)
Understand the combination of data management  with data processing techniques in the same system
 	with	Hadoop that simplify your big data tasks
