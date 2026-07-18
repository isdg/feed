---
title: ClickHouse projections now behave like true secondary indexes
url: https://clickhouse.com/blog/projections-secondary-indices
published: "2026-01-13T00:00:00Z"
feed: clickhouse
guid: https://clickhouse.com/blog/projections-secondary-indices
---

# ClickHouse projections now behave like true secondary indexes

ClickHouse now supports granule-level pruning for lightweight projections, enabling them to act as true secondary indexes that work together to dramatically speed up multi-filter queries without duplicating full table data.
