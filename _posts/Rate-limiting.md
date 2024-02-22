---
title: Rate limiting
date: 2021-12-30 22:37:25
categories:
 - IT Technology
tags:
 - SystemDesign
 - RateLimiting
---

### Rate limiting

- Token bucket
- Leaking bucket
- Fixed window counter
- Sliding window log
- Sliding window counter

<!-- more -->

![rate limiting](draft.png)

### A comparison between token bucket and leaky bucket

#### Token bucket

Can handle bursty traffic because the bucket can store tokens, allowing for temporary bursts of data as long as there are tokens in the bucket, rate of token addition and size of the bucket can be adjusted to control allowed bursty.

#### Leaky bucket
Requests are added to a queue/bucket, and they are released at a steady, constant rate. If the bucket is full, incoming requests are rejected.

It can be useful for smoothing out the traffic, and ideal for maintaining a uniform output rate. however overflow requests can result in rejection, which does not allow too much of traffic bursts.
