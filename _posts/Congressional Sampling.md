---
title: Congressional data sampling overview
date: 2024-02-02 9:17:25
categories:
 - IT Technology
tags:
 - Sampling
---

### Why sampling?

Consider maintaining a highly concurrent service, and with 3k-5k requests per second hitting one server. this will generate a large number of request logs. Among all those request logs, normally data plane apis (data related) have a much larger volume than control plane apis (management related).

We want to understand how healthy the service is running, how healthy every apis are, note that a api with lower volume does not make it less important. 

How do we do that? when the request logs data is large, it is often advantageous to choose a smaller subset of data which could summarize the original dataset, this is called sampling. Main idea is to take a statistically significant sample of data and then analyse this sample rather than having to use the whole original data set.

By querying sampling data, the system is able to provide a efficient result which is approximate to the real answer.

<!-- more -->

### Requirement

An agent running on a host is collecting metrics as a sidecar process. and this agent dynamically adjust its sampling rate to attempt to keep the total number of request metrics below/close to a specified maximum count. 

This agent will report the metrics data together with a sample rate, so when analyzing, system is able to estimate what the real numbers are.

The classic queries on the dataset are:
- how healthy the service as a whole, by analyzing latency and fault rate.
- how healthy every apis this service provides, by analyzing latency, fault rate per api/group.

### How to sample?

We could consider the metrics for each api are grouped by api name, and per api analysis is like a group-by query on name field.

Main idea is from paper: "Congressional Samples for Approximate Answering of Group-By Queries". Acharya, Gibbons and Poosala in SIGMOD 2000

#### House sampling / Uniform sampling

For house sampling, each group gets an equal sampling rate (i.e. a data group that represents 50% of all incoming traffic makes up 50% of the sampled data)

Selection is done so that all the points of data have the same chance to be chosen to the sample.

This approach is simple and fast with O(n) complexity. it works perfectly with statistical data of whole data set.

However, key problem is per-api/group-by data. Given the data size difference between apis is large, control plane apis(management related) will not have many points of data been selected, results in per-api/group-by calculation will not be accurate for apis with lower sampled data. 

#### Senate sampling

For senate sampling, each group gets an equal percent of the output no matter what percent of the input it represents. 

The division is done by dividing the sample size by the number of groups.

This approach favors smaller groups of the data set, and smaller groups get disproportionately large amounts of
points of data into samples.

Senate sampling performs worse at query on overall statistical data (no group-by query).

#### Congressional Sampling

Congressional sampling is a biased sampling method, and it is a combination of both uniform/house and senate sampling. 

##### How it works? 

Congressional sampling considers all the possible groups in the dataset. It uses the maximum of the uniform/house and senate sampling rates. This will result in a sum across all sets that exceeds the requested maximum rate so the result for each set is scaled down to ensure the total is the requested rate.

Below is an example. 

Assume the maximum sample is 200 and there are 5 groups with
data size of A=800, B=50, C=50, D=50, E=50.

|Group|Original size|House|Senate|Congress|Finalized congress|
|--|--|--|--|--|--|
|A|800|160|40|160|100|
|B|50|10|40|40|25|
|C|50|10|40|40|25|
|D|50|10|40|40|25|
|E|50|10|40|40|25|
|Sum|1000|200|200|320|200|

### Summary
In a nutshell: 
- House sampling favours no group-by query. 
- Senate sampling favours smaller groups' group-by queries and performs poorly with no group-by query. 
- Congressional sampling performs better in all of the cases as it does not focus on any particular aspect.