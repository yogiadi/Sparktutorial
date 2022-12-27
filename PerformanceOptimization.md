# Spark Hierarchy

Action -> Jobs -> Stages -> Tasks = Slots = Cores

1 Task 1 partition 1 Slot 1 Core

Spark UI is arranged as per hierarchy.

## Understand your hierarchy

* Core, Count and Speed
* Memory per core (Working & Storage)
* Local Disk Type, Count, Size and Speed
* Network Speed & Topology
* Data Lake properties (rate limits)
* Cost/Core/Hour

## Get a baseline

* Is your action efficient ?
  * Long stages, spills 
* CPU Utilization
  * Ganglia/Yarn etc.
  * Tails
  
## Minimize Data Scans(Lazy load)

* Data skipping
  * Hive partitions
  * Bucketing
* No Lazy loading
  * Stages running for very long time but 0 tasks completed.
  * Check quartiles in stages and spills.

## Partitions 

TBD
 
## Spark Partitions
 
* Input
  * Controls - Size
    * spark.default.parallelism(don't use)
    * spark.sql.files.maxPartitionBytes(mutable)
      - assuming source has sufficient partitions
* Shuffle
  - Controls = Count
    * spark.sql.shuffle.partitions
* Output
  - Control = Size
    * Coalesce(n) to shrink
    * Repartition(n) to increase and/or balance(shuffle)
    * df.write.option("maxRecordsPerFile",N)

Partitions - Shuffle - Default

Default = 200 Shuffle Partitions

Partitions - Right Sizing - Shuffle - Master Equation

* Largest Shuffle Stage
  * Target Size <= 200 MB/partition
  * Partition Count = Stage i/p data/Target Size
   * Solve for Partition Count

Example

Shuffle stage i/p = 210 gb

x = 210000 mb/200mb = 1050

spark.conf.set("spark.sql.shuffle.partitions", 1050)

BUT -> if cluster has 2000 cores

spark.conf.set("spark.sql.shuffle.partitions", 2000)


## Summary Metrics

Check summary metrics in stages.

Check for spills. 

Min, 25th, Median, 75th and Max should be consistent. 

Above is calculated by dividing "Shuffle Read"/200 partitions.

## Input Partitions - Right Sizing

* Use Spark Default(128 mb) unless
  * Increase parallelism
  * Heavily nested/repetitive data
  * Generating data i.e. Explode
  * Source structure is not optimal(upstream)
  * UDFs
  * spark.conf.set("spark.sql.files.maxPartitionBytes", 16777216)
 
## Output Partitions - Right Sizing

* Write once -> Read many
  * More time to write but faster to read
* Perfect writes limit parallelism
  * Compactions (minor and major)

## Output Partitions - Composition

* df.write.option("maxRecordsPerFile",n)
* df.coalesce(n).write
* df.repartition(n).write
* df.repartition(n, [ColA...]).write
* spark.sql.shuffle.partitions(n)
* df.localCheckpoint(..).repartition(n).write

## Partitions - Why so serious ?

* Avoid the spill
* Maximize parallelism
  * Utilize All Cores
  * Provision only the cores you need

## Advanced Optimizations

* Finding imbalance
  * Maximizing Resource requires balance
  * Task Duration
  * Partition size
* Skew
  * Check line of various stages in a job.
  * Check in Summary metrics - min, max and Median.
  * Minimized Data Scans
* Persisting
  * Cache only when required.
  * Unpersist once done.
  * Cache super set dataset.
* Join optimization
  * Sort Merge Joins(Standard)(Both sides are large)
  * Broadcast joins(Fastest)(One side is small)
  * Skew joins
  * Range joins
  * Broadcasted Nested Loop Joins
* Handling skews
  * Skew join optimization
    - Salting Technique - Add column to each side with random int between 0 and (spark.sql.shuffle.partitions - 1) to both sides.
* Expensive Operations
  * Repartition
    - Use Coalesce or shuffle partitions
  * Count
  * DistinctCount
  * If distincts are required.
* UDFs
* Multi-Dimensional Parallelism
  - Driver
  - Horizontal
  - Executor

# Physical Plans in Spark SQL

Logical Planning -> Physical Plan -> Execution(RDD)

Logical info doesn't contain precise information.
Logical Plan is converted to Physical Plan.

Logical Plan (Join) -> Physical Plan(Sort Merge join)
                    -> Physical Plan(Broadcast Hash join)
                    
df.queryExecution.sparkPlan
df.queryExecution.ExecutedPlan

#### Plan on Spark UI

== Parsed Logical Plan ==
== Analyzed Logical Plan ==
== Optimized Logical Plan ==
== Physical Plan ==

## Operators

- FileScan
  * Pair above numbers with info in Jobs and Stages tab in Spark UI for example-
    * Number of files read should be equal to number of Tasks.
    * Filesystem read data size total is equal to Input size.
    * Size of files read total.
    * Rows o/p.
  * FileScan(String Representation)
    - DataFilters
    - PartitionFilters
    - PushedFilters
    - SelectedBucket(bucket pruning)
    - Partition (Partition pruning)
- Exchange
  * Represents shuffle - physical data movement on the cluster which is quie expensive.
- HashAggregate, SortAggregate, ObjectHashAggregate
- SortMerge Join
  * Represents joining 2 dataframes.
  * Exchanges and sort often before Sort Merge join.
- BroadcastHash Join

## Ensure Requirements

TBD

## Reuse Exchange

TBD

## Final Performance Tuning

* Check for Spills in Stages with Summary metrics.
  - Change shuffle partitions.
  - Use Broadcast hash join
* Caching only when necessary
* Use CLUSTER BY when required.
* Pushed Predicate Filters.
* Connect Spark UI numbers with jobs and tasks.
