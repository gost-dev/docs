---
comments: true
---

# Configuration Overview

!!! tip
    Before using the configuration file, it is recommended to understand some basic concepts and architecture, which will be very helpful for understanding the configuration file.

    You can use `-O` in command line mode to output the current configuration at any time.
	
!!! note "Default Configuration File"
    If neither `-C` nor `-L` parameters are specified, GOST will look for `gost.yml` or `gost.json` file in the following locations: current working directory, `/etc/gost/`, `$HOME/gost/`, and use it as the configuration file if it exists.

There are two ways to run GOST: run directly in the command line, and run through a configuration file. The command-line mode is sufficient for most use cases, such as simply starting a proxy or forwarding service. If you need more elaborate configuration, you can use the configuration file. The configuration file supports `YAML` and `JSON` formats.

For detailed configuration specification, please refer to:

* [CLI](../reference/configuration/cmd.md)
* [File](../reference/configuration/file.md)

There is a conversion relationship between the command line mode and the configuration file, for example:

```bash
gost -L http://gost:gost@localhost:8080?foo=bar -F socks5+tls://gost:gost@192.168.1.1:8080?bar=baz
```

The corresponding configuration file:

=== "YAML"

	```yaml
	services:
	- name: service-0
	  addr: "localhost:8080"
	  handler:
		type: http
		chain: chain-0
		auth:
		  username: gost
		  password: gost
		metadata:
		  foo: bar
	  listener:
		type: tcp
		metadata:
		  foo: bar
	chains:
	- name: chain-0
	  hops:
	  - name: hop-0
		nodes:
		- name: node-0
		  addr: 192.168.1.1:8080
		  connector:
			type: socks5
			auth:
			  username: gost
			  password: gost
			metadata:
			  bar: baz
		  dialer:
			type: tls
			metadata:
			  bar: baz
	```
=== "JSON"

	```json
	{
	  "services": [
		{
		  "name": "service-0",
		  "addr": "localhost:8080",
		  "handler": {
			"type": "http",
			"chain": "chain-0",
			"auth": {
			  "username": "gost",
			  "password": "gost"
			},
			"metadata": {
			  "foo": "bar"
			}
		  },
		  "listener": {
			"type": "tcp",
			"metadata": {
			  "foo": "bar"
			}
		  }
		}
	  ],
	  "chains": [
		{
		  "name": "chain-0",
		  "hops": [
			{
			  "name": "hop-0",
			  "nodes": [
				{
				  "name": "node-0",
				  "addr": "192.168.1.1:8080",
				  "connector": {
					"type": "socks5",
					"auth": {
					  "username": "gost",
					  "password": "gost"
					  },
					"metadata": {
					  "bar": "baz"
					}
				  },
				  "dialer": {
					"type": "tls",
					"metadata": {
					  "bar": "baz"
					}
				  }
				}
			  ]
			}
		  ]
		}
	  ]
	}
	```

- All `-L` parameters are converted to a list of `services` in order, each service will automatically generate a name `name` attribute.

    * The scheme will be parsed as `handler` and `listener`, for example `http` will be converted to http handler and tcp listener.
	* The `localhost:8080` corresponds to the field `addr` of the service.
    * The authentication `gost:gost` is converted to `handler.auth`.
	* The option `foo=bar` is converted to `handler.metadata` and `listener.metadata`.
	* If a forwarding chain exists, it is referenced by `handler.chain` (via the `name` field).

- If there are one or more `-F` parameters, a forwarding chain is generated in the `chains` list, a `-F` corresponds to an item in the `hops` list of the forwarding chain, and the `-F` parameters are converted in order to the nodes in the corresponding hop.

    * The scheme will be parsed as `connector` and `dialer`, for example `socks5+tls` will be converted to socks5 connector and tls dialer.
    * The `192.168.1.1:8080` corresponds to the node field `addr`.
    * The authentication `gost:gost` is converted to `connector.auth`.
	* The option `foo=bar` is converted to `connector.metadata` and `dialer.metadata`

## Environment Variables

GOST supports the following environment variables:

* GOST_LOGGER_LEVEL - log level.
* GOST_API - Web API service address.
* GOST_METRICS - Metrics service address.
* GOST_PROFILING - Profiling service address.

## Hot Reload

:material-tag: 3.1.0

The configuration can be reloaded by sending `SIGHUP` signal to the process. If there is an error in the configuration parsing, it will not be applied.

You can also reload the configuration by sending an HTTP POST request to `/config/reload` via the [web API](../tutorials/api/overview.md). This method of reloading will ignore the three global services: api, metrics and profiling.

!!! note "Configuration Priority"
	When using configuration files, command line parameters, and environment variables at the same time, the priority is: command line parameters > environment variables > configuration files. A configuration with a higher priority will override the corresponding configuration with a lower priority.
	
	When reloading the configuration, if the corresponding settings in the command line parameters or environment variables are modified or deleted in the configuration file, they will not be affected.

	For example, if you start the Web API service through the command line parameter `gost -C gost.yaml -api :18080`, when you modify or delete the `api` configuration in gost.yaml and reload it, the api service will still run on :18080.

## System Service

GOST supports running as a system service

### Windows

Create a Windows service through the `sc` command:

```bash
sc create gost binpath= "C:\gost.exe -L :8080" start= auto
```

### Linux

Create `/etc/systemd/system/gost.service` file:

```
[Unit]
Description=GO Simple Tunnel
After=network.target
Wants=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/gost -L=:8080
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable service:

```bash
systemctl enable gost
```

Start service:

```bash
systemctl start gost
```