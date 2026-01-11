# Lecture_5.1_NoSQL_Key_Value


## Slide 1

### NoSQL and Key/Value Stores


## Slide 2

### SQL databases


## Slide 3

### A little bit of history


Most databases became SQL-like in the 1980s
Term first used in 1998 - NoSQL stands for “Not Only SQL”
In 2006 Google published their BigTable paper
It was not SQL
It was designed to scale to petabytes of data (1000s of gigabytes) on thousands of nodes
Solved scaling by relaxing availability

In 2007 Amazon published their Dynamo paper
Again not SQL; Similarly solves the problem of scaling
Solved scaling through relaxing consistency

By 2009 there were tons of systems like these
Now when you have hundreds of nodes, NoSQL is the normal solution


## Slide 4

### Reached a noticeable awareness in April 2009


## Slide 5

### Why NoSQL ?


Databases which may not be relational and can scale to tons and tons of servers
Building distributed RDBMSs is a very complex (expensive) task
Sacrifice SQL compatibility to get higher read/write/storage rates
Schema free - Handling Unstructured Data


## Slide 6

### SQL vs NoSQL


SQL systems are typically good at consistency
If data is written to a row, all reads will get that write
This can slow down transactions
The vast majority of databases (not only SQL) are ACID:
Atomic
Consistent
Isolated
Durable

ACID is analogous to the properties of a global variable in a single threaded program


## Slide 7

### NoSQL Database Types


Key Value Store (Redis, Riak, LocalStorage, Amazon DynamoDB, Datastore on GAE)
Document Oriented (MongoDB, CourchDB, RethinkDB)
Columnar Storage (HBase, Apache Cassandra, ClickHouse, Pinot, and Druid)
Multivalue databases (InfinityDB)
Graph (AllegroGraph, Neo4J)


## Slide 8

### Key Value Store


These store key value pairs really really well
Can be used as a distributed cache
Key-value databases are generally useful for storing session information, user profiles, preferences, shopping cart data
Some document stores are key value stores under the hood
{
Key1: val1, Key2: val2
}


## Slide 9

### Document Oriented Databases


They store complex structures like:
JSON
XML
YAML

These work really well when most queries are for one item instead of aggregations

Typically provide their own unique query languages

These extend the idea of key value stores to more complex types


## Slide 11

### Document Vs Key Value Databases


Both address objects with a key:
Document DBs cluster documents within collections
Key value stores mainly have only one collection
Key value stores are faster (Smaller values and less structure)
Document DBs support more extensive query languages in general
If you do not need complex objects, use a key value store


## Slide 12

### Columnar/Column based Databases


Columns are stored together instead of rows
A row is can be split amongst many machines
Makes aggregations really fast since a single column normally resides on one machine => This makes a big difference for analytical data processing
Usually does not support joins (or joins are very slow)


## Slide 13

### Row based databases


Columnar based databases


## Slide 14

### Relaxing Constraints


All of the above types can be implemented using a normal SQL database a backend

They can also be implemented as ACID (Atomicity, Consistency, Isolation, and Durability) databases

What if we specified you did not need strong consistency?


## Slide 15

### Eventual Consistency


Making everything consistent immediately means clients need to queue
Say you have 3 clients, one writing and the other two reading


Write X


Read X


Read X


## Slide 16

### Eventual Consistency


Making everything consistent immediately means clients need to queue
Say you have 3 clients, one writing and the other two reading
The true ordering is


T=0 | T=1 | T=2
Read X | Write X | Read X


## Slide 17

### Eventual Consistency


But you could receive this order because of network delays


T=0 | T=1 | T=2
Read X | Read X | Write X


## Slide 18

### Eventual Consistency


Or if you have two servers, one could receive the true ordering and one the out of order ordering
Server 1 sees


Server 2 Sees


T=0 | T=1 | T=2
Read X | Read X | Write X


T=0 | T=1 | T=2
Read X | Write X | Read X


## Slide 19

### Eventual Consistency


But sometimes we can afford old values being read for a little while. This means we can read and write at the same time.
Server 1 sees


Server 2 Sees


T=0 | T=1 | T=2
Read X | Read X | Write X


T=0 | T=1 | T=2
Read X | Write X | Read X


## Slide 20

### CAP Theorem


Consistency
All reads receive the most recent write or error
Availability
Every read/write receives a non error
Partition Tolerance
Everything keeps working if the network starts dropping messages

Pick 2


## Slide 21

### CAP Theorem


Consistency
All reads receive the most recent write or error
Availability
Every read/write recieves a non error
Partition Tolerance
Everything keeps working if the network starts dropping messages

Pick 2
Each of these have a non strict version
But you cannot guarantee all 3 in all scenarios


## Slide 22

### From Ofirm


## Slide 23

### CA Systems


Consistency and availability
They will always respond with the latest write

Most SQL databases are CA systems.

SQL Systems
MySQL
MSSQL
SQLite
PostgreSQL


## Slide 24

### CP Systems


Consistency and Partition Tolerance
Will give you the latest write or give you an error if not possible
Can survive half the network going down
HBase, BigTable, MongoDB


## Slide 25

### Is a CP system
Used in HDFS
Linear and modular scalability.
Strictly consistent reads and writes.
Automatic and configurable sharding of table


Everything is still a table
Can return an error since it is CP


## Slide 26

### Is a CP system
Extremely easy to set up
The defaults are insecure
Document Oriented DB
Only stores JSON objects
No longer a simple table
Is a key value store


## Slide 27

### Is Retains some friendly properties of SQL. (query, index)
Queries are javascript expressions
Run arbitrary javascript functions server-side
Master-slave replication
Sharding built-in
Concurrency: Update in Place
Has geospatial indexing
Stores documents as BSON (Binary JSON)
Provides CP (remember CAP theorem)
Enterprise class support


## Slide 31

### CP System
Is a key value store
Lets you write data structures into memory and share them
Very fast: Often used as a caching layer


## Slide 32

### CP System
Resembles a key-value store
Allows indexing, but this involves data-replication
Very fast reads
Scales well


## Slide 33

### CP System
Global SQL Database


Google Spanner


## Slide 34

### AP Systems


Available and partition tolerant
Never returns an error even when half the network is dead


## Slide 35

### From Ofirm


## Slide 36

### AP system
Column-Oriented
Super high availability and super high throughput
Used by Reddit, Facebook and others


## Slide 37

### AP system
Key value store
Can be faster than MongoDB but sacrifices consistency


## Slide 38

### When to use SQL vs NoSQL


By default use an SQL database

Use NoSQL when you need more than one server AND you have a super high write rate
NoSQL happens when you can sacrifice CA and need some other pair from CAP


## Slide 39

### Usage of NoSQL DBs at scale


Facebook
RocksDB - Embeddable persistent key-value store.
Cassandra - Column-based database
Memcached - An in-memory key-value store
Pokemon GO
Google Cloud Datastore’s NoSQL database.
Clash of Clans
DynomoDB Amazon Web Services(AWS)
