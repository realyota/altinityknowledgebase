---
title: "Replication queue"
linkTitle: "Replication queue"
description: >
    Replication queue
---

Use this query to summarize `system.replication_queue` by table and task type when replication work starts to pile up.

Start with [`system.replicas`](https://clickhouse.com/docs/operations/system-tables/replicas) on the affected replicas to confirm whether the backlog is isolated to one replica or more widespread. Then use this query as a drill-down after the replication-queue monitoring check in [ClickHouse® Monitoring](../altinity-kb-monitoring/). For a broader troubleshooting workflow, see [Replication problems](../altinity-kb-check-replication-ddl-queue/).

```sql
SELECT
    database,
    table,
    type,
    max(last_exception),
    max(postpone_reason),
    min(create_time),
    max(last_attempt_time),
    max(last_postpone_time),
    max(num_postponed) AS max_postponed,
    max(num_tries) AS max_tries,
    min(num_tries) AS min_tries,
    countIf(last_exception != '') AS count_err,
    countIf(num_postponed > 0) AS count_postponed,
    countIf(is_currently_executing) AS count_executing,
    count() AS count_all
FROM system.replication_queue
GROUP BY
    database,
    table,
    type
ORDER BY count_all DESC
```
