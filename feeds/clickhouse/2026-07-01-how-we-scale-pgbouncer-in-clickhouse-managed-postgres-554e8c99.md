---
title: How we scale PgBouncer in ClickHouse Managed Postgres
url: https://clickhouse.com/blog/pgbouncer-clickhouse-managed-postgres
published: "2026-07-01T00:00:00Z"
feed: clickhouse
guid: https://clickhouse.com/blog/pgbouncer-clickhouse-managed-postgres
---

# How we scale PgBouncer in ClickHouse Managed Postgres

PgBouncer is single-threaded, so a single process caps out at one CPU core no matter the box size. See how ClickHouse Managed Postgres runs a peered fleet of PgBouncer processes with so\_reuseport to scale pooling across every core — with benchmarks showin
