---
title: Designing the new async-native ClickHouse Python client
url: https://clickhouse.com/blog/python-async-native-client
published: "2026-03-16T00:00:00Z"
feed: clickhouse
guid: https://clickhouse.com/blog/python-async-native-client
---

# Designing the new async-native ClickHouse Python client

clickhouse-connect v0.12.0 ships a ground-up async-native Python client using the half-sync/half-async pattern, delivering 1.16x better throughput and dramatically more stable tail latency under high concurrency.
