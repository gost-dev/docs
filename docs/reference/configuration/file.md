# 配置文件

GOST配置文件使用yaml或json格式，完整的配置文件的结构如下：

=== "yaml格式"

    ```yaml
    services:
    - name: service-0
      addr: ":8080"
      interface: eth0
      sockopts:
        mark: 1
      admission: admission-0
      bypass: bypass-0
      resolver: resolver-0
      hosts: hosts-0
      handler:
        type: http
        auth:
          username: user
          password: pass
        auther: auther-0
        chain: chain-0
        retries: 1
        metadata: 
          foo: bar
          bar: baz
      listener:
        type: tcp
        auth:
          username: user
          password: pass
        auther: auther-0
        chain: chain-0
        tls:
          certFile: cert.pem
          keyFile: key.pem
          caFile: ca.pem
        metadata:
          abc: xyz
          def: 456
      forwarder:
        nodes:
        - name: target-0
          addr: 192.168.1.1:1234
        - name: target-1
          addr: 192.168.1.2:2345
        selector:
          strategy: rand
          maxFails: 1
          failTimeout: 30s

    chains:
    - name: chain-0
      selector:
        strategy: round
        maxFails: 1
        failTimeout: 30s
      hops:
      - name: hop-0
        interface: 192.168.1.2
        sockopts:
          mark: 1
        selector:
          strategy: rand
          maxFails: 3
          failTimeout: 60s
        bypass: bypass-0
        nodes:
        - name: node-0
          addr: ":1080"
          interface: eth1
          sockopts:
            mark: 1
          bypass: bypass-0
          connector:
            type: socks5
            auth:
              username: user
              password: pass
            metadata:
              foo: bar
          dialer:
            type: tcp
            auth:
              username: user
              password: pass
            tls:
              caFile: "ca.pem"
              secure: true
              serverName: "example.com"
            metadata:
              bar: baz 

    tls:
      certFile: "cert.pem"
      keyFile: "key.pem"
      caFile: "ca.pem"

    authers:
    - name: auther-0
      auths:
      - username: user1
        password: pass1
      - username: user2
        password: pass2

    admissions:
    - name: admission-0
      whitelist: false
      matchers:
      - 127.0.0.1
      - 192.168.0.0/16

    bypasses:
    - name: bypass-0
      whitelist: false
      matchers:
      - "*.example.com"
      - .example.org
      - 0.0.0.0/8

    resolvers:
    - name: resolver-0
      nameservers:
      - addr: udp://8.8.8.8:53
        chain: chain-0
        ttl: 60s
        prefer: ipv4
        clientIP: 1.2.3.4
        timeout: 3s
      - addr: tcp://1.1.1.1:53
      - addr: tls://1.1.1.1:853
      - addr: https://1.0.0.1/dns-query
        hostname: cloudflare-dns.com

    hosts:
    - name: hosts-0
      mappings:
      - ip: 127.0.0.1
        hostname: localhost
      - ip: 192.168.1.10
        hostname: foo.mydomain.org
        aliases:
        - foo
      - ip: 192.168.1.13
        hostname: bar.mydomain.org
        aliases:
        - bar
        - baz

    log:
      output: stderr
      level: debug
      format: json
      rotation:
        maxSize: 100
        maxAge: 10
        maxBackups: 3
        localTime: false
        compress: false

    profiling:
      addr: ":6060"
    
    api:
      addr: ":18080"
      pathPrefix: /api
      accesslog: true
      auth:
        username: user
        password: pass
      auther: auther-0

    metrics:
      addr: :9000
      path: /metrics
    ```

=== "json格式"

    ```json
    {
      "services": [
        {
          "name": "service-0",
          "addr": ":8080",
          "interface": "eth0",
          "admission": "admission-0",
          "bypass": "bypass-0",
          "resolver": "resolver-0",
          "hosts": "hosts-0",
          "handler": {
            "type": "http",
            "auth": {
              "username": "gost",
              "password": "gost"
            },
            "auther": "auther-0",
            "retries": 1,
            "chain": "chain-0",
            "metadata": {
              "bar": "baz",
              "foo": "bar"
            }
          },
          "listener": {
            "type": "tcp",
            "auth": {
              "username": "user",
              "password": "pass"
            },
            "auther": "auther-0",
            "chain": "chain-0",
            "tls": {
              "certFile": "cert.pem",
              "keyFile": "key.pem",
              "caFile": "ca.pem"
            },
            "metadata": {
              "abc": "xyz",
              "def": 456
            }
          },
          "forwarder": {
            "nodes": [
              {
                "name": "target-0",
                "addr": "192.168.1.1:1234"
              },
              {
                "name": "target-1",
                "addr": "192.168.1.2:2345"
              }
            ],
            "selector": {
              "strategy": "round",
              "maxFails": 1,
              "failTimeout": 30
            }
          }
        }
      ],
      "chains": [
        {
          "name": "chain-0",
          "selector": {
            "strategy": "round",
            "maxFails": 1,
            "failTimeout": 30
          },
          "hops": [
            {
              "name": "hop-0",
              "interface": "192.168.1.2",
              "selector": {
                "strategy": "rand",
                "maxFails": 3,
                "failTimeout": 60
              },
              "bypass": "bypass-0",
              "nodes": [
                {
                  "name": "node-0",
                  "addr": ":1080",
                  "interface": "eth1",
                  "bypass": "bypass-0",
                  "connector": {
                    "type": "socks5",
                    "auth": {
                      "username": "user",
                      "password": "pass"
                    },
                    "metadata": {
                      "foo": "bar"
                    }
                  },
                  "dialer": {
                    "type": "tcp",
                    "auth": {
                      "username": "user",
                      "password": "pass"
                    },
                    "tls": {
                      "caFile": "ca.pem",
                      "secure": true,
                      "serverName": "example.com"
                    },
                    "metadata": {
                      "bar": "baz"
                    }
                  }
                }
              ]
            }
          ]
        }
      ],
      "authers": [
        {
          "name": "auther-0",
          "auths": [
            {
              "username": "user1",
              "password": "pass1"
            },
            {
              "username": "user2",
              "password": "pass2"
            }
          ]
        }
      ],
      "admissions": [
        {
          "name": "admission-0",
          "whitelist": false,
          "matchers": [
            "127.0.0.1",
            "192.168.0.0/16"
          ]
        }
      ],
      "bypasses": [
        {
          "name": "bypass-0",
          "whitelist": false,
          "matchers": [
            "*.example.com",
            ".example.org",
            "0.0.0.0/8"
          ]
        }
      ],
      "resolvers": [
        {
          "name": "resolver-0",
          "nameservers": [
            {
              "addr": "udp://8.8.8.8:53",
              "chain": "chain-0",
              "prefer": "ipv4",
              "clientIP": "1.2.3.4",
              "ttl": 60,
              "timeout": 30
            },
            {
              "addr": "tcp://1.1.1.1:53"
            },
            {
              "addr": "tls://1.1.1.1:853"
            },
            {
              "addr": "https://1.0.0.1/dns-query",
              "hostname": "cloudflare-dns.com"
            }
          ]
        }
      ],
      "hosts": [
        {
          "name": "hosts-0",
          "mappings": [
            {
              "ip": "127.0.0.1",
              "hostname": "localhost"
            },
            {
              "ip": "192.168.1.10",
              "hostname": "foo.mydomain.org",
              "aliases": [
                "foo"
              ]
            },
            {
              "ip": "192.168.1.13",
              "hostname": "bar.mydomain.org",
              "aliases": [
                "bar",
                "baz"
              ]
            }
          ]
        }
      ],
      "tls": {
        "certFile": "cert.pem",
        "keyFile": "key.pem",
        "caFile": "ca.pem"
      },
      "log": {
        "output": "stderr",
        "level": "debug",
        "format": "json",
        "rotation": {
          "maxSize": 100,
          "maxAge": 10,
          "maxBackups": 3,
          "localTime": false,
          "compress": false
        }
      },
      "profiling": {
        "addr": ":6060",
        "enabled": true
      },
      "api": {
        "addr": ":18080",
        "pathPrefix": "/api",
        "accesslog": true,
        "auth": {
          "username": "user",
          "password": "password"
        },
        "auther": "auther-0"
      },
      "metrics": {
        "addr": ":9000",
        "path": "/metrics"
      }
    }
    ```

## 服务(Service)

`name` (string, required)
:    服务名称

`addr` (string, required)
:    服务地址

`interface` (string)
:    网络接口名或IP地址

`sockopts` (object)
:    Socket参数

`admission` (string, ref)
:    admission名称，引用`admissions.name`

`bypass` (string, ref)
:    bypass名称，引用`bypasses.name`

`resolver` (string, ref)
:    resolver名称，引用`resolvers.name`

`hosts` (string, ref)
:    hosts名称，对应`hosts.name`

`handler` (object, required)
:    处理器对象

`listener` (object, required)
:    监听器对象

`forwarder` (object)
:    转发器对象，用于端口转发

### 处理器(Handler)

`type` (string, required)
:    处理器类型

`auther` (string)
:    认证器名称，引用`authers.name`

`auth` (object)
:    认证信息，如果设置了`auther`，此字段无效。

`chain` (string, ref)
:    转发链名称，引用`chains.name`

`retries` (int, default=0)
:    请求处理失败后重试次数

`metadata` (map)
:    处理器实例相关参数

### 监听器(Listener)
    
`type` (string, required) 
:    监听器类型

`chain` (string, ref)
:    转发链名称，对应`chains.name`

`auther` (string)
:    认证器名称，引用`authers.name`

`auth` (object)
:    认证信息，如果设置了`auther`，此字段无效。

`tls` (object)
:    监听器实例TLS配置

`metadata` (map)
:    监听器实例相关参数

### 转发器(Forwarder)

`nodes` (objects)
:    转发目标节点列表

`selector` (object)
:    负载均衡策略

## 转发链(Chain)

`name` (string, required)
:    转发链名称

`selector` (object)
:    转发链层级节点选择器，用于负载均衡

`hops` (hop-list)
:    跳跃点列表

## 跳跃点(Hop)

`name` (string, required)
:    跳跃点名称

`interface` (string)
:    网络接口名或IP地址

`sockopts` (object)
:    Socket参数

`selector` (object)
:    跳跃点层级节点选择器，如果设置，则覆盖转发链层级选择器

`bypass` (string, ref)
:    bypass名称，引用`bypasses.name`

`nodes` (node-list)
:    节点列表

## 节点(Node)

`name` (string, required)
:    节点名称

`addr` (string, required)
:    节点地址

`interface` (string)
:    网络接口名或IP地址，如果设置，则会覆盖`hop.interface`

`sockopts` (object)
:    Socket参数，如果设置，则会覆盖`hop.sockopts`

`bypass` (string, ref)
:    bypass名称，引用`bypasses.name`。

`connector` (object)
:    连接器对象

`dialer` (object)
:    拨号器对象

### 连接器(Connector)

`type` (string, required)
:    连接器类型

`auth` (object)
:    认证信息

`metadata` (map)
:    连接器实例相关参数

### 拨号器(Dialer)

`type` (string, required)
:    拨号器类型

`auth` (object)
:    认证信息

`tls` (object)
:    TLS配置

`metadata` (map)
:    拨号器实例相关参数

## TLS

`certFile` (string)
:    证书公钥文件

`keyFile` (string)
:    证书私钥文件

`caFile` (string)
:    CA证书文件

`secure` (bool, default=false)
:    开启服务器证书和域名校验

`serverName` (string)
:    服务器域名，用于域名校验

## 认证器(Auther)

`name` (string, required)
:    名称

`auths` (list)
:    认证信息列表

## 认证信息(Auth)

`username` (string)
:    用户名

`password` (string)
:    密码

## 节点选择器(Selector)

`strategy` (string, default=round)
:   节点选择策略：

    * `round`, `rr` - 轮询
    * `random`, `rand` - 随机
    * `fifo` - 主备模式

`maxFails` (int, default=1)
:    节点连接最大失败次数

`failTimeout` (duration, default=30s)
:    节点失败标记超时时长

## 准入控制器(Admission)

`name` (string, required)
:    admission名称

`whitelist` (bool, default=false)
:    切换为白名单

`matchers` (strings)
:    地址列表，支持IP，CIDR

## 分流器(Bypass)

`name` (string, required)
:    bypass名称

`whitelist` (bool, default=false)
:    切换为白名单

`matchers` (strings)
:    地址列表，支持IP，CIDR，域名或域名通配符

## 域名解析器(Resolver)

`name` (string, required)
:    名称

`nameservers` (list)
:    域名服务列表

### 域名服务(Nameserver)

`addr` (string, required)
:    域名地址

`chain` (string, ref)
:    转发链名称，引用`chains.name`

`prefer` (string, default=ipv4)
:    IP地址类型优先级

     * `ipv4` - IPv4优先
     * `ipv6` - IPv6优先

`clientIP` (string)
:    客户端IP，设置后会开启ECS(EDNS Client Subnet)扩展功能。

`ttl` (duration)
:    DNS缓存有效期，默认使用DNS查询返回结果中的TTL。当设置为负值，则不使用缓存。

`timeout` (duration)

:     DNS请求超时时长

## 主机映射器(Hosts)

主机名-IP地址静态映射表

`name` (string, required)
:    映射表名称

`mappings` (list)
:    映射列表

### 映射列表项(mapping)

`ip` (string)
:    IP地址

`hostname` (string)
:    主机名

`aliases` (strings)
:    主机别名列表

## Socket参数(SockOpts)

`mark` (int)
:    Linux Socket SO_MARK参数选项

## 日志(log)

日志配置，设置日志级别，格式和输出方式。

`level` (string, default=info)
:    日志级别，支持的选项：`trace`，`debug`，`info`，`warn`，`error`，`fatal`。

`format` (string, default=json)
:    日志格式，支持的格式：`json`，`text`。

`output` (string, default=stderr)
:    日志输出方式：
  
     * `none` - 丢弃日志。
     * `stderr` - 标准错误流
     * `stdout` - 标准输出流
     * `/path/to/file` - 指定的文件路径

`rotation.maxSize` (int, default=100)
:    文件存储大小，单位为MB。

`rotation.maxAge` (int)
:    备份日志文件保存天数，默认不根据时间清理旧文件。

`rotation.maxBackups` (int)
:    备份日志文件数量，默认保存所有文件。

`rotation.localTime` (bool, default=false)
:    备份文件名是否使用本地时间格式。默认使用UTC时间。

`rotation.compress` (bool, default=false)
:    备份文件是否(使用gzip)压缩。

## Profiling

`addr` (string)
:    服务地址

`enabled` (bool, default=false)
:    是否开启

## API

`addr` (string)
:    WebAPI服务地址，设置后将开启WebAPI服务

`pathPrefix` (string)
:    设置API路径前缀

`accesslog` (bool, default=false)
:    开启API访问日志

`auth` (object)
:    认证信息，如果设置了`auther`，此字段无效。

`auther` (string)
:    认证器名称，引用`authers.name`

## Metrics

`addr` (string)
:    服务地址

`path` (string, default=/metrics)
:    访问路径