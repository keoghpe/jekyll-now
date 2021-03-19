---
layout: post
title: Understanding memory
---

## Memory basics with PS

We can run PS with the -m option to order our processes by memory. 

We can get the top 10 process with: 

```
ps aux -m | head -n 11
```

RSS - resident set size - How much memory allocated and in RAM. Doesn't include swap.
VSZ - virtual size - Includes all memory that process can access - swap, in use and allocated.


https://stackoverflow.com/questions/7880784/what-is-rss-and-vsz-in-linux-memory-management

## Paging

What on earth is paging?

## Valgrind?

## Read book on operating systems
