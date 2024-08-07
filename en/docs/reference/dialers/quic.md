# QUIC

名称: `quic`

状态： GA

QUIC拨号器使用[QUIC协议](https://github.com/lucas-clemente/quic-go)与QUIC服务建立数据通道。

=== "命令行"
    ```
    gost -L :8080 -F http+quic://:8443
    ```
=== "配置文件"
    ```yaml
    services:
   	- name: service-0
      addr: ":8080"
      handler:
        type: auto
        chain: chain-0
      listener:
        type: tcp
    chains:
    - name: chain-0
      hops:
      - name: hop-0
        nodes:
        - name: node-0
          addr: :8443
          connector:
            type: http
          dialer:
            type: quic
    ```

## 参数列表

`keepAlive` (bool, default=false)
:    enable keepalive.

`ttl` (duration, default=10s)
:    keepalive period.

`handshakeTimeout` (duration, default=5s)
:    handshake timeout

`maxIdleTimeout` (duration, default=30s)
:    max idle timeout

TLS配置请参考[TLS配置说明](/tutorials/tls/)。