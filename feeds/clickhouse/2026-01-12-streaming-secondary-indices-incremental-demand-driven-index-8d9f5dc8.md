---
title: 'Streaming secondary indices: incremental, demand-driven index evaluation'
url: https://clickhouse.com/blog/streaming-secondary-indices
published: "2026-01-12T00:00:00Z"
feed: clickhouse
guid: https://clickhouse.com/blog/streaming-secondary-indices
---

# Streaming secondary indices: incremental, demand-driven index evaluation

ClickHouse now streams secondary index evaluation alongside data reads instead of scanning indexes upfront, making index usage incremental and demand-driven, reducing startup latency, unnecessary work, and memory usage.
