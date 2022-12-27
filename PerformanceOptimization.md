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
