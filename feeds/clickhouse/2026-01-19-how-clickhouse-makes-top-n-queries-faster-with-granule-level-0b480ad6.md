---
title: How ClickHouse makes Top-N queries faster with granule-level data skipping
url: https://clickhouse.com/blog/clickhouse-top-n-queries-granule-level-data-skipping
published: "2026-01-19T00:00:00Z"
feed: clickhouse
guid: https://clickhouse.com/blog/clickhouse-top-n-queries-granule-level-data-skipping
---

# How ClickHouse makes Top-N queries faster with granule-level data skipping

A deep dive into how ClickHouse uses data-skipping metadata to turn Top-N queries into a pruning problem, cutting I/O and speeding up queries at scale.
