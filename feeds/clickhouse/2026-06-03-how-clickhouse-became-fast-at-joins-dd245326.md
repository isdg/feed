---
title: How ClickHouse became fast at joins
url: https://clickhouse.com/blog/clickhouse-fast-joins
published: "2026-06-03T00:00:00Z"
feed: clickhouse
guid: https://clickhouse.com/blog/clickhouse-fast-joins
---

# How ClickHouse became fast at joins

Over two years of focused join engineering, ClickHouse became 26× faster on the TPC-H SF100 join-heavy workload. Here’s how parallel hash joins, runtime filters, lazy column replication, and smarter join planning got us there.
