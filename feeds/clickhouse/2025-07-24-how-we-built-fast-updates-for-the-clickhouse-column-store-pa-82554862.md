---
title: 'How we built fast UPDATEs for the ClickHouse column store – Part 2: SQL-style UPDATEs'
url: https://clickhouse.com/blog/updates-in-clickhouse-2-sql-style-updates
published: "2025-07-24T00:00:00Z"
feed: clickhouse
guid: https://clickhouse.com/blog/updates-in-clickhouse-2-sql-style-updates
---

# How we built fast UPDATEs for the ClickHouse column store – Part 2: SQL-style UPDATEs

ClickHouse can do fast, declarative UPDATEs. Patch parts make it possible, with minimal I/O, instant query visibility, and high parallelism. This post breaks down the mechanics that make it all work.
