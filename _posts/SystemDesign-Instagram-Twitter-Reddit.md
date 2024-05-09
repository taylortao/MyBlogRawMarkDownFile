---
title: System Design-Instagram/Twitter/Reddit
date: 2024-05-09 10:10:18
categories:
 - IT Technology
tags:
 - SystemDesign
---

# Requirement
- Load who I am following, and who follows me
- Load all post from a single user
- Low latency news feeds
- New post is visible to public or close friends  
- Nested comments

Read >> Write

<!-- more -->

# Capacity
## Load estimation
1 million daily active users (for cache size)
and cache 100 new feeds makes it 100 x 1 million = 100MB which could be easily fit on a single host.

## Storage estimation
one post = 100 charaters
100 bytes test + 100 bytes metadata
1 billion posts per day = 1 billion x 200 bytes x 365 = 73TB per year
avg(# of followers) = 100 with celebrity has millions
comments = 100 bytes test + 100 bytes metadata
comments per post = 1 million x 200 bytes = 200MB

# System interface

```
Query user info:
getUser(userId)

Query following relationship:
getFollowers(userId)
getFollowing(userId)

Posts:
sendPost(userId)

Load news feeds:
getNewsFeeds(userId)

Comments:
postComment(postId, comment)
```

### Data model
These tables are source of truth tables, higher data durability will be needed:

```
User table:
userId, some other user's metadata

UserFollower table:
userId, followerUserId, Security permission(public/close friend)

Posts table:
userId, Post

Comment table:
comment id, parent comment id, other comment properties
```

# High-level design / Detailed design

## Follower/Following
In order to support data locality search all followers and followings, we will be needing two tables:

```
UserFollower table: (who follows me, source of truth table)
userId, followerUserId

UserFollowing table: (who I am following, derived data, could be a in-memory Redis if data size is small)
userId, Follower
```

To maintain the consistency:
(1) Two phase commit will be very slow and not good fault tolerence since it needs both table available.
(2) Change data capture (CDC): the process of identifying and capturing changes made to data in a database and then delivering those changes in real-time to a downstream process or system.

```
UserFollower Table -> Kafka (distributed log) -> Flink (processer) -> derived data in UserFollowing.
```

For UserFollower:
- UserFollower is the source of truth.
- UserFollower take all writes on critical path (need to be fast: LSM-tree based)
- Write conflict is not an issue, for example, user 1 got follower A and B, requests hit different host, then at the end of the day we could merge (user1, A) and (user2, B) together into (user1, {A, B})
- Kafka could be partitioned based on user id.

## Load/Post news feed
The most native way is for read request: query UserFollowing cache first and then query Posts table one by one.

This is not ideal since the read latency will be very slow, in expedite the read requests, we want to have news feeds aggregated for all active users.
This means for each new post, we need to publish the same post to all their followers, assume average followers count is 100, then each post will make 100 copies.

Storage size for it:
Assume we have 1 billion / day x 200 bytes x 100(followers) = 20TB of storage per day.

And to make load news feed fast, if we put them all in memory:
20TB/256GB = 80 hosts (in-memory nodes) 

From Posts table to the aggregated feeds cache will be through an offline pipeline: (Push model in the eye of news feed reader)

```
Client -> Posts DB -> Stream -> Flink(Processor) -> Aggregated feed cache
```

Processor:
- Keep a local copy of all followers for a given poster, avoid making constant call to db.
- Partitioned by user id to have high concurrency and no write conflicts.
- Posting pipeline honors security level: whether they are close friends or not

There is a trade-off here: change security level would not impact historical posts and propagating permission change is not synchronous.

Posts DB:
- Write needs to be fast. 
- Partition by userid, Sort key is timestamp. so that query by user id ordered by timestamp will be fast (load recent posts from user X)
- Cassendra could be a good choice.

### Popular users
Assuming in the system we have less than 10K popular users, by nature they have millions of followers. 
And the above Posting workflow will be very slow when they publish a new post. 
In this case, actually we will know if this post will be a popular posts based on the follower count. 

So in this case, we will have another popular posts workflow: (Pull mode in the eye of news feed reader)

```
Popular User client -> Posts DB -> Stream -> Flink -> Popular posts cache

Popular posts cache:
user id, post
```

### Load news feeds
News feeds service will be like: 

```
Reader client X -> News Feed Service -> (Popular user ids X is following) -> (popular posts cache) 
                                     -> (News feed cache for user )
                                     -> Aggregate above results then sort and return
```

## Nest comments
Tree structure comments, let's assume comments size will be 200MB per post. (1 million x 200 characters/comment)

The main theme for comments are:
- Casually dependencies on comments. 
- Read >> Write
- Single leader replication for a single post: considering a case that if a client read comment "1" and publish a reply "11", then "11" makes no sense if "1" is not there. therefore, for each post, we would prefer a single leader replication.

And data structure to represent a tree would be: 

```
Comments table: (Sourse of Truth data)
Id, ParentId, comment
```

Two ways to implement:
(1) Native graph database, it has pointers on disk from node to node, which makes it faster than non native graph database.
However jumping from disk to disk is not that fast due to lack of data locality. data could be on different partitions.

(2) Use general relational database
To speed up the search, we could generate a DFS index like (similar thoughts from GeoHash):

Comments tree:

```
A
|----------|
AA         AB
|----|     |
AAA AAB    ABA
```

The comments are natually co-locate by the prefix and could be loaded in order with good data locality:

```
A
AA
AAA
AAB
AB
ABA
```

In terms of prefix, if we use `{0-9a-z}{4,4}` which makes `34^4=1.7millions` children per level.
The con of this approach is BFS(or other generalized queries) are not efficient. cause it is purely optimized for DFS search.

### Top comments (Derived data view)
The real world example is to sort by hottest or most controversial(which has lots of up and down votes) comments.

Two ways to implement this are:
(1) Denormalized graph based secondary index on disk

```
Comment-1
| (12) ----|(6)
2          3
| (5)
4
```

If another 3 people upvotes comment 3, it will add 6 and 3 make it a 9 upvotes.
It will maintain a sorted list of edges on write path. read will grab edges accordingly.

(2) Normalized graph based secondary index in memory (could be Redis)
Instead of holding the entire comments, will only store comment ids in the in-memory trees.
We could have a HashMap with key as comment id and value as edge.

### Synchronous update VS Derived data
For the secondary index update, we probably would not want to do it synchronously due to it might require two phase commits. 
And those data does not need to be perfectly consistent, all they need is eventually consistent or casually consistent.
So Derived data is better. now let's discuss casually consistent first:
(1) Partitioned by post id, all these comments/upvotes/downvotes will go to the same kafka shard.
Upvotes and downvots could have a high tps and overwhelm kafka partition.
(2) Wait for comments to propagation to all replicas and then we could shard upvotes/downvotes based on comment id.
(3) Have separate workflows for comments and votes

```
Comment DB -> Kafka(shard on postId) -> Flink(Consumer) -> Derived comments.
UpVotes/DownVotes DB -> Kafka(shard on postId+commentId) -> Flink(Consumer) -> Wait until comments is present before exporting votes to it -> Derived comments.

UpVotes/DownVotes DB
User,PostId,CommentId,Upvote/Downvote
```
