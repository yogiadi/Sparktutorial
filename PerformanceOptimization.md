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
