---
title: "Cutting P99 Latency by 60% on Our Search Service"
date: 2026-06-10
author: Manoj Kumar
category: "Performance"
excerpt_override: "How we traced a tail-latency problem to connection pooling and fixed it without adding a single server."
# image: /assets/img/search-latency.png   # optional — uncomment to add a hero
---

When our search API started missing its latency SLO, the averages looked fine — it
was the tail that hurt. This is a walkthrough of how we found the cause and the
change that brought P99 down from 820 ms to 320 ms.

## The symptom

Dashboards showed a healthy median (~45 ms) but a P99 that spiked unpredictably to
nearly a second. Tail latency like this almost never comes from CPU; it comes from
*waiting* — locks, queues, or pools.

## Finding the bottleneck

We added per-request spans around three stages: auth, query planning, and the
database call. The data was unambiguous:

| Stage          | Median | P99    |
|----------------|--------|--------|
| Auth           | 2 ms   | 4 ms   |
| Query planning | 8 ms   | 11 ms  |
| Database call  | 30 ms  | **790 ms** |

The database itself was fast. Requests were spending most of their time *waiting to
acquire a connection* from a pool that was sized for average load, not peak.

## The fix

Three changes, in order of impact:

1. **Right-sized the pool** based on `peak_concurrency × avg_hold_time`, not a guessed number.
2. **Added a short acquire timeout** so a saturated pool fails fast instead of queueing.
3. **Made slow queries observable** with a log line above 100 ms.

```python
pool = ConnectionPool(
    min_size=10,
    max_size=64,          # was 20
    timeout=0.25,         # fail fast instead of queueing
)
```

## Results

> P99 dropped from 820 ms to 320 ms within an hour of deploy — and stayed flat
> through the next traffic peak.

No new servers, no query rewrites. The lesson we keep relearning: **tail latency is
usually a queueing problem, not a compute problem.**
