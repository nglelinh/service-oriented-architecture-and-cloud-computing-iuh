---
layout: post
title: 06 Spark Streaming - Real-time Data Processing
chapter: '06'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter06
---

Spark Streaming is Apache Spark's scalable and fault-tolerant stream processing engine that enables processing of live data streams. It extends Spark's core capabilities to handle real-time data ingestion and processing, making it possible to build end-to-end streaming applications.

## Introduction to Stream Processing

### Batch vs Stream Processing
Traditional batch processing works with finite datasets, while stream processing handles continuous, unbounded data streams in real-time.

```
Batch Processing:
┌─────────────────────────────────────────────────────────────┐
│                    Static Dataset                            │
├─────────────────────────────────────────────────────────────┤
│ Input → Process → Output                                    │
│                                                             │
│ • Fixed size data                                           │
│ • High latency (minutes to hours)                           │
│ • High throughput                                           │
│ • Complete results                                          │
└─────────────────────────────────────────────────────────────┘

Stream Processing:
┌─────────────────────────────────────────────────────────────┐
│                 Continuous Data Stream                       │
├─────────────────────────────────────────────────────────────┤
│ Input Stream → Real-time Process → Output Stream           │
│                                                             │
│ • Unbounded data                                            │
│ • Low latency (milliseconds to seconds)                     │
│ • Variable throughput                                       │
│ • Incremental results                                       │
└─────────────────────────────────────────────────────────────┘
```

### Real-world Streaming Use Cases
```javascript
// Common streaming applications
const streamingUseCases = {
  // Financial Services
  fraudDetection: {
    input: "Credit card transactions",
    processing: "Real-time anomaly detection",
    output: "Fraud alerts within 100ms",
    impact: "Prevent financial losses"
  },
  
  // E-commerce
  recommendationEngine: {
    input: "User clicks and purchases",
    processing: "Real-time ML inference",
    output: "Personalized recommendations",
    impact: "Increase conversion rates"
  },
  
  // IoT and Manufacturing
  predictiveMaintenance: {
    input: "Sensor data from machinery",
    processing: "Anomaly detection algorithms",
    output: "Maintenance alerts",
    impact: "Reduce downtime costs"
  },
  
  // Social Media
  trendingTopics: {
    input: "Social media posts and interactions",
    processing: "Real-time aggregation and ranking",
    output: "Trending topics and hashtags",
    impact: "Engage users with relevant content"
  },
  
  // Gaming
  realTimeAnalytics: {
    input: "Player actions and game events",
    processing: "Real-time metrics calculation",
    output: "Live dashboards and alerts",
    impact: "Optimize game experience"
  }
};
```

## Spark Streaming Architecture

### Core Concepts
Spark Streaming works by discretizing the continuous input stream into batches and processing them using Spark's batch processing engine.

```
Spark Streaming Architecture:
┌─────────────────────────────────────────────────────────────┐
│                    Input Sources                             │
├─────────────────┬─────────────────┬─────────────────────────┤
│     Kafka       │      Flume      │    TCP Sockets          │
│   (Messages)    │     (Logs)      │   (Network Data)        │
└─────────────────┴─────────────────┴─────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                 Spark Streaming                             │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              DStream (Discretized Stream)            │   │
│  │                                                     │   │
│  │  [Batch 1] → [Batch 2] → [Batch 3] → [Batch 4]    │   │
│  │     RDD        RDD        RDD        RDD           │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                               │
│                            ▼                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │            Spark Core Engine                        │   │
│  │         (Batch Processing)                          │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   Output Sinks                              │
├─────────────────┬─────────────────┬─────────────────────────┤
│   Databases     │   File Systems  │    Message Queues       │
│  (HDFS, S3)     │   (Local, NFS)  │   (Kafka, RabbitMQ)     │
└─────────────────┴─────────────────┴─────────────────────────┘
```

### DStreams (Discretized Streams)
DStreams are the fundamental abstraction in Spark Streaming, representing a continuous sequence of RDDs.

```python
# DStream concept illustration
from pyspark.streaming import StreamingContext
from pyspark import SparkContext

class DStreamExample:
    def __init__(self):
        self.sc = SparkContext("local[2]", "StreamingExample")
        self.ssc = StreamingContext(self.sc, 1)  # 1 second batch interval
    
    def create_dstream_from_socket(self):
        """Create DStream from TCP socket"""
        # Connect to localhost:9999
        lines = self.ssc.socketTextStream("localhost", 9999)
        
        # Each 'lines' represents a batch of data received in 1 second
        return lines
    
    def demonstrate_dstream_operations(self):
        """Show various DStream operations"""
        lines = self.create_dstream_from_socket()
        
        # Transformation: Split lines into words
        words = lines.flatMap(lambda line: line.split(" "))
        
        # Transformation: Map each word to (word, 1)
        pairs = words.map(lambda word: (word, 1))
        
        # Transformation: Count words in each batch
        word_counts = pairs.reduceByKey(lambda x, y: x + y)
        
        # Action: Print results
        word_counts.pprint()
        
        return word_counts
    
    def demonstrate_windowed_operations(self):
        """Show windowed operations"""
        lines = self.create_dstream_from_socket()
        words = lines.flatMap(lambda line: line.split(" "))
        pairs = words.map(lambda word: (word, 1))
        
        # Window operation: Count words over last 30 seconds, 
        # updated every 10 seconds
        windowed_word_counts = pairs.reduceByKeyAndWindow(
            lambda x, y: x + y,      # Reduce function
            lambda x, y: x - y,      # Inverse reduce function
            30,                      # Window duration (30 seconds)
            10                       # Slide duration (10 seconds)
        )
        
        windowed_word_counts.pprint()
        return windowed_word_counts
    
    def start_streaming(self):
        """Start the streaming context"""
        self.ssc.start()             # Start the computation
        self.ssc.awaitTermination()  # Wait for termination
```

### Micro-batch Processing Model
```
Micro-batch Timeline:
Time:    0s    1s    2s    3s    4s    5s    6s
         │     │     │     │     │     │     │
Input:   ████  ████  ████  ████  ████  ████  ████
         │     │     │     │     │     │     │
Batch:   [B1]  [B2]  [B3]  [B4]  [B5]  [B6]  [B7]
         │     │     │     │     │     │     │
Process:  ▼     ▼     ▼     ▼     ▼     ▼     ▼
         RDD1  RDD2  RDD3  RDD4  RDD5  RDD6  RDD7

Characteristics:
• Batch Interval: 1 second (configurable)
• Each batch becomes an RDD
• Processing happens in parallel
• Results available after each batch
```

## Input Sources and Data Ingestion

### Built-in Input Sources
```scala
// Scala examples of various input sources
import org.apache.spark.streaming._
import org.apache.spark.streaming.kafka010._

object InputSources {
  val ssc = new StreamingContext(sparkConf, Seconds(1))
  
  // 1. File-based sources
  def fileSource(): DStream[String] = {
    // Monitor a directory for new files
    ssc.textFileStream("hdfs://namenode:port/streaming/input/")
  }
  
  // 2. Socket-based sources  
  def socketSource(): DStream[String] = {
    // TCP socket connection
    ssc.socketTextStream("localhost", 9999)
  }
  
  // 3. Kafka integration
  def kafkaSource(): DStream[ConsumerRecord[String, String]] = {
    val kafkaParams = Map[String, Object](
      "bootstrap.servers" -> "localhost:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "streaming-consumer-group",
      "auto.offset.reset" -> "latest"
    )
    
    val topics = Array("user-events", "system-logs")
    
    KafkaUtils.createDirectStream[String, String](
      ssc,
      PreferConsistent,
      Subscribe[String, String](topics, kafkaParams)
    )
  }
  
  // 4. Custom receiver
  def customSource(): DStream[String] = {
    ssc.receiverStream(new CustomReceiver("custom-source-url"))
  }
}

// Custom receiver implementation
class CustomReceiver(url: String) extends Receiver[String](StorageLevel.MEMORY_AND_DISK_2) {
  
  def onStart(): Unit = {
    // Start the thread that receives data
    new Thread("Custom Receiver") {
      override def run(): Unit = { receive() }
    }.start()
  }
  
  def onStop(): Unit = {
    // Cleanup resources
  }
  
  private def receive(): Unit = {
    var userInput: String = null
    try {
      // Simulate receiving data
      while (!isStopped() && { userInput = receiveData(); userInput != null }) {
        store(userInput)  // Store received data
      }
    } catch {
      case e: Exception => restart("Error receiving data", e)
    }
  }
  
  private def receiveData(): String = {
    // Implement actual data receiving logic
    Thread.sleep(1000)
    s"Data received at ${System.currentTimeMillis()}"
  }
}
```

### Kafka Integration Deep Dive
```python
# Python Kafka integration example
from pyspark.sql import SparkSession
from pyspark.streaming import StreamingContext
from pyspark.sql.functions import *
from pyspark.sql.types import *

class KafkaStreamingProcessor:
    def __init__(self):
        self.spark = SparkSession.builder \
            .appName("KafkaStreamingApp") \
            .config("spark.sql.streaming.checkpointLocation", "/tmp/checkpoint") \
            .getOrCreate()
        
        self.spark.sparkContext.setLogLevel("WARN")
    
    def create_kafka_stream(self):
        """Create streaming DataFrame from Kafka"""
        kafka_df = self.spark \
            .readStream \
            .format("kafka") \
            .option("kafka.bootstrap.servers", "localhost:9092") \
            .option("subscribe", "user-events,system-logs,transactions") \
            .option("startingOffsets", "latest") \
            .load()
        
        return kafka_df
    
    def process_user_events(self):
        """Process user event stream"""
        kafka_df = self.create_kafka_stream()
        
        # Define schema for user events
        user_event_schema = StructType([
            StructField("user_id", StringType(), True),
            StructField("event_type", StringType(), True),
            StructField("timestamp", LongType(), True),
            StructField("properties", MapType(StringType(), StringType()), True)
        ])
        
        # Parse JSON messages
        parsed_df = kafka_df \
            .filter(col("topic") == "user-events") \
            .select(
                from_json(col("value").cast("string"), user_event_schema).alias("data"),
                col("timestamp").alias("kafka_timestamp"),
                col("partition"),
                col("offset")
            ) \
            .select("data.*", "kafka_timestamp", "partition", "offset")
        
        # Real-time aggregations
        user_activity = parsed_df \
            .withWatermark("timestamp", "10 minutes") \
            .groupBy(
                window(col("timestamp"), "5 minutes", "1 minute"),
                col("event_type")
            ) \
            .agg(
                count("*").alias("event_count"),
                countDistinct("user_id").alias("unique_users")
            )
        
        return user_activity
    
    def detect_anomalies(self):
        """Real-time anomaly detection"""
        kafka_df = self.create_kafka_stream()
        
        # Transaction schema
        transaction_schema = StructType([
            StructField("transaction_id", StringType(), True),
            StructField("user_id", StringType(), True),
            StructField("amount", DoubleType(), True),
            StructField("merchant", StringType(), True),
            StructField("timestamp", LongType(), True)
        ])
        
        transactions = kafka_df \
            .filter(col("topic") == "transactions") \
            .select(
                from_json(col("value").cast("string"), transaction_schema).alias("data")
            ) \
            .select("data.*")
        
        # Detect large transactions (simple rule-based)
        anomalies = transactions \
            .filter(col("amount") > 10000) \
            .withColumn("anomaly_type", lit("large_transaction")) \
            .withColumn("detected_at", current_timestamp())
        
        return anomalies
    
    def start_streaming_queries(self):
        """Start all streaming queries"""
        
        # User activity monitoring
        user_activity = self.process_user_events()
        activity_query = user_activity \
            .writeStream \
            .outputMode("update") \
            .format("console") \
            .option("truncate", False) \
            .trigger(processingTime="30 seconds") \
            .start()
        
        # Anomaly detection
        anomalies = self.detect_anomalies()
        anomaly_query = anomalies \
            .writeStream \
            .outputMode("append") \
            .format("console") \
            .option("truncate", False) \
            .start()
        
        # Wait for termination
        self.spark.streams.awaitAnyTermination()

# Usage
if __name__ == "__main__":
    processor = KafkaStreamingProcessor()
    processor.start_streaming_queries()
```

## Transformations and Operations

### Stateless Transformations
```python
# Stateless transformations example
class StatelessTransformations:
    
    def basic_transformations(self, dstream):
        """Basic stateless transformations"""
        
        # Map: Transform each element
        mapped = dstream.map(lambda x: x.upper())
        
        # Filter: Select elements based on condition
        filtered = dstream.filter(lambda x: len(x) > 5)
        
        # FlatMap: Transform and flatten
        words = dstream.flatMap(lambda line: line.split(" "))
        
        # Union: Combine multiple DStreams
        combined = dstream.union(another_dstream)
        
        return mapped, filtered, words, combined
    
    def aggregation_transformations(self, dstream):
        """Aggregation transformations"""
        
        # ReduceByKey: Aggregate by key within each batch
        word_pairs = dstream.map(lambda word: (word, 1))
        word_counts = word_pairs.reduceByKey(lambda a, b: a + b)
        
        # CountByValue: Count occurrences of each value
        value_counts = dstream.countByValue()
        
        # Reduce: Aggregate all elements in each batch
        total = dstream.reduce(lambda a, b: a + b)
        
        return word_counts, value_counts, total
    
    def join_operations(self, dstream1, dstream2):
        """Join operations between DStreams"""
        
        # Join: Inner join on keys
        joined = dstream1.join(dstream2)
        
        # LeftOuterJoin: Left outer join
        left_joined = dstream1.leftOuterJoin(dstream2)
        
        # CoGroup: Group values from both streams by key
        cogrouped = dstream1.cogroup(dstream2)
        
        return joined, left_joined, cogrouped
```

### Stateful Transformations
```scala
// Stateful transformations in Scala
import org.apache.spark.streaming.State
import org.apache.spark.streaming.StateSpec

object StatefulTransformations {
  
  // UpdateStateByKey: Maintain state across batches
  def updateStateByKey(dstream: DStream[(String, Int)]): DStream[(String, Int)] = {
    
    def updateFunction(newValues: Seq[Int], runningCount: Option[Int]): Option[Int] = {
      val newCount = newValues.sum + runningCount.getOrElse(0)
      Some(newCount)
    }
    
    dstream.updateStateByKey(updateFunction)
  }
  
  // MapWithState: More efficient state management
  def mapWithState(dstream: DStream[(String, Int)]): DStream[(String, Int)] = {
    
    def mappingFunction(key: String, value: Option[Int], state: State[Int]): Option[(String, Int)] = {
      val currentCount = value.getOrElse(0)
      val previousCount = state.getOption().getOrElse(0)
      val newCount = currentCount + previousCount
      
      state.update(newCount)
      Some((key, newCount))
    }
    
    val stateSpec = StateSpec.function(mappingFunction)
      .initialState(initialRDD)  // Optional initial state
      .numPartitions(10)         // Number of partitions for state
      .timeout(Minutes(30))      // Timeout inactive keys
    
    dstream.mapWithState(stateSpec)
  }
  
  // Session-based analytics example
  def sessionAnalytics(userEvents: DStream[(String, Event)]): DStream[(String, Session)] = {
    
    case class Event(timestamp: Long, action: String, page: String)
    case class Session(startTime: Long, endTime: Long, pageViews: Int, actions: List[String])
    
    def updateSession(key: String, value: Option[Event], state: State[Session]): Option[(String, Session)] = {
      val event = value.get
      val currentSession = state.getOption()
      
      currentSession match {
        case Some(session) =>
          // Update existing session
          val updatedSession = session.copy(
            endTime = event.timestamp,
            pageViews = session.pageViews + 1,
            actions = session.actions :+ event.action
          )
          state.update(updatedSession)
          Some((key, updatedSession))
          
        case None =>
          // Create new session
          val newSession = Session(
            startTime = event.timestamp,
            endTime = event.timestamp,
            pageViews = 1,
            actions = List(event.action)
          )
          state.update(newSession)
          Some((key, newSession))
      }
    }
    
    val stateSpec = StateSpec.function(updateSession)
      .timeout(Minutes(30))  // Session timeout
    
    userEvents.mapWithState(stateSpec)
  }
}
```

## Window Operations

### Time-based Windows
```python
# Window operations example
from pyspark.streaming import StreamingContext
from datetime import datetime, timedelta

class WindowOperations:
    
    def __init__(self, ssc):
        self.ssc = ssc
    
    def sliding_window_example(self, dstream):
        """Sliding window operations"""
        
        # Count elements in sliding window
        # Window: 30 seconds, Slide: 10 seconds
        windowed_counts = dstream.countByWindow(30, 10)
        
        # Reduce over sliding window
        windowed_sum = dstream.reduceByWindow(
            lambda x, y: x + y,    # Reduce function
            lambda x, y: x - y,    # Inverse reduce function (optional)
            30,                    # Window duration
            10                     # Slide duration
        )
        
        return windowed_counts, windowed_sum
    
    def keyed_window_operations(self, pair_dstream):
        """Window operations on key-value pairs"""
        
        # ReduceByKeyAndWindow: Aggregate by key over window
        windowed_word_counts = pair_dstream.reduceByKeyAndWindow(
            lambda x, y: x + y,    # Reduce function
            lambda x, y: x - y,    # Inverse reduce function
            60,                    # Window duration (60 seconds)
            20                     # Slide duration (20 seconds)
        )
        
        # CountByValueAndWindow: Count values over window
        windowed_value_counts = pair_dstream.countByValueAndWindow(60, 20)
        
        return windowed_word_counts, windowed_value_counts
    
    def advanced_window_analytics(self, user_events):
        """Advanced window-based analytics"""
        
        def calculate_metrics(time, rdd):
            """Calculate custom metrics for each window"""
            if not rdd.isEmpty():
                # Convert to DataFrame for complex operations
                df = rdd.toDF(["user_id", "event_type", "timestamp"])
                
                # Calculate various metrics
                total_events = df.count()
                unique_users = df.select("user_id").distinct().count()
                event_distribution = df.groupBy("event_type").count().collect()
                
                print(f"Window ending at {time}:")
                print(f"  Total events: {total_events}")
                print(f"  Unique users: {unique_users}")
                print(f"  Event distribution: {event_distribution}")
        
        # Apply custom function to each window
        user_events.window(60, 20).foreachRDD(calculate_metrics)
    
    def real_time_dashboard_data(self, metrics_stream):
        """Generate data for real-time dashboard"""
        
        # 5-minute window, updated every minute
        dashboard_metrics = metrics_stream.window(300, 60).transform(
            lambda time, rdd: self.calculate_dashboard_metrics(time, rdd)
        )
        
        return dashboard_metrics
    
    def calculate_dashboard_metrics(self, time, rdd):
        """Calculate metrics for dashboard"""
        if rdd.isEmpty():
            return rdd
        
        # Convert to DataFrame for SQL operations
        df = rdd.toDF(["metric_name", "value", "timestamp"])
        
        # Register as temporary table
        df.createOrReplaceTempView("metrics")
        
        # Calculate aggregated metrics using SQL
        result = self.ssc.sparkContext.sql("""
            SELECT 
                'avg_response_time' as metric,
                AVG(CASE WHEN metric_name = 'response_time' THEN value END) as value,
                MAX(timestamp) as timestamp
            FROM metrics
            UNION ALL
            SELECT 
                'total_requests' as metric,
                SUM(CASE WHEN metric_name = 'request_count' THEN value END) as value,
                MAX(timestamp) as timestamp
            FROM metrics
            UNION ALL
            SELECT 
                'error_rate' as metric,
                SUM(CASE WHEN metric_name = 'error_count' THEN value END) / 
                SUM(CASE WHEN metric_name = 'request_count' THEN value END) * 100 as value,
                MAX(timestamp) as timestamp
            FROM metrics
        """)
        
        return result
```

## Output Operations and Sinks

### Built-in Output Operations
```java
// Java output operations example
import org.apache.spark.streaming.api.java.*;
import org.apache.spark.api.java.function.*;

public class OutputOperations {
    
    public void basicOutputs(JavaDStream<String> dstream) {
        
        // Print: Output to console (for debugging)
        dstream.print();
        dstream.print(20); // Print first 20 elements
        
        // SaveAsTextFiles: Save each RDD to text files
        dstream.saveAsTextFiles("hdfs://output/prefix", "suffix");
        
        // SaveAsObjectFiles: Save as serialized objects
        dstream.saveAsObjectFiles("hdfs://output/objects", "obj");
        
        // SaveAsHadoopFiles: Save using Hadoop OutputFormat
        dstream.saveAsHadoopFiles(
            "hdfs://output/hadoop",
            "part",
            Text.class,
            IntWritable.class,
            TextOutputFormat.class
        );
    }
    
    public void customOutputs(JavaDStream<String> dstream) {
        
        // ForEach: Apply custom function to each RDD
        dstream.foreachRDD(new VoidFunction<JavaRDD<String>>() {
            @Override
            public void call(JavaRDD<String> rdd) throws Exception {
                // Custom processing for each RDD
                if (!rdd.isEmpty()) {
                    // Save to database, send to external system, etc.
                    saveToDatabase(rdd.collect());
                }
            }
        });
        
        // Transform and save
        JavaDStream<String> processed = dstream.map(new Function<String, String>() {
            @Override
            public String call(String s) throws Exception {
                return processRecord(s);
            }
        });
        
        processed.foreachRDD(rdd -> {
            // Partition-wise processing for better performance
            rdd.foreachPartition(partition -> {
                // Initialize connection per partition
                DatabaseConnection conn = new DatabaseConnection();
                
                while (partition.hasNext()) {
                    String record = partition.next();
                    conn.insert(record);
                }
                
                conn.close();
            });
        });
    }
    
    private void saveToDatabase(List<String> records) {
        // Database saving logic
        System.out.println("Saving " + records.size() + " records to database");
    }
    
    private String processRecord(String record) {
        // Record processing logic
        return record.toUpperCase();
    }
}
```

### Advanced Output Patterns
```python
# Advanced output patterns
import json
from datetime import datetime
from pyspark.sql import SparkSession
from pyspark.sql.functions import *

class AdvancedOutputs:
    
    def __init__(self):
        self.spark = SparkSession.builder.appName("AdvancedOutputs").getOrCreate()
    
    def multi_sink_output(self, processed_stream):
        """Output to multiple sinks simultaneously"""
        
        def multi_sink_foreach_batch(df, epoch_id):
            """Process each micro-batch"""
            
            # Cache the DataFrame since we'll use it multiple times
            df.cache()
            
            try:
                # Sink 1: Save to Parquet for analytics
                df.write \
                    .mode("append") \
                    .partitionBy("date") \
                    .parquet("s3://data-lake/processed-events/")
                
                # Sink 2: Save alerts to database
                alerts = df.filter(col("severity") == "HIGH")
                if alerts.count() > 0:
                    alerts.write \
                        .format("jdbc") \
                        .option("url", "jdbc:postgresql://db:5432/alerts") \
                        .option("dbtable", "real_time_alerts") \
                        .option("user", "admin") \
                        .option("password", "password") \
                        .mode("append") \
                        .save()
                
                # Sink 3: Send metrics to monitoring system
                metrics = df.groupBy("event_type") \
                    .agg(count("*").alias("count")) \
                    .collect()
                
                self.send_metrics_to_monitoring(metrics, epoch_id)
                
                # Sink 4: Update real-time dashboard cache
                dashboard_data = df.groupBy("region", "event_type") \
                    .agg(
                        count("*").alias("event_count"),
                        avg("processing_time").alias("avg_processing_time")
                    )
                
                self.update_dashboard_cache(dashboard_data)
                
            finally:
                df.unpersist()
        
        # Start the streaming query with custom foreach batch
        query = processed_stream.writeStream \
            .foreachBatch(multi_sink_foreach_batch) \
            .outputMode("update") \
            .trigger(processingTime="10 seconds") \
            .start()
        
        return query
    
    def exactly_once_delivery(self, stream):
        """Implement exactly-once delivery semantics"""
        
        def idempotent_write(df, epoch_id):
            """Idempotent write operation"""
            
            # Add unique identifier for each batch
            df_with_batch_id = df.withColumn("batch_id", lit(epoch_id))
            
            # Use upsert operation (merge) instead of append
            df_with_batch_id.createOrReplaceTempView("batch_data")
            
            # Merge logic to handle duplicates
            self.spark.sql("""
                MERGE INTO target_table t
                USING batch_data s
                ON t.id = s.id AND t.batch_id = s.batch_id
                WHEN NOT MATCHED THEN
                    INSERT (id, data, batch_id, processed_at)
                    VALUES (s.id, s.data, s.batch_id, current_timestamp())
            """)
        
        query = stream.writeStream \
            .foreachBatch(idempotent_write) \
            .option("checkpointLocation", "/tmp/checkpoint/exactly-once") \
            .start()
        
        return query
    
    def send_metrics_to_monitoring(self, metrics, batch_id):
        """Send metrics to external monitoring system"""
        
        monitoring_data = {
            "timestamp": datetime.now().isoformat(),
            "batch_id": batch_id,
            "metrics": [{"event_type": row["event_type"], "count": row["count"]} 
                       for row in metrics]
        }
        
        # Send to monitoring system (e.g., Prometheus, CloudWatch)
        print(f"Sending metrics: {json.dumps(monitoring_data, indent=2)}")
    
    def update_dashboard_cache(self, dashboard_data):
        """Update real-time dashboard cache"""
        
        # Write to Redis or similar cache
        dashboard_data.write \
            .format("org.apache.spark.sql.redis") \
            .option("table", "dashboard_metrics") \
            .option("key.column", "region") \
            .mode("overwrite") \
            .save()
```

## Performance Optimization

### Tuning Parameters
```scala
// Performance tuning configuration
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.SparkConf

object PerformanceTuning {
  
  def optimizedSparkConf(): SparkConf = {
    new SparkConf()
      .setAppName("OptimizedStreamingApp")
      
      // Memory settings
      .set("spark.executor.memory", "4g")
      .set("spark.executor.memoryFraction", "0.8")
      .set("spark.streaming.receiver.maxRate", "10000")  // Max records per second per receiver
      
      // Batch interval optimization
      .set("spark.streaming.blockInterval", "200ms")     // Block interval for receivers
      
      // Backpressure (dynamic rate limiting)
      .set("spark.streaming.backpressure.enabled", "true")
      .set("spark.streaming.backpressure.initialRate", "1000")
      
      // Kafka-specific optimizations
      .set("spark.streaming.kafka.maxRatePerPartition", "2000")
      
      // Checkpointing
      .set("spark.streaming.stopGracefullyOnShutdown", "true")
      
      // Serialization
      .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
      
      // Garbage collection
      .set("spark.executor.extraJavaOptions", 
           "-XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:+PrintGCDetails")
  }
  
  def createOptimizedStreamingContext(): StreamingContext = {
    val conf = optimizedSparkConf()
    val ssc = new StreamingContext(conf, Seconds(2))  // 2-second batch interval
    
    // Set checkpoint directory for fault tolerance
    ssc.checkpoint("hdfs://namenode:port/streaming/checkpoint")
    
    ssc
  }
  
  // Optimization techniques
  def optimizationTechniques(ssc: StreamingContext): Unit = {
    
    // 1. Appropriate batch interval
    // Rule of thumb: processing time should be < batch interval
    
    // 2. Parallelism optimization
    val inputDStream = ssc.socketTextStream("localhost", 9999)
    
    // Repartition if needed to increase parallelism
    val repartitioned = inputDStream.repartition(8)
    
    // 3. Caching for iterative operations
    val words = repartitioned.flatMap(_.split(" "))
    words.cache()  // Cache if used multiple times
    
    // 4. Efficient state management
    val wordCounts = words.map((_, 1))
      .reduceByKeyAndWindow(
        _ + _,           // Reduce function
        _ - _,           // Inverse reduce function (more efficient)
        Seconds(60),     // Window duration
        Seconds(20)      // Slide duration
      )
    
    // 5. Broadcast variables for lookup tables
    val broadcastLookup = ssc.sparkContext.broadcast(loadLookupTable())
    
    val enriched = words.map { word =>
      val lookup = broadcastLookup.value
      (word, lookup.getOrElse(word, "unknown"))
    }
  }
  
  def loadLookupTable(): Map[String, String] = {
    // Load lookup data
    Map("hello" -> "greeting", "world" -> "earth")
  }
}
```

### Memory Management
```python
# Memory management strategies
class MemoryManagement:
    
    def configure_memory_settings(self):
        """Configure memory settings for streaming"""
        
        spark_conf = {
            # Executor memory settings
            "spark.executor.memory": "8g",
            "spark.executor.memoryFraction": "0.75",
            "spark.executor.cores": "4",
            
            # Storage memory settings
            "spark.sql.streaming.stateStore.maintenanceInterval": "60s",
            "spark.sql.streaming.stateStore.minDeltasForSnapshot": "10",
            
            # Shuffle settings
            "spark.sql.shuffle.partitions": "200",
            "spark.serializer": "org.apache.spark.serializer.KryoSerializer",
            
            # Garbage collection
            "spark.executor.extraJavaOptions": 
                "-XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:+PrintGCTimeStamps"
        }
        
        return spark_conf
    
    def memory_efficient_processing(self, stream):
        """Memory-efficient stream processing patterns"""
        
        # 1. Process data in smaller batches
        def process_partition(partition):
            """Process partition efficiently"""
            batch_size = 1000
            batch = []
            
            for record in partition:
                batch.append(record)
                
                if len(batch) >= batch_size:
                    # Process batch
                    yield self.process_batch(batch)
                    batch = []
            
            # Process remaining records
            if batch:
                yield self.process_batch(batch)
        
        # 2. Use mapPartitions for memory efficiency
        efficient_stream = stream.mapPartitions(process_partition)
        
        # 3. Avoid collecting large datasets
        def safe_foreach_batch(df, epoch_id):
            """Safe processing without collecting all data"""
            
            # Process in chunks instead of collecting all
            total_count = df.count()
            chunk_size = 10000
            
            for i in range(0, total_count, chunk_size):
                chunk = df.limit(chunk_size).offset(i)
                self.process_chunk(chunk.collect())
        
        return efficient_stream
    
    def process_batch(self, batch):
        """Process a batch of records"""
        # Implement batch processing logic
        return [record.upper() for record in batch]
    
    def process_chunk(self, chunk):
        """Process a chunk of data"""
        # Implement chunk processing logic
        print(f"Processing chunk of {len(chunk)} records")
```

## Fault Tolerance and Checkpointing

### Checkpointing Mechanism
```java
// Checkpointing and fault tolerance
import org.apache.spark.streaming.api.java.JavaStreamingContext;

public class FaultTolerance {
    
    public static JavaStreamingContext createStreamingContext(String checkpointDir) {
        
        // Function to create new StreamingContext
        Function0<JavaStreamingContext> createContext = () -> {
            SparkConf conf = new SparkConf()
                .setAppName("FaultTolerantStreamingApp")
                .setMaster("local[*]");
            
            JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(2));
            
            // Set checkpoint directory
            jssc.checkpoint(checkpointDir);
            
            // Define streaming computation
            JavaDStream<String> lines = jssc.socketTextStream("localhost", 9999);
            
            JavaDStream<String> words = lines.flatMap(line -> 
                Arrays.asList(line.split(" ")).iterator());
            
            JavaPairDStream<String, Integer> pairs = words.mapToPair(word -> 
                new Tuple2<>(word, 1));
            
            JavaPairDStream<String, Integer> wordCounts = pairs.reduceByKey((a, b) -> a + b);
            
            wordCounts.print();
            
            return jssc;
        };
        
        // Get or create StreamingContext from checkpoint
        JavaStreamingContext jssc = JavaStreamingContext.getOrCreate(checkpointDir, createContext);
        
        return jssc;
    }
    
    public void gracefulShutdown(JavaStreamingContext jssc) {
        // Add shutdown hook for graceful termination
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.out.println("Shutting down streaming application gracefully...");
            jssc.stop(true, true);  // Stop gracefully, wait for completion
            System.out.println("Application stopped.");
        }));
        
        jssc.start();
        jssc.awaitTermination();
    }
    
    public void recoverFromFailure() {
        String checkpointDir = "hdfs://namenode:port/streaming/checkpoint";
        
        try {
            JavaStreamingContext jssc = createStreamingContext(checkpointDir);
            gracefulShutdown(jssc);
            
        } catch (Exception e) {
            System.err.println("Failed to start streaming context: " + e.getMessage());
            
            // Implement retry logic
            int maxRetries = 3;
            int retryCount = 0;
            
            while (retryCount < maxRetries) {
                try {
                    Thread.sleep(5000);  // Wait before retry
                    JavaStreamingContext jssc = createStreamingContext(checkpointDir);
                    gracefulShutdown(jssc);
                    break;
                    
                } catch (Exception retryException) {
                    retryCount++;
                    System.err.println("Retry " + retryCount + " failed: " + 
                                     retryException.getMessage());
                }
            }
        }
    }
}
```

## Real-world Example: Real-time Analytics Pipeline

### Complete End-to-End Example
```python
# Complete real-time analytics pipeline
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from pyspark.sql.types import *
import json

class RealTimeAnalyticsPipeline:
    
    def __init__(self):
        self.spark = SparkSession.builder \
            .appName("RealTimeAnalyticsPipeline") \
            .config("spark.sql.streaming.checkpointLocation", "/tmp/checkpoint") \
            .config("spark.sql.streaming.stateStore.maintenanceInterval", "60s") \
            .getOrCreate()
        
        self.spark.sparkContext.setLogLevel("WARN")
    
    def setup_schemas(self):
        """Define schemas for different event types"""
        
        self.user_event_schema = StructType([
            StructField("user_id", StringType(), True),
            StructField("session_id", StringType(), True),
            StructField("event_type", StringType(), True),
            StructField("page_url", StringType(), True),
            StructField("timestamp", LongType(), True),
            StructField("user_agent", StringType(), True),
            StructField("ip_address", StringType(), True)
        ])
        
        self.transaction_schema = StructType([
            StructField("transaction_id", StringType(), True),
            StructField("user_id", StringType(), True),
            StructField("amount", DoubleType(), True),
            StructField("currency", StringType(), True),
            StructField("merchant_id", StringType(), True),
            StructField("timestamp", LongType(), True),
            StructField("payment_method", StringType(), True)
        ])
    
    def create_input_streams(self):
        """Create input streams from Kafka"""
        
        # User events stream
        user_events_raw = self.spark \
            .readStream \
            .format("kafka") \
            .option("kafka.bootstrap.servers", "localhost:9092") \
            .option("subscribe", "user-events") \
            .option("startingOffsets", "latest") \
            .load()
        
        # Parse user events
        self.user_events = user_events_raw \
            .select(
                from_json(col("value").cast("string"), self.user_event_schema).alias("data"),
                col("timestamp").alias("kafka_timestamp")
            ) \
            .select("data.*", "kafka_timestamp") \
            .withColumn("event_time", from_unixtime(col("timestamp"))) \
            .withWatermark("event_time", "10 minutes")
        
        # Transaction events stream
        transactions_raw = self.spark \
            .readStream \
            .format("kafka") \
            .option("kafka.bootstrap.servers", "localhost:9092") \
            .option("subscribe", "transactions") \
            .option("startingOffsets", "latest") \
            .load()
        
        # Parse transactions
        self.transactions = transactions_raw \
            .select(
                from_json(col("value").cast("string"), self.transaction_schema).alias("data")
            ) \
            .select("data.*") \
            .withColumn("transaction_time", from_unixtime(col("timestamp"))) \
            .withWatermark("transaction_time", "5 minutes")
    
    def real_time_user_analytics(self):
        """Real-time user behavior analytics"""
        
        # Page view analytics
        page_views = self.user_events \
            .filter(col("event_type") == "page_view") \
            .groupBy(
                window(col("event_time"), "5 minutes", "1 minute"),
                col("page_url")
            ) \
            .agg(
                count("*").alias("view_count"),
                countDistinct("user_id").alias("unique_visitors"),
                countDistinct("session_id").alias("unique_sessions")
            ) \
            .withColumn("avg_views_per_visitor", 
                       col("view_count") / col("unique_visitors"))
        
        # User session analytics
        session_analytics = self.user_events \
            .groupBy(
                window(col("event_time"), "10 minutes", "2 minutes"),
                col("session_id"),
                col("user_id")
            ) \
            .agg(
                count("*").alias("events_in_session"),
                collect_list("page_url").alias("page_sequence"),
                min("event_time").alias("session_start"),
                max("event_time").alias("session_end")
            ) \
            .withColumn("session_duration_minutes",
                       (unix_timestamp("session_end") - unix_timestamp("session_start")) / 60)
        
        return page_views, session_analytics
    
    def fraud_detection(self):
        """Real-time fraud detection"""
        
        # Detect suspicious transaction patterns
        suspicious_transactions = self.transactions \
            .groupBy(
                window(col("transaction_time"), "1 minute"),
                col("user_id")
            ) \
            .agg(
                count("*").alias("transaction_count"),
                sum("amount").alias("total_amount"),
                countDistinct("merchant_id").alias("unique_merchants"),
                collect_list("payment_method").alias("payment_methods")
            ) \
            .filter(
                (col("transaction_count") > 10) |  # Too many transactions
                (col("total_amount") > 10000) |    # High amount
                (col("unique_merchants") > 5)      # Too many different merchants
            ) \
            .withColumn("fraud_score", 
                       col("transaction_count") * 0.3 + 
                       col("total_amount") / 1000 * 0.4 + 
                       col("unique_merchants") * 0.3) \
            .filter(col("fraud_score") > 5.0)
        
        return suspicious_transactions
    
    def real_time_recommendations(self):
        """Generate real-time recommendations"""
        
        # User behavior patterns
        user_patterns = self.user_events \
            .filter(col("event_type").isin(["page_view", "click", "purchase"])) \
            .groupBy(
                window(col("event_time"), "30 minutes", "5 minutes"),
                col("user_id")
            ) \
            .agg(
                collect_list("page_url").alias("visited_pages"),
                count("*").alias("activity_level")
            )
        
        # Join with transaction data for purchase behavior
        purchase_behavior = self.transactions \
            .groupBy(
                window(col("transaction_time"), "30 minutes", "5 minutes"),
                col("user_id")
            ) \
            .agg(
                collect_list("merchant_id").alias("purchased_from"),
                avg("amount").alias("avg_purchase_amount")
            )
        
        # Combine for recommendations
        recommendation_data = user_patterns.join(
            purchase_behavior,
            ["window", "user_id"],
            "left_outer"
        )
        
        return recommendation_data
    
    def setup_outputs(self):
        """Setup output sinks"""
        
        page_views, session_analytics = self.real_time_user_analytics()
        suspicious_transactions = self.fraud_detection()
        recommendations = self.real_time_recommendations()
        
        # Output 1: Page views to console (for monitoring)
        page_views_query = page_views \
            .writeStream \
            .outputMode("update") \
            .format("console") \
            .option("truncate", False) \
            .trigger(processingTime="30 seconds") \
            .start()
        
        # Output 2: Fraud alerts to database
        fraud_query = suspicious_transactions \
            .writeStream \
            .outputMode("update") \
            .format("console") \
            .option("truncate", False) \
            .trigger(processingTime="10 seconds") \
            .start()
        
        # Output 3: Session analytics to Parquet files
        session_query = session_analytics \
            .writeStream \
            .outputMode("append") \
            .format("parquet") \
            .option("path", "/tmp/session-analytics") \
            .option("checkpointLocation", "/tmp/checkpoint/sessions") \
            .trigger(processingTime="60 seconds") \
            .start()
        
        # Output 4: Recommendations to Kafka
        recommendations_query = recommendations \
            .select(to_json(struct("*")).alias("value")) \
            .writeStream \
            .format("kafka") \
            .option("kafka.bootstrap.servers", "localhost:9092") \
            .option("topic", "recommendations") \
            .option("checkpointLocation", "/tmp/checkpoint/recommendations") \
            .start()
        
        return [page_views_query, fraud_query, session_query, recommendations_query]
    
    def run_pipeline(self):
        """Run the complete pipeline"""
        
        self.setup_schemas()
        self.create_input_streams()
        queries = self.setup_outputs()
        
        # Wait for all queries to terminate
        for query in queries:
            query.awaitTermination()

# Usage
if __name__ == "__main__":
    pipeline = RealTimeAnalyticsPipeline()
    pipeline.run_pipeline()
```

## Conclusion

Spark Streaming provides a powerful platform for real-time data processing with the following key advantages:

### Key Benefits
1. **Unified Platform**: Same API for batch and stream processing
2. **Fault Tolerance**: Automatic recovery from failures
3. **Scalability**: Linear scalability with cluster size
4. **Integration**: Rich ecosystem integration (Kafka, HDFS, databases)
5. **Exactly-once Processing**: Strong consistency guarantees

### Best Practices
- Choose appropriate batch intervals based on latency requirements
- Use structured streaming for complex event processing
- Implement proper checkpointing for fault tolerance
- Monitor and tune performance continuously
- Design for exactly-once semantics when needed

### When to Use Spark Streaming
- **Real-time Analytics**: Dashboard updates, metrics calculation
- **ETL Pipelines**: Continuous data transformation and loading
- **Fraud Detection**: Real-time anomaly detection
- **IoT Processing**: Sensor data analysis
- **Log Processing**: Real-time log analysis and alerting

In the next chapter, we'll explore Spark SQL for structured data processing and how it complements streaming workloads.
