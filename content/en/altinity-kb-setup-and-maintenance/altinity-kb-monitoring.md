---
title: "ClickHouse® Monitoring"
linkTitle: "ClickHouse® Monitoring"
description: >
    Tracking potential issues in your cluster before they cause a critical error
keywords: 
  - clickhouse monitoring
  - clickhouse metrics
---

What to read / watch on the subject:
* Altinity webinar "ClickHouse Monitoring 101: What to monitor and how". [Watch the video](https://www.youtube.com/watch?v=W9KlehhgwLw) or [download the slides](https://www.slideshare.net/Altinity/clickhouse-monitoring-101-what-to-monitor-and-how).
* [The ClickHouse docs](https://clickhouse.com/docs/en/operations/monitoring/)

## What should be monitored

The following metrics should be collected / monitored

* For Host Machine:
  * CPU: saturation, load average, and iowait
  * Memory: pressure and available memory
  * Network: throughput, packets, errors, and drops
  * Storage: latency, throughput, IOPS, and queue depth
  * Disk Space: free / used

* For ClickHouse:
  * Query workload:
    * Connections and number of queries running
    * Query rate, query duration, and long-running queries
    * Read / Write / Return (bytes/rows)
    * Query read amplification: selected rows / bytes / marks / ranges / parts
  * Memory / cache / contention:
    * Cache hit rates: mark cache, query cache, and page / filesystem cache if used
  * Parts / background work:
    * Merges (queue length, memory used)
    * Mutations
    * Part growth, max parts per partition, and detached parts
  * Replication / distributed execution:
    * Replication queue length, lag, and failed fetch / check events
    * Read-only replicas
    * Keeper / ZooKeeper wait time on the ClickHouse side
    * Keeper / ZooKeeper client metrics on the ClickHouse side: in-flight requests, sessions / watches, operation rates by type, init / close churn, and exceptions
    * DDL queue length and Distributed tables backlog
  * Optional integrations:
    * S3 errors and remote-disk latency (if used)
    * Kafka consumer health (if used)

* For ClickHouse Keeper (if used):
  * Quorum / leader election stability, leader churn, and quorum uptime
  * Follower / observer sync, proposal size, and proposal / ack / commit / propagation latency
  * Outstanding requests and backlog in prep / sync / commit / final processing queues
  * Sessions, connection rejects / drops, and watch growth if your workload uses watches heavily
  * Fsync time / rate, snapshot time, open file descriptors, and other disk-pressure signals
  * TLS handshake or ensemble-auth failures if enabled
  * [See also clickhouse-keeper](../altinity-kb-zookeeper/clickhouse-keeper/)

* For ZooKeeper (if used):
  * Session health, outstanding requests, connection churn, and watch counts
  * Znode count / growth and approximate data size
  * Packets sent / received, leader election, quorum uptime, follower sync time, and request latency
  * Snapshot / fsync pressure, unrecoverable errors, and digest mismatches
  * JVM heap / GC / pause and thread health
  * [See separate article](../altinity-kb-zookeeper/zookeeper-monitoring/)


## ClickHouse monitoring tools

### ClickHouse internal dashboards

Built-in ClickHouse web dashboards are useful for local troubleshooting and ad hoc checks. Do not treat them as a replacement for production monitoring, alerting, retention, or access-control design. Do not expose these endpoints publicly.

* Advanced dashboard: `http://localhost:8123/dashboard`. Current ClickHouse docs describe it as a built-in dashboard for query rate, CPU, merges, reads, memory, inserts, and part counts. It is backed by rows from [`system.dashboards`](https://clickhouse.com/docs/operations/system-tables/dashboards) and mostly charts history from `system.metric_log` and `system.asynchronous_metric_log`; if those logs are disabled or empty, many graphs will be empty. See the upstream [monitoring docs](https://clickhouse.com/docs/operations/monitoring#built-in-advanced-observability-dashboard) and [advanced dashboard example](https://clickhouse.com/blog/common-issues-you-can-solve-using-advanced-monitoring-dashboards#how-to-get-started-with-the-advanced-dashboard).
* Custom dashboard definitions can be served by the same `/dashboard` page from any table with the same schema as `system.dashboards`. This is useful for local one-off panels, but keep long-term dashboards in your normal observability system.
* ClickStack UI: starting with ClickHouse 26.2, ClickStack / HyperDX is embedded in the ClickHouse binary at `http://localhost:8123/clickstack`. Use it to explore local logs, traces, metrics, or ClickHouse system tables. The embedded version is intended for local development and learning, not production deployments; it does not provide persistent state storage, alerting, or saved dashboard/query persistence. See [Introducing ClickStack embedded in ClickHouse](https://clickhouse.com/blog/clickstack-embedded-clickhouse).
* Keeper dashboard: `http://localhost:9182/dashboard`, only when `keeper_server.http_control.port` is enabled. The same HTTP control interface exposes commands and storage APIs, so restrict it with network controls. See [Keeper HTTP API and Dashboard](https://clickhouse.com/docs/operations/utilities/clickhouse-keeper-http-api).
* jemalloc UI: starting with ClickHouse 26.2, `http://localhost:8123/jemalloc` shows allocator statistics and can fetch heap profiles. Use it for allocation and memory debugging, not steady-state monitoring; jemalloc profiling can add overhead. See [allocation profiling](https://clickhouse.com/docs/operations/allocation-profiling#jemalloc-web-ui).

### Prometheus + Grafana

Use Prometheus for production monitoring and alerting. Scrape ClickHouse Server and ClickHouse Keeper as separate targets when Keeper is used.

* ClickHouse Server: enable the built-in [Prometheus endpoint](https://clickhouse.com/docs/en/operations/server-configuration-parameters/settings/#server_configuration_parameters-prometheus) in `clickhouse-server` config. It can expose metrics from `system.metrics`, `system.asynchronous_metrics`, `system.events`, and `system.errors`; newer versions can also expose histograms and dimensional metrics through the [Prometheus protocol handler](https://clickhouse.com/docs/interfaces/prometheus). Common dashboards: [14192](https://grafana.com/grafana/dashboards/14192) and [13500](https://grafana.com/grafana/dashboards/13500).
* ClickHouse Keeper: starting with ClickHouse 22.12, Keeper has its own Prometheus endpoint. Configure `prometheus.port` and `prometheus.endpoint` in the Keeper config and scrape every Keeper node; the release example uses port `9369` and `/metrics`. These are Keeper server metrics, not the same thing as ClickHouse Server metrics about ZooKeeper / Keeper client activity. See the [22.12 release note](https://clickhouse.com/blog/clickhouse-release-22-12#clickhouse-keeper---prometheus-endpoint-antonio-andelic).
* Altinity Kubernetes Operator: if ClickHouse is deployed by the operator, use the operator-managed metrics exporter, dashboards, and alerts. See the operator [Prometheus setup](https://github.com/Altinity/clickhouse-operator/blob/eb3fc4e28514d0d6ea25a40698205b02949bcf9d/docs/prometheus_setup.md), [Grafana setup](https://github.com/Altinity/clickhouse-operator/blob/eb3fc4e28514d0d6ea25a40698205b02949bcf9d/docs/grafana_setup.md), [dashboard](https://github.com/Altinity/clickhouse-operator/tree/master/grafana-dashboard), and [alert rules](https://github.com/Altinity/clickhouse-operator/blob/master/deploy/prometheus/prometheus-alert-rules-clickhouse.yaml).
* Operator-compatible metrics without the operator: if you do not run ClickHouse in Kubernetes but want to reuse the operator Grafana dashboard, expose a `FORMAT Prometheus` query through an HTTP handler. See [Compatibility layer for the Altinity Kubernetes Operator for ClickHouse](../monitoring-operator-exporter-compatibility/).
* Legacy external exporter: [clickhouse_exporter](https://github.com/ClickHouse/clickhouse_exporter) with dashboard [882](https://grafana.com/grafana/dashboards/882) exists, but is unmaintained. Prefer the built-in exporter or the operator exporter for new deployments.

### Grafana dashboards querying ClickHouse directly

Grafana can query ClickHouse directly through a ClickHouse datasource. This is useful for `system.query_log` analysis and ad hoc operational dashboards, but it is different from Prometheus monitoring: every refresh runs SQL on ClickHouse. Use a restricted read-only user, keep panels time-bounded, and avoid expensive high-cardinality queries on production clusters.

* Altinity / Vertamedia datasource: prefer [Altinity plugin for ClickHouse](https://grafana.com/grafana/plugins/vertamedia-clickhouse-datasource/) for new direct-query dashboards. It was initially developed by Vertamedia and has been maintained by Altinity since 2020. For modern Grafana use current 3.x versions; old pre-3.x versions were Angular-based. You can use the [operator queries dashboard](https://github.com/Altinity/clickhouse-operator/blob/master/grafana-dashboard/ClickHouse_Queries_dashboard.json) as a starting point.
* Official Grafana ClickHouse datasource: [Grafana ClickHouse datasource](https://grafana.com/grafana/plugins/grafana-clickhouse-datasource/) is an alternative when your Grafana stack standardizes on Grafana-maintained datasource plugins or needs its logs, traces, alerting, and OpenTelemetry-oriented workflows. Current plugin docs also list built-in dashboards for query, data, cluster, and OpenTelemetry analysis: [ClickHouse datasource docs](https://grafana.com/docs/plugins/grafana-clickhouse-datasource/latest/).
* Older direct-query dashboards: [ClickHouse Performance Monitor 13606](https://grafana.com/grafana/dashboards/13606) and [ClickHouse Queries 2515](https://grafana.com/grafana/dashboards/2515) query ClickHouse directly. Treat them as import examples to review and adapt, not drop-in production defaults. Dashboard 13606 states it was built for ClickHouse 20.8.7; dashboard 2515 depends on `system.query_log`.

### Other monitoring integrations

These are secondary paths. Prefer Prometheus/Grafana for production monitoring and the Altinity Grafana datasource plugin for new direct-query dashboards unless your environment already standardizes on one of these tools.

* Zabbix: use the official [Zabbix ClickHouse by HTTP template](https://www.zabbix.com/integrations/clickhouse) for current Zabbix deployments.
* Graphite-compatible pipelines: ClickHouse can push `system.metrics`, `system.events`, and `system.asynchronous_metrics` to Graphite with `<graphite>` in `config.xml`; multiple `<graphite>` sections are supported for different intervals. See the ClickHouse [Graphite configuration](https://clickhouse.com/docs/en/operations/server-configuration-parameters/settings/#server_configuration_parameters-graphite). Do not confuse this monitoring exporter with the [GraphiteMergeTree](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/graphitemergetree) table engine, which stores Graphite time-series data in ClickHouse.
* InfluxDB / Telegraf: for InfluxDB stacks, prefer the [Telegraf ClickHouse input plugin](https://docs.influxdata.com/telegraf/v1/input-plugins/clickhouse/) or scrape the ClickHouse Prometheus endpoint through Telegraf. The old InfluxDB v1 [Graphite protocol](https://docs.influxdata.com/influxdb/v1/supported_protocols/graphite/) path is mainly for legacy Graphite-compatible pipelines.
* Nagios / Icinga: keep these checks coarse: `/ping`, `/replicas_status`, host checks, and a small number of thresholded SQL checks. If you write custom plugins, follow the standard [Monitoring Plugins guidelines](https://www.monitoring-plugins.org/doc/guidelines.html) for return codes, thresholds, timeouts, and one-line output. Do not rely on unmaintained ClickHouse-specific plugins without reviewing them first.
* Commercial platforms: [Datadog](https://docs.datadoghq.com/integrations/clickhouse/?tab=host), [Sematext](https://sematext.com/docs/integration/clickhouse/), [IBM Instana](https://www.ibm.com/docs/en/instana-observability?topic=technologies-monitoring-clickhouse), [Site24x7](https://www.site24x7.com/plugins/clickhouse-monitoring.html), and [Acceldata Pulse](https://docs.acceldata.io/pulse/user-guide/clickhouse) have ClickHouse monitoring integrations or documented ClickHouse monitoring workflows. Validate exact metric coverage, ClickHouse version support, and ClickHouse Keeper coverage before relying on a vendor dashboard as the only monitoring source.

### "Build your own" ClickHouse monitoring

Use custom checks for smoke tests, Nagios / Icinga-style checks, or legacy monitoring systems. They are not a replacement for Prometheus / Grafana metric retention, dashboards, and alerting.

The HTTP examples assume the default HTTP interface on port `8123`; adjust the scheme, host, and port for HTTPS, load balancers, or non-default ports.

Enable rows for optional engines or Keeper-backed features only where those features are configured.

| Check | Query or endpoint | Alert when | Severity |
| --- | --- | --- | --- |
| Server availability | `GET /ping` | HTTP status is not `200` or response body is not `Ok.` | Critical |
| Concurrent queries | `SELECT value FROM system.metrics WHERE metric = 'Query'` | Value approaches your configured concurrency limits | Critical |
| Disk space / disk state | `SELECT name, free_space, unreserved_space, total_space, keep_free_space, is_read_only, is_broken FROM system.disks`<br>`SELECT metric, value FROM system.metrics WHERE metric IN ('BrokenDisks', 'ReadonlyDisks')` | Free or unreserved space is below your threshold, or a disk becomes read-only / broken | Critical |
| Insert / parts pressure | `SELECT metric, value FROM system.metrics WHERE metric = 'DelayedInserts'`<br>`SELECT event, value FROM system.events WHERE event = 'RejectedInserts'`<br>`SELECT metric, value FROM system.asynchronous_metrics WHERE metric IN ('MaxPartCountForPartition', 'TotalPartsOfMergeTreeTables')` | Inserts are delayed or rejected, or part counts keep growing | Critical |
| Async insert backlog | `SELECT count(), sum(total_bytes), min(first_update) FROM system.asynchronous_inserts`<br>`SELECT metric, value FROM system.metrics WHERE metric IN ('PendingAsyncInsert', 'AsynchronousInsertQueueSize', 'AsynchronousInsertQueueBytes', 'AsynchronousInsertThreadsScheduled')` | Queue size, bytes, or oldest pending async insert age keeps growing | High |
| Replicated table status | `GET /replicas_status?verbose=1` | HTTP status is not `200`, or verbose output reports a read-only replicated table. Current ClickHouse uses `min_absolute_delay_to_close` / `min_relative_delay_to_close` for the delay part of this handler, not `max_replica_delay_for_distributed_queries` | High |
| Read-only replicas | `SELECT value FROM system.metrics WHERE metric = 'ReadonlyReplica'` | Value is greater than `0` | High |
| Replicated fetches | `SELECT count(), max(elapsed) FROM system.replicated_fetches`<br>`SELECT value FROM system.metrics WHERE metric = 'ReplicatedFetch'` | Fetch count or max elapsed time is above your normal baseline | High |
| Distributed insert backlog | `SELECT metric, value FROM system.metrics WHERE metric IN ('DistributedFilesToInsert', 'DistributedBytesToInsert', 'BrokenDistributedFilesToInsert', 'BrokenDistributedBytesToInsert')` | Pending files / bytes keep growing, or broken files / bytes are greater than `0` | High |
| Distribution queue | `SELECT database, table, is_blocked, error_count, data_files, broken_data_files, last_exception FROM system.distribution_queue` | `is_blocked = 1`, errors / broken files are present, or queued files keep growing | High |
| Distributed DDL queue | `SELECT entry, host, status, exception_code, exception_text, query_create_time FROM system.distributed_ddl_queue WHERE status NOT IN ('Finished', 'Removing') OR (exception_code IS NOT NULL AND exception_code != 0)` | ON CLUSTER DDL is stuck or has non-zero exception codes | High |
| Replication queue stuck | `SELECT count() FROM system.replication_queue WHERE num_tries > 100 OR num_postponed > 1000` | Count is greater than `0`; see also [Replication queue](../altinity-kb-replication-queue/) | High |
| S3Queue failures | `SELECT count() FROM system.s3queue_metadata_cache WHERE status = 'Failed'` | Failed S3Queue files are present, or failed count keeps increasing | High |
| S3 disk / object storage errors | `SELECT metric, value FROM system.metrics WHERE metric IN ('DiskS3NoSuchKeyErrors', 'S3Requests')` | `DiskS3NoSuchKeyErrors` increases, or S3 requests stay elevated unexpectedly for your workload | High |
| Kafka consumers | `SELECT database, table, consumer_id, exceptions.time, exceptions.text, last_poll_time, last_commit_time, missing_dependencies FROM system.kafka_consumers`<br>`SELECT metric, value FROM system.metrics WHERE metric IN ('KafkaConsumers', 'KafkaConsumersInUse', 'KafkaConsumersWithAssignment', 'KafkaAssignedPartitions', 'KafkaBackgroundReads')` | Recent exceptions appear, polling / commits stop, dependencies are missing, or expected consumers / assignments disappear | High |
| Dictionary errors | `SELECT database, name, status, error_count, last_exception FROM system.dictionaries WHERE status IN ('FAILED', 'FAILED_AND_RELOADING') OR error_count > 0` | A dictionary failed to load / reload, or error count increases | High |
| Data loss or replica mismatch | `SELECT event, value FROM system.events WHERE event IN ('ReplicatedDataLoss', 'DataAfterMergeDiffersFromReplica', 'DataAfterMutationDiffersFromReplica')` | Any counter increases | High |
| Recent server exceptions in logs | `SELECT event_time, level, logger_name, message FROM system.text_log WHERE event_time >= now() - INTERVAL 5 MINUTE AND level IN ('Fatal', 'Critical', 'Error')` | New fatal / critical / error log messages appear; requires `system.text_log` to be enabled at a suitable level | High |
| Keeper / ZooKeeper client state | `SELECT metric, value FROM system.metrics WHERE metric IN ('ZooKeeperSession', 'ZooKeeperSessionExpired', 'ZooKeeperConnectionLossStartedTimestampSeconds', 'ZooKeeperRequest', 'ZooKeeperWatch')`<br>`SELECT event, value FROM system.events WHERE event IN ('ZooKeeperHardwareExceptions', 'ZooKeeperUserExceptions', 'ZooKeeperOtherExceptions')` | Connection-loss timestamp or expired sessions appear, sessions / watches move unexpectedly, or error counters increase | High |
| Error counters | `SELECT name, value, last_error_time, last_error_message FROM system.errors WHERE value > 0 AND last_error_time >= now() - INTERVAL 5 MINUTE ORDER BY last_error_time DESC` | Unexpected error counters increase; do not alert on every non-zero value without a baseline because some counters can increase during successful queries | Medium |
| Detached parts | `SELECT count() FROM system.detached_parts` | Count is greater than your normal baseline; use this table instead of filesystem globs when available | Medium |
| Restart detection | `SELECT value FROM system.asynchronous_metrics WHERE metric = 'Uptime'` | Uptime drops below the expected value | Medium |

For deeper dashboards or incident drill-downs, include these system tables as inspection sources:

* [`system.metrics`](https://clickhouse.com/docs/operations/system-tables/metrics): current counters for active server state, queues, background work, and integration-specific gauges.
* [`system.asynchronous_metrics`](https://clickhouse.com/docs/operations/system-tables/asynchronous_metrics): periodically refreshed metrics such as uptime, part counts, and disk usage.
* [`system.events`](https://clickhouse.com/docs/operations/system-tables/events): cumulative event counters, including insert rejections, replication data-loss events, and ZooKeeper / Keeper client exceptions.
* [`system.replicas`](https://clickhouse.com/docs/operations/system-tables/replicas): replicated table state, queue size, delay, and session status.
* [`system.merges`](https://clickhouse.com/docs/operations/system-tables/merges): currently running merges and progress.
* [`system.mutations`](https://clickhouse.com/docs/operations/system-tables/mutations): pending and running mutations.
* [`system.detached_parts`](https://clickhouse.com/docs/operations/system-tables/detached_parts): detached parts for MergeTree tables, including reason and disk path when available.
* [`system.asynchronous_inserts`](https://clickhouse.com/docs/operations/system-tables/asynchronous_inserts): pending async inserts in the server memory queue.
* [`system.kafka_consumers`](https://clickhouse.com/docs/operations/system-tables/kafka_consumers): Kafka consumer assignments, offsets, recent exceptions, and dependencies.

{{% alert title="Warning" color="warning" %}}
Scraped metrics are not a complete history. Short-lived states between scrapes can be missed.
{{% /alert %}}

Interpret these tables by signal type:

* `system.metrics` is a point-in-time view of current values. For example, `Query` is the number of queries running when the table is read.
* `system.asynchronous_metrics` is also a snapshot, but values are calculated periodically in the background.
* `system.events` contains cumulative counters since server start. Alert on deltas or rates between scrapes, not on the raw value alone, except for rare counters where any increase is meaningful.

If you need a full picture of query volume, latency, errors, or short-lived query spikes, use [`system.query_log`](https://clickhouse.com/docs/operations/system-tables/query_log) / [`system.query_thread_log`](https://clickhouse.com/docs/operations/system-tables/query_thread_log) in addition to scraped metrics.

## Monitoring ClickHouse logs

[ClickHouse logs](/altinity-kb-setup-and-maintenance/logging/) can be another important source of information. There are 2 logs enabled by default
* /var/log/clickhouse-server/clickhouse-server.err.log (error & warning, you may want to keep an eye on that or send it to some monitoring system)
* /var/log/clickhouse-server/clickhouse-server.log (trace logs,  very detailed, useful for debugging, usually too verbose to monitor).

The server log level is controlled by `logger.level` and optional per-output / per-logger overrides. In the upstream default config `logger.level` is `trace`, which is very verbose. `system.text_log` has its own `<text_log><level>` filter, but it only receives messages that already passed the server logger level. Setting `<text_log><level>trace</level>` will not recover trace / debug messages if the server logger is configured at `information`, `warning`, or another less verbose level. Valid levels include `fatal`, `critical`, `error`, `warning`, `notice`, `information`, `debug`, and `trace`. Allowing `trace` or `debug` in both places is useful for troubleshooting, but it can make `system.text_log` grow quickly.

Since ClickHouse 24.8, the upstream default config enables `system.text_log` with `<level>trace</level>`. In older versions, or in custom packages/configs, you may still need to enable it manually. Ensure that you will not expose sensitive log messages to users who should not see them.

{{% alert title="Warning" color="warning" %}}
With the default `trace` level, `system.text_log` can grow quickly. If you keep it enabled in production, set an appropriate `level`, `partition_by`, `order_by`, and `ttl`. Without a TTL, system log table growth is not bounded by retention.
{{% /alert %}}

Check the current volume before using `system.text_log` for monitoring:

```sql
SELECT
    level,
    count(),
    min(event_time),
    max(event_time)
FROM system.text_log
GROUP BY level
ORDER BY level;
```

Example configuration with fewer rows:

```
$ cat /etc/clickhouse-server/config.d/text_log.xml
<clickhouse>
    <text_log>
        <database>system</database>
        <table>text_log</table>
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
        <level>warning</level>
        <partition_by>toYYYYMM(event_date)</partition_by>
        <order_by>event_date, event_time, level, logger_name</order_by>
        <ttl>event_date + INTERVAL 30 DAY DELETE</ttl>
    </text_log>
</clickhouse>
```

## Other sources

* [OpenTelemetry support](https://clickhouse.com/docs/en/operations/opentelemetry/)
* [Monitor ClickHouse with Datadog](https://www.datadoghq.com/blog/monitor-clickhouse/)
* [Unsorted notes on monitor and Alerts](https://docs.google.com/spreadsheets/d/1K92yZr5slVQEvDglfZ88k_7bfsAKqahY9RPp_2tSdVU/edit#gid=521173956)
* [Tencent Cloud ClickHouse Monitoring Metrics](https://intl.cloud.tencent.com/document/product/1026/36887)
* [Tinybird experience (scroll to monitoring section)](https://www.tinybird.co/blog/what-i-learned-operating-clickhouse-part-ii)
