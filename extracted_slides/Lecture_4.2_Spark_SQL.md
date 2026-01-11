# Lecture_4.2_Spark_SQL


## Slide 1

### Spark SQL


## Slide 2

### Traditional Databases
SQL
Optimizations
Spark SQL


Outline


## Slide 3

### Traditional Databases
SQL
Optimizations
Spark SQL


Outline


## Slide 4

### RDBMS


Relational Database Management Systems
Systems that deal with relational data (data that points to other data)
A database management system manages how the data is stored and retrieved.  Usually the data is modified with SQL


E.g: MySQL, PostgreSQL, OracleDB, etc


## Slide 5

### http://www.mytecbits.com/wp-content/uploads/RDBMS.png


## Slide 6

### Other Features


RDBMS handles data backups, logically storing data, distributing data to leader  followers, permissions, data integrity, handling and load balancing queries, and  optimization.
RDBMSs do all of this “under the hood” (mostly)


https://docs.oracle.com/cd/B28359_01/network.111/b28316/img/netag089.gif


## Slide 7

### RDBMS Types of Data


RDBMSs like simple data: INTEGERS, STRINGS, etc
They don’t like handling JSON, HASHMAP, LISTS
Complex data types are more difficult for the SQL engine to optimize  against
If you think you need advanced data type functionality:
Seriously rethink your application design
If you are absolutely sure that you need it:
You should probably use another application server.


## Slide 8

### Traditional Databases
SQL
Optimizations
Spark SQL


Outline


## Slide 9

### Structured Query Language


Most of you have had some interaction with SQL
SQL was made for both programmers and for accountants who were used to  spreadsheets
We can imagine taking data from spreadsheets, join from different sheets etc


https://udemy-images.udemy.com/course/750x422/743174_8046_5.jpg


## Slide 10

### Basic Commands - Data Definition Language (DDL)


DDL lets you create, destroy, alter, and modify constraints on data
You can think of them as operations that set up where data will go


CREATE TABLE (
id INTEGER, name VARCHAR(255), location VARCHAR(255)
);
ALTER TABLE ADD status INTEGER;
ALTER TABLE ADD blah INTEGER NOT NULL;
DROP TABLE;


## Slide 11

### Data Modification Language - DML


This adds, deletes, selects, and updates data (basic CRUD operations)
This lets you put data into the database tables


INSERT INTO table (col1, col2, ..) VALUES (v1, v2, ..), ..
DELETE FROM table where col1 = …
UPDATE table SET col1=’asdf’ WHERE col2=’asd’
SELECT * FROM table


## Slide 12

### Data Modification Language Extensions


The data modification language also lets you do more powerful things when  retrieving data
We can have data GROUP BY a certain column(s)
Have data ORDER BY some column(s)
We can JOIN multiple spreadsheets based on a column

We can have SQL calculate functions or aggregations on the fly
Usually RDBMSs are optimized for read-heavy workloads


## Slide 13

### https://dsin.files.wordpress.com/2013/03/sqljoins_cheatsheet.png


## Slide 14

### SQL Prepared Statements


Actual interactions with the database.
INSERT INTO table VALUES (`+userid+`);

What if userid = “1; SELECT * FROM table WHERE col1 NOT IN (“?
INSERT INTO table VALUES (1); SELECT * FROM table  WHERE col1 NOT IN ();
This will give us back all the results from the database!


## Slide 15

### SQL Prepared Statements


To avoid this, we have prepared statements
INSERT INTO table VALUES (?) and, send userid separately
This avoids the injection problem but doesn’t let SQL server optimize database  queries


## Slide 16

### Traditional Databases
SQL
Optimizations
Spark SQL


Outline


## Slide 17

### SQL Turing Completeness


Every SQL statement (in ANSI SQL) will terminate
The Non-Turing Completeness of SQL let’s us optimize many portions of  queries


## Slide 18

### User tips for optimizing SQL queries


Don’t use `SELECT *` statements, you usually are selecting more rows than  need be


If you have multiple levels of joins then you may want to consider staging your  data into an intermediate table in order to reduce communication overhead


Add indices! Indices can slow updates but drastically speed up complex  queries if the indices are on the appropriate columns


## Slide 19

### SQL Optimizer: Prediction


Consider a query like `select col1 from table where col1=1 AND col2=2;`
Your server has the choice of filtering by col2 and then col1 or by col1 then  col2.
If the server knows that there are a lot of NULL values in col2 which would  reduce the number of rows in consideration a lot, it will filter based on col2 first  and then filter on col1 because the complexity will be NUM_ROWS *  SMALL_NUMBER


## Slide 20

### SQL Optimizer: Lazy Joins


A join is when you combine two tables on a column


c1 | c2
1 | 2
2 | 4


c3 | c4
1 | 1
2 | 1
3 | 2


## Slide 21

### Example Join


SELECT * FROM t1 JOIN t2 USING (c1, c4);


c1 | c2 | c3 | c4
1 | 2 | 1 | 1
1 | 2 | 2 | 1
2 | 4 | 3 | 2


## Slide 22

### Lazy Join


SQL may filter the data before joining, may group by before joining if you know  that one of the columns is in one of the table
This is very ad-hoc prediction because SQL usually doesn’t keep track of super  in depth statistics
As a SQL server runs longer, then it gets better at this prediction
The main reason that it can’t keep track of all of this information is due to concurrency  bottlenecks so it makes static analyses instead


## Slide 23

### Traditional Databases
SQL
Optimizations
Spark SQL


Outline


## Slide 24

### Spark SQL


Distributed in-memory computation on massive scale (Just like Spark!)
Can use all data sources that Spark supports natively:
Can import data from RDDs
JSON/CSV files can be loaded with inferred schema
Parquet files - Column-based storage format
Supported by many Apache systems (big surprise!)
Hive Table import
A popular data warehousing platform by Apache


## Slide 25

### Spark SQL


SQL using Spark as a “Database”
Spark SQL is best optimized for retrieving data
Don’t UPDATE, INSERT, or DELETE
Optimization handled by a newer optimization engine, Catalyst
Creates physical execution plan and compiles directly to JVM bytecode
Can function as a compatibility layer for firms that use RDBMS systems


## Slide 26

### Spark DataFrames


Dataset organized into named columns
Similar to structure as Dataframes in Python (i.e. Pandas) or R
Lazily evaluated like normal RDDs
Tends to be more performant than raw RDD operations


https://databricks.com/blog/2015/02/17/introducing-dataframes-in-spark-for-large-scale-data-science.html


## Slide 27

### Pandas DataFrame


Does in-memory computing, but:
Not scalable by itself.
Not fault tolerant.

import pandas as	pd
df = pd.read_csv("/path/to/data.json")  df


first_name last_name


age	preTestScore  4	25,000
24	94,000
57


postTestScore


0
1
2


Jason Miller  Molly Jacobson


Tina	.	36


42
52
31


3 | Jake | Milner | 24 | . | 62
4 | Amy | Cooze 73 | . | 70 | 


## Slide 28

### Spark DataFrames


When to prefer RDDs over DataFrames:
Need low-level access to data
Data is mostly unstructured or schemaless

When to prefer DataFrames over RDDs:
Operations on structured data
If higher-level abstractions are useful (i.e. joins, aggregation, etc.)
High-performance is desired, and workload fits within DataFrame APIs
Catalyst optimization makes DataFrames more performant on  average


## Slide 29

### Spark DataSets


Strongly-typed DataFrames
Only accessible in Spark2+ using Scala
Operations on DataFrames are all statically typed, so you catch type errors at  compile-time


https://databricks.com/blog/2016/07/14/a-tale-of-three-apache-spark-apis-rdds-dataframes-and-datasets.html


## Slide 30

### Data Ingest (RDD)


from pyspark.sql import SQLContext  sqlContext = SQLContext(sc)
users_rdd = sc.parallelize([[1, 'Alice', 10], [2, 'Bob', 8]])  users = sqlContext.createDataFrame(
users_rdd,
['id',	'name',	'num_posts'])  users.printSchema()
#root
# |-- id: long (nullable = true)
# |-- name: string (nullable = true)
# |-- num_posts: long (nullable = true)


## Slide 31

### Data Ingest (JSON)


from pyspark.sql import SQLContext  sqlContext = SQLContext(sc)
users = sqlContext.read.json("/path/to/users.json")  users.printSchema()


# | root | 
# | |-- | id: long (nullable = true)
# | |-- | name: string (nullable = true)
# | |-- | num_posts: long (nullable = true)


## Slide 32

### SQL API


# Register users DataFrame as a table called "users"  users.createOrReplaceTempView( 'users')

# Query the table  sqlContext.sql(
'SELECT * FROM users WHERE name="Bob"'
).collect()


# [Row(id=2, name='Bob', num_posts=8)]


## Slide 33

### DataFrame API


# Same query can be done with DataFrame API

users.filter(users.name=='Bob').collect()  # [Row(id=2, name='Bob', num_posts=8)]

users.filter(users.name=='Eve').select('num_posts').collect()  # [Row(num_posts=10)]
