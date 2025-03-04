---
name: Prometheus
publisher: Kong Inc.
desc: Expose metrics related to Kong and proxied upstream services in Prometheus exposition format
description: |
  Expose metrics related to Kong and proxied upstream services in [Prometheus](https://prometheus.io/docs/introduction/overview/) exposition format, which can be scraped by a Prometheus Server.
type: plugin
categories:
  - analytics-monitoring
kong_version_compatibility:
  community_edition:
    compatible: true
  enterprise_edition:
    compatible: true
params:
  name: prometheus
  service_id: true
  route_id: false
  protocols:
    - name: http
    - name: https
    - name: tcp
    - name: tls
    - name: grpc
    - name: grpcs
  dbless_compatible: 'yes'
  dbless_explanation: |
    The database will always be reported as reachable in Prometheus with DB-less.
    Additionally, the DB entity count metric (`kong_db_entities_total`) is not
    emitted in DB-less mode.
  config:
    - name: per_consumer
      required: false
      datatype: boolean
      default: '`false`'
      description: |
        A boolean value that determines if per-consumer metrics should be
        collected.
        If enabled, the `kong_http_requests_total` and `kong_bandwidth_bytes`
        metrics fill in the consumer label when available.
    - name: status_code_metrics
      required: false
      datatype: boolean
      default: '`false`'
      description: |
        A boolean value that determines if status code metrics should be
        collected.
        If enabled, `http_requests_total`, `stream_sessions_total` metrics will be exported.
    - name: latency_metrics
      required: false
      datatype: boolean
      default: '`false`'
      description: |
        A boolean value that determines if status code metrics should be
        collected.
        If enabled, `kong_latency_ms`, `upstream_latency_ms` and `request_latency_ms`
        metrics will be exported.
    - name: bandwidth_metrics
      required: false
      datatype: boolean
      default: '`false`'
      description: |
        A boolean value that determines if status code metrics should be
        collected.
        If enabled, `bandwidth_bytes` and `stream_sessions_total` metrics will be exported.
    - name: upstream_health_metrics
      required: false
      datatype: boolean
      default: '`false`'
      description: |
        A boolean value that determines if status code metrics should be
        collected.
        If enabled, `upstream_target_health` metric will be exported.
---

Metrics tracked by this plugin are available on both the Admin API and Status
API at the `http://localhost:<port>/metrics`
endpoint. Note that the URL to those APIs will be specific to your
installation; see [Accessing the metrics](#accessing-the-metrics).

This plugin records and exposes metrics at the node level. Your Prometheus
server will need to discover all Kong nodes via a service discovery mechanism,
and consume data from each node's configured `/metrics` endpoint.

## Grafana dashboard

Metrics exported by the plugin can be graphed in Grafana using a drop in
dashboard: [https://grafana.com/dashboards/7424](https://grafana.com/dashboards/7424).

## Available metrics

- **DB reachability**: A gauge type with a value of 0 or 1, which represents
  whether DB can be reached by a Kong node.
- **Connections**: Various Nginx connection metrics like active, reading,
  writing, and number of accepted connections.
- **Dataplane Status**: The last seen timestamp, config hash, config sync status and certificate expiration timestamp for
data plane nodes is exported to control plane.
- **Enterprise License Information**: The {{site.base_gateway}} license expiration date, features and
license signature. Those metrics are only exported on {{site.base_gateway}}.
- **DB Entity Count** <span class="badge enterprise"></span> : A gauge metric that
    measures the current number of database entities.
- **Number of Nginx timers** : A gauge metric that measures the total number of Nginx
    timers, in Running or Pending state.

Following metrics are disabled by default as it may create high cardinality of metrics and may
cause performance issues:

When `status_code_metrics` is set to true:
- **Status codes**: HTTP status codes returned by upstream services.
  These are available per service, across all services, and per route per consumer.

When `latency_metrics` is set to true:
- **Latencies Histograms**: Latency (in ms), as measured at Kong:
   - **Request**: Total time taken by Kong and upstream services to serve
     requests.
   - **Kong**: Time taken for Kong to route a request and run all configured
     plugins.
   - **Upstream**: Time taken by the upstream service to respond to requests.

When `bandwidth_metrics` is set to true:
- **Bandwidth**: Total Bandwidth (egress/ingress) flowing through Kong.
  This metric is available per service and as a sum across all services.

When `upstream_health_metrics` is set to true:
- **Target Health**: The healthiness status (`healthchecks_off`, `healthy`, `unhealthy`, or `dns_error`) of targets
  belonging to a given upstream as well as their subsystem (`http` or `stream`).

Here is an example of output you could expect from the `/metrics` endpoint:

```bash
curl -i http://localhost:8001/metrics
```

Response:
```sh
HTTP/1.1 200 OK
Server: openresty/1.15.8.3
Date: Tue, 7 Jun 2020 16:35:40 GMT
Content-Type: text/plain; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Access-Control-Allow-Origin: *

# HELP kong_bandwidth_bytes Total bandwidth (ingress/egress) throughput in bytes
# TYPE kong_bandwidth_bytes counter
kong_bandwidth_bytes{service="google",route="google.route-1",direction="egress",consumer=""} 264
kong_bandwidth_bytes{service="google",route="google.route-1",direction="ingress",consumer=""} 93
# HELP kong_datastore_reachable Datastore reachable from Kong, 0 is unreachable
# TYPE kong_datastore_reachable gauge
kong_datastore_reachable 1
# HELP kong_http_requests_total HTTP status codes per consumer/service/route in Kong
# TYPE kong_http_requests_total counter
kong_http_requests_total{service="google",route="google.route-1",code="200",source="service",consumer=""} 1
# HELP kong_node_info Kong Node metadata information
# TYPE kong_node_info gauge
kong_node_info{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",version="3.0.0"} 1
# HELP kong_kong_latency_ms Latency added by Kong and enabled plugins for each service/route in Kong
# TYPE kong_kong_latency_ms histogram
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="5"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="7"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="10"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="15"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="20"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="30"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="50"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="75"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="100"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="200"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="500"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="750"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="1000"} 1
kong_kong_latency_ms_bucket{service="google",route="google.route-1",le="+Inf"} 1
kong_kong_latency_ms_count{service="google",route="google.route-1"} 1
kong_kong_latency_ms_sum{service="google",route="google.route-1"} 4
# HELP kong_memory_lua_shared_dict_bytes Allocated slabs in bytes in a shared_dict
# TYPE kong_memory_lua_shared_dict_bytes gauge
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong",kong_subsystem="http"} 40960
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_cluster_events",kong_subsystem="http"} 40960
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_core_db_cache",kong_subsystem="http"} 823296
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_core_db_cache_miss",kong_subsystem="http"} 90112
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_db_cache",kong_subsystem="http"} 794624
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_db_cache_miss",kong_subsystem="http"} 86016
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_healthchecks",kong_subsystem="http"} 40960
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_locks",kong_subsystem="http"} 61440
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_process_events",kong_subsystem="http"} 40960
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_rate_limiting_counters",kong_subsystem="http"} 86016
kong_memory_lua_shared_dict_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="prometheus_metrics",kong_subsystem="http"} 57344
# HELP kong_memory_lua_shared_dict_total_bytes Total capacity in bytes of a shared_dict
# TYPE kong_memory_lua_shared_dict_total_bytes gauge
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong",kong_subsystem="http"} 5242880
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_cluster_events",kong_subsystem="http"} 5242880
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_core_db_cache",kong_subsystem="http"} 134217728
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_core_db_cache_miss",kong_subsystem="http"} 12582912
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_db_cache",kong_subsystem="http"} 134217728
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_db_cache_miss",kong_subsystem="http"} 12582912
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_healthchecks",kong_subsystem="http"} 5242880
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_locks",kong_subsystem="http"} 8388608
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_process_events",kong_subsystem="http"} 5242880
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="kong_rate_limiting_counters",kong_subsystem="http"} 12582912
kong_memory_lua_shared_dict_total_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",shared_dict="prometheus_metrics",kong_subsystem="http"} 5242880
# HELP kong_memory_workers_lua_vms_bytes Allocated bytes in worker Lua VM
# TYPE kong_memory_workers_lua_vms_bytes gauge
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21173",kong_subsystem="http"} 64329517
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21174",kong_subsystem="http"} 46314808
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21175",kong_subsystem="http"} 46681598
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21176",kong_subsystem="http"} 46637209
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21177",kong_subsystem="http"} 46234336
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21178",kong_subsystem="http"} 46180420
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21179",kong_subsystem="http"} 46161105
kong_memory_workers_lua_vms_bytes{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",pid="21180",kong_subsystem="http"} 46366877
# HELP kong_nginx_connections_total Number of connections by subsystem
# TYPE kong_nginx_connections_total gauge
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="accepted"} 296
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="active"} 9
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="handled"} 296
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="reading"} 0
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="total"} 296
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="waiting"} 0
kong_nginx_connections_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http",state="writing"} 9
# HELP kong_nginx_metric_errors_total Number of nginx-lua-prometheus errors
# TYPE kong_nginx_metric_errors_total counter
kong_nginx_metric_errors_total 0
# HELP kong_nginx_requests_total Total number of requests
# TYPE kong_nginx_requests_total gauge
kong_nginx_requests_total{node_id="849373c5-45c1-4c1d-b595-fdeaea6daed8",subsystem="http"} 296
# HELP kong_nginx_timers Number of Nginx timers
# TYPE kong_nginx_timers gauge
kong_nginx_timers{state="pending"} 1
kong_nginx_timers{state="running"} 39
# HELP kong_request_latency_ms Total latency incurred during requests for each service/route in Kong
# TYPE kong_request_latency_ms histogram
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="25"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="50"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="80"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="100"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="250"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="400"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="700"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="1000"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="2000"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="5000"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="10000"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="30000"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="60000"} 1
kong_request_latency_ms_bucket{service="google",route="google.route-1",le="+Inf"} 1
kong_request_latency_ms_count{service="google",route="google.route-1"} 1
kong_request_latency_ms_sum{service="google",route="google.route-1"} 6
# HELP kong_upstream_latency_ms Latency added by upstream response for each service/route in Kong
# TYPE kong_upstream_latency_ms histogram
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="25"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="50"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="80"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="100"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="250"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="400"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="700"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="1000"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="2000"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="5000"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="10000"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="30000"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="60000"} 1
kong_upstream_latency_ms_bucket{service="google",route="google.route-1",le="+Inf"} 1
kong_upstream_latency_ms_count{service="google",route="google.route-1"} 1
kong_upstream_latency_ms_sum{service="google",route="google.route-1"} 2

```

{:.note}
> **Note:** Upstream targets' health information is exported once per subsystem. If both
stream and HTTP listeners are enabled, targets' health will appear twice. Health metrics
have a `subsystem` label to indicate which subsystem the metric refers to.

## Accessing the metrics

In most configurations, the Kong Admin API will be behind a firewall or would
need to be set up to require authentication. Here are a couple of options to
allow access to the `/metrics` endpoint to Prometheus:


1. If the [Status API](/gateway/latest/reference/configuration/#status_listen)
   is enabled, then its `/metrics` endpoint can be used.
   This is the preferred method.

1. The `/metrics` endpoint is also available on the Admin API, which can be used
   if the Status API is not enabled. Note that this endpoint is unavailable
   when [RBAC](/gateway/latest/admin-api/rbac/reference/) is enabled on the
   Admin API (Prometheus does not support Key-Auth to pass the token).

---

## Changelog

**{{site.base_gateway}} 3.0.x**
* High cardinality metrics are now disabled by default.
* Decreased performance penalty to proxy traffic when collecting metrics.
* The following metric names were adjusted to add units to standardize where possible:
  * `http_status` to `http_requests_total`
  * `latency` to `kong_request_latency_ms`/`kong_upstream_latency_ms`/`kong_kong_latency_ms`
  * `kong_bandwidh` to `kong_bandwidth_bytes`
  * `nginx_http_current_connections`/`nginx_stream_current_connections` to `nginx_connections_total`
  * Removed: `http_consumer_status`
* New metric: `session_duration_ms` for monitoring stream connections
* New metric: `node_info` is a single gauge set to 1 that outputs the node's ID and {{site.base_gateway}} version
* Latency was split into four different metrics: `kong_latency_ms`, `upstream_latency_ms`, `request_latency_ms` (HTTP), and `session_duration_ms` (stream). Buckets details follow:
  * Kong Latency and Upstream Latency can operate at orders of different magnitudes. Separate these buckets to reduce memory overhead.
* `request_count` and `consumer_status` were merged into `http_requests_total`. If the `per_consumer` config is set to false, the `consumer` label will be empty.  If the `per_consumer` config is true, the `consumer` label will be filled.
* `http_requests_total` has a new label [`source`](/gateway/latest/plugin-development/pdk/kong.response/#kongresponseget_source/). It can be set to `exit`, `error`, or `service`.
* All Memory metrics have a new label, `node_id`.
* Plugin version bumped to 3.0.0
* The `node_id` label was added to memory metrics.

**{{site.base_gateway}} 2.8.x**
* Adds a new metric:
  * `kong_nginx_timers` (gauge): total number of Nginx timers, in Running or Pending state.
* Add two new metrics:
  * `kong_db_entities_total` (gauge): total number of entities in the database
  * `kong_db_entity_count_errors` (counter): measures the number of errors
      encountered during the measurement of `kong_db_entities_total`

**{{site.base_gateway}} 2.5.x**
* New `data_plane_cluster_cert_expiry_timestamp` metric
* Added `subsystem` label to Upstream Target health metrics

**{{site.base_gateway}} 2.4.x**
* Added the `per_consumer` configuration parameter to export per-consumer status.

**{{site.base_gateway}} 2.3.x**
* The plugin can now export Kong Enterprise licensing information.
