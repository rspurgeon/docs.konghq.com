---
name: Rate Limiting
publisher: Kong Inc.
desc: Rate-limit how many HTTP requests can be made in a period of time
description: |
  Rate limit how many HTTP requests can be made in a given period of seconds, minutes, hours, days, months, or years.
  If the underlying Service/Route (or deprecated API entity) has no authentication layer,
  the **Client IP** address will be used; otherwise, the Consumer will be used if an
  authentication plugin has been configured.

  **Tip:** The [Rate Limiting Advanced](/hub/kong-inc/rate-limiting-advanced/)
    plugin provides the ability to apply
    [multiple limits in sliding or fixed windows](/hub/kong-inc/rate-limiting-advanced/#multi-limits-windows).
type: plugin
categories:
  - traffic-control
kong_version_compatibility:
  community_edition:
    compatible: true
  enterprise_edition:
    compatible: true
params:
  name: rate-limiting
  service_id: true
  route_id: true
  consumer_id: true
  protocols:
    - name: http
    - name: https
  dbless_compatible: partially
  dbless_explanation: |
    The plugin will run fine with the `local` policy (which doesn't use the database) or
    the `redis` policy (which uses an independent Redis, so it is compatible with DB-less).

    The plugin will not work with the `cluster` policy, which requires writes to the database.
  config:
    - name: second
      required: semi
      value_in_examples: 5
      datatype: number
      description: The number of HTTP requests that can be made per second.
    - name: minute
      required: semi
      datatype: number
      description: The number of HTTP requests that can be made per minute.
    - name: hour
      required: semi
      value_in_examples: 10000
      datatype: number
      description: The number of HTTP requests that can be made per hour.
    - name: day
      required: semi
      datatype: number
      description: The number of HTTP requests that can be made per day.
    - name: month
      required: semi
      datatype: number
      description: The number of HTTP requests that can be made per month.
    - name: year
      required: semi
      datatype: number
      description: The number of HTTP requests that can be made per year.
    - name: limit_by
      required: false
      default: '`consumer`'
      datatype: string
      description: |
        The entity that is used when aggregating the limits. Available values are:
        - `consumer`
        - `credential`
        - `ip`
        - `service` (The `service.id` or `service.name` configuration must be provided if you're adding the plugin to a service through the top-level `/plugins` endpoint.)
        - `header` (The `header_name` configuration must be provided.)
        - `path` (The `path` configuration must be provided.)

        If the entity value for aggregating the limits cannot be determined, the system falls back to `ip`.
    - name: header_name
      required: semi
      datatype: string
      description: Header name to be used if `limit_by` is set to `header`.
    - name: path
      required: semi
      datatype: string
      description: Path to be used if `limit_by` is set to `path`.
    - name: policy
      required: false
      value_in_examples: local
      default: '`local`'
      datatype: string
      description: |
        The rate-limiting policies to use for retrieving and incrementing the
        limits. Available values are:
        - `local`: Counters are stored locally in-memory on the node.
        - `cluster`: Counters are stored in the Kong data store and shared across
        the nodes.
        - `redis`: Counters are stored on a Redis server and shared
        across the nodes.

        In DB-less and hybrid modes, the `cluster` config policy is not supported.
        For DB-less mode, use one of `redis` or `local`; for hybrid mode, use
        `redis`, or `local` for data planes only.

        In Konnect, the default policy is `redis`.

        For details on which policy should be used, refer to the
        [implementation considerations](#implementation-considerations).
    - name: fault_tolerant
      required: true
      default: '`true`'
      datatype: boolean
      description: |
        A boolean value that determines if the requests should be proxied even if Kong has troubles connecting a third-party data store. If `true`, requests will be proxied anyway, effectively disabling the rate-limiting function until the data store is working again. If `false`, then the clients will see `500` errors.
    - name: hide_client_headers
      required: true
      default: '`false`'
      datatype: boolean
      description: Optionally hide informative response headers.
    - name: redis_host
      required: semi
      datatype: string
      description: |
        When using the `redis` policy, this property specifies the address to the Redis server.
    - name: redis_port
      required: false
      default: '`6379`'
      datatype: integer
      description: |
        When using the `redis` policy, this property specifies the port of the Redis server. By default is `6379`.
    - name: redis_username
      minimum_version: "2.8.x"
      required: false
      datatype: string
      referenceable: true
      description: |
        When using the `redis` policy, this property specifies the username to connect to the Redis server when ACL authentication is desired.
    - name: redis_password
      minimum_version: "2.7.x"
      required: false
      datatype: string
      referenceable: true
      description: |
        When using the `redis` policy, this property specifies the password to connect to the Redis server.
    - name: redis_ssl
      minimum_version: "2.7.x"
      required: true
      default: '`false`'
      datatype: boolean
      description: |
        When using the `redis` policy, this property specifies if SSL is used to connect to the Redis server.
    - name: redis_ssl_verify
      required: true
      default: '`false`'
      datatype: boolean
      description: |
        When using the `redis` policy with `redis_ssl` set to `true`, this property specifies it server SSL certificate is validated. Note that you need to configure the lua_ssl_trusted_certificate to specify the CA (or server) certificate used by your Redis server. You may also need to configure lua_ssl_verify_depth accordingly.
    - name: redis_server_name
      required: false
      datatype: string
      description: |
        When using the `redis` policy with `redis_ssl` set to `true`, this property specifies the server name for the TLS extension Server Name Indication (SNI)
    - name: redis_timeout
      required: false
      default: '`2000`'
      datatype: number
      description: |
        When using the `redis` policy, this property specifies the timeout in milliseconds of any command submitted to the Redis server.
    - name: redis_database
      required: false
      default: '`0`'
      datatype: integer
      description: |
        When using the `redis` policy, this property specifies the Redis database to use.
    - name: error_code
      minimum_version: "3.1.x"
      required: false
      default: 429
      datatype: number
      description: |
        Set a custom error code to return when the rate limit is exceeded.
    - name: error_message
      minimum_version: "3.1.x"
      required: false
      default: rate limit exceeded
      datatype: string
      description: |
        Set a custom error message to return when the rate limit is exceeded.


  extra: '<div class="alert alert-warning"> <strong>Note:</strong> At least one limit (`second`, `minute`, `hour`, `day`, `month`, `year`) must be configured. Multiple limits can be configured. </div>'
---

## Headers sent to the client

When this plugin is enabled, Kong sends additional headers
to show the allowed limits, number of available requests,
and the time remaining (in seconds) until the quota is reset. Here's an example header:

```
RateLimit-Limit: 6
RateLimit-Remaining: 4
RateLimit-Reset: 47
```

The plugin also sends headers to show the time limit and the minutes still available:

```
X-RateLimit-Limit-Minute: 10
X-RateLimit-Remaining-Minute: 9
```

If more than one time limit is set, the header contains all of these:

```
X-RateLimit-Limit-Second: 5
X-RateLimit-Remaining-Second: 4
X-RateLimit-Limit-Minute: 10
X-RateLimit-Remaining-Minute: 9
```

When a limit is reached, the plugin returns an `HTTP/1.1 429` status code, with the following JSON body:

```json
{ "message": "API rate limit exceeded" }
```

{:.warning}
> **Warning**: The headers `RateLimit-Limit`, `RateLimit-Remaining`, and `RateLimit-Reset` are based on the Internet-Draft [RateLimit Header Fields for HTTP](https://tools.ietf.org/html/draft-polli-ratelimit-headers-01). These could change if the specification is updated.

## Implementation considerations

The plugin supports three policies.

| Policy    | Pros | Cons   |
| --------- | ---- | ------ |
| `local`   | Minimal performance impact. | Less accurate. Unless there's a consistent-hashing load balancer in front of Kong, it diverges when scaling the number of nodes.
| `cluster` | Accurate, no extra components to support. | Each request forces a read and a write on the data store. Therefore, relatively, the biggest performance impact. |
| `redis`   | Accurate, less performance impact than a `cluster` policy. | Needs a Redis installation. Bigger performance impact than a `local` policy. ||

Two common use cases are:

1. _Every transaction counts_. The highest level of accuracy is needed. An example is a transaction with financial
   consequences.
2. _Backend protection_. Accuracy is not as relevant. The requirement is
   only to protect backend services from overloading that's caused either by specific
   users or by attacks.

{:.warning}
> **Note**: **Enterprise-Only**: The Kong Community Edition of this Rate Limiting plugin does not
include [Redis Sentinel](https://redis.io/topics/sentinel) support. Only [{{site.base_gateway}} Enterprise](https://www.konghq.com/kong) customers can use Redis Sentinel with Kong Rate Limiting, enabling them to deliver highly available primary-replica deployments.

### Every transaction counts

In this scenario, because accuracy is important, the `local` policy is not an option. Consider the support effort you might need
for Redis, and then choose either `cluster` or `redis`.

You could start with the `cluster` policy, and move to `redis`
if performance reduces drastically.

Do remember that you cannot port the existing usage metrics from the data store to Redis.
This might not be a problem with short-lived metrics (for example, seconds or minutes)
but if you use metrics with a longer time frame (for example, months), plan
your switch carefully.

### Backend protection

If accuracy is of lesser importance, choose the `local` policy. You might need to experiment a little
before you get a setting that works for your scenario. As the cluster scales to more nodes, more user requests are handled.
When the cluster scales down, the probability of false negatives increases. So, adjust your limits when scaling.

For example, if a user can make 100 requests every second, and you have an
equally balanced 5-node Kong cluster, setting the `local` limit to something like 30 requests every second
should work. If you see too many false negatives, increase the limit.

To minimise inaccuracies, consider using a consistent-hashing load balancer in front of
Kong. The load balancer ensures that a user is always directed to the same Kong node, thus reducing
inaccuracies and preventing scaling problems.

### Fallback to IP

When the selected policy cannot be retrieved, the plugin falls back
to limiting usage by identifying the IP address. This can happen for several reasons, such as the
selected header was not sent by the client or the configured service was not found.

---

## Changelog

**{{site.base_gateway}} 3.1.x**
* Added the ability to customize the error code and message with
the configuration parameters `error_code` and `error_message`.

**{{site.base_gateway}} 2.8.x**

* Added the `redis_username` configuration parameter.

**{{site.base_gateway}}  2.7.x**

* Added the `redis_ssl`, `redis_ssl_verify`, and `redis_server_name` configuration parameters.

[api-object]: /gateway/latest/admin-api/#api-object
[configuration]: /gateway/latest/reference/configuration
[consumer-object]: /gateway/latest/admin-api/#consumer-object
