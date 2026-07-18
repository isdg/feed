---
title: 'How we built fast UPDATEs for the ClickHouse column store – Part 1: Purpose-built engines'
url: https://clickhouse.com/blog/updates-in-clickhouse-1-purpose-built-engines
published: "2025-07-22T00:00:00Z"
feed: clickhouse
guid: https://clickhouse.com/blog/updates-in-clickhouse-1-purpose-built-engines
---

# How we built fast UPDATEs for the ClickHouse column store – Part 1: Purpose-built engines

ClickHouse is a column store, but that doesn’t mean updates are slow. In this post, we explore how purpose-built engines like ReplacingMergeTree deliver fast, efficient UPDATE-like behavior through smart insert semantics.
