---
title: How we configure huge pages in ClickHouse Managed Postgres
url: https://clickhouse.com/blog/huge-pages-clickhouse-managed-postgres
published: "2026-07-14T00:00:00Z"
feed: clickhouse
guid: https://clickhouse.com/blog/huge-pages-clickhouse-managed-postgres
---

# How we configure huge pages in ClickHouse Managed Postgres

How ClickHouse Managed Postgres reserves, enforces, and sizes huge pages so shared\_buffers avoids the per-connection page-table tax that eats RAM and stalls reads on 4KB pages.
