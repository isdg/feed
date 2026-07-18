---
title: How we scaled raw GROUP BY to 100 B+ rows in under a second
url: https://clickhouse.com/blog/clickhouse-parallel-replicas
published: "2025-09-29T00:00:00Z"
feed: clickhouse
guid: https://clickhouse.com/blog/clickhouse-parallel-replicas
---

# How we scaled raw GROUP BY to 100 B+ rows in under a second

ClickHouse Cloud now scales analytical queries with parallel replicas, fanning a single query across thousands of cores for terabyte-per-second throughput. This post dives into the internals and lets you see and feel the speed.
