# HTTP前台应用

## HTTP前台应用

Ceph对象网关支持两个可以使用配置的嵌入式HTTP前端库`rgw_frontends`。有关 语法的详细信息，请参见“ [配置参考](https://docs.ceph.com/docs/nautilus/radosgw/config-ref) ”。

### Beast

Mimic版本中的新功能。

的`beast`前端使用Boost.Beast库HTTP分析和Boost.Asio的库用于异步网络I / O。

#### [选项](https://docs.ceph.com/docs/nautilus/radosgw/frontends/#id4)

`port` 和 `ssl_port`描述

设置ipv4和ipv6侦听端口号。可以像中一样多次指定。`port=80 port=8000`类型

整数默认

`80`

`endpoint` 和 `ssl_endpoint`描述

以形式设置侦听地址`address[:port]`，其中地址是点分十进制形式的IPv4地址字符串，或者是用方括号括起来的十六进制表示形式的IPv6地址。指定IPv6端点将仅侦听v6。可选端口默认为80 `endpoint`和443 `ssl_endpoint`。可以像中一样多次指定 。`endpoint=[::1] endpoint=192.168.0.100:8000`类型

整数默认

没有

`ssl_certificate`描述

用于启用SSL的端点的SSL证书文件的路径。类型

串默认

没有

`ssl_private_key`描述

用于启用SSL的端点的私钥文件的可选路径。如果未给出，则将该`ssl_certificate`文件用作私钥。类型

串默认

没有

`tcp_nodelay`描述

如果设置了套接字选项，将禁用连接上的Nagle算法，这意味着将尽快发送数据包，而不是等待完整的缓冲区或超时发生。

`1` 对所有套接字禁用Nagel算法。

`0` 保持默认值：启用Nagel算法。类型

整数（0或1）默认

0

`max_connection_backlog`描述

可选值，用于定义等待接受的连接队列的最大大小。如果未配置，`boost::asio::socket_base::max_connections`将使用from值。类型

整数默认

没有

### [CIVETWEB ](https://docs.ceph.com/docs/nautilus/radosgw/frontends/#id5)

Firefly版本中的新功能。

的`civetweb`前端使用Civetweb HTTP库，它是猫鼬的叉子。

#### [选项](https://docs.ceph.com/docs/nautilus/radosgw/frontends/#id6)

`port`描述

设置监听端口号。对于启用了SSL的端口，请添加 `s`后缀，如`443s`。要绑定特定的IPv4或IPv6地址，请使用形式`address:port`。多个端点可以通过分离`+`，如`127.0.0.1:8000+443s`，或通过提供多个选项中。`port=8000 port=443s`类型

串默认

`7480`

`num_threads`描述

设置Civetweb产生的用于处理传入HTTP连接的线程数。这有效地限制了前端可以服务的并发连接数。类型

整数默认

`rgw_thread_pool_size`

`request_timeout_ms`描述

Civetweb在放弃之前将等待更多传入数据的时间（以毫秒为单位）。类型

整数默认

`30000`

`ssl_certificate`描述

用于启用SSL的端口的SSL证书文件的路径。类型

串默认

没有

`access_log_file`描述

访问日志文件的路径。完整路径或相对于当前工作目录的路径。如果不存在（默认），则不记录访问。类型

串默认

`EMPTY`

`error_log_file`描述

错误日志文件的路径。完整路径或相对于当前工作目录的路径。如果不存在（默认），则不记录错误。类型

串默认

`EMPTY`

以下是设置了`/etc/ceph/ceph.conf`其中一些选项的文件示例：

```text
[client.rgw.gateway-node1]
rgw_frontends = civetweb request_timeout_ms=30000 error_log_file=/var/log/radosgw/civetweb.error.log access_log_file=/var/log/radosgw/civetweb.access.log
```

支持的选项的完整列表可在《[Civetweb用户手册》中找到](https://civetweb.github.io/civetweb/UserManual.html)。

### [通用选项](https://docs.ceph.com/docs/nautilus/radosgw/frontends/#id7)

一些前端选项是通用的，并且受所有前端支持：

`prefix`描述

在所有请求的URI中插入的前缀字符串。例如，仅swift前端可以提供uri前缀`/swift`。类型

串默认

没有

