---
title: How to choose data storage (Study notes)
date: 2024-05-04 11:17:49
categories:
 - IT Technology
tags:
 - Database
 - Storage
 - SystemDesign
---

There are lots of different options for databases. How can we decide which one to choose? There are couple aspects to consider.
- (1) consistency level: ACID or BASE
- (2) index: B-tree or LSM-tree
- (3) replication

<!-- more -->

# Database Consistency Models: ACID vs BASE 
## ACID
ACID - most relational database supports it
Transaction - an abstraction that provide ACID guarantees 

### Atomicity
each write in the transaction will either be committed or aborted entirely

### Consistency
a transaction brings the database from one valid state to another valid state.

### Isolation
concurrent transactions do not interfere with each other(serializability), each transaction can pretend it is the only one running on the database which has performance penalty

### Durability
Once a transaction is committed, it remains so, data will be on disk.

### Ways to implement Serializable Isolation

#### Actual Serial Execution
Use a single thread for all database queries with limitation of a single CPU core and all data needs to be on single partition.

#### Two Phase Locking - pessimistic
Each object has a lock, either shared mode or exclusive mode.
- Shared lock(read lock): multiple transactions can concurrently read from a row.
- Exclusive lock(write lock): no other transactions are currently holding a shared or exclusive lock.

we also needs predicate lock for rows that does not exist. predicate lock performs poorly due to going through a bunch of rows. 

Two phase locking performance is bad due to deadlocks. database must detect when transactions are having a deadlock, and abort one of them, let the other finish, and then retry the aborted one.

A dead lock scenario. both transactions try to update single row, T1 grab read lock, T2 grab read lock, now both transactions could not acquire write lock, which forms a deadlock.

#### Serializable Snapshot Isolation - optimistic
Transactions run concurrently based on snapshot of the database and revert a transaction if a concurrency issue detected.
Keep track of which transactions have read an item, if another transaction writes to the item, abort all of the transactions that read it.

### Weaker isolation
Serializable Isolation comes at a very high performance cost, and many databases have chosen to implement weaker forms of isolation.

#### Concurrency bug - Dirty Reads
Clients reads uncommitted data which might gets rolled back later on.
Solution: Database remembers old value of a write until write request is committed, no lock needed.

#### Concurrency bug - Dirty Writes
When writing data, we can only overwrite committed data.
Solution: Have a lock on the object, transaction must grab the lock first.

#### Solution - Read committed 
Read committed isolation prevents dirty reads and dirty writes. (weakest isolation).

#### Concurrency bug - Read skew (Non-repeatable reads)
When making multiple reads within a transaction, while throughout the course of those reads, database changes and client sees the database in an inconsistent state.

This is problematic for things like analytics queries, as the data will make no sense if some pieces of it are more or less updated than others.

Solution: Snapshot Isolation
- (1) Every transaction is assigned an increasing transaction ID
- (2) When writing a value, the transaction ID is also saved
- (3) When a transaction performs a read, it takes the value with the highest transaction ID that is less than the reader ID.

For example: if reader's transaction id is 13, then all updates/writes after transaction id 13 won't be seen.

#### Concurrency bug - Lost Updates
When two threads read an object, perform an operation on it, and then write it back, one of the threads’ update may be lost.

```
T1 and T2:
read balance = 100

T1 and T2: 
update balance' = balance + 100

then after them:
balance = 200. (we lost 100)
```

Solution:
(1) use exclusive locks to do atomic write.
(2) locks in application code, bug prone.
(3) snapshot isolation, the 2nd transaction could see the value that it was about to write has since changed, it will rollback and retry (Recommended)

These techniques do not work in multileader/leaderless replication, as they assume one copy of the data, instead better to store conflicts as siblings and use custom resolution logic to solve.

#### Solution - Snapshot Isolation
Prevents read skew (non-repeatable read), detect lost updates

#### Concurrency bug - Write Skew
Two transactions each read the same set of objects in order to make a decision on whether to make a write, and then the write itself will break the invariant.

For example: a table has three seat.
both T1 and T2 will select people on table, it returns P1 and P2.
then both T1 and T2 inserted a different people, P3 and P4 which break the invariant that at maximum 3 people are allowed.

Solution: lock on all the rows that you read to form the predicate, and then only one transaction can read them at a time.

#### Concurrency bug - Phantoms
What if those rows don’t all yet exist? for example this table could only one people. then invariants are broken when both transactions create a new row, and there is nothing to put a lock on!

Solution: materializing conflicts: create a blank row prematurely, so that transactions can apply a lock to it.

#### Solution - Predicate Lock and Materializing Conflict
used to prevent write skew and phantoms

## BASE
BasicallyAvailable: system is available most of the time.
Soft State: state of the system may change over time, data consistency is developer's problem and would not be handled by database.
Eventually Consistent: the system’s updated state is gradually replicated across all nodes.

## Summary
ACID: transactions must be reliable and consistent. correctness is of more importance than speed. 
BASE: high availability and scalability are necessary, and some degree of data inconsistency is acceptable.

ACID use cases: banking applications, financial systems, job scheduling

# Database index
Index is used for the purpose of speeding up reads.

(1) LSM Trees + SSTables
Writes first go to a balanced binary search tree in memory
Tree flushed to a sorted table on disk when it gets too big
Can binary search SSTables for the value of a key
If there are too many SSTables they can be merged together (old values of keys will be discarded) 

Pro: Fast writes to memory and WAL
Con: Might have to search many SSTables for value of key, thus slow read

(2) B-Trees
A binary tree using pointers on disk
Writes iterate through the binary tree and either overwrite the existing key value or create a new page on disk and modify the parent pointer to the new page

# Replication
Replication is the process of having multiple copies of data in order to be more reliable.

(1) Single leader replication: all writes go to one database, reads come from any database
(2) Multi leader replication: writes can go to a small subset of leader databases reads can come from any database
(3) Leaderless replication: writes go to all databases, reads come from all databases

Single leader replication pro:
it is easy to ensure that there are no data conflicts due to all writes will go to one node

Leaderless and multileader replication pro: 
supports higher write throughput beyond just one database node (at the cost of potential write conflicts)

# Database types and key features
## Relational database

Relational database: mysql, oracle, sql server, postgres

Key features:
(1) Fixed schema, columns must be decided and chosen and it can be altered later, but it involves modifying the whole database and potentially going offline.
(2) Relational/Normalized data - changes to one table may require changes to others, for example: adding an author and their books to different tables on different nodes, this will require two phase commit, which is expensive and with performance penalty
(3) Could guarantee ACID, which makes query excessively slow if you don’t need them due to two phase locking etc.
(4) typically use B-trees as indexes

Reasons to use SQL database:
(1) when correctness is of more importance than speed and scalability
for example: e-commerce, bank and financial applications

Reasons to not use SQL database:
(1) Vertical scaling by upgrading to better hardware is pricy.
(2) Horizontal scaling by dding more nodes is hard. why hard?
The way to support horizontal scaling is to shard/partition data, and it is hard.
For example transfer money between two customers, and they are on different partitions, we need distributed transactions which is hard to implement and also two phases locking over network would be very slow.

## Document data model (NoSQL)
Document database: mongodb, riak, couchbase, rethinkdb

Key features:
(1) data is written in large nested documents and data is denormalized
(2) each document can have an entirely different structure. 
(2) data has better data locality 

## Key value database (NoSQL)
Key-Value Stores: Data is stored in an array of key-value pairs, include Redis, Voldemort, DynamoDb and LevelDb.

## Wide-Column database (NoSQL)
Wide colume database supports things like nested-key/value, it includes Apache Cassandra, hbase, etc.
It normally has a shard key and a sort key and allows for flexible schemas.
Great for applications with high write volume, consistency is not as important, all writes and reads go to the same shard (no transactions supported)

Apache Cassandra / Riak
Multileader/Leaderless replication (configurable)
Index: LSM tree and SSTables (Super fast writes)
Conflict resolution: 
- Apache Cassandra: last write win.
- Riak: supports CRDTs (conflict free replicated data types), it allows for implementing counters and sets in a conflict free way, custom code to handle conflicts
Use case: chat application etc.

Apache HBase:
Single leader replication
Index: LSM tree and SSTables (Fast writes)
Column oriented storage: column compression and increased data locality within columns of data
Use case: multiple thumbnails of a youtube video, sensor readings etc

## Graph database
Graph Databases are used to store data whose relations are best represented in a graph, which has nodes (entities), properties (information about the entities), and edges (connections between the entities). 
Examples: Neo4j, Titan and InfiniteGraph

Key Features:
Data is represented as nodes and edges, which has pointers from one address on disk to another for quicker lookups
Comparing with using other db index, it needs O(log n) to binary search, but using direct pointers is O(1)

Use cases:
For data naturally represented in graph formats, for example: map data, friends on social media

## Text search database
Optimized for storing and searching against text, includes: elasticsearch, solr, lucene. their index is based on inverted Index.

## In-memory database (Cache)
Key value stores implemented in memory and it uses a hashmap under the hood.
Which includes: Memcached and Redis, Redis has richer feature set.

Useful for data that needs to be written and retrieved extremely quickly, memory is expensive so good for small datasets
For example caches, geo spatial index.

## Time series database
Use LSM trees for fast ingestion, but break table into many small indexes by both ingestion source and timestamp
By putting the whole index in cache for better performance
Quick deletion of whole index when no longer relevant (as opposed to typical tombstone)
Timestamps should be sequential and similar and should be able to be compressed
Column based, having all column values in one file reduces disk I/O and allows faster aggregations of column data, generally we only need one or two columns at a time for graphing

It includes: TimeScaleDB, Apache Druid
It could be used for: sensor data, metrics, etc.

## New SQL
Still has SQL syntax but to improve more on scalability.

VoltDb:
SQL but completely in memory, single threaded execution for no locking, it is expensive and only allows for small datasets

Spanner:
SQL, uses GPS clocks in data center to avoid locking by using timestamps to determine order of writes which is very expensive
