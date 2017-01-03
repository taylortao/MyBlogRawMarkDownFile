---
title: How the performance of a web page is calculated
date: 2017-01-03 21:37:25
categories:
 - IT Technology
tags:
 - Performance
 - Measurement
---

There are two angles to measure a web page's performace data, one model is from browser's perspective, and the other is from server's perspective.
 
### Browser measured latency
For browser measured latencies, its overall diagram of web request is like below:

<!-- more -->

![W3C PLT model](browser1.png)

The timestamp used in Client side is implemented by pseudo-code like below

```html
<head>
st = new Date() /* baseline timestamp */
</head>
<body onload="onPageLoad()">
bst = new Date()
// Render html Page
bed = new Date()
 
function onPageLoad()
{
plst = new Date()
// execute script
pled = new Date()
}

</body>
```

![Browser side latency](browser2.png)

### Server measured latency

About instrumentation, the timestamp used in Server side could use a long type to store.
Below is an example of a web service based on chunked transfer encode (2 chunks to render).

![Server side latency](server.png)