---
title: Cache 101
date: 2022-07-25 16:16:25
categories:
 - IT Technology
tags:
 - SystemDesign
---

Cache helps on availability and resiliency by for example, improving request latency then service is more able to handle incoming traffic. as well as decrease load on downstream dependencies.

On the flip side, cache introduces modal behavior for your service, with differing behavior depending on whether a given object is cached. 

<!-- more -->


### When to use cache?
Two things to consider:
(1) are there any hot-keys/hot partitions? would it have a good cache hit ratio?
(2) could the system tolerate eventual consistency?

### Local cache vs. external cache
Depends on where the cache lives, local cache also called on-box cache. could be simply implemented as in-memory hash table.

Pros of local cache, it is easy and low risk with no operational overhead.
Cons of local cache, (1) inconsistent between hosts, client might get inconsistent results. (2) the traffic volume to down stream services will be proportional to the fleet size. (3) code start. (4) single host storage size limit.

External caches use a separate fleet to manage cache data. which could address cons of local cache, however brings its own cons around complexity, client needs to deal with cache fleet unavailabity case. also need to consider if cache fleet is down, how to prevent downstream services been brown out. (load shedding / cap maximum request rate). mass refresh is something very friendly to downstream.

### Inline vs. cache aside
Inline caches, embeded cache management into the main data access api.
Cache aside(or side caches), client code acts as transaction manager and manipulates the cache before or after calls to the data source. 

### Different type of caches

#### Read-through vs. Refresh-ahead

Read-through: application for a cache miss, then querying the database, populating the cache, and continuing application processing. In this case, application is resilient to cache failures with performance penalty.

Refresh-ahead: application automatically and asynchronously reload recently accessed cache entry before its expiration. The asynchronous refresh is triggered when an object that is sufficiently close to its expiration time.

![read cache](readcache.png)

#### Write-through, write-behind vs. write-around

Write-through: when the application updates data in the cache, the operation will not complete both cache and database have the updated value. which induces extra write latency, however paired with Read-Through, it has better consistency guarantee.

Write-behind: confirm to client right after write to cache, only write to db after specific intervals or certain conditions met. it will have data loss in case of crash.

Write-around: write to db directly.

![write cache](writecache.png)

### Cache expiration
There are multiple ways of cache expiration

(0) TTL based.
(1) Fifo
(2) Lifo
(3) LRU: least recently used.
(4) MRU: most recently used.
(5) LFU: least frequently used.
(6) RR: random replacement.

Real-world traffic patterns can differ from what modeled so it is important to track the actual performance of cache. For example, by emitting metrics on cache hits and misses, total cache size and number of requests to downstream services.

To protect downstream service brown out as well as resiliency when downstream services are unavailable is to use two TTLs: a soft TTL and a hard TTL. Cache should be refreshed on Soft TTL expiry. But, if downstream service does not respond when cache is refreshed, service could continue to serve the stale data until the hard TTL.

### Caching penetration: negative cache vs. bloom filter

Issue: data that cannot be found from either the cache or the database that has been requested in huge volume. 

To solve this problem:

(1) Cache keys with null value. Set a short TTL (Time to Live) for keys with null value.  
(2) Using Bloom filter. only if the key exists, the request first goes to the cache/database.

Negative cache could also extend to error case to protect downstream services.

### when deploy a cache, the service becomes addicted to it
Last thing is when deploy a cache, the service becomes addicted to it. for example: dependencies notice reduced call volume, and scale down and lower quata over time. Service serves more traffic because its latency and CPU utilization goes down. Therefore if your cache breaks, there will be a high blast radius outage. we need to take cache seriously.
