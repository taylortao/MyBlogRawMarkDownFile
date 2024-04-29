---
title: Distributed key value storage overview (Study notes)
date: 2024-04-26 13:37:29
categories:
 - IT Technology
tags:
 - Key value storage
---

# High level components
Will discuss through couple aspects, includes: Index, Replication, Failure detection and Consistency, the last parts will be some existing database example.


## Index
A database index is used for speeding up reads based on a specific key. Index will slow down database write and speed up read.

### Hash index
Hash index is kept in memory hash table of key mapped to the memory location of the data, occasionally write to disk for persistence. however, it works poorly on disk.

Pros: easy to implement and veryfast (RAM is fast)
Cons: all keys must fit in memory and it is bad for range queries.

Scenarios: it is fast but only useful on small datasets.

<!-- more -->

### SSTable and LSM Tree

SSTable: Sorted String Table
LSM Tree: Log structured merge tree.
WAL: Write-ahead Log

How it works?
Write first goes to an in-memory balanced binary search tree(also called MemTable) and when tree becomes too large, system writes contents (sorted by key name) to an SSTable.
To improve its persistency, it keeps logs on disk of all MemTable writes in WAL(Write ahead logs)
There is a compaction process, which works periodically to merge and compact SSTable. 

For read requests: it goes to MemTable first and then search from newest to oldest SSTable. for old records, read could be slow, to improve this, a sparse in-memory hash table is used. 

Pros: high write throughput due to writes goes to in-memory buffer. also it is good for range queries due to internal sorting of data in the index.
Cons: slow reads, especially for old key, and merge process of log segments can take up background resources.

### B-Tree

This type of index models the data to form a tree on disk.

For read requests: it traverses through the tree and find the value
For update requests: it traverses through the tree and change the value
For write requests: it traverses throught the tree, if there is extra space then add the key, otherwise you have to split the location block into two blocks then add the key, after that, update the parent block to reflect this action. the process can me made durable by using write ahead log.

Pros: relatively fast reads due to most B-trees can be stored in only 3-4 levels. it is good for range queries, as data is kept internally sorted.
Cons: relatively slow writes cause it has to write to disk as opposed to memory.

To guarantee durability and data integrity, each modification to the system is first written to an WAL on the disk.  WAL is always sequentially appended, so it is fast.

Segmented Log: as the file grows, it can also become a performance bottleneck, so we can split it into multiple parts. 

## Replication
Replication is the process of having multiple copies of data in order to make sure that if a database goes down the data wouldn't lost.  

Types:
(1) Single leader replication: all writes go to one instance, reads come from any instance
(2) Multi leader replication: writes can go to a small subset of leader instances and reads can come from any instance
(3) Leaderless replication: writes go to all instances, reads come from all instances (for example: quorum)

## Failure detection
Lease: a time-bound leases to grant clients rights on resources.

Split Brain: we cannot truly know if the leader has stopped for good or has experienced an intermittent failure like a stop-the-world GC pause or a temporary network disruption. if new leader is picked before original leader(zombie leader) has fully died, we have two active leaders.

Generation Clock: a monotonically increasing number to indicate a server's generation, which is used to solve split brain.

Fencing Tokens: Everytime the majority of nodes make a decision, assign it a monotonically increasing ID. and if the downstream service ever comes across a request with a lower fencing token value than the current one, reject it.

Fencing: Put a 'Fence' around the previous leader to prevent it from doing any damage or causing corruption.
(1) Resource fencing by blocking the previously active leader from accessing resources.
(2) Node fencing: by stoping the previously active leader from accessing all resources. 

## Consistency

### CAP theorem
Any distributed system needs to pick two out of the three properties:
(1) C: Consistency: all nodes see the same data at the same time. 
(2) A: Availability: requests received should result in a response even when network failures occur
(3) P: Partition tolerance: A partition is a communication break (or a network failure) between any two nodes in the system.

However since no point of only having CA, the theorem can really be: In the presence of a network partition, a distributed system must choose either Consistency or Availability.

- AP systems: DynamoDB, Cassandra, etc
- CP systems: BigTable, HBase

### PACELC theorem
This theorem is to solve what happens when there is no network partition? What choices does a distributed system have when there is no partition?

PACELC: 
If there is a partition ('P'), a distributed system can tradeoff between availability and consistency ('A' and 'C');
else ('E'), when the system is running normally without partitions, the system can tradeoff between latency ('L') and consistency ('C').

- PA/EL systems: DynamoDB, Cassandra, etc, which pick availability over consistency when a partition occurs; otherwise, pick lower latency.
- PC/EC systems: BigTable, HBase, etc. always pick consistency, give up availability and lower latency.
- PA/EC systems: default configuration of MongoDB. MongoDB works in a primary/secondaries configuration. For default configuration, replication is done asynchronously, when there is a network partition in which primary is becomes isolated, it is a loss of consistency, cause server returns success after primary persists data, but otherwise it guarantees consistency. Alternately, when MongoDB is configured to write and read on majority replicas, it could be categorized as PC/EC.

### Strong consistency
There are couple ways to implement strong consistency.
- Read from leader: all read must from leader.
- Sync: when write, it must be synced to all replicas.
- Quorum reads and writes.


# Examples

## Cassandra
Wide-column database for high read and write throughput, high availability, and it is optimized for single partition read and write.

Storage engine: LSM tree + SSTable. it uses bloom filter to optimize reading through SSTables.

Partition key works with consistent hashing to determine which replica to send data.

Replication:
Quorum based. write is sent to every single replicas after it reaches a minimum nodes server returns succeed to client. 

Conflict handling: last write wins. 

Read repair on reads: if client sees a replica with outdated value it will update it.

Anti-entropy process: it runs a background process to ensure eventual consistency of replicas.

Hinted handoff: when replica could not handle write, coordinator stores the write and send to them later after replica is back online.

Failure detection: failures are detected via gossip protocol and heartbeat.

Local cluster keys: denormalize data to keep multiple copies for different cluster key.

Use case: 
Write heavy applications and it is self contained, data only needs to be fetched with other data from its partition.
Such as: chat messages, user activity tracking, etc.

Pitfalls: 
Lack of strong consistency, lack of support of data relationships. lack of global secondary indexes.