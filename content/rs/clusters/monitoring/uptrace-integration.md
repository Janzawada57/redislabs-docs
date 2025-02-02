---
Title: Uptrace integration with Redis Enterprise Software
linkTitle: Uptrace integration
description: To collect, display, and monitor metrics data from your databases and other cluster components, you can connect Uptrace to your Redis Enterprise cluster using OpenTelemetry Collector.
weight: 50
alwaysopen: false
categories: ["RS"]
---

To collect, display, and monitor metrics data from your databases and other cluster components, you can connect Uptrace to your Redis Enterprise cluster using OpenTelemetry Collector.

[Uptrace](https://uptrace.dev/get/) is an open source application performance monitoring (APM) tool that supports distributed tracing, metrics, and logs. You can use it to monitor applications and set up automatic alerts to receive notifications.

With [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/), you can receive, process, and export telemetry data to any monitoring tool. You can use it to scrape Prometheus metrics provided by Redis and then export those metrics to Uptrace.

You can use Uptrace to:

- Collect and display data metrics not available in the [admin console]({{< relref "/rs/references/metrics" >}}).
- Use prebuilt dashboard templates maintained by the Uptrace community.
- Set up automatic alerts and receive notifications via email, Slack, Telegram, and others.
- Monitor your app performance and logs using [tracing](https://uptrace.dev/opentelemetry/distributed-tracing.html).

![uptrace-redis-nodes](/images/rs/uptrace-redis-nodes.png)

## Install Collector and Uptrace

Because installing OpenTelemetry Collector and Uptrace can take some time, you can use the [docker-compose](https://github.com/uptrace/uptrace/tree/master/example/redis-enterprise) example that also comes with Redis Enterprise cluster and AlertManager.

After you download the Docker example, you can edit the following configuration files in the `uptrace/example/redis-enterprise` directory before you start the Docker containers:

- `otel-collector.yaml` - Configures `/etc/otelcol-contrib/config.yaml` in the OpenTelemetry Collector container.
- `uptrace.yml` - Configures`/etc/uptrace/uptrace.yml` in the Uptrace container.

You can also install OpenTelemetry and Uptrace from scratch using the following guides:

- [Getting started with OpenTelemetry Collector](https://uptrace.dev/opentelemetry/collector.html)
- [Getting started with Uptrace](https://uptrace.dev/get/install.html)

After you install Uptrace, you can access the Uptrace UI at [http://localhost:14318/](http://localhost:14318/).

## Scrape Prometheus metrics

Redis Enterprise cluster exposes a Prometheus scraping endpoint on `http://localhost:8070/`. You can scrape that endpoint by adding the following lines to the OpenTelemetry Collector config:

```yaml
# /etc/otelcol-contrib/config.yaml

prometheus_simple/cluster1:
  collection_interval: 10s
  endpoint: "localhost:8070" # Redis Cluster endpoint
  metrics_path: "/"
  tls:
    insecure: false
    insecure_skip_verify: true
    min_version: "1.0"
```

Next, you can export the collected metrics to Uptrace using OpenTelemetry protocol (OTLP):

```yaml
# /etc/otelcol-contrib/config.yaml

receivers:
  otlp:
    protocols:
      grpc:
      http:

exporters:
  otlp:
    # Uptrace is accepting metrics on this port
    endpoint: localhost:14317
    headers: { "uptrace-dsn": "http://project1_secret_token@localhost:14317/1" }
    tls:
      insecure: true

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp]
    metrics:
      receivers: [otlp, prometheus_simple/cluster1]
      processors: [batch]
      exporters: [otlp]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp]
```

Don't forget to restart the Collector and then check logs for any errors:

```shell
docker-compose logs otel-collector

# or

sudo journalctl -u otelcol-contrib -f
```

You can also check the full OpenTelemetry Collector config [here](https://github.com/uptrace/uptrace/blob/master/example/redis-enterprise/otel-collector.yaml).

## View metrics

When metrics start arriving to Uptrace, you should see a couple of dashboards in the Metrics tab. In total, Uptrace should create 3 dashboards for Redis Enterprise metrics:

- "Redis: Nodes" dashboard displays a list of cluster nodes. You can select a node to view its metrics.

- "Redis: Databases" displays a list of Redis databases in all cluster nodes. To find a specific database, you can use filters or sort the table by columns.

- "Redis: Shards" contains a list of shards that you have in all cluster nodes. You can filter or sort shards and select a shard for more details.

## Monitor metrics

To start monitoring metrics, you need to add some alerting rules.

For example, the following rule uses the `group by node` expression to create an alert whenever an individual Redis shard is down:

```yaml
# /etc/uptrace/uptrace.yml

alerting:
  rules:
    - name: Redis shard is down
      metrics:
        - redis_up as $redis_up
      query:
        - group by cluster # monitor each cluster,
        - group by bdb # each database,
        - group by node # and each shard
        - $redis_up == 0
      # shard should be down for 5 minutes to trigger an alert
      for: 5m
      # only monitor Uptrace projects with these ids
      projects: [1]
```

You can also create queries with more complex expressions.

For example, the following rules create an alert when the keyspace hit rate is lower than 75% or memory fragmentation is too high:

```yaml
# /etc/uptrace/uptrace.yml

alerting:
  rules:
    - name: Redis read hit rate < 75%
      metrics:
        - redis_keyspace_read_hits as $hits
        - redis_keyspace_read_misses as $misses
      query:
        - group by cluster
        - group by bdb
        - group by node
        - $hits / ($hits + $misses) < 0.75
      for: 5m
      projects: [1]

    - name: Memory fragmentation is too high
      metrics:
        - redis_used_memory as $mem_used
        - redis_mem_fragmentation_ratio as $fragmentation
      query:
        - group by cluster
        - group by bdb
        - group by node
        - $mem_used > 32mb and $fragmentation > 3
      for: 5m
      projects: [1]
```

You can learn more about the query language [here](https://uptrace.dev/get/querying-metrics.html).

## Send notifications

Uptrace does not manage notifications by itself and instead provides an [integration](https://uptrace.dev/get/alerting.html) with AlertManager.

AlertManager handles alerts sent by client applications such as Uptrace and takes care of deduplicating, grouping, and routing notifications to configured receivers via email, Slack, Telegram, and many others.

To connect Uptrace to AlertManager and enable alert notifications, add the AlertManager API endpoint to `uptrace.yml`:

```yaml
# /etc/uptrace/uptrace.yml

##
## AlertManager client configuration.
## See https://uptrace.dev/get/alerting.html for details.
##
## Note that this is NOT an AlertManager config and you need to configure AlertManager separately.
## See https://prometheus.io/docs/alerting/latest/configuration/ for details.
##
alertmanager_client:
  # AlertManager API endpoints that Uptrace uses to manage alerts.
  urls:
    - "http://localhost:9093/api/v2/alerts"
```
