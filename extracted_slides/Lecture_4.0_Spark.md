# Lecture_4.0_Spark


## Slide 1

### Spark


## Slide 2

### Learning objectives


Be familiar with big data processing models using  multiple nodes/clusters
Understand the Spark programming model for big  data processing
Able to perform practical programming features with  Apache Spark


Leaning objective


## Slide 3

### Why Spark ?
Spark Architecture and Spark Core
RDD and its operations
Lazy evaluation in Spark
Spark vs. Hadoop
Spark Use Cases
Spark Programming


Outline


## Slide 4

### Is it big


Snapshot from: https://data.cityofnewyork.us/Transportation/2018-Yellow-Taxi-Trip-Data/t29m-gskq


e.g., 112M rows (2021 check)


a 5.31GB CSV file containing data on taxi rides (fare amount, number of passengers, pickup time, and pickup and dropoff locations)


Is it big ?


## Slide 5

### Is it big ?


## Slide 6

### Is it big


What happen if we use traditional tools such as Panda to read a big dataset ?


Question ?


Pandas loads the entire dataset into RAM, which is fine for small datasets but fails for large ones. 
If the dataset is larger than the available memory, Python will either: 
Crash with a MemoryError. 
Start swapping (using disk as memory), which slows performance significantly.
Pandas is single-threaded by default, meaning it doesn’t take full advantage of multiple CPU cores. 
Large datasets cause slow processing and high CPU usage.


## Slide 7

### Better than Pandas ?


DuckDb
columnar-vectorized engine
Polars
parallelization of DataFrame from the beginning
Rust-based with Python bindings, which has outstanding performance comparable to C
Lazy evaluation: plan (not execute) the query until triggered
Vaex
Modin


## Slide 8

### Motivations / History


MapReduce began to show it’s downsides:
It isn’t fast enough
It’s inefficient on iterative workloads
It relies too heavily on on-disk operations - Most of the Hadoop applications, they spend more than 90% of the time doing HDFS read-write operations


Research group at UC Berkeley develop Spark
Started in 2009
Initial versions outperformed MapReduce by 10-20x


## Slide 9

### Apache Spark


Open Source, Distributed general-purpose computing framework.


Managed by Apache Foundation
Written in Scala
Robust high-level APIs for different languages
Allows iterative computations
-	Graph algorithms, ML, and more


## Slide 10

### Computational Framework


You write framework code, as distinct from “normal code”

Code that tells the framework what to do, not how to do it
The framework can handle optimization / data transfer internally

Other computational frameworks:
Tensorflow, Caffe, etc.


## Slide 11

### Computational Framework


## Slide 12

### Spark submit


The spark-submit command is a utility to run or submit a Spark or PySpark application program (or job) to the cluster by specifying options and configurations


## Slide 13

### Spark UI


Apache Spark provides a suite of Web UIs (Jobs, Stages, Tasks, Storage, Environment, Executors, and SQL) to monitor the status of your Spark application


## Slide 14

### Architecture


Worker


Driver

SparkContext |  | 
 |  | Cluster Manager
 |  | 
 |  | 


Executor


TASK


TASK


CACHE


Worker


Executor


TASK


TASK


CACHE


Executor


TASK


TASK


CACHE


Application


## Slide 16

### Architecture


Driver
One per job, where the job is submit to
Handles DAG scheduling, schedules tasks on executors
Tracks data as it flows through the job
Failure recovery and retries

Executor: JVM processes launched on worker nodes.
Possibly many per job, Possibly many per worker node
Stores RDD data in memory
Performs tasks (operations on RDDs), and transfers data as needed


## Slide 17

### Executor Allocation


Traditional Static Allocation
Create all Executors at beginning of job
Executors are online until end of job
Only option in early versions of Spark

Dynamic Executor Allocation
Jobs can scale up/down number of executors as needed
More efficient for clusters running multiple apps concurrently
The cluster manager must support dynamic allocation (e.g., YARN, Kubernetes, Mesos, standalone cluster manager with an external shuffle service)


## Slide 18

### Dynamic Executor Allocation


## Slide 19

### Resilient Distributed Dataset (RDD)


Data Abstraction in Spark.
Resilient
Fault-tolerant using a data lineage graph
RDDs know where they came from, and how they were computed
Distributed
Data lives on multiple nodes
RDDs know where they’re stored, so computation can be done “close” to the data
Dataset
A collection of partitioned data


## Slide 20

### Resilient Distributed Dataset (RDD)


What does an RDD look like?
A large set of arbitrary data (tuples, objects, values, etc)

Features of RDDs:
Stored in-memory, Cacheable
Stored on executors
Immutable
Once created, cannot be edited
Must be transformed into a new descendent RDD
Parallel via partitioning
Similar to how Hadoop partitions map inputs


## Slide 21

### Example with RDD


VendorID,tpep_pickup_datetime,tpep_dropoff_datetime,passenger_count,trip_distance,RatecodeID,store_and_fwd_fl  ag,PULocationID,DOLocationID,payment_type,fare_amount,extra,mta_tax,tip_amount,tolls_amount,improvement_sur  charge,total_amount
2,11/04/2084 12:32:24 PM,11/04/2084 12:47:41 PM,1,1.34,1,N,238,236,2,10,0,0.5,0,0,0.3,10.8
2,11/04/2084 12:32:24 PM,11/04/2084 12:47:41 PM,1,1.34,1,N,238,236,2,10,0,0.5,0,0,0.3,10.8
2,11/04/2084 12:25:53 PM,11/04/2084 12:29:00 PM,1,0.32,1,N,238,238,2,4,0,0.5,0,0,0.3,4.8


as a text file


## Slide 23

### RDD transformations and actions


Transformations - create a new RDD/DataFrame without immediately executing


map
filter
sample
intersection
groupByKey
join


Actions – trigger execution and returns result


reduce()
collect()
count()
write()
take()
saveAs…File()


RDD Transformations and Actions


## Slide 24

### Spark Application Components


Job
An application can have multiple jobs
a complete computation triggered by a RDD action (e.g. collect, count, show)
Stage
A group of potentially many operations, e.g. a set of tasks that can be executed without shuffle dependencies.
Many executors work on tasks in a single stage
Narrow transformations (like map, filter, select) are pipelined into a single stage. 
Wide transformations (like groupBy, reduce, join) create shuffle boundaries, resulting in additional stages.
Task
The “simplest unit of work” in Spark
One operation on a partition of an RDD
Fully parrallel


## Slide 25

### The Spark Flow


JA Spark job is submitted to Spark for processing.
The job is divided into stages based on the transformations and dependencies in the computation.
Each stage consists of multiple tasks that can be executed in parallel.
Tasks are assigned to worker nodes, which perform the required computations on the data partitions assigned to them.
Intermediate and final results are produced as tasks complete their computations.
The overall job completes when all stages and tasks finish execution


## Slide 26

### Monitoring Spark: Executors and tasks


## Slide 27

### Executors and tasks


## Slide 28

### RDD operations


## Slide 29

### RDD Operations


Transformations
The process of taking an input RDD, doing some computation, and producing a new RDD
Done lazily (only ever executed if an “action” depends on the output)
i.e. Higher order function like Map, ReduceByKey, FlatMap
Actions
Triggers computation by asking for some type of output
i.e. Output to text file, Count of RDD items, Min, Max, 
Collect, show, save


## Slide 30

### Lazy evaluation in Spark


is a fundamental concept that optimizes data processing by delaying the execution of transformations until an action is performed.
Deferred Execution
Transformations (e.g., map, filter) are not executed immediately
Spark records these transformations as a logical plan
analyzes and optimizes the logical plan before execution, combining or reordering operations
Only when an action (e.g., count, collect, save) is called does Spark execute the transformations


## Slide 31

### What does it do?
Computes an execution DAG

Determines the preferred locations (Executors) to run each task

Handles failure due to lost shuffle output files

Performs operation optimizing
Groups multiple operations (e.g. maps and filters) into the same stage


DAGScheduler


## Slide 32

### DAGScheduler - Directed Acyclic Graph Scheduler


## Slide 33

### RDD Workflow


## Slide 34

### Storage for Spark


Again, much more flexibility.
HDFS
S3
Cassandra
HBase
Etc.


While Hadoop (mostly) limited to HDFS, Spark can bring in data from anywhere


## Slide 35

### RDD Operations - Cache


Persistence:
     cache() uses default storage level, i.e., MEMORY_ONLY.
    persist() can be provided with any of the possible storage levels memory/disk
    When an RDD is persisted, each node in the cluster stores the partitions (of the RDD) in memory (RAM).
    When there are multiple transformations or actions on an RDD, persistence helps to cut down the latency by the time required to load the data from file storage to memory.


## Slide 36

### Example – Without Cache


## Slide 37

### Example – With Cache


## Slide 38

### Resource managers for Spark


Allocating resources (CPU cores, memory, or GPUs) to the tasks and jobs submitted to the cluster
Job/task scheduling
Reassigning tasks from failed jobs etc.


## Slide 39

### Resource managers for Spark


Standalone Cluster Manager
Apache Hadoop YARN (Yet Another Resource Negotiator)
Apache Mesos
Kubernetes (K8s)


## Slide 40

### Spark deployment modes - Local mode


Executing a program on your laptop using single JVM. It can be java, scala or python program where you have defined & used spark context object, imported spark libraries and processed data residing in your system
Spark runs the driver and workers in the same JVM 
both the Master and Worker(s) run as threads within a single Java Virtual Machine process


## Slide 41

### Spark deployment modes - Cluster mode


Standalone: Master -> Workers
Spark distribution comes with its own resource manager also. When your program uses spark's resource manager, execution mode is called Standalone.  There are multiple JVMs
YARN/Mesos/K8s – ResourceManager + NodeManager + 
In reality Spark programs are meant to process data stored across machines. Executors process data stored on these machines. We need a utility to monitor executors and manage resources on these machines( clusters).


## Slide 42

### Deployment modes submit commands


## Slide 43

### Much more flexibility than Hadoop


Aims for easy and interactive analytics
Distributed Platform for complex multi-stage applications, (e.g. real-time ML)


Spark Core


Spark  Standalone


YARN


MESOS


SQL


Streaming


MLlib


GraphX


Infrastructure  Layer


Framework Layer


Library/Application  Layer


## Slide 44

### Related Frameworks


Spark Core is tightly integrated with several key libraries


Spark Core


SQL


Streaming


MLlib


GraphX


## Slide 45

### Related Frameworks


Spark Streaming
Stream live data into Spark cluster
Send it out to databases or HDFS
Spark SQL
Integrates relational database programming (SQL) with Spark
Spark MLlib
Large-Scale Machine Learning
GraphX
Graph and graph-parallel computations


Interactions between the frameworks allow multi-stage data applications.


## Slide 46

### Catalyst Optimizer in Spark SQL


Optimizes logical plans for DataFrames and Datasets. 
Performs rule-based and cost-based optimization.


## Slide 47

### Catalyst Optimizer in Spark SQL


Not calculate 1 + 2 for all  
Filter t2.id > 50 before JOIN


## Slide 49

### Spark Overview
Spark Core
Related Framework
Spark vs. Hadoop
Spark Use Cases
Spark Programming


Outline


## Slide 50

### Hadoop vs. Spark


Spark can be more than 100x  faster, especially when performing  computationally intensive tasks.


## Slide 51

### Hadoop vs. Spark


Step 1. Build something
Step 2. Prove its 100x faster than Hadoop  Step 3. ???
Step 4. Profit!


## Slide 52

### Hadoop + Spark


Spark on HDFS
What can they bring to the table for each other?
Hadoop
Huge Datasets under control by commodity hardware.
Low cost operations
Spark
Real-time, in-memory processing for those data sets.
High-speed, advanced analytics to a multiple stage operations.

Spark cannot yet completely replace Hadoop.


## Slide 56

### Spark Overview
Spark Core
Related Framework
Spark vs. Hadoop
Spark Use Cases
Deploying Spark


Outline


## Slide 57

### When to use Spark


Building a data pipeline
Interactive analysis and multi-stage data application.
Allows real-time interaction / experimentation with data

Streaming Data : Spark Streaming  
Machine Learning: Spark MLlib


## Slide 58

### Spark in Industry


Commerce Industry
eBay
Provide targeted offers, enhance customer experience, etc.
eBay runs Spark on top of YARN.
-	2000 nodes, 20,000 cores, and 100TB of RAM
Alibaba
Feature extraction on image data, Aggregate data on the platform
Millions of Merchant-User Interaction is represented in graphs.


## Slide 59

### Spark in Industry


Finance Industry
Real-Time Fraud Detection (stolen credit card swipe or stolen card number)
Check with previous fraud footprint
Triggers call center, etc.
Validate incoming transactions
Risk-based assessment
Collecting and archiving logs
Spark can easily be combined with external data source and pipelines.


## Slide 60

### Spark Overview
Spark Core
Related Framework
Spark vs. Hadoop
Spark Use Cases
Spark Programming


Outline


## Slide 61

### What Does Spark Code Look Like?


count = len([line for line in \  open('file.txt') \
if 'pattern' in line])  print(count)


file = sparkContext.textFile("file.txt")  matcher = lambda x: x.contains("pattern")  count = file.filter(matcher).count()  print(count)


## Slide 62

### Example


Sentient Spark Object


Transformation  Function
file = sparkContext.textFile("file.txt")
matcher = lambda x: x.contains("pattern")  count = file.filter(matcher).count()  print(count)


Action
Transformation


## Slide 63

### Word Count Example


# Load data
textData = sparkContext.textFile("input.txt")


# Split into words
WORD_RE = re.compile(r"[\w']+")
words = textData.flatMap(lambda line: WORD_RE.findall(line))


# Get count by word
counts = words.map(lambda w: (w, 1)).countByKey()


print(counts.collect())


## Slide 64

### Join Example


favColors = sc.parallelize([('bob', 'red'), ('alice', 'blue')])
favNumbers = sc.parallelize([('bob', 1), ('alice', 2)])

joined = favColors.join(favNumbers)  joined.collect()


# [('bob', ('red', 1)), ('alice', ('blue', 2))]


## Slide 65

### ReduceByKey Example


nums = sc.parallelize([('a', 1), ('b', 2), ('a', 3), ('b', 4)])  reduced = nums.reduceByKey(lambda v1, v2: v1 + v2)  reduced.collect()


# [('b', 6), ('a', 4)]
