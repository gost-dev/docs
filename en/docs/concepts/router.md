---
comments: true
---

# Router

!!! tip "Dynamic configuration"
    Router supports dynamic configuration via [Web API](../tutorials/api/overview.md).

!!! note "Limitation"
    Router currently can only be used in [TUN device](../tutorials/tuntap.md) and [routing tunnel](../tutorials/routing-tunnel.md).

A router is composed of a routing table. Each routing item is a mapping from the target network to the gateway.

## Data Source

Router can configure multiple data sources, currently supported data sources are: inline, file, redis.

#### Inline

An inline data source means setting the data directly in the configuration file via the `routes` options.

```yaml
routers:
- name: router-0
  routes:
  - dst: 10.0.0.1
    gateway: 192.168.123.1
  - dst: 192.168.1.0/24
    gateway: 192.168.123.2
  - dst: 172.10.0.0/16
    gateway: 192.168.123.3
```

### File

Specify an external file as the data source. Specify the file path via the `file.path` property.

```yaml
routers:
- name: router-0
  file:
    path: /path/to/file
```

The file format is mapping items separated by lines, each line is an net-gateway pair separated by spaces, and the part starting with `#` is the comment information.

```text
# dst gateway

10.0.0.1  192.168.123.1
192.168.1.0/24  192.168.123.2
172.10.0.0/16  192.168.123.3
```

### Redis

Specify the redis service as the data source, and the redis data type can be [Hash](https://redis.io/docs/data-types/hashes/) or [Set](https://redis.io/docs/data-types/sets/).

```yaml
routers:
- name: router-0
  redis:
    addr: 127.0.0.1:6379
    db: 1
    username: user
    password: 123456
    key: gost:routers:router-0
    type: hash
```

`addr` (string, required)
:    redis addr.

`db` (int, default=0)
:    database name.

`username` (string)
:    username.

`password` (string)
:    password.

`key` (string, default=gost)
:    redis key.

`type` (string, default=hash)
:    data type: `hash` or `set`.

```redis
> HGETALL gost:routers:router-0
1) "10.0.0.1"
2) "192.168.123.1"
3) "192.168.1.0/24"
4) "192.168.123.2"
5) "172.10.0.0/16"
6) "192.168.123.3"
```

### HTTP

Specify an HTTP service as the data source. For the requested URL, if the HTTP status code is 200, it is considered valid, and the returned data format is the same as that of the file data source.

```yaml
routers:
- name: router-0
  http:
    url: http://127.0.0.1:8000
    timeout: 10s
```

`url` (string, required)
:    request URL.

`timeout` (duration, default=0)
:    request timeout.

## Priority

When configuring multiple data sources at the same time, the priority from high to low is: HTTP, redis, file, inline.

## Hot Reload

File, redis, HTTP data sources support hot reloading. Enable hot loading by setting the `reload` option, which specifies the period for synchronizing the data source data.

```yaml hl_lines="3"
routers:
- name: router-0
  reload: 10s
  file:
    path: /path/to/file
  redis:
    addr: 127.0.0.1:6379
	db: 1
	password: 123456
	key: gost:routers:router-0
  http:
    url: http://127.0.0.1:8000
    timeout: 10s
```

## Plugin

Router can be configured to use an external [plugin](plugin.md) service, and it will forward the request to the plugin server for processing. Other parameters are invalid when using plugin.

```yaml
routers:
- name: router-0
  plugin:
    type: grpc
    addr: 127.0.0.1:8000
    tls: 
      secure: false
      serverName: example.com
```

`type` (string, default=grpc)
:    plugin type: `grpc`, `http`.

`addr` (string, required)
:    plugin server address.

`tls` (object, default=null)
:    TLS encryption will be used for transmission, TLS encryption is not used by default.

### HTTP Plugin

```yaml
routers:
- name: router-0
  plugin:
    type: http
    addr: http://127.0.0.1:8000/router
```

#### Example

```bash
curl -XGET "http://127.0.0.1:8000/router?dst=192.168.1.2&id=1b564300-a2c5-406d-af0c-b7bdb039b0e3"
```

`dst` (string, required)
:    destination address.

`id` (UUID, optional)
:    Optional router ID, only valid when using [routing tunnel](../tutorials/routing-tunnel.md).

```json
{"dst":"192.168.1.0/24","gateway":"192.168.123.2"}
```
