---
title: System design questions template
date: 2020-01-18 20:08:00
categories:
 - IT Technology
tags:
 - SystemDesign
---

This article is about the process of defining archiecture, components, modules, interfaces and data for a system to satisfy specified requirement, the structure mainly used for system design interview.

### Clarify the problem (Scenario)
Clarify the problem or in another word, scenarios and use cases (include corner cases).
Below is an example for a video streaming service:
 - Register/log in, play video, or video recommendation
 - Prioritize all the scenarios and deep dive into it.
 - Play video
    + (1) Generate channel
    + (2) Get all videos in the channel 
    + (3) Play videos in the channel

Requirements include functional and non-functional requirements. non-functional requirements includes: scalability, performance, availability, security.  

Due to the limit of time, we should prioritize and identify which ones are critical. and for non-functional, we could discuss some trade-offs. For example, a system highly optimized for read operations might have slower write operations.

<!-- more -->

### Constraints or scale (Necessary)
This part is more about constraints and asssumptions, we could follow this category: 

(1) Load estimation
expected number of requests per second, throughput per second, or user traffic for the system.

(2) Storage estimation
amount of storage required to handle

(3) Bandwidth estimation
determine the network bandwidth needed to support the expected traffic and data transfer.
single host xx Gigabit/s

(4) Latency estimation
predict the response time and latency of the system based on architecture and components.

(5) Resource estimation
estimate the number of servers, CPUs, or memory required.

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

### System interface
Mainly to discuss concrete interface: input / output.

for example:

```
postTweet(user_id, tweet_data, tweet_location, user_location, timestamp, …)  
```

### Data model
What is data structure?
How data will flow between different system components.
This will help future discussion around data partition, storage, transportation, encryption etc.

for example:

```
Tweet: TweetID, Content, TweetLocation, NumberOfLikes, TimeStamp, etc.
```

### High-level design / Detailed design
This part is the main components or core components
How the services talk with each other, what algorithm need to apply.
Also which part of data is the source of truth(authority), and which part of data is derived data.
we should be able to draw diagrams of components of your designed system and its connections and justify your design with use cases, constraints and corner cases.

Typical component design
 - load balancer
 - Web gateway / API gateway / Web server
 - Application service layer
 - Data cache
 - Data storage
 - Message queue, notification center
 - Logs, storage, analytics 
 - Search

Data parts:
Data cache (reddis, memcached, cache key, LRU etc)
Data storage (DB, files, single db, master slave, sharding, partition, replication)

### Trade offs (Evolve / Bottleneck)
Evaluate the whole design in terms of performance, reliability (robustness), scalability
 - Bottlenecks: most likely database, cache + master-slave + sharding
 - Monolithic: modular -> multiple processors + RPC -> service oriented 
 - Performance tuning: cache, asynchronous process (message queue)
