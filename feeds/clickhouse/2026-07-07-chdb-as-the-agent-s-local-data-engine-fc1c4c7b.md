---
title: chDB as the Agent's Local Data Engine
url: https://clickhouse.com/blog/chdb-agents-local-data-engine
published: "2026-07-07T00:00:00Z"
feed: clickhouse
guid: https://clickhouse.com/blog/chdb-agents-local-data-engine
---

# chDB as the Agent's Local Data Engine

chDB embeds a full ClickHouse query engine inside an agent's own process, turning data access, memory, and federation into local function calls instead of network round trips, cutting the latency, retries, and token waste that come with remote queries.
