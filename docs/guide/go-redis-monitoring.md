---
title: Go Redis Monitoring Performance [OpenTelemetry 2022]
---

<CoverImage title="Monitoring Go Redis Performance and Errors" />

[[toc]]

## What is OpenTelemetry?

[OpenTelemetry](https://uptrace.dev/opentelemetry/) is an open-source observability framework for
[distributed tracing](https://uptrace.dev/opentelemetry/distributed-tracing.html) (including logs
and errors) and [metrics](https://uptrace.dev/opentelemetry/metrics.html).

Otel allows developers to collect and export telemetry data in a vendor agnostic way. With
OpenTelemetry, you can instrument your application once and then add or change vendors without
changing the instrumentation, for example, here is a list
[popular DataDog alternatives](https://uptrace.dev/get/compare/datadog-competitors.html) that
support OpenTelemetry.

OpenTelemetry is available for most programming languages and provides interoperability across
different languages and environments.

## OpenTelemetry instrumentation

go-redis comes with an OpenTelemetry instrumentation called
[redisotel](https://github.com/go-redis/redis/tree/master/extra/redisotel) that is distributed as a
separate module:

```shell
go get github.com/go-redis/redis/extra/redisotel/v8
```

To instrument Redis client, you need to add the hook provided by redisotel:

```go
import (
    "github.com/go-redis/redis/v8"
    "github.com/go-redis/redis/extra/redisotel/v8"
)

rdb := redis.NewClient(&redis.Options{...})

rdb.AddHook(redisotel.NewTracingHook())
```

For Redis Cluster and Ring you need to instrument each node separately:

```go
rdb := redis.NewClusterClient(&redis.ClusterOptions{
    // ...

    NewClient: func(opt *redis.Options) *redis.Client {
        node := redis.NewClient(opt)
        node.AddHook(redisotel.NewTracingHook())
        return node
    },
})

rdb.AddHook(redisotel.NewTracingHook())
```

To make tracing work, you must pass the active
[trace context](https://uptrace.dev/opentelemetry/go-tracing.html#context) to go-redis commands, for
example:

```go
ctx := req.Context()
val, err := rdb.Get(ctx, "key").Result()
```

## Uptrace

Uptrace is an open source APM tool with an intuitive query builder, rich dashboards, automatic
alerts, and integrations for most languages and frameworks.

You can [install Uptrace](https://uptrace.dev/get/install.html) by downloading a DEB/RPM package or
a pre-compiled binary.

As expected, redisotel creates
[spans](https://uptrace.dev/opentelemetry/distributed-tracing.html#spans) for processed Redis
commands and records any errors as they occur. Here is how the collected information is displayed:

![Redis trace](/redis-monitoring/trace.png)

You can find a runnable example at
[GitHub](https://github.com/go-redis/redis/tree/master/example/otel).

## Prometheus

You can send OpenTelemetry metrics to Prometheus using
[OpenTelemetry Prometheus exporter](https://uptrace.dev/opentelemetry/prometheus-metrics.html).

## Monitoring Redis Server performance

In addition to monitoring go-redis client, you can also
[monitor Redis Server performance](https://uptrace.dev/opentelemetry/redis-performance-monitoring.html)
using OpenTelemetry Collector Agent.

## See also

- [Debugging Go Redis: pool size, timeouts](/guide/go-redis-debugging.html)
- [Getting started with OpenTelemetry, Gin, and GORM](https://uptrace.dev/get/opentelemetry-gin-gorm.html)
