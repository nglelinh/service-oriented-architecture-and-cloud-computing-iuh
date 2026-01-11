# Lecture_4.1_Spark_Streaming


## Slide 1

### Spark
Streaming


## Slide 2

### Streaming Overview
Spark Streaming
Spark Streaming Programming
Final Project Announcement


Outline


## Slide 3

### Streaming


Why streaming?
Lots of data is not fixed, in practice
We have new data coming in all the time; it makes sense to respond in real time

What is streaming?
Clients push “events” in real time to an interface with our streaming system
The streaming system distributes input across the cluster (like batch processing)
Some resultant data is generated, which can be saved or streamed out of the  system


## Slide 4

### Batch Processing vs Stream Processing


 | Batch Processing | Stream Processing
Data Size | Large batches of data;  Most data in a data set | “An event”; Micro-batches of records
Nominal Latency | Minutes to Hours | In the order of milliseconds
Analysis | Complex Algorithm/Analytics | Simple functions, aggregation, rolling  metrics


## Slide 5

### Application of Streaming Data


Real-Time Machine Learning
e.g. Twitter’s “trending” topics, disaster monitoring, etc.

Tracking changes in the finance markets in real-time

Processing sensor data in large industrial settings
(Also scientific settings)


## Slide 6

### Stream Sources


File System

Internet of Things

Network Traffic

Embedded devices on a radio frequency


## Slide 7

### Apache Storm - A pure streaming system


Purpose-built for real-time stream computation

Three main concepts:
“Stream”: An unbounded sequence of tuples (like an infinite RDD)
“Spout”: A source of tuples
“Bolt”: A transformation operation on a stream


## Slide 8

### Apache Storm - A pure streaming system


Spouts, Bolts, and Streams define a “Topology”


Spout


Spout


Bolt


Bolt


Bolt


Bolt


Output


Stream


Stream


Stream


## Slide 9

### Apache Storm


Powerful language-agnostic tool/framework

Open-sourced by Twitter
Used to power Twitter’s real-time tweet analytics
Now Twitter uses Heron

Handles fault tolerance
Keeps track of which tuples have been fully processed
If a “Bolt” fails, unprocessed / partially processed tuples are reprocessed


## Slide 10

### Streaming Overview
Spark Streaming
Spark Streaming Programming
Final Project Announcement


Outline


## Slide 11

### Spark Streaming


## Slide 12

### Spark Looks a lot like Apache Storm...


Spout


Spout


Bolt


Bolt


Bolt


Bolt


Output


Stream


Stream


Stream


Apache Storm


## Slide 13

### Spark Looks a lot like Apache Storm...


Data  Source A


Transformation


Transformation


Transformation


Transformation


Output


RDD


RDD


RDD


Action


Data  Source B


Spark Streaming


## Slide 14

### How Spark Handles Streaming


Spark Core has a robust way for creating computation graphs on batch data

How can we extend this to streaming data?


## Slide 15

### How Spark Handles Streaming


## Slide 16

### How Spark Handles Streaming


Stream  Source


Transformation


Transformation


Transformation


Transformation


Output


DStream


DStream


DStream


Action


Stream  Source


## Slide 17

### How Spark Handles Streaming


Stream  Source


Transformation


Transformation


Transformation


Transformation


Output


DStream


DStream


DStream


Action


Stream  Source


RDD (t=1)


RDD (t=1)


RDD (t=1)


## Slide 18

### How Spark Handles Streaming


Stream  Source


Transformation


Transformation


Transformation


Transformation


Output


DStream


DStream


DStream


Action


Stream  Source


RDD (t=2)


RDD (t=2)


RDD (t=2)


RDD (t=1)


RDD (t=1)


RDD (t=1)


## Slide 19

### How Spark Handles Streaming


Stream  Source


Transformation


Transformation


Transformation


Transformation


Output


DStream


DStream


DStream


Action


Stream  Source


RDD (t=3)


RDD (t=3)


RDD (t=3)


RDD (t=2)


RDD (t=2)


RDD (t=2)


RDD (t=1)


## Slide 20

### DStream
“Discretized Stream” which represents an infinite stream of data
In actuality, it’s a (endless) sequence of RDDs

Spark Streaming Context
Similar to the Spark Context, except it handles DStreams
Has a user-defined batch interval
Defines the window size for RDDs

Job Processing
Executed in multiples of the batch interval


How Spark Handles Streaming


## Slide 21

### Built-in Data Sources
File Stream - Load new files in a given directory
Socket Stream - Listen on a TCP connection for new data

Additional Supported Stream Sources


Kafka
Flume
Kinesis


(Apache)
(Apache, in the Hadoop ecosystem)  (AWS)


Spark Stream Sources


## Slide 22

### Streaming Overview
Spark Streaming
Spark Streaming Programming
Final Project Announcement


Outline


## Slide 23

### Transformations
Operations on DStreams (mostly) identical to RDDs
Functions: Map, Filter, Join, etc.

Windowed Operations
Applied transformation on a time-based window of data
Functions: CountByWindow, ReduceByWindow, etc.


Spark Streaming Programming


## Slide 24

### Stateful Operations
So far, we’re limited to “seeing” data within our current window
What if we need arbitrary state?
Functions: “UpdateStateByKey”
Uses an RDD to keep a persistent state

The “Transform” Function
Allows you get access to the underlying DStream RDD
Used for “combining” DStream data with arbitrary RDDs
i.e. Join streamed data on precomputed data


Spark Streaming Programming


## Slide 25

### Spark Streaming Programming Example


from pyspark import SparkContext
from pyspark.streaming import StreamingContext


# local streaming context with two threads
sc = SparkContext("local[2]",	"NetworkWordCount")
ssc = StreamingContext(sc, 1)	# batch interval of 1 second


# DStream that pulls from localhost:8888
lines = ssc.socketTextStream("localhost", 8888)


## Slide 26

### Spark Streaming Programming Example


# We can use the lines DStream almost like a normal RDD  filtered = lines.filter( lambda l: 'cloud' in l).flatMap( lambda  key_on_word = filtered.map( lambda w: (w, 1))


x: x.split()


# Count in window lengths of 30 seconds, evaluated every 10 seconds  windowed = key_on_word.countByValueAndWindow(30, 10)

# We can call an action on DStreams like RDDs  windowed.pprint()

# Start stream processing  ssc.start()
