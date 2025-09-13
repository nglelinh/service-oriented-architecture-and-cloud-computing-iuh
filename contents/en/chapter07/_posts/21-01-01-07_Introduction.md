---
layout: post
title: 07 Spark SQL - Structured Data Processing
chapter: '07'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter07
---

Spark SQL is Apache Spark's module for working with structured data. It provides a programming interface for working with structured and semi-structured data using SQL queries, DataFrames, and Datasets, combining the power of Spark's distributed computing with the familiarity of SQL.

## Introduction to Spark SQL

### What is Spark SQL?
Spark SQL is a Spark module for structured data processing that provides:
- **SQL Interface**: Write SQL queries against Spark data
- **DataFrame API**: Programmatic interface inspired by data frames in R and Python pandas
- **Dataset API**: Type-safe interface for Scala and Java
- **Catalyst Optimizer**: Advanced query optimization engine
- **Unified Data Access**: Read from various data sources (Parquet, JSON, Hive, JDBC, etc.)

```
Spark SQL Architecture:
┌─────────────────────────────────────────────────────────────┐
│                    SQL Interface                             │
├─────────────────────────────────────────────────────────────┤
│              DataFrame/Dataset API                           │
├─────────────────────────────────────────────────────────────┤
│                 Catalyst Optimizer                           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │   Parser    │ │  Analyzer   │ │     Optimizer       │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                 Tungsten Execution                           │
├─────────────────────────────────────────────────────────────┤
│                    Spark Core                                │
└─────────────────────────────────────────────────────────────┘
```

### Evolution from RDDs to DataFrames
```python
# Evolution of Spark APIs
from pyspark import SparkContext, SparkConf
from pyspark.sql import SparkSession, Row
from pyspark.sql.functions import *

class SparkAPIEvolution:
    
    def __init__(self):
        self.spark = SparkSession.builder \
            .appName("SparkSQLDemo") \
            .config("spark.sql.adaptive.enabled", "true") \
            .getOrCreate()
        
        self.sc = self.spark.sparkContext
    
    def rdd_approach(self):
        """Traditional RDD approach - more verbose"""
        
        # Sample data
        data = [
            ("Alice", 25, "Engineering"),
            ("Bob", 30, "Sales"),
            ("Charlie", 35, "Engineering"),
            ("Diana", 28, "Marketing")
        ]
        
        # Create RDD
        rdd = self.sc.parallelize(data)
        
        # Filter and transform using RDD operations
        engineering_employees = rdd.filter(lambda x: x[2] == "Engineering")
        names_and_ages = engineering_employees.map(lambda x: (x[0], x[1]))
        
        # Collect results
        result = names_and_ages.collect()
        print("RDD Result:", result)
        
        return result
    
    def dataframe_approach(self):
        """Modern DataFrame approach - more readable and optimized"""
        
        # Sample data
        data = [
            ("Alice", 25, "Engineering"),
            ("Bob", 30, "Sales"),
            ("Charlie", 35, "Engineering"),
            ("Diana", 28, "Marketing")
        ]
        
        # Create DataFrame with schema
        columns = ["name", "age", "department"]
        df = self.spark.createDataFrame(data, columns)
        
        # Filter and select using DataFrame operations
        result_df = df.filter(col("department") == "Engineering") \
                     .select("name", "age")
        
        # Show results
        result_df.show()
        
        # Or use SQL
        df.createOrReplaceTempView("employees")
        sql_result = self.spark.sql("""
            SELECT name, age 
            FROM employees 
            WHERE department = 'Engineering'
        """)
        
        sql_result.show()
        
        return result_df
    
    def performance_comparison(self):
        """Compare performance between RDD and DataFrame"""
        
        # Create larger dataset
        large_data = [(f"Person_{i}", i % 100, f"Dept_{i % 10}") 
                     for i in range(1000000)]
        
        # RDD approach
        import time
        start_time = time.time()
        
        rdd = self.sc.parallelize(large_data)
        rdd_result = rdd.filter(lambda x: x[1] > 50) \
                       .map(lambda x: (x[0], x[1])) \
                       .count()
        
        rdd_time = time.time() - start_time
        
        # DataFrame approach
        start_time = time.time()
        
        df = self.spark.createDataFrame(large_data, ["name", "age", "dept"])
        df_result = df.filter(col("age") > 50) \
                     .select("name", "age") \
                     .count()
        
        df_time = time.time() - start_time
        
        print(f"RDD Time: {rdd_time:.2f}s, DataFrame Time: {df_time:.2f}s")
        print(f"DataFrame is {rdd_time/df_time:.2f}x faster")
        
        return rdd_time, df_time
```

## DataFrames and Datasets

### Creating DataFrames
```scala
// Scala DataFrame creation examples
import org.apache.spark.sql.{SparkSession, DataFrame, Row}
import org.apache.spark.sql.types._
import org.apache.spark.sql.functions._

object DataFrameCreation {
  
  val spark = SparkSession.builder()
    .appName("DataFrameExamples")
    .master("local[*]")
    .getOrCreate()
  
  import spark.implicits._
  
  def createFromCollections(): DataFrame = {
    // From Scala collections
    val data = Seq(
      ("Alice", 25, "Engineering", 75000.0),
      ("Bob", 30, "Sales", 65000.0),
      ("Charlie", 35, "Engineering", 85000.0),
      ("Diana", 28, "Marketing", 70000.0)
    )
    
    val df = data.toDF("name", "age", "department", "salary")
    df
  }
  
  def createWithExplicitSchema(): DataFrame = {
    // Define explicit schema
    val schema = StructType(Array(
      StructField("name", StringType, nullable = false),
      StructField("age", IntegerType, nullable = false),
      StructField("department", StringType, nullable = false),
      StructField("salary", DoubleType, nullable = false)
    ))
    
    val rowData = Seq(
      Row("Alice", 25, "Engineering", 75000.0),
      Row("Bob", 30, "Sales", 65000.0),
      Row("Charlie", 35, "Engineering", 85000.0)
    )
    
    val rdd = spark.sparkContext.parallelize(rowData)
    val df = spark.createDataFrame(rdd, schema)
    df
  }
  
  def createFromFiles(): DataFrame = {
    // From JSON files
    val jsonDF = spark.read
      .option("multiline", "true")
      .json("path/to/employees.json")
    
    // From CSV files
    val csvDF = spark.read
      .option("header", "true")
      .option("inferSchema", "true")
      .csv("path/to/employees.csv")
    
    // From Parquet files
    val parquetDF = spark.read.parquet("path/to/employees.parquet")
    
    jsonDF
  }
  
  def createFromDatabases(): DataFrame = {
    // From JDBC sources
    val jdbcDF = spark.read
      .format("jdbc")
      .option("url", "jdbc:postgresql://localhost:5432/company")
      .option("dbtable", "employees")
      .option("user", "admin")
      .option("password", "password")
      .option("driver", "org.postgresql.Driver")
      .load()
    
    jdbcDF
  }
}
```

### DataFrame Operations
```python
# Comprehensive DataFrame operations
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from pyspark.sql.types import *

class DataFrameOperations:
    
    def __init__(self):
        self.spark = SparkSession.builder \
            .appName("DataFrameOperations") \
            .getOrCreate()
    
    def create_sample_data(self):
        """Create sample datasets for demonstrations"""
        
        # Employee data
        employees_data = [
            (1, "Alice", 25, "Engineering", 75000.0, "2020-01-15"),
            (2, "Bob", 30, "Sales", 65000.0, "2019-03-20"),
            (3, "Charlie", 35, "Engineering", 85000.0, "2018-07-10"),
            (4, "Diana", 28, "Marketing", 70000.0, "2021-02-01"),
            (5, "Eve", 32, "Sales", 68000.0, "2020-11-15")
        ]
        
        employees_schema = StructType([
            StructField("id", IntegerType(), False),
            StructField("name", StringType(), False),
            StructField("age", IntegerType(), False),
            StructField("department", StringType(), False),
            StructField("salary", DoubleType(), False),
            StructField("hire_date", StringType(), False)
        ])
        
        self.employees_df = self.spark.createDataFrame(employees_data, employees_schema)
        
        # Department data
        departments_data = [
            ("Engineering", "Tech", "Alice Johnson"),
            ("Sales", "Business", "Bob Smith"),
            ("Marketing", "Business", "Carol Davis"),
            ("HR", "Support", "David Wilson")
        ]
        
        self.departments_df = self.spark.createDataFrame(
            departments_data, 
            ["department", "category", "manager"]
        )
        
        return self.employees_df, self.departments_df
    
    def basic_operations(self):
        """Basic DataFrame operations"""
        
        df = self.employees_df
        
        # Select columns
        names_and_salaries = df.select("name", "salary")
        
        # Filter rows
        high_earners = df.filter(col("salary") > 70000)
        
        # Add new columns
        df_with_bonus = df.withColumn("bonus", col("salary") * 0.1)
        
        # Rename columns
        df_renamed = df.withColumnRenamed("name", "employee_name")
        
        # Drop columns
        df_minimal = df.drop("hire_date")
        
        # Sort data
        df_sorted = df.orderBy(col("salary").desc())
        
        # Group by and aggregate
        dept_stats = df.groupBy("department") \
                      .agg(
                          avg("salary").alias("avg_salary"),
                          count("*").alias("employee_count"),
                          max("age").alias("max_age")
                      )
        
        return {
            "names_and_salaries": names_and_salaries,
            "high_earners": high_earners,
            "with_bonus": df_with_bonus,
            "dept_stats": dept_stats
        }
    
    def advanced_transformations(self):
        """Advanced DataFrame transformations"""
        
        df = self.employees_df
        
        # Window functions
        from pyspark.sql.window import Window
        
        # Rank employees by salary within department
        window_spec = Window.partitionBy("department").orderBy(col("salary").desc())
        
        df_with_rank = df.withColumn("salary_rank", 
                                   row_number().over(window_spec))
        
        # Calculate running totals
        running_total_spec = Window.partitionBy("department") \
                                  .orderBy("salary") \
                                  .rowsBetween(Window.unboundedPreceding, Window.currentRow)
        
        df_with_running_total = df.withColumn("running_salary_total",
                                            sum("salary").over(running_total_spec))
        
        # Pivot operations
        pivot_df = df.groupBy("department") \
                    .pivot("age") \
                    .agg(avg("salary"))
        
        # Complex expressions
        df_with_categories = df.withColumn(
            "salary_category",
            when(col("salary") > 80000, "High")
            .when(col("salary") > 65000, "Medium")
            .otherwise("Low")
        )
        
        # Date operations
        df_with_dates = df.withColumn("hire_date_parsed", 
                                    to_date(col("hire_date"), "yyyy-MM-dd")) \
                         .withColumn("years_employed", 
                                   datediff(current_date(), col("hire_date_parsed")) / 365)
        
        return {
            "with_rank": df_with_rank,
            "with_running_total": df_with_running_total,
            "pivot": pivot_df,
            "with_categories": df_with_categories,
            "with_dates": df_with_dates
        }
    
    def join_operations(self):
        """Various join operations"""
        
        emp_df = self.employees_df
        dept_df = self.departments_df
        
        # Inner join
        inner_joined = emp_df.join(dept_df, "department", "inner")
        
        # Left outer join
        left_joined = emp_df.join(dept_df, "department", "left_outer")
        
        # Right outer join
        right_joined = emp_df.join(dept_df, "department", "right_outer")
        
        # Full outer join
        full_joined = emp_df.join(dept_df, "department", "full_outer")
        
        # Join with complex conditions
        complex_join = emp_df.alias("e").join(
            dept_df.alias("d"),
            (col("e.department") == col("d.department")) & 
            (col("e.salary") > 70000),
            "inner"
        )
        
        # Self join example
        emp_pairs = emp_df.alias("e1").join(
            emp_df.alias("e2"),
            (col("e1.department") == col("e2.department")) & 
            (col("e1.id") < col("e2.id")),
            "inner"
        ).select(
            col("e1.name").alias("employee1"),
            col("e2.name").alias("employee2"),
            col("e1.department")
        )
        
        return {
            "inner": inner_joined,
            "left": left_joined,
            "complex": complex_join,
            "self_join": emp_pairs
        }
```

## SQL Interface and Catalyst Optimizer

### SQL Queries in Spark
```java
// Java SQL interface examples
import org.apache.spark.sql.*;
import org.apache.spark.sql.types.*;
import static org.apache.spark.sql.functions.*;

public class SparkSQLInterface {
    
    private SparkSession spark;
    
    public SparkSQLInterface() {
        this.spark = SparkSession.builder()
            .appName("SparkSQLInterface")
            .master("local[*]")
            .getOrCreate();
    }
    
    public void basicSQLOperations() {
        // Create sample DataFrame
        Dataset<Row> employees = createEmployeeDataFrame();
        
        // Register as temporary view
        employees.createOrReplaceTempView("employees");
        
        // Basic SELECT queries
        Dataset<Row> allEmployees = spark.sql("SELECT * FROM employees");
        allEmployees.show();
        
        // Filtering and sorting
        Dataset<Row> highEarners = spark.sql("""
            SELECT name, department, salary
            FROM employees 
            WHERE salary > 70000 
            ORDER BY salary DESC
        """);
        
        // Aggregations
        Dataset<Row> deptStats = spark.sql("""
            SELECT 
                department,
                COUNT(*) as employee_count,
                AVG(salary) as avg_salary,
                MAX(salary) as max_salary,
                MIN(salary) as min_salary
            FROM employees 
            GROUP BY department
            ORDER BY avg_salary DESC
        """);
        
        deptStats.show();
    }
    
    public void advancedSQLQueries() {
        Dataset<Row> employees = createEmployeeDataFrame();
        employees.createOrReplaceTempView("employees");
        
        // Window functions
        Dataset<Row> rankedEmployees = spark.sql("""
            SELECT 
                name,
                department,
                salary,
                ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as rank,
                DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dense_rank,
                LAG(salary) OVER (PARTITION BY department ORDER BY salary) as prev_salary
            FROM employees
        """);
        
        // Common Table Expressions (CTEs)
        Dataset<Row> cteQuery = spark.sql("""
            WITH dept_averages AS (
                SELECT department, AVG(salary) as avg_dept_salary
                FROM employees
                GROUP BY department
            ),
            employee_comparison AS (
                SELECT 
                    e.name,
                    e.department,
                    e.salary,
                    d.avg_dept_salary,
                    e.salary - d.avg_dept_salary as salary_diff
                FROM employees e
                JOIN dept_averages d ON e.department = d.department
            )
            SELECT * FROM employee_comparison
            WHERE salary_diff > 0
            ORDER BY salary_diff DESC
        """);
        
        // Subqueries
        Dataset<Row> subqueryResult = spark.sql("""
            SELECT name, department, salary
            FROM employees e1
            WHERE salary > (
                SELECT AVG(salary) 
                FROM employees e2 
                WHERE e2.department = e1.department
            )
        """);
        
        rankedEmployees.show();
        cteQuery.show();
        subqueryResult.show();
    }
    
    public void complexAnalytics() {
        Dataset<Row> employees = createEmployeeDataFrame();
        employees.createOrReplaceTempView("employees");
        
        // Percentile calculations
        Dataset<Row> percentiles = spark.sql("""
            SELECT 
                department,
                PERCENTILE_APPROX(salary, 0.25) as q1_salary,
                PERCENTILE_APPROX(salary, 0.5) as median_salary,
                PERCENTILE_APPROX(salary, 0.75) as q3_salary,
                PERCENTILE_APPROX(salary, 0.9) as p90_salary
            FROM employees
            GROUP BY department
        """);
        
        // Pivot operations
        Dataset<Row> pivotResult = spark.sql("""
            SELECT * FROM (
                SELECT department, age, salary
                FROM employees
            ) PIVOT (
                AVG(salary) as avg_salary
                FOR age IN (25, 28, 30, 32, 35)
            )
        """);
        
        // Array and map operations
        Dataset<Row> arrayOperations = spark.sql("""
            SELECT 
                department,
                COLLECT_LIST(name) as employee_names,
                COLLECT_SET(age) as unique_ages,
                MAP_FROM_ARRAYS(
                    COLLECT_LIST(name), 
                    COLLECT_LIST(salary)
                ) as name_salary_map
            FROM employees
            GROUP BY department
        """);
        
        percentiles.show();
        pivotResult.show();
        arrayOperations.show();
    }
    
    private Dataset<Row> createEmployeeDataFrame() {
        // Implementation to create sample DataFrame
        return spark.emptyDataFrame(); // Placeholder
    }
}
```

### Catalyst Optimizer Deep Dive
```python
# Understanding Catalyst Optimizer
from pyspark.sql import SparkSession
from pyspark.sql.functions import *

class CatalystOptimizerDemo:
    
    def __init__(self):
        self.spark = SparkSession.builder \
            .appName("CatalystDemo") \
            .config("spark.sql.adaptive.enabled", "true") \
            .config("spark.sql.adaptive.coalescePartitions.enabled", "true") \
            .getOrCreate()
    
    def demonstrate_optimization_phases(self):
        """Show different phases of Catalyst optimization"""
        
        # Create sample data
        df = self.spark.range(1000000).toDF("id") \
                      .withColumn("value", col("id") * 2) \
                      .withColumn("category", col("id") % 10)
        
        # Complex query that will be optimized
        result = df.filter(col("id") > 100) \
                  .filter(col("value") < 1000000) \
                  .select("id", "category") \
                  .groupBy("category") \
                  .count() \
                  .orderBy("count")
        
        # Show the logical plan
        print("=== LOGICAL PLAN ===")
        result.explain(True)
        
        # Show the optimized physical plan
        print("\n=== OPTIMIZED PHYSICAL PLAN ===")
        result.explain("formatted")
        
        return result
    
    def predicate_pushdown_example(self):
        """Demonstrate predicate pushdown optimization"""
        
        # Create DataFrames
        large_df = self.spark.range(1000000).toDF("id") \
                            .withColumn("dept_id", col("id") % 100)
        
        small_df = self.spark.range(100).toDF("dept_id") \
                            .withColumn("dept_name", concat(lit("Department_"), col("dept_id")))
        
        # Query with filter after join - Catalyst will push filter down
        result = large_df.join(small_df, "dept_id") \
                        .filter(col("dept_id") < 10) \
                        .select("id", "dept_name")
        
        print("=== PREDICATE PUSHDOWN EXAMPLE ===")
        result.explain(True)
        
        return result
    
    def column_pruning_example(self):
        """Demonstrate column pruning optimization"""
        
        # Create DataFrame with many columns
        df = self.spark.range(100000).toDF("id")
        for i in range(20):
            df = df.withColumn(f"col_{i}", col("id") + i)
        
        # Only select few columns - Catalyst will prune unused columns
        result = df.select("id", "col_1", "col_5") \
                  .filter(col("id") > 1000)
        
        print("=== COLUMN PRUNING EXAMPLE ===")
        result.explain(True)
        
        return result
    
    def constant_folding_example(self):
        """Demonstrate constant folding optimization"""
        
        df = self.spark.range(10000).toDF("id")
        
        # Query with constant expressions - will be folded at compile time
        result = df.withColumn("computed", col("id") + (10 * 5 + 20)) \
                  .filter(col("id") > (100 - 50)) \
                  .select("id", "computed")
        
        print("=== CONSTANT FOLDING EXAMPLE ===")
        result.explain(True)
        
        return result
    
    def join_reordering_example(self):
        """Demonstrate join reordering optimization"""
        
        # Create three DataFrames of different sizes
        large_df = self.spark.range(1000000).toDF("id") \
                            .withColumn("large_value", col("id"))
        
        medium_df = self.spark.range(10000).toDF("id") \
                             .withColumn("medium_value", col("id"))
        
        small_df = self.spark.range(100).toDF("id") \
                            .withColumn("small_value", col("id"))
        
        # Multi-way join - Catalyst will reorder for optimal execution
        result = large_df.join(medium_df, "id") \
                        .join(small_df, "id") \
                        .select("id", "large_value", "medium_value", "small_value")
        
        print("=== JOIN REORDERING EXAMPLE ===")
        result.explain(True)
        
        return result
    
    def adaptive_query_execution(self):
        """Demonstrate Adaptive Query Execution (AQE) features"""
        
        # Enable AQE features
        self.spark.conf.set("spark.sql.adaptive.enabled", "true")
        self.spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
        self.spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
        
        # Create skewed data
        skewed_data = []
        for i in range(1000):
            if i < 10:
                # Create skew - first 10 keys have many records
                skewed_data.extend([(i, f"value_{j}") for j in range(10000)])
            else:
                # Other keys have few records
                skewed_data.append((i, f"value_{i}"))
        
        df1 = self.spark.createDataFrame(skewed_data, ["key", "value1"])
        df2 = self.spark.range(1000).toDF("key") \
                       .withColumn("value2", concat(lit("val2_"), col("key")))
        
        # Join that will trigger skew handling
        result = df1.join(df2, "key") \
                   .groupBy("key") \
                   .count()
        
        print("=== ADAPTIVE QUERY EXECUTION ===")
        result.explain(True)
        
        # Execute to see AQE in action
        result.collect()
        
        return result
```

## Working with Different Data Sources

### File Formats
```scala
// Comprehensive data source examples
import org.apache.spark.sql.{SparkSession, SaveMode}
import org.apache.spark.sql.types._

object DataSources {
  
  val spark = SparkSession.builder()
    .appName("DataSources")
    .getOrCreate()
  
  def workWithJSON(): Unit = {
    // Read JSON files
    val jsonDF = spark.read
      .option("multiline", "true")
      .option("mode", "PERMISSIVE")  // Handle malformed records
      .json("path/to/data.json")
    
    // Write JSON with options
    jsonDF.write
      .mode(SaveMode.Overwrite)
      .option("compression", "gzip")
      .json("path/to/output.json")
    
    // Handle nested JSON
    val nestedJsonDF = spark.read.json("path/to/nested.json")
    val flattenedDF = nestedJsonDF
      .select(
        col("id"),
        col("user.name").as("user_name"),
        col("user.email").as("user_email"),
        explode(col("orders")).as("order")
      )
      .select(
        col("id"),
        col("user_name"),
        col("user_email"),
        col("order.order_id"),
        col("order.amount")
      )
  }
  
  def workWithParquet(): Unit = {
    // Read Parquet files
    val parquetDF = spark.read
      .option("mergeSchema", "true")  // Merge schemas from multiple files
      .parquet("path/to/data.parquet")
    
    // Write Parquet with partitioning
    parquetDF.write
      .mode(SaveMode.Overwrite)
      .partitionBy("year", "month")
      .option("compression", "snappy")
      .parquet("path/to/partitioned_output.parquet")
    
    // Read specific partitions
    val filteredParquet = spark.read
      .parquet("path/to/partitioned_output.parquet")
      .filter(col("year") === 2023 && col("month") >= 6)
  }
  
  def workWithCSV(): Unit = {
    // Read CSV with schema inference
    val csvDF = spark.read
      .option("header", "true")
      .option("inferSchema", "true")
      .option("sep", ",")
      .option("quote", "\"")
      .option("escape", "\\")
      .option("nullValue", "NULL")
      .option("dateFormat", "yyyy-MM-dd")
      .csv("path/to/data.csv")
    
    // Read CSV with explicit schema
    val schema = StructType(Array(
      StructField("id", IntegerType, nullable = false),
      StructField("name", StringType, nullable = true),
      StructField("age", IntegerType, nullable = true),
      StructField("salary", DoubleType, nullable = true)
    ))
    
    val csvWithSchemaDF = spark.read
      .option("header", "true")
      .schema(schema)
      .csv("path/to/data.csv")
    
    // Write CSV
    csvDF.write
      .mode(SaveMode.Overwrite)
      .option("header", "true")
      .csv("path/to/output.csv")
  }
  
  def workWithDelta(): Unit = {
    // Delta Lake operations (requires delta-core dependency)
    import io.delta.tables._
    
    // Read Delta table
    val deltaDF = spark.read.format("delta").load("path/to/delta-table")
    
    // Write Delta table
    deltaDF.write
      .format("delta")
      .mode(SaveMode.Overwrite)
      .save("path/to/new-delta-table")
    
    // Delta table operations
    val deltaTable = DeltaTable.forPath(spark, "path/to/delta-table")
    
    // Merge operation (UPSERT)
    val updatesDF = spark.read.parquet("path/to/updates.parquet")
    
    deltaTable.as("target")
      .merge(updatesDF.as("source"), "target.id = source.id")
      .whenMatched()
      .updateAll()
      .whenNotMatched()
      .insertAll()
      .execute()
    
    // Time travel
    val historicalDF = spark.read
      .format("delta")
      .option("timestampAsOf", "2023-01-01 00:00:00")
      .load("path/to/delta-table")
  }
}
```

### Database Connectivity
```python
# Database connectivity examples
from pyspark.sql import SparkSession
from pyspark.sql.functions import *

class DatabaseConnectivity:
    
    def __init__(self):
        self.spark = SparkSession.builder \
            .appName("DatabaseConnectivity") \
            .config("spark.jars", "postgresql-42.3.1.jar,mysql-connector-java-8.0.28.jar") \
            .getOrCreate()
    
    def postgresql_operations(self):
        """PostgreSQL database operations"""
        
        # Connection properties
        postgres_props = {
            "user": "admin",
            "password": "password",
            "driver": "org.postgresql.Driver",
            "stringtype": "unspecified"  # Handle VARCHAR columns properly
        }
        
        # Read from PostgreSQL
        employees_df = self.spark.read \
            .jdbc(
                url="jdbc:postgresql://localhost:5432/company",
                table="employees",
                properties=postgres_props
            )
        
        # Read with custom query
        custom_query = """
            (SELECT e.*, d.department_name 
             FROM employees e 
             JOIN departments d ON e.dept_id = d.id 
             WHERE e.salary > 50000) as high_earners
        """
        
        high_earners_df = self.spark.read \
            .jdbc(
                url="jdbc:postgresql://localhost:5432/company",
                table=custom_query,
                properties=postgres_props
            )
        
        # Write to PostgreSQL
        processed_df = employees_df.withColumn("bonus", col("salary") * 0.1)
        
        processed_df.write \
            .mode("overwrite") \
            .jdbc(
                url="jdbc:postgresql://localhost:5432/company",
                table="employee_bonuses",
                properties=postgres_props
            )
        
        # Parallel reading with partitioning
        partitioned_df = self.spark.read \
            .jdbc(
                url="jdbc:postgresql://localhost:5432/company",
                table="large_table",
                column="id",  # Partition column
                lowerBound=1,
                upperBound=1000000,
                numPartitions=10,
                properties=postgres_props
            )
        
        return employees_df, high_earners_df
    
    def mysql_operations(self):
        """MySQL database operations"""
        
        mysql_props = {
            "user": "root",
            "password": "password",
            "driver": "com.mysql.cj.jdbc.Driver"
        }
        
        # Read from MySQL
        orders_df = self.spark.read \
            .jdbc(
                url="jdbc:mysql://localhost:3306/ecommerce",
                table="orders",
                properties=mysql_props
            )
        
        # Incremental data loading
        max_id_query = """
            (SELECT COALESCE(MAX(id), 0) as max_id FROM processed_orders) as max_id_table
        """
        
        max_id_df = self.spark.read \
            .jdbc(
                url="jdbc:mysql://localhost:3306/ecommerce",
                table=max_id_query,
                properties=mysql_props
            )
        
        max_id = max_id_df.collect()[0]["max_id"]
        
        # Read only new records
        new_orders_query = f"""
            (SELECT * FROM orders WHERE id > {max_id}) as new_orders
        """
        
        new_orders_df = self.spark.read \
            .jdbc(
                url="jdbc:mysql://localhost:3306/ecommerce",
                table=new_orders_query,
                properties=mysql_props
            )
        
        return orders_df, new_orders_df
    
    def nosql_operations(self):
        """NoSQL database operations"""
        
        # MongoDB (requires mongo-spark-connector)
        mongo_df = self.spark.read \
            .format("mongo") \
            .option("uri", "mongodb://localhost:27017/mydb.mycollection") \
            .load()
        
        # Cassandra (requires spark-cassandra-connector)
        cassandra_df = self.spark.read \
            .format("org.apache.spark.sql.cassandra") \
            .option("keyspace", "mykeyspace") \
            .option("table", "mytable") \
            .load()
        
        # Elasticsearch (requires elasticsearch-spark connector)
        es_df = self.spark.read \
            .format("es") \
            .option("es.nodes", "localhost") \
            .option("es.port", "9200") \
            .option("es.resource", "myindex/mytype") \
            .load()
        
        return mongo_df, cassandra_df, es_df
    
    def cloud_data_sources(self):
        """Cloud data source operations"""
        
        # AWS S3
        s3_df = self.spark.read \
            .option("header", "true") \
            .csv("s3a://my-bucket/data/*.csv")
        
        # Google Cloud Storage
        gcs_df = self.spark.read \
            .parquet("gs://my-bucket/data/*.parquet")
        
        # Azure Blob Storage
        azure_df = self.spark.read \
            .json("abfss://container@account.dfs.core.windows.net/data/*.json")
        
        # BigQuery (requires spark-bigquery-connector)
        bq_df = self.spark.read \
            .format("bigquery") \
            .option("table", "project.dataset.table") \
            .load()
        
        return s3_df, gcs_df, azure_df, bq_df
```

## Performance Optimization

### Query Optimization Techniques
```java
// Performance optimization strategies
import org.apache.spark.sql.*;
import org.apache.spark.sql.functions.*;
import org.apache.spark.storage.StorageLevel;

public class PerformanceOptimization {
    
    private SparkSession spark;
    
    public PerformanceOptimization() {
        this.spark = SparkSession.builder()
            .appName("PerformanceOptimization")
            .config("spark.sql.adaptive.enabled", "true")
            .config("spark.sql.adaptive.coalescePartitions.enabled", "true")
            .config("spark.sql.adaptive.skewJoin.enabled", "true")
            .config("spark.sql.cbo.enabled", "true")  // Cost-based optimizer
            .getOrCreate();
    }
    
    public void cachingStrategies() {
        Dataset<Row> largeDF = createLargeDataFrame();
        
        // Cache frequently accessed DataFrames
        largeDF.cache();
        
        // Or use specific storage levels
        largeDF.persist(StorageLevel.MEMORY_AND_DISK_SER());
        
        // Use the cached DataFrame multiple times
        Dataset<Row> result1 = largeDF.filter(col("category").equalTo("A"));
        Dataset<Row> result2 = largeDF.groupBy("department").count();
        
        // Show cache statistics
        System.out.println("Is cached: " + largeDF.storageLevel().useMemory());
        
        // Unpersist when no longer needed
        largeDF.unpersist();
    }
    
    public void partitioningOptimization() {
        Dataset<Row> df = createLargeDataFrame();
        
        // Repartition for better parallelism
        Dataset<Row> repartitioned = df.repartition(200, col("department"));
        
        // Coalesce to reduce number of partitions
        Dataset<Row> coalesced = df.coalesce(50);
        
        // Write with partitioning for faster reads
        df.write()
          .mode(SaveMode.Overwrite)
          .partitionBy("year", "month")
          .parquet("path/to/partitioned_data");
        
        // Read with partition pruning
        Dataset<Row> filtered = spark.read()
            .parquet("path/to/partitioned_data")
            .filter(col("year").equalTo(2023))
            .filter(col("month").geq(6));
    }
    
    public void joinOptimization() {
        Dataset<Row> largeTable = createLargeDataFrame();
        Dataset<Row> smallTable = createSmallDataFrame();
        
        // Broadcast small tables for broadcast joins
        Dataset<Row> broadcastJoin = largeTable.join(
            broadcast(smallTable),
            "department"
        );
        
        // Use appropriate join hints
        Dataset<Row> hintedJoin = largeTable.hint("shuffle_hash")
            .join(smallTable.hint("broadcast"), "department");
        
        // Bucketed joins for pre-partitioned data
        largeTable.write()
            .bucketBy(10, "department")
            .saveAsTable("bucketed_large_table");
        
        smallTable.write()
            .bucketBy(10, "department")
            .saveAsTable("bucketed_small_table");
        
        // This will use bucket join (no shuffle needed)
        Dataset<Row> bucketJoin = spark.table("bucketed_large_table")
            .join(spark.table("bucketed_small_table"), "department");
    }
    
    public void columnOptimization() {
        Dataset<Row> df = createWideDataFrame();
        
        // Project only needed columns early
        Dataset<Row> projected = df.select("id", "name", "salary", "department");
        
        // Use columnar formats like Parquet
        projected.write()
            .mode(SaveMode.Overwrite)
            .option("compression", "snappy")
            .parquet("path/to/columnar_data");
        
        // Predicate pushdown with columnar formats
        Dataset<Row> filtered = spark.read()
            .parquet("path/to/columnar_data")
            .filter(col("salary").gt(50000))  // This filter is pushed down
            .select("name", "salary");        // Only these columns are read
    }
    
    public void aggregationOptimization() {
        Dataset<Row> df = createLargeDataFrame();
        
        // Use approximate aggregations for large datasets
        Dataset<Row> approxStats = df.agg(
            approx_count_distinct("user_id").as("approx_unique_users"),
            expr("percentile_approx(salary, 0.5)").as("median_salary")
        );
        
        // Pre-aggregate when possible
        Dataset<Row> preAggregated = df.groupBy("department", "year")
            .agg(
                sum("salary").as("total_salary"),
                count("*").as("employee_count")
            );
        
        // Cache pre-aggregated results
        preAggregated.cache();
        
        // Use pre-aggregated data for further analysis
        Dataset<Row> departmentTrends = preAggregated
            .groupBy("department")
            .agg(avg("total_salary").as("avg_yearly_salary"));
    }
    
    public void memoryOptimization() {
        // Configure memory settings
        spark.conf().set("spark.sql.execution.arrow.pyspark.enabled", "true");
        spark.conf().set("spark.sql.columnVector.offheap.enabled", "true");
        
        Dataset<Row> df = createLargeDataFrame();
        
        // Use efficient data types
        Dataset<Row> optimized = df
            .withColumn("id", col("id").cast("int"))
            .withColumn("salary", col("salary").cast("float"))
            .withColumn("is_active", col("is_active").cast("boolean"));
        
        // Avoid wide transformations when possible
        Dataset<Row> efficient = df
            .filter(col("department").equalTo("Engineering"))  // Narrow transformation
            .select("id", "name", "salary")                    // Narrow transformation
            .orderBy("salary");                                // Wide transformation (minimize these)
    }
    
    private Dataset<Row> createLargeDataFrame() {
        // Implementation to create large DataFrame
        return spark.emptyDataFrame();
    }
    
    private Dataset<Row> createSmallDataFrame() {
        // Implementation to create small DataFrame
        return spark.emptyDataFrame();
    }
    
    private Dataset<Row> createWideDataFrame() {
        // Implementation to create wide DataFrame
        return spark.emptyDataFrame();
    }
}
```

## Real-world Example: Data Warehouse ETL Pipeline

### Complete ETL Pipeline
```python
# Complete data warehouse ETL pipeline
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from pyspark.sql.types import *
from datetime import datetime, timedelta
import logging

class DataWarehouseETL:
    
    def __init__(self):
        self.spark = SparkSession.builder \
            .appName("DataWarehouseETL") \
            .config("spark.sql.adaptive.enabled", "true") \
            .config("spark.sql.adaptive.coalescePartitions.enabled", "true") \
            .config("spark.sql.sources.partitionOverwriteMode", "dynamic") \
            .getOrCreate()
        
        self.logger = self._setup_logging()
        
    def _setup_logging(self):
        """Setup logging configuration"""
        logging.basicConfig(level=logging.INFO)
        return logging.getLogger(__name__)
    
    def extract_data(self, extraction_date):
        """Extract data from various sources"""
        
        self.logger.info(f"Starting data extraction for {extraction_date}")
        
        # Extract from transactional database
        transactions_df = self._extract_transactions(extraction_date)
        
        # Extract from user management system
        users_df = self._extract_users()
        
        # Extract from product catalog
        products_df = self._extract_products()
        
        # Extract from web logs
        web_logs_df = self._extract_web_logs(extraction_date)
        
        return {
            "transactions": transactions_df,
            "users": users_df,
            "products": products_df,
            "web_logs": web_logs_df
        }
    
    def _extract_transactions(self, extraction_date):
        """Extract transaction data"""
        
        # Calculate date range for incremental load
        start_date = extraction_date
        end_date = extraction_date + timedelta(days=1)
        
        query = f"""
            (SELECT 
                transaction_id,
                user_id,
                product_id,
                quantity,
                unit_price,
                total_amount,
                transaction_timestamp,
                payment_method,
                status
             FROM transactions 
             WHERE transaction_timestamp >= '{start_date}' 
             AND transaction_timestamp < '{end_date}') as daily_transactions
        """
        
        transactions_df = self.spark.read \
            .jdbc(
                url="jdbc:postgresql://prod-db:5432/ecommerce",
                table=query,
                properties={
                    "user": "etl_user",
                    "password": "etl_password",
                    "driver": "org.postgresql.Driver"
                }
            )
        
        self.logger.info(f"Extracted {transactions_df.count()} transactions")
        return transactions_df
    
    def _extract_users(self):
        """Extract user dimension data"""
        
        users_df = self.spark.read \
            .jdbc(
                url="jdbc:mysql://user-db:3306/users",
                table="users",
                properties={
                    "user": "readonly_user",
                    "password": "readonly_password",
                    "driver": "com.mysql.cj.jdbc.Driver"
                }
            )
        
        self.logger.info(f"Extracted {users_df.count()} users")
        return users_df
    
    def _extract_products(self):
        """Extract product dimension data"""
        
        products_df = self.spark.read \
            .format("mongo") \
            .option("uri", "mongodb://product-db:27017/catalog.products") \
            .load()
        
        self.logger.info(f"Extracted {products_df.count()} products")
        return products_df
    
    def _extract_web_logs(self, extraction_date):
        """Extract web log data from S3"""
        
        log_path = f"s3a://web-logs/year={extraction_date.year}/month={extraction_date.month:02d}/day={extraction_date.day:02d}/*.log"
        
        # Define log schema
        log_schema = StructType([
            StructField("timestamp", StringType(), True),
            StructField("ip_address", StringType(), True),
            StructField("method", StringType(), True),
            StructField("url", StringType(), True),
            StructField("status_code", IntegerType(), True),
            StructField("response_size", LongType(), True),
            StructField("user_agent", StringType(), True)
        ])
        
        # Parse log files
        raw_logs_df = self.spark.read.text(log_path)
        
        # Extract log fields using regex
        log_pattern = r'(\S+) \S+ \S+ \[([\w:/]+\s[+\-]\d{4})\] "(\S+) (\S+) \S+" (\d{3}) (\d+) "([^"]*)" "([^"]*)"'
        
        web_logs_df = raw_logs_df.select(
            regexp_extract(col("value"), log_pattern, 1).alias("ip_address"),
            regexp_extract(col("value"), log_pattern, 2).alias("timestamp"),
            regexp_extract(col("value"), log_pattern, 3).alias("method"),
            regexp_extract(col("value"), log_pattern, 4).alias("url"),
            regexp_extract(col("value"), log_pattern, 5).cast("int").alias("status_code"),
            regexp_extract(col("value"), log_pattern, 6).cast("long").alias("response_size"),
            regexp_extract(col("value"), log_pattern, 8).alias("user_agent")
        ).filter(col("ip_address") != "")
        
        self.logger.info(f"Extracted {web_logs_df.count()} web log entries")
        return web_logs_df
    
    def transform_data(self, raw_data):
        """Transform and clean the extracted data"""
        
        self.logger.info("Starting data transformation")
        
        # Transform transactions
        fact_sales = self._transform_transactions(raw_data["transactions"])
        
        # Transform dimensions
        dim_users = self._transform_users(raw_data["users"])
        dim_products = self._transform_products(raw_data["products"])
        
        # Create aggregated tables
        daily_sales_summary = self._create_daily_sales_summary(fact_sales)
        user_behavior_metrics = self._create_user_behavior_metrics(
            raw_data["web_logs"], 
            raw_data["transactions"]
        )
        
        return {
            "fact_sales": fact_sales,
            "dim_users": dim_users,
            "dim_products": dim_products,
            "daily_sales_summary": daily_sales_summary,
            "user_behavior_metrics": user_behavior_metrics
        }
    
    def _transform_transactions(self, transactions_df):
        """Transform transaction data into fact table"""
        
        fact_sales = transactions_df \
            .filter(col("status") == "completed") \
            .withColumn("sale_date", to_date(col("transaction_timestamp"))) \
            .withColumn("sale_hour", hour(col("transaction_timestamp"))) \
            .withColumn("revenue", col("quantity") * col("unit_price")) \
            .withColumn("discount_amount", 
                       when(col("total_amount") < col("revenue"), 
                            col("revenue") - col("total_amount"))
                       .otherwise(0)) \
            .select(
                col("transaction_id").alias("sale_id"),
                col("user_id"),
                col("product_id"),
                col("sale_date"),
                col("sale_hour"),
                col("quantity"),
                col("unit_price"),
                col("revenue"),
                col("discount_amount"),
                col("total_amount"),
                col("payment_method")
            )
        
        return fact_sales
    
    def _transform_users(self, users_df):
        """Transform user data into dimension table"""
        
        dim_users = users_df \
            .withColumn("age_group", 
                       when(col("age") < 25, "18-24")
                       .when(col("age") < 35, "25-34")
                       .when(col("age") < 45, "35-44")
                       .when(col("age") < 55, "45-54")
                       .otherwise("55+")) \
            .withColumn("registration_year", year(col("registration_date"))) \
            .withColumn("is_premium", col("subscription_type") == "premium") \
            .select(
                col("user_id"),
                col("username"),
                col("email"),
                col("age"),
                col("age_group"),
                col("gender"),
                col("country"),
                col("city"),
                col("registration_date"),
                col("registration_year"),
                col("subscription_type"),
                col("is_premium")
            )
        
        return dim_users
    
    def _transform_products(self, products_df):
        """Transform product data into dimension table"""
        
        # Flatten nested MongoDB structure
        dim_products = products_df \
            .select(
                col("_id").alias("product_id"),
                col("name").alias("product_name"),
                col("category.main").alias("main_category"),
                col("category.sub").alias("sub_category"),
                col("price"),
                col("brand"),
                col("description"),
                col("attributes.weight").alias("weight"),
                col("attributes.dimensions").alias("dimensions"),
                col("created_date"),
                col("is_active")
            ) \
            .withColumn("price_tier",
                       when(col("price") < 50, "Budget")
                       .when(col("price") < 200, "Mid-range")
                       .otherwise("Premium"))
        
        return dim_products
    
    def _create_daily_sales_summary(self, fact_sales):
        """Create daily sales summary table"""
        
        daily_summary = fact_sales \
            .groupBy("sale_date") \
            .agg(
                count("sale_id").alias("total_transactions"),
                sum("quantity").alias("total_items_sold"),
                sum("revenue").alias("total_revenue"),
                sum("discount_amount").alias("total_discounts"),
                sum("total_amount").alias("net_revenue"),
                countDistinct("user_id").alias("unique_customers"),
                countDistinct("product_id").alias("unique_products"),
                avg("total_amount").alias("avg_order_value")
            ) \
            .withColumn("avg_items_per_transaction", 
                       col("total_items_sold") / col("total_transactions")) \
            .withColumn("discount_rate", 
                       col("total_discounts") / col("total_revenue"))
        
        return daily_summary
    
    def _create_user_behavior_metrics(self, web_logs_df, transactions_df):
        """Create user behavior metrics"""
        
        # Process web logs
        web_sessions = web_logs_df \
            .withColumn("timestamp_parsed", 
                       to_timestamp(col("timestamp"), "dd/MMM/yyyy:HH:mm:ss Z")) \
            .withColumn("date", to_date(col("timestamp_parsed"))) \
            .filter(col("status_code") == 200)
        
        # Extract user ID from URL (assuming it's in the URL)
        user_web_activity = web_sessions \
            .withColumn("user_id", 
                       regexp_extract(col("url"), r"/user/(\d+)", 1)) \
            .filter(col("user_id") != "") \
            .groupBy("user_id", "date") \
            .agg(
                count("*").alias("page_views"),
                countDistinct("url").alias("unique_pages"),
                min("timestamp_parsed").alias("first_visit"),
                max("timestamp_parsed").alias("last_visit")
            ) \
            .withColumn("session_duration_minutes",
                       (unix_timestamp("last_visit") - unix_timestamp("first_visit")) / 60)
        
        # Combine with transaction data
        user_transactions = transactions_df \
            .withColumn("transaction_date", to_date(col("transaction_timestamp"))) \
            .groupBy("user_id", "transaction_date") \
            .agg(
                count("*").alias("transactions_count"),
                sum("total_amount").alias("daily_spend")
            )
        
        # Join web activity with transactions
        user_behavior = user_web_activity.alias("web") \
            .join(
                user_transactions.alias("trans"),
                (col("web.user_id") == col("trans.user_id")) & 
                (col("web.date") == col("trans.transaction_date")),
                "full_outer"
            ) \
            .select(
                coalesce(col("web.user_id"), col("trans.user_id")).alias("user_id"),
                coalesce(col("web.date"), col("trans.transaction_date")).alias("date"),
                coalesce(col("page_views"), lit(0)).alias("page_views"),
                coalesce(col("unique_pages"), lit(0)).alias("unique_pages"),
                coalesce(col("session_duration_minutes"), lit(0)).alias("session_duration_minutes"),
                coalesce(col("transactions_count"), lit(0)).alias("transactions_count"),
                coalesce(col("daily_spend"), lit(0.0)).alias("daily_spend")
            ) \
            .withColumn("conversion_rate",
                       when(col("page_views") > 0, 
                            col("transactions_count") / col("page_views"))
                       .otherwise(0))
        
        return user_behavior
    
    def load_data(self, transformed_data, load_date):
        """Load transformed data into data warehouse"""
        
        self.logger.info("Starting data loading")
        
        # Load fact table
        self._load_fact_sales(transformed_data["fact_sales"], load_date)
        
        # Load dimension tables
        self._load_dim_users(transformed_data["dim_users"])
        self._load_dim_products(transformed_data["dim_products"])
        
        # Load summary tables
        self._load_daily_sales_summary(transformed_data["daily_sales_summary"], load_date)
        self._load_user_behavior_metrics(transformed_data["user_behavior_metrics"], load_date)
        
        self.logger.info("Data loading completed")
    
    def _load_fact_sales(self, fact_sales_df, load_date):
        """Load fact sales table with partitioning"""
        
        # Add partition columns
        partitioned_df = fact_sales_df \
            .withColumn("year", year(col("sale_date"))) \
            .withColumn("month", month(col("sale_date")))
        
        # Write to data warehouse (Parquet format with partitioning)
        partitioned_df.write \
            .mode("append") \
            .partitionBy("year", "month") \
            .option("compression", "snappy") \
            .parquet("s3a://data-warehouse/fact_sales/")
        
        # Also load to analytical database for real-time queries
        partitioned_df.write \
            .mode("append") \
            .jdbc(
                url="jdbc:postgresql://analytics-db:5432/warehouse",
                table="fact_sales",
                properties={
                    "user": "warehouse_user",
                    "password": "warehouse_password",
                    "driver": "org.postgresql.Driver"
                }
            )
    
    def _load_dim_users(self, dim_users_df):
        """Load user dimension with SCD Type 2"""
        
        # Implement Slowly Changing Dimension Type 2
        # This is a simplified version - in practice, you'd need more complex logic
        
        dim_users_with_metadata = dim_users_df \
            .withColumn("effective_date", current_date()) \
            .withColumn("end_date", lit(None).cast("date")) \
            .withColumn("is_current", lit(True))
        
        # Write to dimension table
        dim_users_with_metadata.write \
            .mode("overwrite") \
            .option("compression", "snappy") \
            .parquet("s3a://data-warehouse/dim_users/")
    
    def _load_dim_products(self, dim_products_df):
        """Load product dimension"""
        
        dim_products_df.write \
            .mode("overwrite") \
            .option("compression", "snappy") \
            .parquet("s3a://data-warehouse/dim_products/")
    
    def _load_daily_sales_summary(self, daily_summary_df, load_date):
        """Load daily sales summary"""
        
        daily_summary_df.write \
            .mode("append") \
            .option("compression", "snappy") \
            .parquet("s3a://data-warehouse/daily_sales_summary/")
    
    def _load_user_behavior_metrics(self, user_behavior_df, load_date):
        """Load user behavior metrics"""
        
        partitioned_behavior = user_behavior_df \
            .withColumn("year", year(col("date"))) \
            .withColumn("month", month(col("date")))
        
        partitioned_behavior.write \
            .mode("append") \
            .partitionBy("year", "month") \
            .option("compression", "snappy") \
            .parquet("s3a://data-warehouse/user_behavior_metrics/")
    
    def run_etl_pipeline(self, extraction_date):
        """Run the complete ETL pipeline"""
        
        try:
            self.logger.info(f"Starting ETL pipeline for {extraction_date}")
            
            # Extract
            raw_data = self.extract_data(extraction_date)
            
            # Transform
            transformed_data = self.transform_data(raw_data)
            
            # Load
            self.load_data(transformed_data, extraction_date)
            
            # Data quality checks
            self._run_data_quality_checks(transformed_data)
            
            self.logger.info("ETL pipeline completed successfully")
            
        except Exception as e:
            self.logger.error(f"ETL pipeline failed: {str(e)}")
            raise
        
        finally:
            # Cleanup
            self.spark.catalog.clearCache()
    
    def _run_data_quality_checks(self, transformed_data):
        """Run data quality checks"""
        
        self.logger.info("Running data quality checks")
        
        # Check for null values in critical columns
        fact_sales = transformed_data["fact_sales"]
        
        null_checks = fact_sales.select(
            sum(when(col("sale_id").isNull(), 1).otherwise(0)).alias("null_sale_ids"),
            sum(when(col("user_id").isNull(), 1).otherwise(0)).alias("null_user_ids"),
            sum(when(col("total_amount") <= 0, 1).otherwise(0)).alias("invalid_amounts")
        ).collect()[0]
        
        if null_checks["null_sale_ids"] > 0:
            self.logger.warning(f"Found {null_checks['null_sale_ids']} null sale IDs")
        
        if null_checks["null_user_ids"] > 0:
            self.logger.warning(f"Found {null_checks['null_user_ids']} null user IDs")
        
        if null_checks["invalid_amounts"] > 0:
            self.logger.warning(f"Found {null_checks['invalid_amounts']} invalid amounts")
        
        self.logger.info("Data quality checks completed")

# Usage
if __name__ == "__main__":
    etl = DataWarehouseETL()
    
    # Run for yesterday's data
    extraction_date = datetime.now().date() - timedelta(days=1)
    etl.run_etl_pipeline(extraction_date)
```

## Conclusion

Spark SQL provides a powerful and unified interface for structured data processing, offering:

### Key Advantages
1. **Unified API**: Single interface for batch and streaming data
2. **Performance**: Advanced optimization through Catalyst and Tungsten
3. **Compatibility**: Standard SQL support with extensions
4. **Flexibility**: Support for various data sources and formats
5. **Scalability**: Distributed processing with automatic optimization

### Best Practices
- Use DataFrames/Datasets over RDDs for better performance
- Leverage Catalyst optimizer through proper query structure
- Cache frequently accessed data appropriately
- Use columnar formats (Parquet) for analytical workloads
- Implement proper partitioning strategies
- Monitor and tune performance continuously

### When to Use Spark SQL
- **Data Warehousing**: ETL pipelines and data transformation
- **Analytics**: Complex analytical queries on large datasets
- **Data Integration**: Combining data from multiple sources
- **Reporting**: Generating reports from structured data
- **Machine Learning**: Data preparation and feature engineering

In the next chapter, we'll explore Spark MLlib for machine learning workloads and how it integrates with Spark SQL for end-to-end ML pipelines.
