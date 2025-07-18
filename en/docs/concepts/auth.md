---
comments: true
---

# Authentication

Authentication can be performed by setting single authentication information or authenticator.

!!! tip "Dynamic configuration"
    Authenticator supports dynamic configuration via [Web API](../tutorials/api/overview.md).

## Single Authentication

If multi-user authentication is not required, single-user authentication can be performed by directly setting the single authentication information.

**Server**

=== "CLI"

	Set directly by `username:password`:

    ```bash
	gost -L http://user:pass@:8080
	```

	If the authentication information contains special characters, it can also be set through the `auth` option. The value of `auth` is a base64 encoded value in the form of `username:password`.

	```bash
	echo -n user:pass | base64
	```

	```bash
	gost -L http://:8080?auth=dXNlcjpwYXNz
	```

=== "File (YAML)"

    ```yaml linenums="1" hl_lines="6 7 8"
	services:
	- name: service-0
	  addr: ":8080"
	  handler:
		type: http
		auth:
		  username: user
		  password: pass
	  listener:
		type: tcp
	```

	Single authentication information is set via the `auth` property on the service's handler or listener.

**Client**

=== "CLI"

	Set directly by `username:password`:

    ```
	gost -L http://:8080 -F socks5://user:pass@:1080
	```

	If the authentication information contains special characters, it can also be set through the `auth` option. The value of `auth` is a base64 encoded value in the form of `username:password`.

	```
	gost -L http://:8080 -F socks5://:1080?auth=dXNlcjpwYXNz
	```

=== "File (YAML)"

    ```yaml linenums="1" hl_lines="18 19 20"
	services:
	- name: service-0
	  addr: ":8080"
	  handler:
		type: http
		chain: chain-0
	  listener:
		type: tcp
	chains:
	- name: chain-0
	  hops:
	  - name: hop-0
		nodes:
		- name: node-0
		  addr: :1080
		  connector:
			type: socks5
			auth:
			  username: user
			  password: pass
		  dialer:
		    type: tcp
	```

	Single authentication information is set via the `auth` property on the node's connector or dialer.

### Loading From File

:material-tag: 3.1.0

Client can also specify the file path through the `auth.file` option and load the authentication information from the file.

=== "File (YAML)"

    ```yaml hl_lines="18 19"
	services:
	- name: service-0
	  addr: ":8080"
	  handler:
		type: http
		chain: chain-0
	  listener:
		type: tcp
	chains:
	- name: chain-0
	  hops:
	  - name: hop-0
		nodes:
		- name: node-0
		  addr: :1080
		  connector:
			type: socks5
			auth:
			  file: /path/to/auth/file
		  dialer:
		    type: tcp
	```

The file format is the authentication information separated by lines, each line of authentication information is a user-pass pair separated by spaces, and the lines starting with `#` are commented out.


```text
# username password

user1 pass1
```

If there are multiple lines of authentication information, only the first one will be used.

## Authenticator

An authenticator contains one or more sets of authentication information. Service can achieve the multi-user authentication function through the authenticator.

!!! note 
    Authenticator only supports the configuration file method.

=== "File (YAML)"

    ```yaml linenums="1" hl_lines="6 10"
    services:
    - name: service-0
      addr: ":8080"
      handler:
        type: http
		auther: auther-0
      listener:
        type: tcp
	authers:
	- name: auther-0
	  auths:
	  - username: user1
	    password: pass1
	  - username: user2
        password: pass2
	```

Use the specified authenticator by referencing the authenticator name via the `auther` property on the service's handler or listener.

!!! caution "Priority"
	If an authenticator is used, single authentication information will be ignored.

	If the `auth` option is set, the authentication information set directly in the path will be ignored.

!!! caution "Shadowsocks Handler"
	The Shadowsocks handler cannot use authenticator, and only supports setting single authentication information as encryption parameter.

## Authenticator Group

Use multiple authenticators by specifying a list of authenticators using the `authers` option. When any one of the authenticators passes the authentication, it means the authentication is passed.

=== "File (YAML)"

    ```yaml linenums="1" hl_lines="6 7 8 12 16"
    services:
    - name: service-0
      addr: ":8080"
      handler:
        type: http
		authers:
		- auther-0
		- auther-1
      listener:
        type: tcp
	authers:
	- name: auther-0
	  auths:
	  - username: user1
	    password: pass1
	- name: auther-1
	  auths:
	  - username: user2
        password: pass2
	```

## Data Source

Authenticator can configure multiple data sources, currently supported data sources are: inline, file, redis.

### Inline

An inline data source means setting the data directly in the configuration file via the `auths` property.

```yaml
authers:
- name: auther-0
  auths:
  - username: user1
    password: pass1
  - username: user2
    password: pass2
```

### File

Specify an external file as the data source. Specify the file path via the `file.path` property.

```yaml
authers:
- name: auther-0
  file:
    path: /path/to/auth/file
```

The file format is the authentication information separated by lines, each line of authentication information is a user-pass pair separated by spaces, and the lines starting with `#` are commented out.

```text
# username password

admin           #123456
test\user001    123456
test.user@002   12345678
```

### Redis

Specify the redis service as the data source, and the redis data type must be [Hash](https://redis.io/docs/manual/data-types/#hashes).

```yaml
authers:
- name: auther-0
  redis:
    addr: 127.0.0.1:6379
    db: 1
    username: user
    password: 123456
    key: gost:authers:auther-0
```

`addr` (string, required)
:    redis server address

`db` (int, default=0)
:    database name

`username` (string)
:    username

`password` (string)
:    password

`key` (string, default=gost)
:    redis key

```redis
> HGETALL gost:authers:auther-0
1) "admin"
2) "#123456"
3) "test\user001"
4) "123456"
```

### HTTP

Specify the HTTP service as the data source. For the requested URL, if HTTP returns a 200 status code, it is considered valid, and the returned data format is the same as the file data source.

```yaml
authers:
- name: auther-0
  http:
    url: http://127.0.0.1:8000
    timeout: 10s
```

`url` (string, required)
:    request URL

`timeout` (duration, default=0)
:    request timeout

## Priority

When configuring multiple data sources at the same time, the priority from high to low is: HTTP, redis, file, inline. If the same username exists in different data sources, the data with higher priority will overwrite the data with lower priority.

## Hot Reload

File, redis and HTTP data sources support hot reloading. Enable hot loading by setting the `reload` property, which specifies the period for synchronizing the data source data.

```yaml linenums="1" hl_lines="3"
authers:
- name: auther-0
  reload: 10s
  file:
    path: /path/to/auth/file
  redis:
    addr: 127.0.0.1:6379
	db: 1
	password: 123456
	key: gost:authers:auther-0
```

!!! note 
	Authentication information set via the command line applies only to the handler or connector, and for ssh and sshd services it applies to the listener and dialer.

	If the configuration file is automatically generated through the command line, this parameter item will not appear in the metadata.

## Plugin

Authenticator can be configured to use an external [plugin](plugin.md) service, and authenticator will forward the request to the plugin server for processing. Other parameters are invalid when using plugin.

```yaml
authers:
- name: auther-0
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
authers:
- name: auther-0
  plugin:
    type: http
    addr: http://127.0.0.1:8000/auth
```

#### Example

```bash
curl -XPOST http://127.0.0.1:8000/auth -d '{"username":"gost", "password":"gost", "client":"127.0.0.1:12345"}'
```

```json
{"ok": true, "id":"gost"}
```

`client` (string)
:    client address

`id` (string)
:    plugin service can optionally return the user ID, and this information will be passed to other subsequent plugin services (Bypass, HostMapper, Resolver) for user identification.