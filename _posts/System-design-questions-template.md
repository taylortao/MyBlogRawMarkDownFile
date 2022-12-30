---
title: System design questions template
date: 2020-01-18 20:08:00
categories:
 - IT Technology
tags:
 - SystemDesign
---

This article is about the process of defining archiecture, components, modules, interfaces and data for a system to satisfy specified requirement.
Also the whole processes could be shorten as SNAKE.

### Clarify the problem (Scenario)
Clarify the problem or in another word, scenarios and use cases (include corner cases).
Below is an example for a video streaming service:
 - Register/log in, play video, or video recommendation
 - Prioritize all the scenarios and deep dive into it.
 - Play video
    + (1) Generate channel
    + (2) Get all videos in the channel 
    + (3) Play videos in the channel

Later in micro design section, we could discuss concrete interface: input / output.

<!-- more -->

### Constraints or scale (Necessary)
This part is more about constraints and asssumptions.
For example:
 - Number of users, like daily active user(DAU)
 - Frequency of user actions, like:
     + Concurrent user TPS
     + Concurrent user count = DAU / seconds in a day x average online time
     + Peak concurrent user TPS
     + Future TPS, could be peak TPS x 3 (estimation)
 - Read/write ratio, read heavy application / write heavy application
 - How large the data set is
     + Memory: if per client, we need 10K memory, then the total memory would be peak TPS x 10K
     + Disk usage.
 - Network traffic
     + bandwidth: if per client we need 3mbps(Mbit/s), then peak network traffic would be peak TPS x 3mbps

### Abstract design (Application)
This part is the main components or core components
How the services talk with each other, what algorithm need to apply.
Also which part of data is the source of truth(authority), and which part of data is derived data.
we should be able to draw diagrams of components of your designed system and its connections and justify your design with use cases, constraints and corner cases.

Typical component design
 - Web gateway(load balancer)
 - Application service layer
 - Data cache
 - Data storage
 - Message queue, notification center
 - Logs, storage, analytics 
 - Search

### Data (Kilobit)
Two parts:
Data cache (reddis, memcached, cache key, LRU etc)
Data storage (DB, files, single db, master slave, sharding, partition, replication)

### Trade offs (Evolve)
Evaluate the whole design in terms of performance, reliability (robustness), scalability
 - Bottlenecks: most likely database, cache + master-slave + sharding
 - Monolithic: modular -> multiple processors + RPC -> service oriented 
 - Performance tuning: cache, asynchronous process (message queue)
