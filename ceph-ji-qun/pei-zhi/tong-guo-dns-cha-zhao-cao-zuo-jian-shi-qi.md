# 通过DNS查找操作监视器

## 通过DNS查找操作监视器

从11.0.0版开始，RADOS支持通过DNS查找监视器。

这样，守护程序和客户端就不需要在其ceph.conf配置文件中使用_mon主机_配置指令。

使用DNS SRV TCP记录，客户端可以查找监视器。

这样可以减少客户端和监视器上的配置。使用DNS更新，可以使客户端和守护程序了解监视器拓扑中的更改。

默认情况下，客户端和守护程序将查找名为_ceph -mon_的TCP服务，该服务由_mon\_dns\_srv\_name_配置指令配置。

`mon dns srv name`描述

用于查询DNS的监视器主机/地址的服务名称类型

串默认

`ceph-mon`

### 例子

将DNS搜索域设置为_example.com时_，DNS区域文件可能包含以下元素。

首先，为监视器创建记录，即IPv4（A）或IPv6（AAAA）。

```text
mon1.example.com. AAAA 2001:db8::100
mon2.example.com. AAAA 2001:db8::200
mon3.example.com. AAAA 2001:db8::300
```

```text
mon1.example.com. A 192.168.0.1
mon2.example.com. A 192.168.0.2
mon3.example.com. A 192.168.0.3
```

现在有了这些记录，我们可以创建名称为_ceph-mon_的SRV TCP记录，指向三个Monitor。

```text
_ceph-mon._tcp.example.com. 60 IN SRV 10 60 6789 mon1.example.com.
_ceph-mon._tcp.example.com. 60 IN SRV 10 60 6789 mon2.example.com.
_ceph-mon._tcp.example.com. 60 IN SRV 10 60 6789 mon3.example.com.
```

在这种情况下，监视器运行在端口_6789上_，它们的优先级和权重分别为_10_和_60_。

客户端和守护程序中的当前实现将_仅_遵循SRV记录中设置的优先级，并且它们将仅连接到优先级最低的监视器。具有相同优先级的目标将被随机选择。

