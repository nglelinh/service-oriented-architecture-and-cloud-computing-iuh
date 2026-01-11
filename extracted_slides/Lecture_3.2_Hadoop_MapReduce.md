# Lecture_3.2_Hadoop_MapReduce


## Slide 1

### MapReduce


## Slide 2

### A bit about Distributed Systems
MapReduce Overview
MapReduce in Industry
Programming Hadoop MapReduce Jobs
Mappers and Reducers
Operating Model


Outline


## Slide 3

### A bit about Distributed Systems
MapReduce Overview
MapReduce in Industry
Programming Hadoop MapReduce Jobs
Mappers and Reducers
Operating Model


Outline


## Slide 4

### Running computation on large amounts of data
Want a Framework that scales from 10GB => 10TB => 10PB


High throughput data processing
Not only processing lots of data, but doing so in a reasonable timeframe


Cost efficiency in data processing
Workloads typically run weekly/daily/hourly (not one-off)
Need to be mindful of costs (hardware or otherwise)


Our Primary Concerns:


## Slide 5

### What traditionally restricts performance?


Processor frequency (Computation-intensive tasks)
Fastest commodity processor runs at 3.7 - 4.0 Ghz
Rough correlation with instruction throughput


Network/Disk bandwidth (Data-intensive tasks)
Often, data processing is computationally simple
Jobs become bottlenecked by network performance, instead of computational  resources


## Slide 6

### Moore’s Law


The number of transistors  in a dense integrated  circuit doubles  approximately every two  years


It’s failing!


## Slide 7

### Parallelism


If Moore’s law is slowing down how can we process more data at local  scale?
More CPU cores per processor
More efficient multithreading / multiprocessing

However, there are limits to local parallelism…
Physical limits: CPU heat distribution, processor complexity
Pragmatic limits: Price per processor, what if the workload isn’t  CPU limited?


## Slide 8

### Distributed Systems from a Cloud Perspective


Mindset shift from vertical scaling to horizontal scaling
Don’t increase performance of each computer
Instead, use a pool of computers - (a datacenter, “the cloud”)
Increase performance by adding new computer to pool
(Or, buy purchasing more resources from a cloud vendor)


## Slide 9

### Distributed Systems from a Cloud Perspective


Vertical Scaling - “The old way”
Need more processing power?
Add more CPU cores to your existing machines
Need more memory?
Add more physical memory to your existing machines
Need more network bandwidth?
Buy/install more expensive networking equipment


## Slide 10

### Distributed Systems from a Cloud Perspective


Horizontal Scaling
Standardize on commodity hardware
Still server-grade, but before diminishing returns kicks in
Need more CPUs / Memory / Bandwidth?
Add more (similarly spec’d) machines to your total resource pool
Still need to invest in good core infrastructure (machine interconnection)
However, commercial clouds are willing to do this work for you

Empirically, horizontal scaling works really well if done right:
This is how Google, Facebook, Amazon, Twitter, et al. achieve high performance
Also changes how we write code
We can no longer consider our code to only run sequentially on one computer


## Slide 11

### A bit about Distributed Systems
MapReduce Overview
MapReduce in Industry
Programming Hadoop MapReduce Jobs
Mappers and Reducers
Operating Model


Outline


## Slide 12

### Motivation


Large amount of raw data to process

Conceptually straightforward computation

Distributed computations → Finish in reasonable time

Problems:
Parallelize computations
Distribute data
Handle failures


Motivation


## Slide 13

### MapReduce


What it is:
A programming paradigm/model to break data processing jobs into distinct stages which  can be run in a distributed setting
Big Idea:
Restrict programming model to get parallelism “for free”

Most large-scale data processing is free of “data dependencies”
Results of processing one piece of data not tightly coupled with results of  processing another piece of data
Increase throughput by distributing chunks of the input dataset to different  machines, so the job can execute in parallel


## Slide 14

### MapReduce: Overview


First version in 2003 by Google
Significant growth of usage
Applications:
Large-scale machine learning problems
Clustering problems for the Google News
Extraction of data in queries &
Extraction of properties of web pages
Large-scale graph computations
…
Rewrite the production indexing system for the Google web search service


MapReduce: Overview


## Slide 15

### What is MapReduce


Inspired by the map and reduce primitives
In Lisp & other languages


Advantages:
Allow user defined computations
Hides messy details in a library:
Parallelization
Fault-tolerance
Data distribution
Load balancing


What is MapReduce


## Slide 16

### MapReduce


A job is defined by 2 distinct stages:
Map - Transformation / Filtering
Reduce - Aggregation

Data is described by key/value pairs
Key - An identifier of data
I.e. User ID, time period, record identifier, etc.
Value - Workload specific data associated with key
I.e. number of occurences, text, measurement, etc.


## Slide 17

### Map & Reduce


Map


A function to process input key/value pairs to generate a set of intermediate key/value pairs.
Values are grouped together by intermediate key and sent to the Reduce function.


Reduce


A function that merges all the intermediate values associated with the same intermediate key  into some output key/value per intermediate key


<key_out, val_out>


<key_input, val_input>	⇒	<key_inter, val_inter>	⇒
Map	Reduce


## Slide 18

### Map & Reduce - Word Count


Problem: Given a “large” amount of text data, how many occurences of each  individual word are there? Not fit into a single computer memory?
Essentially a “count by key” operation

Generalizes to other tasks:
Counting user engagements, aggregating log entries by machine, etc.

Map Phase:
Split text into words, emitting (“word”, 1) pairs
Reduce Phase:
Calculate the sum of occurrences per word


## Slide 19

### Map & Reduce - Word Count


Input Data:
“ABCAACBCD”


Mapper


Reducer


Output Data


## Slide 20

### Map & Reduce - Word Count


Input Data:
“ABCAACBCD”


Mapper


Reducer


Output Data


“A B C”


“A A C”


“B C D”


## Slide 21

### Map & Reduce - Word Count


Input Data:
“ABCAACBCD”


Mapper


“A”


“B”
Reducer

“C”


“D”


Output Data


“A B C”


“A A C”


“B C D”


(“A”, 1)


(“B”, 1)


(“C”, 1)


“Shuffle and Sort”


## Slide 22

### Map & Reduce - Word Count


Input Data:
“ABCAACBCD”


Mapper


“A”


“B”
Reducer

“C”


“D”


Output Data


“A B C”


“A A C”


“B C D”


(“A”, 1)


(“B”, 1)


(“C”, 1)


(“C”, 1)


(“A”, 1)
(“A”, 1)


“Shuffle and Sort”


## Slide 23

### Map & Reduce - Word Count


Input Data:
“ABCAACBCD”


Mapper


“A”


“B”
Reducer

“C”


“D”


Output Data


“A B C”


“A A C”


“B C D”


(“A”, 1)


(“B”, 1)


(“C”, 1)


(“C”, 1)


(“B”, 1)


(“C”, 1)


(“D”, 1)


“Shuffle and Sort”


(“A”, 1)
(“A”, 1)


## Slide 24

### Map & Reduce - Word Count


Input Data:
“ABCAACBCD”


Mapper


Reducer


Output Data


“A B C”


“A A C”


“B C D”


(“A”, 1)


(“B”, 1)


(“C”, 1)


(“C”, 1)


“A”


“B”


“C”


(“B”, 1)


“D”


(“C”, 1)


(“D”, 1)


“Shuffle and Sort”


(“A”, 3)


(“B”, 2)


(“C”, 3)


(“D”, 1)


(“A”, 1)
(“A”, 1)


## Slide 25

### Map & Reduce - Word Count


Input Data:
“ABCAACBCD”


Output Data


“Shuffle and Sort”


Node 2


Node 1


Node 3


Node 4


Node 5


Node 6


Node 7


Map Phase


Reduce  Phase


## Slide 26

### Map & Reduce - Word Count


Input Data:
“ABCAACBCD”


Output Data


“Shuffle and Sort”


Node 2


Node 1


Node 3


Node 4


Node 5


Map Phase


Reduce  Phase


## Slide 27

### Map & Reduce


Why is Map parallelizable?
Input data split into independent chunks which can be transformed / filtered
independently of other data

Why is Reduce parallelizable?
The aggregate value per key is only dependent on values associated with that key
All values associated with a certain key are processed on the same node

What do we give up in using MR?
Can’t “cheat” and have results depend on side-effects, global state, or partial  results of another key


## Slide 28

### Map & Reduce - Shuffle/Sort In-Depth


Combiner - Optional
Optional step at end of Map Phase to pre-combine intermediate values before  sending to reducer
Use combiners to perform partial aggregation during the Map phase, reducing data transfer to the Reduce phase.
Like a reducer, but run by the mapper (usually to reduce bandwidth)
Partition / Shuffle
Mappers send intermediate data to reducers by key (key determines which reducer  is the recipient)
“Shuffle” because intermediate output of each mapper is broken up by key and  redistributed to reducers
Secondary Sort - Optional
Sort within keys by value
Value stream to reducers will be in sorted order


## Slide 29

### Map & Reduce - Shuffle/Sort - Combiner


Map


Reduce


Mapper 1:  “ABABAA”




Mapper 2:  “BBCCC”





Mapper 3  “CCCC”


Reducer 1






Reducer 2





Reducer 3


## Slide 30

### Map & Reduce - Shuffle/Sort - Combiner


Map


Reduce


(“A”,


Mapper 1:  “ABABAA”


Mapper 2:  “BBCCC”


Mapper 3  “CCCC”


(“A”,


1) (“A”,  1)(“A”,
1)	1)


(“B”, 1)


(“B”, 1)


(“B”, 1)  (“B”, 1)
(“C”, 1)
(“C”, 1)(“C”, 1)


(“C”, 1)


(“C”, 1)


(“C”, 1) (“C”, 1)


Reducer 1






Reducer 2





Reducer 3


(“A”, 4)


(“B”, 4)


(“C”, 7)


## Slide 31

### Map & Reduce - Shuffle/Sort - Combiner


Map


Reduce


Mapper 1:  “ABABAA”


Mapper 2:  “BBCCC”


Mapper 3  “CCCC”


Reducer 1






Reducer 2





Reducer 3


Combiner


Combiner


Combiner


## Slide 32

### Map & Reduce - Shuffle/Sort - Combiner


Map


Reduce


(“A”,


Mapper 1:  “ABABAA”


Mapper 2:  “BBCCC”


Mapper 3  “CCCC”


1) (“A”,


(“A”,


1)(“A”,


1)	1)	
(“B”, 1)  (“B”, 1)


(“B”, 1)  (“B”, 1)


(“C”, 1)
(“C”, 1)(“C”, 1)


(“C”, 1)


(“C”, 1)


(“C”, 1) (“C”, 1)


Reducer 1






Reducer 2





Reducer 3


Combiner


Combiner


Combiner


## Slide 33

### Map & Reduce - Shuffle/Sort - Combiner


Map


Reduce


(“A”,


(“A”,


Reducer 1






Reducer 2





Reducer 3


Mapper 1:  “ABABAA”




Mapper 2:  “BBCCC”





Mapper 3  “CCCC” | 1) (“A”,	1)(“A”,
 	1)	1)	
(“B”, 1)  (“B”, 1)

(“B”, 1)  (“B”, 1)
(“C”, 1)
(“C”, 1)(“C”, 1)


 	
(“C”, 1)	(“C”, 1)
(“C”, 1) (“C”, 1) | Combiner





Combiner





Combiner | (“A”, 4)
 |  |  | (“B”, 2)
 |  |  | (“B”, 2)
 |  |  | (“C”, 3)
 |  |  | (“C”, 4)


(“A”, 4)


(“B”, 4)


(“C”, 7)


## Slide 34

### Map & Reduce - Caveats


What if we need data that is dependent on another key?
Solution: Chain MapReduce jobs together
Job 1: Calculate necessary subconditions per each key
Job 2: Determine final aggregate value

Chaining MapReduce Jobs
Output of nth job is the input to the (n+1)th job
Very useful in practice!
Try to minimize number of stages, because bandwidth overhead per stage is high
MapReduce tends to be naive in this area


## Slide 35

### A bit about Distributed Systems
MapReduce Overview
Structure & Execution Overview
MapReduce in Industry
Programming Hadoop MapReduce Jobs
Mappers and Reducers
Operating Model


Outline


## Slide 36

### Structure


Single master
A set of workers


Structure


## Slide 37

### Structure


M: Partition input data into M splits
Typically 16-64 MB per piece
R: Partition intermediate key space into R pieces
Using a partitioning function
E.g. (hash(key) mod R)
All specified by user


Execution


## Slide 38

### One master program
The rest are workers
Master assign map/reduce tasks to idle workers


Execution


## Slide 39

### Execution Overview


One master program
The rest are workers
Master assign map/reduce tasks to idle workers


Execution


## Slide 40

### Execution Overview


Workers perform Map task
Intermediate key/value pairs buffered in memory
Periodically write buffered pairs into local disk
Locations of buffered pairs → Master


Execution


## Slide 41

### Execution Overview


Workers perform Map task
Intermediate key/value pairs buffered in memory
Periodically write buffered pairs into local disk
Locations of buffered pairs → Master


Execution


## Slide 42

### Execution Overview


Master sends these locations to reduce workers
Reduce worker reads intermediate data


Sorts data by intermediate keys →
Intermediates with same key group together
If intermediate data too large:
External sort is used

Atomically renames temporary output file → Final output file
Final file system: Data from one execution of each reduce task
Therefore, R output files


Execution


## Slide 43

### Execution Overview


Master sends these locations to reduce workers
Reduce worker reads intermediate data


Sorts data by intermediate keys →
Intermediates with same key group together
If intermediate data too large:
External sort is used


Execution


## Slide 44

### Execution Overview


Master sends these locations to reduce workers
Reduce worker reads intermediate data


Sorts by intermediate keys →
Intermediates with same key group together
If intermediate data too large:
External sort is used


Execution


## Slide 45

### Execution Overview


Master sends these locations to reduce workers
Reduce worker reads intermediate data


Sorts by intermediate keys →
Intermediates with same key group together
If intermediate data too large:
External sort is used

Atomically renames temporary output file → Final output file
Final file system: Data from one execution of each reduce task
Therefore, R output files


Execution


## Slide 46

### Execution Overview


When all tasks completed
Master wakes up the user program


Execution


## Slide 47

### Execution Overview


Master data structures:
State of each map & reduce task:
Idle, in-progress, completed

Identity of worker machine
R intermediate file locations for each completed map task


Execution


## Slide 48

### Fault Tolerance


Why important:
Commodity machines: Failures are common
Hundreds and thousands of machines: Tolerate faults gracefully
Primary mechanism:	Re-execution


Worker Failure
Master Failure


Fault Tolerance


## Slide 49

### Fault Tolerance


Worker Failure
Master pings worker periodically
No response → Mark worker failed
Reset Map task completed by failed worker → idle state
Reset in-progress Map and Reduce tasks by failed worker → idle state
Idle tasks → eligible for rescheduling


Fault Tolerance


## Slide 50

### Fault Tolerance


Worker Failure
Master pings worker periodically
No response → Mark worker failed
Reset Map task completed by failed worker → idle state
Reset in-progress Map and Reduce tasks by failed worker → idle state
Idle tasks → eligible for rescheduling


Fault Tolerance


## Slide 51

### Fault Tolerance


Worker Failure
Master pings worker periodically
No response → Mark worker failed
Reset Map task completed by failed worker → idle state
Reset in-progress Map and Reduce tasks → idle state
Idle tasks → eligible for rescheduling


Fault Tolerance


## Slide 52

### Fault Tolerance


Worker Failure
Master pings worker periodically
No response → Mark worker failed
Reset Map task completed by failed worker → idle state
Reset in-progress Map and Reduce tasks → idle state
Idle tasks → eligible for rescheduling


Fault Tolerance


## Slide 53

### Fault Tolerance


Worker Failure
Completed Map tasks are re-executed on failure
Map outputs: on local disks of workers
Completed Reduce tasks do not need
Reduce outputs: in global file system

All the workers will be notified of a re-execution
Reduce worker read data from new location
Resilient to large-scale worker failures


Fault Tolerance


## Slide 54

### Fault Tolerance


Worker Failure
Completed Map tasks are re-executed on failure
Map outputs: on local disks of workers
Completed Reduce tasks do not need
Reduce outputs: in global file system

All the workers will be notified of a re-execution
Reduce worker read data from new location
Resilient to large-scale worker failures


Fault Tolerance


## Slide 55

### Fault Tolerance


Master Failure
Single master, rare failure
If master fails, aborts the MapReduce computation
Client can retry


Fault Tolerance


## Slide 56

### A bit about Distributed Systems
MapReduce Overview
MapReduce in Industry
Programming Hadoop MapReduce Jobs
Mappers and Reducers
Operating Model


Outline


## Slide 57

### Hadoop MapReduce


What it is: Specific implementation of a MapReduce system

What Hadoop MapReduce gives us:
A means of automatically distributing work across machines
Scheduling of jobs
Fault tolerance
Cluster monitoring and job tracking

How? An underlying resource manager (more on this later)


## Slide 58

### Hadoop MapReduce


How fast is it?

Benchmarks based on sorting large datasets (synthetic load)
Hadoop Record: 1.42TB / min
Record set in 2013 using a 2100 node cluster
Since 2014, Spark (and others) have been faster


## Slide 59

### Compelling Use Cases:
Batch Processing
Analyzing data “at rest” (i.e. daily/hourly jobs, not streaming data)
i.e. Log Processing, User data transformation / analysis, web scraping

Workloads that can be broken into a single (or few) distinct Map/Reduce  phases
Poor results on iterative workloads
Non-parallizable workloads => Many MR stages => High bandwitdh  overhead


MapReduce in Industry


## Slide 60

### Google
Released MapReduce whitepaper in 2004, detailing their use of MR to process  large datasets
Inspired Hadoop MapReduce (Open source implementation)
Google later developed more advanced systems like Google Dataflow to improve on MapReduce's limitations.

Twitter
Uses MapReduce to “process tweets, log files, and many other types of data”


MapReduce in Industry


## Slide 61

### Facebook
Maintains 2 Hadoop clusters with 1400 total machines and 10,000+ processing  cores, 15PB of storage

Spotify
Runs 20k+ Hadoop jobs daily
Uses Hadoop for “content generation, data aggregation, reporting, analysis”


MapReduce in Industry


## Slide 62

### LinkedIn
LinkedIn used Hadoop MapReduce for processing log data, building recommendation systems, and performing large-scale data analytics.
Netflix
Netflix used MapReduce to process viewing history, analyze user behavior, and generate personalized movie recommendations.
Amazon (AWS EMR)
Amazon provides a managed service called Amazon EMR (Elastic MapReduce) that allows companies to run MapReduce jobs for data processing, machine learning, and log analysis.


MapReduce in Industry


## Slide 63

### ●
○
○
●
●


## Slide 64

### ●
○
○
●
●


## Slide 65

### ⇒	⇒


●
○
○
●
○
○


## Slide 66

### ●
○
■

●
○
■

○
■
■


## Slide 67

### ●
○
○
○


num_posts.csv  Alice;10  Bob;15


num_friends.csv  Alice;105  Bob;85


Mapper


<Alice, (10, POSTS)>
<Bob, (15, POSTS)>
<Alice, (105, FRIENDS)>,
<Bob, (85, FRIENDS)>


Reducer


<User, (Posts, Friends)>
<Alice, (10, 105)>
<Bob, (15, 85)>


## Slide 68

### ●
○
■
■


## Slide 69

### ●
○
# Want to count number of unique values  def reduce(self, key, values):
# 'unique' will store all distinct values we see  unique = set()
for v in values:
if v not in unique:  unique.add(v)
# yield size of unique value set
yield (key, len(unique))


## Slide 70

### ●
○
■
■


○
■
■
■


## Slide 71

### ●
○
# Want to count number of unique values
def reduce(self, key, values):  prev, count = None, 0


# Here we assume that values is sorted
for v in values:
if v != prev:
count += 1
prev = v


yield (key, count)


## Slide 72

### ●
○


## Slide 73

### ●
○
■
■
■


Source: https://research.google.com/pubs/pub36249.html


## Slide 74

### ●
○
○
■
■
●
●

■


## Slide 75

### ●
○
○
○


## Slide 76

### ●
○
○
●
●


## Slide 77

### ●
●
●


## Slide 78

### ●
○
○
■
○
■
■


■


## Slide 79

### ●
○
○
■
■
●


○
■
○


## Slide 80

### ●
$ echo $DATA | ./mapper | sort -k1,1 | ./reducer > output
○
○


## Slide 81

### ●
○
○
○
○
■
■
■
■


## Slide 82

### ●
○
■
■


## Slide 83

### ●
○
○
●
●


## Slide 84

### <... new york weather …>
<... illinois weather …>
<... kansas news …>
<... illinois news …>


Mapper


<””, ... illinois weather …>
<””, … illinois news …>


Reducer


<””, ... illinois weather …>
<””, … illinois news …>


## Slide 85

### <illinois.edu HTML>
<stanford.edu HTML>
<amazon.com HTML>


Mapper


<twitter.com, illinois.edu>
<twitter.com, stanford.edu>
<twitter.com, amazon.com>
...


Reducer


<twitter.com, (illinois.edu,  stanford.edu, amazon.edu)>
...


## Slide 86

### Source: https://research.google.com/pubs/pub36249.html
