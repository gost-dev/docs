# TUN

名称: `tun`

状态： GA

=== "命令行"
    ```
	gost -L tun://:8421?net=192.168.123.2/24
	```
=== "配置文件"
    ```yaml
	services:
	- name: service-0
	  addr: ":8421"
	  handler:
		type: tun
		metadata:
		  bufferSize: 1500
	  listener:
		type: tun
		metadata:
		  net: 192.168.123.2/24
	```

## 参数列表

`bufferSize` (int, default=1500)
:    数据读写缓冲区字节大小 

`keepAlive` (bool, default=false)
:    开启心跳，仅客户端有效

`ttl` (duration, default=10s)
:    心跳间隔时长，`keepAlive`为true时有效

!!! note "限制"
    TUN处理器只能与[TUN监听器](/reference/listeners/tun/)一起使用，构建基于TUN设备的VPN。