# 代理转发和通道

在GOST中最常用到的两个功能是代理和转发，而这两个功能都需要一个载体，这个载体就是通道，或称为数据通道。这三个概念彼此之间有差异，又有类似的地方，甚至三者之间可以互相转换。

## 代理

通常意义上指的是代理协议，例如HTTP, SOCKS5等，是一种应用层数据交换协议，和一般的协议不同的是，服务端在这里充当中间人或代理者的角色，客户端的请求目标地址不是代理服务，而是通过代理协议协商的第三方服务。与第三方服务建立了连接后，代理服务在这里就只是一个数据转发的作用。

代理由于使用了特定协议，因此可以实现许多额外功能，例如身份认证，权限管理等。

## 转发

一般指的是端口转发或端口映射，在两个不同的端口之间建立一种联系，一般是单向映射，发送到其中一个端口的数据最终会原封不动的发到另一个端口，但反过来是不行的。转发可以不使用任何应用协议(纯TCP转发)，也可以使用特定的转发协议(Relay)。转发也可以看作是一种定向透明代理，客户端无法指定目标地址，甚至不需要区分转发服务与实际的目标服务。

## 通道

通道或叫数据通道，是指一个可以双向传输的数据流，通道的两端都可以同时收发数据，以实现全双工通信。
任何可以实现此功能定义的通信协议都可以用来作为数据通道，例如TCP/UDP协议，Websocket，HTTP/2，QUIC等，甚至代理和转发采用一些手段也可以被用作通道。

## 逻辑分层

虽然三者紧密相关，但在GOST中还是做了稍微严格的划分，一个GOST服务或节点被分为两层，数据通道层和数据处理层。数据通道层对应的是拨号器和监听器，数据处理层对应的是连接器，处理器和转发器，这里又根据是否使用转发器来区分是代理还是转发。

这是一种逻辑上的划分，具体到协议是没有这些限制的。例如HTTP/2协议既可以作为数据通道也可以作为代理，Relay协议兼具代理和转发功能，甚至HTTP也被用来作为数据通道(pht)。

=== "Relay代理模式"

    服务端
	```
	gost -L relay+wss://gost:gost@:8420
	```

	客户端
	```
	gost -L http://:8080 -F relay+wss://gost:gost@:8420
	```

	客户端使用TCP数据通道通过HTTP代理协议接收请求，使用Websocket数据通道通过relay协议转发给服务端处理，并开启认证。这里数据在两层代理(HTTP代理和Relay代理)之间进行传输。

=== "Relay转发模式"

    服务端
	```
	gost -L relay+wss://:8420/:18080
	```

    客户端
	```
	gost -L tcp://:8080 -F relay+wss://:8420
	```

	客户端使用TCP数据通道进行转发，再使用Websocket数据通道通过relay协议进行转发，服务端最终将数据发送给18080端口。这里使用了两层转发，最终将客户端的8080端口映射到服务端的18080端口，访问客户端的8080端口和直接访问服务端的18080端口是没有区别的。

## 协同效应

代理和转发都可以单独工作，但把三者组合使用会产生一些不同的效果。

### 使用代理进行端口转发

某些情况下，端口转发中的两个端口之间不能直接建立连接，这时可以通过转发链利用代理服务来进行中转。

```
gost -L tcp://:8080/192.168.1.1:80 -F http://192.168.1.2:8080
```

8080端口通过转发链中的192.168.1.2:8080代理节点间接映射到192.168.1.1:80。

### 添加数据通道

通过转发可以为已存在的服务动态增加数据通道。

#### HTTP-over-TLS

```
gost -L tls://:8443/:8080 -L http://:8080
```

通过使用TLS数据通道的端口转发，给8080端口的HTTP代理服务增加了TLS加密数据通道。

此时8443端口等同于：

```
gost -L https://:8443
```

#### Shadowsocks-over-KCP

```
gost -L kcp://:8338/:8388 -L ss://:8388
```

通过使用KCP数据通道的端口转发，给8388端口的shadowsocks代理服务增加了KCP数据通道。

此时8338端口等同于：

```
gost -L ss+kcp://:8338
```

### 去除数据通道

与上面的例子相反，也可以通过转发将现有服务的数据通道去除。

#### HTTPS to HTTP

将HTTPS代理服务转成HTTP代理服务

```
gost -L https://:8443
```

```
gost -L tcp://:8080 -F forward+tls://:8443
```

此时8080端口等同于：
```
gost -L http://:8080
```

#### Shadowsocks-over-KCP to Shadowsocks

```
gost -L ss+kcp://:8338
```

```
gost -L ss://:8388 -F forward+kcp://:8338
```

此时8388端口等同于：
```
gost -L ss://:8388
```
