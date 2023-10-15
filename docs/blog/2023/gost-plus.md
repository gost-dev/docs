---
template: blog.html
author: ginuerzh
author_gh_user: ginuerzh
read_time: 10min
publish_date: 2023-10-15 22:00
---

[反向代理隧道](https://gost.run/tutorials/reverse-proxy-advanced/)是GOST中新增的一个较大功能，同时也是一个很重要的功能，借助于反向代理和内网穿透，可以很方便的将内网Web服务暴露到公网，随时随地都能访问。

为了能够对此功能进行更全面的测试，同时也为了能够给需要临时暴露内网服务的用户提供一种快捷的方式，特公开推出`GOST.PLUS`公共反向代理测试服务。此服务面向所有用户开放，无需注册。

本服务以测试为主要目的，每个连接限速1Mbit/s，所有公共访问点均为临时访问点，有效期为24小时。

## 使用方法

假如本地有一个Web服务`192.168.1.1:80`需要临时暴露到公网，只需在本地机器上运行以下命令：

```bash
gost -L rtcp://:0/192.168.1.1:80 -F tunnel+wss://tunnel.gost.plus:443?tunnel.id=f8baa731-4057-4300-ab75-c4e603834f1b
```

!!! tip "隧道ID"
    每个隧道通过`tunnel.id`指定的隧道ID来唯一标识，每个隧道ID对应唯一的一个公共访问点。隧道ID是一个合法的UUID，可以通过UUID生成器来生成。

!!! caution
    隧道ID作为隧道和服务的唯一凭证，请妥善保管，防止泄露被滥用。

执行后如果隧道建立成功则会有以下日志输出：

```json
{"connector":"relay","dialer":"wss","hop":"hop-0","kind":"connector","level":"info","msg":"create tunnel on f1bbbb4aa9d9868a:0/tcp OK, tunnel=f8baa731-4057-4300-ab75-c4e603834f1b, connector=df4d62df-8b73-478a-96a2-26826e9cd675","node":"node-0","time":"2023-10-15T14:21:29.580Z"}
```

日志中的`f1bbbb4aa9d9868a`即为此服务的公共访问点，此时通过`https://f1bbbb4aa9d9868a.gost.plus`便可访问到内网的192.168.1.1:80服务。

## 自定义公共访问点

除了自动生成公共访问点，也可以通过自己指定访问点名称：

```bash
gost -L rtcp://hello/192.168.1.1:80 -F tunnel+wss://tunnel.gost.plus:443?tunnel.id=f8baa731-4057-4300-ab75-c4e603834f1b
```

上面的命令中指定了公共访问点为`test`，便可以通过`https://hello.gost.plus`来访问。

!!! note "绑定访问点"
    每个访问点在第一次使用时会注册并绑定到对应的隧道ID，绑定时长为24小时，在此期间其他隧道无法再次绑定并使用此访问点。当超时后绑定将失效，访问点可以再次绑定到不同的隧道。

更多的设置和使用方法请参考[反向代理](https://gost.run/tutorials/reverse-proxy/)。