---
layout: post
title: 02-01 MapReduce Core Concepts and Architecture
chapter: '02'
order: 2
owner: Nguyen Le Linh
lang: en
categories:
- chapter02
lesson_type: required
---

Understanding the core concepts and architecture of MapReduce is essential for effectively designing and implementing distributed data processing solutions. This lesson explores the fundamental building blocks, execution model, and architectural components that make MapReduce a powerful paradigm for big data processing.

## The MapReduce Programming Model

### Functional Programming Foundations
MapReduce is inspired by functional programming concepts, particularly the `map` and `reduce` functions found in languages like Lisp and Python.

```python
# Functional programming inspiration
def traditional_map_reduce():
    """Traditional functional programming example"""
    
    # Sample data: list of numbers
    numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    
    # Map: Apply function to each element
    squared = list(map(lambda x: x * x, numbers))
    print(f"Mapped (squared): {squared}")
    
    # Reduce: Aggregate results
    from functools import reduce
    sum_of_squares = reduce(lambda x, y: x + y, squared)
    print(f"Reduced (sum): {sum_of_squares}")
    
    return sum_of_squares

# MapReduce extends this concept to distributed systems
def distributed_mapreduce_concept():
    """Conceptual distributed version"""
    
    # Data distributed across multiple machines
    data_partitions = [
        [1, 2, 3],    # Partition 1 on Machine A
        [4, 5, 6],    # Partition 2 on Machine B  
        [7, 8, 9, 10] # Partition 3 on Machine C
    ]
    
    # Map phase: Process each partition independently
    mapped_partitions = []
    for partition in data_partitions:
        # This would run on different machines in parallel
        mapped_partition = [x * x for x in partition]
        mapped_partitions.append(mapped_partition)
    
    print(f"Mapped partitions: {mapped_partitions}")
    
    # Shuffle phase: Group by key (implicit here)
    all_mapped_values = []
    for partition in mapped_partitions:
        all_mapped_values.extend(partition)
    
    # Reduce phase: Aggregate all results
    final_result = sum(all_mapped_values)
    print(f"Final result: {final_result}")
    
    return final_result
```

### Key-Value Pair Abstraction
MapReduce operates on key-value pairs throughout the entire processing pipeline.

```
Data Flow with Key-Value Pairs:

Input Data:
┌─────────────────────────────────────────────────────────────┐
│ "Hello World"                                               │
│ "Hello MapReduce"                                           │
│ "World of Big Data"                                         │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
Input Key-Value Pairs:
┌─────────────────────────────────────────────────────────────┐
│ (0, "Hello World")                                          │
│ (1, "Hello MapReduce")                                      │
│ (2, "World of Big Data")                                    │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼ Map Function
Intermediate Key-Value Pairs:
┌─────────────────────────────────────────────────────────────┐
│ ("Hello", 1), ("World", 1)                                 │
│ ("Hello", 1), ("MapReduce", 1)                             │
│ ("World", 1), ("of", 1), ("Big", 1), ("Data", 1)          │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼ Shuffle & Sort
Grouped Key-Value Pairs:
┌─────────────────────────────────────────────────────────────┐
│ ("Big", [1])                                                │
│ ("Data", [1])                                               │
│ ("Hello", [1, 1])                                           │
│ ("MapReduce", [1])                                          │
│ ("World", [1, 1])                                           │
│ ("of", [1])                                                 │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼ Reduce Function
Output Key-Value Pairs:
┌─────────────────────────────────────────────────────────────┐
│ ("Big", 1)                                                  │
│ ("Data", 1)                                                 │
│ ("Hello", 2)                                                │
│ ("MapReduce", 1)                                            │
│ ("World", 2)                                                │
│ ("of", 1)                                                   │
└─────────────────────────────────────────────────────────────┘
```

## Detailed Phase Analysis

### 1. Input Phase
The input phase prepares data for processing by splitting it into manageable chunks.

```javascript
// Input phase simulation
class InputPhase {
  constructor() {
    this.inputSplitSize = 64 * 1024 * 1024; // 64MB default
  }

  splitInput(inputData, splitSize = this.inputSplitSize) {
    const splits = [];
    
    // Split large files into smaller chunks
    for (let i = 0; i < inputData.length; i += splitSize) {
      const split = {
        splitId: Math.floor(i / splitSize),
        startOffset: i,
        length: Math.min(splitSize, inputData.length - i),
        data: inputData.slice(i, i + splitSize),
        location: this.getDataLocation(i) // For data locality
      };
      splits.push(split);
    }
    
    return splits;
  }

  getDataLocation(offset) {
    // Simulate data locality information
    const nodes = ['node1', 'node2', 'node3'];
    return nodes[Math.floor(offset / this.inputSplitSize) % nodes.length];
  }

  createInputRecords(split) {
    // Convert split into key-value pairs
    const records = [];
    const lines = split.data.split('\n');
    
    lines.forEach((line, index) => {
      if (line.trim()) {
        records.push({
          key: split.startOffset + index,
          value: line.trim()
        });
      }
    });
    
    return records;
  }
}

// Example usage
const inputProcessor = new InputPhase();
const sampleData = `Hello World
Hello MapReduce  
World of Big Data
MapReduce is powerful
Big Data processing`;

const splits = inputProcessor.splitInput(sampleData, 20);
console.log('Input splits:', splits.length);

splits.forEach((split, index) => {
  const records = inputProcessor.createInputRecords(split);
  console.log(`Split ${index} records:`, records);
});
```

### 2. Map Phase
The map phase applies a user-defined function to each input record independently.

```python
# Map phase implementation
class MapPhase:
    def __init__(self):
        self.mappers = []
    
    def map_function(self, key, value):
        """
        User-defined map function
        Input: (key, value) pair
        Output: List of (intermediate_key, intermediate_value) pairs
        """
        # Example: Word count mapper
        words = value.lower().split()
        result = []
        
        for word in words:
            # Clean word (remove punctuation)
            clean_word = ''.join(c for c in word if c.isalnum())
            if clean_word:
                result.append((clean_word, 1))
        
        return result
    
    def execute_map_tasks(self, input_splits):
        """Execute map tasks on all input splits"""
        intermediate_results = []
        
        for split_id, split in enumerate(input_splits):
            print(f"Processing split {split_id} on {split.get('location', 'unknown')}")
            
            split_results = []
            records = self.create_records_from_split(split)
            
            for record in records:
                # Apply map function to each record
                mapped_pairs = self.map_function(record['key'], record['value'])
                split_results.extend(mapped_pairs)
            
            intermediate_results.append({
                'split_id': split_id,
                'results': split_results,
                'location': split.get('location')
            })
        
        return intermediate_results
    
    def create_records_from_split(self, split):
        """Convert split data into key-value records"""
        records = []
        lines = split['data'].strip().split('\n')
        
        for i, line in enumerate(lines):
            if line.strip():
                records.append({
                    'key': split['startOffset'] + i,
                    'value': line.strip()
                })
        
        return records

# Example execution
mapper = MapPhase()

# Sample input splits
input_splits = [
    {
        'splitId': 0,
        'startOffset': 0,
        'data': 'Hello World\nHello MapReduce',
        'location': 'node1'
    },
    {
        'splitId': 1,
        'startOffset': 2,
        'data': 'World of Big Data\nMapReduce is powerful',
        'location': 'node2'
    }
]

# Execute map phase
map_results = mapper.execute_map_tasks(input_splits)

for result in map_results:
    print(f"Split {result['split_id']} results: {result['results']}")
```

### 3. Shuffle and Sort Phase
The shuffle and sort phase redistributes intermediate data and groups values by key.

```java
// Shuffle and Sort implementation (Java-style pseudocode)
public class ShuffleAndSort {
    
    public class IntermediatePair {
        String key;
        Object value;
        int partitionId;
        
        public IntermediatePair(String key, Object value) {
            this.key = key;
            this.value = value;
            this.partitionId = calculatePartition(key);
        }
    }
    
    // Partitioning function
    public int calculatePartition(String key) {
        int numReducers = 3; // Example: 3 reduce tasks
        return Math.abs(key.hashCode()) % numReducers;
    }
    
    // Shuffle phase: Partition and distribute intermediate data
    public Map<Integer, List<IntermediatePair>> shuffle(
            List<IntermediatePair> intermediateData) {
        
        Map<Integer, List<IntermediatePair>> partitions = new HashMap<>();
        
        // Initialize partitions
        for (int i = 0; i < 3; i++) {
            partitions.put(i, new ArrayList<>());
        }
        
        // Distribute data to partitions
        for (IntermediatePair pair : intermediateData) {
            int partition = pair.partitionId;
            partitions.get(partition).add(pair);
        }
        
        return partitions;
    }
    
    // Sort phase: Sort within each partition by key
    public Map<Integer, Map<String, List<Object>>> sortAndGroup(
            Map<Integer, List<IntermediatePair>> partitions) {
        
        Map<Integer, Map<String, List<Object>>> sortedPartitions = new HashMap<>();
        
        for (Map.Entry<Integer, List<IntermediatePair>> entry : partitions.entrySet()) {
            int partitionId = entry.getKey();
            List<IntermediatePair> pairs = entry.getValue();
            
            // Sort by key
            pairs.sort((a, b) -> a.key.compareTo(b.key));
            
            // Group values by key
            Map<String, List<Object>> groupedData = new LinkedHashMap<>();
            for (IntermediatePair pair : pairs) {
                groupedData.computeIfAbsent(pair.key, k -> new ArrayList<>())
                          .add(pair.value);
            }
            
            sortedPartitions.put(partitionId, groupedData);
        }
        
        return sortedPartitions;
    }
    
    // Network transfer simulation
    public void transferToReducers(Map<Integer, Map<String, List<Object>>> partitions) {
        for (Map.Entry<Integer, Map<String, List<Object>>> entry : partitions.entrySet()) {
            int reducerId = entry.getKey();
            Map<String, List<Object>> data = entry.getValue();
            
            System.out.println("Transferring to Reducer " + reducerId + ":");
            for (Map.Entry<String, List<Object>> keyValue : data.entrySet()) {
                System.out.println("  Key: " + keyValue.getKey() + 
                                 ", Values: " + keyValue.getValue());
            }
        }
    }
}
```

### 4. Reduce Phase
The reduce phase aggregates values for each unique key to produce final results.

```python
# Reduce phase implementation
class ReducePhase:
    def __init__(self):
        self.reducers = []
    
    def reduce_function(self, key, values):
        """
        User-defined reduce function
        Input: (key, list_of_values)
        Output: (key, aggregated_value)
        """
        # Example: Word count reducer - sum all counts
        total_count = sum(values)
        return (key, total_count)
    
    def execute_reduce_tasks(self, partitioned_data):
        """Execute reduce tasks on partitioned data"""
        final_results = []
        
        for partition_id, grouped_data in partitioned_data.items():
            print(f"Reducer {partition_id} processing...")
            
            partition_results = []
            for key, values in grouped_data.items():
                # Apply reduce function
                result = self.reduce_function(key, values)
                partition_results.append(result)
            
            final_results.extend(partition_results)
        
        return final_results
    
    def sort_final_results(self, results):
        """Sort final results for consistent output"""
        return sorted(results, key=lambda x: x[0])

# Complete MapReduce simulation
def simulate_complete_mapreduce():
    """Simulate complete MapReduce execution"""
    
    # Sample input data
    input_text = [
        "Hello World Hello",
        "MapReduce World Big Data", 
        "Hello Big Data Processing"
    ]
    
    print("=== INPUT DATA ===")
    for i, line in enumerate(input_text):
        print(f"Line {i}: {line}")
    
    # Map Phase
    print("\n=== MAP PHASE ===")
    intermediate_pairs = []
    
    for line_num, line in enumerate(input_text):
        words = line.lower().split()
        for word in words:
            pair = (word, 1)
            intermediate_pairs.append(pair)
            print(f"Map output: {pair}")
    
    # Shuffle and Sort Phase
    print("\n=== SHUFFLE & SORT PHASE ===")
    from collections import defaultdict
    
    # Group by key
    grouped_data = defaultdict(list)
    for key, value in intermediate_pairs:
        grouped_data[key].append(value)
    
    # Sort by key
    sorted_keys = sorted(grouped_data.keys())
    
    print("Grouped and sorted data:")
    for key in sorted_keys:
        values = grouped_data[key]
        print(f"Key: '{key}' -> Values: {values}")
    
    # Reduce Phase
    print("\n=== REDUCE PHASE ===")
    final_results = []
    
    for key in sorted_keys:
        values = grouped_data[key]
        total = sum(values)
        result = (key, total)
        final_results.append(result)
        print(f"Reduce output: {result}")
    
    print("\n=== FINAL RESULTS ===")
    for key, count in final_results:
        print(f"'{key}': {count}")
    
    return final_results

# Execute simulation
if __name__ == "__main__":
    results = simulate_complete_mapreduce()
```

## MapReduce Architecture Components

### 1. JobTracker (Master Node)
The JobTracker coordinates the entire MapReduce job execution.

```javascript
// JobTracker simulation
class JobTracker {
  constructor() {
    this.jobs = new Map();
    this.taskTrackers = new Map();
    this.jobCounter = 0;
  }

  submitJob(jobConfig) {
    const jobId = `job_${++this.jobCounter}`;
    
    const job = {
      id: jobId,
      config: jobConfig,
      status: 'SUBMITTED',
      mapTasks: [],
      reduceTasks: [],
      startTime: Date.now(),
      progress: {
        mapProgress: 0,
        reduceProgress: 0
      }
    };

    // Create map tasks based on input splits
    job.mapTasks = this.createMapTasks(jobConfig.inputSplits, jobId);
    
    // Create reduce tasks based on configuration
    job.reduceTasks = this.createReduceTasks(jobConfig.numReducers, jobId);
    
    this.jobs.set(jobId, job);
    
    // Schedule tasks
    this.scheduleJob(job);
    
    return jobId;
  }

  createMapTasks(inputSplits, jobId) {
    return inputSplits.map((split, index) => ({
      taskId: `${jobId}_map_${index}`,
      type: 'MAP',
      split: split,
      status: 'PENDING',
      assignedTracker: null,
      attempts: 0,
      preferredLocations: [split.location]
    }));
  }

  createReduceTasks(numReducers, jobId) {
    const tasks = [];
    for (let i = 0; i < numReducers; i++) {
      tasks.push({
        taskId: `${jobId}_reduce_${i}`,
        type: 'REDUCE',
        partitionId: i,
        status: 'PENDING',
        assignedTracker: null,
        attempts: 0
      });
    }
    return tasks;
  }

  scheduleJob(job) {
    console.log(`Scheduling job ${job.id}`);
    
    // Schedule map tasks first
    job.mapTasks.forEach(task => {
      this.scheduleTask(task);
    });
    
    job.status = 'RUNNING';
  }

  scheduleTask(task) {
    // Find best TaskTracker for the task
    const bestTracker = this.findBestTaskTracker(task);
    
    if (bestTracker) {
      task.assignedTracker = bestTracker.id;
      task.status = 'RUNNING';
      bestTracker.assignTask(task);
      
      console.log(`Assigned ${task.taskId} to ${bestTracker.id}`);
    } else {
      console.log(`No available TaskTracker for ${task.taskId}`);
    }
  }

  findBestTaskTracker(task) {
    let bestTracker = null;
    let bestScore = -1;

    for (const [trackerId, tracker] of this.taskTrackers) {
      if (tracker.isAvailable()) {
        let score = tracker.getCapacityScore();
        
        // Prefer local data for map tasks
        if (task.type === 'MAP' && 
            task.preferredLocations.includes(tracker.location)) {
          score += 10; // Locality bonus
        }
        
        if (score > bestScore) {
          bestScore = score;
          bestTracker = tracker;
        }
      }
    }

    return bestTracker;
  }

  handleTaskCompletion(taskId, success, output) {
    console.log(`Task ${taskId} completed: ${success ? 'SUCCESS' : 'FAILURE'}`);
    
    // Find the job and task
    for (const [jobId, job] of this.jobs) {
      const task = [...job.mapTasks, ...job.reduceTasks]
                    .find(t => t.taskId === taskId);
      
      if (task) {
        if (success) {
          task.status = 'COMPLETED';
          task.output = output;
          
          // Check if all map tasks are done before starting reduce tasks
          if (task.type === 'MAP') {
            this.checkMapCompletion(job);
          }
          
          // Check if job is complete
          this.checkJobCompletion(job);
        } else {
          this.handleTaskFailure(task, job);
        }
        break;
      }
    }
  }

  checkMapCompletion(job) {
    const allMapsDone = job.mapTasks.every(task => task.status === 'COMPLETED');
    
    if (allMapsDone && job.reduceTasks.every(task => task.status === 'PENDING')) {
      console.log(`All map tasks completed for job ${job.id}. Starting reduce tasks.`);
      
      // Schedule reduce tasks
      job.reduceTasks.forEach(task => {
        this.scheduleTask(task);
      });
    }
  }

  checkJobCompletion(job) {
    const allTasksDone = [...job.mapTasks, ...job.reduceTasks]
                          .every(task => task.status === 'COMPLETED');
    
    if (allTasksDone) {
      job.status = 'COMPLETED';
      job.endTime = Date.now();
      console.log(`Job ${job.id} completed in ${job.endTime - job.startTime}ms`);
    }
  }

  handleTaskFailure(task, job) {
    task.attempts++;
    
    if (task.attempts < 3) { // Max 3 attempts
      task.status = 'PENDING';
      task.assignedTracker = null;
      
      console.log(`Retrying task ${task.taskId} (attempt ${task.attempts})`);
      this.scheduleTask(task);
    } else {
      console.log(`Task ${task.taskId} failed after 3 attempts. Failing job.`);
      job.status = 'FAILED';
    }
  }
}
```

### 2. TaskTracker (Worker Node)
TaskTrackers execute individual map and reduce tasks.

```python
# TaskTracker implementation
import threading
import time
from queue import Queue

class TaskTracker:
    def __init__(self, tracker_id, location, max_tasks=4):
        self.id = tracker_id
        self.location = location
        self.max_tasks = max_tasks
        self.running_tasks = {}
        self.task_queue = Queue()
        self.job_tracker = None
        
        # Start task execution thread
        self.executor_thread = threading.Thread(target=self._task_executor)
        self.executor_thread.daemon = True
        self.executor_thread.start()
        
        # Start heartbeat thread
        self.heartbeat_thread = threading.Thread(target=self._send_heartbeat)
        self.heartbeat_thread.daemon = True
        self.heartbeat_thread.start()
    
    def assign_task(self, task):
        """Receive task assignment from JobTracker"""
        if len(self.running_tasks) < self.max_tasks:
            self.task_queue.put(task)
            return True
        return False
    
    def _task_executor(self):
        """Main task execution loop"""
        while True:
            try:
                task = self.task_queue.get(timeout=1)
                self._execute_task(task)
            except:
                continue
    
    def _execute_task(self, task):
        """Execute a single task"""
        print(f"TaskTracker {self.id} executing {task['taskId']}")
        
        self.running_tasks[task['taskId']] = task
        
        try:
            if task['type'] == 'MAP':
                result = self._execute_map_task(task)
            elif task['type'] == 'REDUCE':
                result = self._execute_reduce_task(task)
            
            # Simulate task execution time
            time.sleep(2)
            
            # Report success to JobTracker
            self.job_tracker.handle_task_completion(
                task['taskId'], True, result
            )
            
        except Exception as e:
            print(f"Task {task['taskId']} failed: {e}")
            self.job_tracker.handle_task_completion(
                task['taskId'], False, None
            )
        
        finally:
            del self.running_tasks[task['taskId']]
    
    def _execute_map_task(self, task):
        """Execute map task"""
        split = task['split']
        
        # Simulate map function execution
        intermediate_data = []
        
        # Process each record in the split
        for record in split['data'].split('\n'):
            if record.strip():
                words = record.lower().split()
                for word in words:
                    intermediate_data.append((word, 1))
        
        # Write intermediate data to local disk
        output_file = f"map_output_{task['taskId']}.tmp"
        self._write_intermediate_data(intermediate_data, output_file)
        
        return {
            'type': 'MAP_OUTPUT',
            'file': output_file,
            'records': len(intermediate_data)
        }
    
    def _execute_reduce_task(self, task):
        """Execute reduce task"""
        partition_id = task['partitionId']
        
        # Collect intermediate data from all map outputs
        intermediate_data = self._collect_intermediate_data(partition_id)
        
        # Group by key
        grouped_data = {}
        for key, value in intermediate_data:
            if key not in grouped_data:
                grouped_data[key] = []
            grouped_data[key].append(value)
        
        # Apply reduce function
        final_results = []
        for key, values in grouped_data.items():
            total = sum(values)
            final_results.append((key, total))
        
        # Write final output
        output_file = f"reduce_output_{partition_id}.txt"
        self._write_final_output(final_results, output_file)
        
        return {
            'type': 'REDUCE_OUTPUT',
            'file': output_file,
            'records': len(final_results)
        }
    
    def _write_intermediate_data(self, data, filename):
        """Write intermediate data to disk"""
        with open(filename, 'w') as f:
            for key, value in data:
                f.write(f"{key}\t{value}\n")
    
    def _collect_intermediate_data(self, partition_id):
        """Collect intermediate data for reduce task"""
        # In real implementation, this would fetch data from map outputs
        # For simulation, return sample data
        return [('hello', 1), ('world', 1), ('hello', 1)]
    
    def _write_final_output(self, results, filename):
        """Write final results to output file"""
        with open(filename, 'w') as f:
            for key, value in results:
                f.write(f"{key}\t{value}\n")
    
    def _send_heartbeat(self):
        """Send periodic heartbeat to JobTracker"""
        while True:
            if self.job_tracker:
                status = {
                    'tracker_id': self.id,
                    'location': self.location,
                    'running_tasks': len(self.running_tasks),
                    'available_slots': self.max_tasks - len(self.running_tasks),
                    'timestamp': time.time()
                }
                self.job_tracker.receive_heartbeat(status)
            
            time.sleep(10)  # Heartbeat every 10 seconds
    
    def is_available(self):
        """Check if TaskTracker can accept more tasks"""
        return len(self.running_tasks) < self.max_tasks
    
    def get_capacity_score(self):
        """Get capacity score for task scheduling"""
        available_slots = self.max_tasks - len(self.running_tasks)
        return available_slots / self.max_tasks
```

## Data Flow and Execution Model

### Complete Execution Timeline
```
MapReduce Job Execution Timeline:

T0: Job Submission
├── Parse job configuration
├── Validate input paths
└── Create job ID

T1: Input Splitting
├── Analyze input files
├── Calculate optimal split size
└── Create InputSplit objects

T2: Task Creation
├── Create Map tasks (one per split)
├── Create Reduce tasks (configurable number)
└── Initialize task tracking

T3: Map Task Scheduling
├── Find available TaskTrackers
├── Consider data locality
└── Assign tasks to workers

T4: Map Task Execution
├── Read input split
├── Apply map function
├── Write intermediate data
└── Report completion

T5: Shuffle and Sort
├── Partition intermediate data
├── Sort by key within partitions
├── Transfer data to reduce nodes
└── Merge sorted streams

T6: Reduce Task Scheduling
├── Wait for all maps to complete
├── Schedule reduce tasks
└── Assign to TaskTrackers

T7: Reduce Task Execution
├── Read partitioned data
├── Apply reduce function
├── Write final output
└── Report completion

T8: Job Completion
├── Verify all tasks completed
├── Clean up temporary files
└── Report final status
```

## Performance Considerations

### Optimization Strategies
```python
# Performance optimization techniques
class MapReduceOptimizer:
    
    @staticmethod
    def optimize_split_size(file_size, cluster_size, block_size=128*1024*1024):
        """Calculate optimal input split size"""
        
        # Rule of thumb: 1-4 splits per node
        target_splits = cluster_size * 2
        
        optimal_split_size = max(
            block_size,  # Don't go below block size
            file_size // target_splits
        )
        
        return optimal_split_size
    
    @staticmethod
    def optimize_reducer_count(intermediate_data_size, 
                             target_output_size=1*1024*1024*1024):
        """Calculate optimal number of reducers"""
        
        # Target: ~1GB output per reducer
        num_reducers = max(1, intermediate_data_size // target_output_size)
        
        # Ensure it's a reasonable number
        return min(num_reducers, 1000)
    
    @staticmethod
    def estimate_resource_requirements(input_size, complexity_factor=1.5):
        """Estimate resource requirements for job"""
        
        # Estimate intermediate data size
        intermediate_size = input_size * complexity_factor
        
        # Estimate memory requirements
        map_memory = min(1024, input_size // (100 * 1024 * 1024))  # MB
        reduce_memory = min(2048, intermediate_size // (50 * 1024 * 1024))  # MB
        
        # Estimate execution time
        estimated_time = (input_size // (100 * 1024 * 1024)) * 2  # minutes
        
        return {
            'intermediate_size': intermediate_size,
            'map_memory_mb': map_memory,
            'reduce_memory_mb': reduce_memory,
            'estimated_time_minutes': estimated_time
        }

# Example optimization
optimizer = MapReduceOptimizer()

# For a 10GB input file on 20-node cluster
file_size = 10 * 1024 * 1024 * 1024  # 10GB
cluster_size = 20

split_size = optimizer.optimize_split_size(file_size, cluster_size)
print(f"Optimal split size: {split_size // (1024*1024)}MB")

reducer_count = optimizer.optimize_reducer_count(file_size * 1.5)
print(f"Recommended reducers: {reducer_count}")

resources = optimizer.estimate_resource_requirements(file_size)
print(f"Resource estimates: {resources}")
```

## Common Patterns and Anti-Patterns

### Design Patterns
```java
// Common MapReduce design patterns

// 1. Filtering Pattern
public class FilteringMapper extends Mapper<LongWritable, Text, Text, Text> {
    @Override
    protected void map(LongWritable key, Text value, Context context) 
            throws IOException, InterruptedException {
        
        String line = value.toString();
        
        // Filter: Only process lines matching criteria
        if (line.contains("ERROR") || line.contains("WARN")) {
            context.write(new Text("alert"), value);
        }
    }
}

// 2. Aggregation Pattern
public class AggregationReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context)
            throws IOException, InterruptedException {
        
        int sum = 0;
        for (IntWritable value : values) {
            sum += value.get();
        }
        
        context.write(key, new IntWritable(sum));
    }
}

// 3. Join Pattern (Map-side join)
public class JoinMapper extends Mapper<LongWritable, Text, Text, Text> {
    private Map<String, String> lookupTable = new HashMap<>();
    
    @Override
    protected void setup(Context context) throws IOException {
        // Load small dataset into memory for joining
        loadLookupTable();
    }
    
    @Override
    protected void map(LongWritable key, Text value, Context context)
            throws IOException, InterruptedException {
        
        String[] fields = value.toString().split(",");
        String joinKey = fields[0];
        
        // Perform join
        String lookupValue = lookupTable.get(joinKey);
        if (lookupValue != null) {
            String joinedRecord = value.toString() + "," + lookupValue;
            context.write(new Text(joinKey), new Text(joinedRecord));
        }
    }
}
```

### Anti-Patterns to Avoid
```python
# Common MapReduce anti-patterns

class MapReduceAntiPatterns:
    
    def small_files_antipattern(self):
        """
        Anti-pattern: Processing many small files
        Problem: Creates too many map tasks, overhead dominates
        """
        # BAD: 1000 files of 1MB each
        small_files = [f"file_{i}.txt" for i in range(1000)]
        
        # GOOD: Combine small files into larger chunks
        combined_files = self.combine_small_files(small_files, target_size=128*1024*1024)
        return combined_files
    
    def reducer_hotspot_antipattern(self):
        """
        Anti-pattern: Skewed data causing reducer hotspots
        Problem: One reducer gets most of the data
        """
        # BAD: Natural key distribution is skewed
        def bad_partitioner(key):
            return hash(key) % num_reducers  # Some keys much more frequent
        
        # GOOD: Add randomization to balance load
        def good_partitioner(key):
            # Add random suffix to popular keys
            if self.is_popular_key(key):
                random_suffix = random.randint(0, 9)
                balanced_key = f"{key}_{random_suffix}"
                return hash(balanced_key) % num_reducers
            return hash(key) % num_reducers
    
    def unnecessary_reduce_antipattern(self):
        """
        Anti-pattern: Using reduce when map-only job would suffice
        Problem: Unnecessary shuffle and sort overhead
        """
        # BAD: Using reducer for simple filtering
        def bad_approach():
            # Map: Filter records
            # Reduce: Just pass through (unnecessary)
            pass
        
        # GOOD: Map-only job with no reducers
        def good_approach():
            # Map: Filter and output directly
            # No reduce phase needed
            pass
    
    def memory_intensive_antipattern(self):
        """
        Anti-pattern: Loading large datasets into memory
        Problem: OutOfMemoryError and poor performance
        """
        # BAD: Load entire dataset into memory
        def bad_mapper():
            large_lookup = {}  # Loads GBs of data
            # Process records...
        
        # GOOD: Stream processing or distributed cache
        def good_mapper():
            # Use distributed cache for lookup tables
            # Process records in streaming fashion
            pass

# Performance monitoring
class MapReduceMonitor:
    
    def analyze_job_performance(self, job_stats):
        """Analyze job performance and identify bottlenecks"""
        
        issues = []
        
        # Check for data skew
        if job_stats['max_reduce_time'] > job_stats['avg_reduce_time'] * 3:
            issues.append("Data skew detected in reduce phase")
        
        # Check for small files
        if job_stats['avg_split_size'] < 64 * 1024 * 1024:  # 64MB
            issues.append("Small input splits detected")
        
        # Check for excessive spilling
        if job_stats['spill_ratio'] > 0.3:  # 30%
            issues.append("Excessive spilling to disk")
        
        # Check for network bottlenecks
        if job_stats['shuffle_time'] > job_stats['total_time'] * 0.5:
            issues.append("Network bottleneck in shuffle phase")
        
        return issues
```

## Conclusion

Understanding MapReduce core concepts and architecture is fundamental for designing efficient distributed data processing solutions. Key takeaways include:

### Essential Concepts
1. **Key-Value Abstraction**: Everything operates on key-value pairs
2. **Functional Programming Model**: Map and reduce operations are side-effect free
3. **Automatic Parallelization**: Framework handles distribution and coordination
4. **Fault Tolerance**: Built-in recovery from hardware failures
5. **Data Locality**: Computation moves to data, not vice versa

### Architectural Components
- **JobTracker**: Coordinates job execution and task scheduling
- **TaskTracker**: Executes individual map and reduce tasks
- **Input/Output Formats**: Handle data serialization and storage
- **Partitioner**: Controls data distribution to reducers

### Performance Considerations
- Optimal split sizing for input data
- Appropriate number of reducers
- Data skew mitigation strategies
- Memory and disk usage optimization

In the next lesson, we'll dive into hands-on MapReduce programming, implementing real-world examples and exploring advanced programming patterns.
