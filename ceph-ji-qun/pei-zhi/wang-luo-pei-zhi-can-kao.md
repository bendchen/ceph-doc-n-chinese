# 网络配置参考

## 网络配置参考

网络配置对于构建高性能[Ceph存储集群](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-storage-cluster)至关重要 。Ceph存储群集不代表[Ceph客户端](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-client)执行请求路由或调度。相反，Ceph客户端直接向Ceph OSD守护程序发出请求。Ceph OSD守护程序代表Ceph客户端执行数据复制，这意味着复制和其他因素会在Ceph存储群集网络上施加额外的负载。

我们的快速入门配置提供了一个简单的[Ceph配置文件](https://docs.ceph.com/docs/nautilus/start/quick-ceph-deploy/#create-a-cluster)，该[文件仅](https://docs.ceph.com/docs/nautilus/start/quick-ceph-deploy/#create-a-cluster)设置监视器IP地址和守护程序主机名。除非您指定群集网络，否则Ceph将假定为单个“公共”网络。Ceph只能在公共网络上正常运行，但是大型集群中的第二个“集群”网络可能会显着改善性能。

可以使用两个网络来运行Ceph存储群集：一个公共（前端）网络和一个群集（后端）网络。但是，这种方法使网络配置（包括硬件和软件）复杂化，并且通常不会对整体性能产生重大影响。因此，我们通常建议双NIC系统在同一网络上配置两个IP或绑定。

如果尽管复杂，但仍然希望使用两个网络，则每个 [Ceph节点](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-node)将需要具有多个NIC。有关其他详细信息，请参见[硬件建议-网络](https://docs.ceph.com/docs/nautilus/start/hardware-recommendations#networks)。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-2452ee22ef7d825a489a08e0b935453f2b06b0e6.png)

### IP表

默认情况下，守护程序[绑定](https://docs.ceph.com/docs/nautilus/rados/configuration/network-config-ref/#bind)到该`6800:7300`范围内的端口。您可以自行决定配置此范围。在配置IP表之前，请检查默认`iptables`配置。

> 须藤iptables -L

某些Linux发行版包含一些规则，这些规则拒绝来自所有网络接口的除SSH之外的所有入站请求。例如：

```text
REJECT all -- anywhere anywhere reject-with icmp-host-prohibited
```

首先，您将需要在公共网络和群集网络上都删除这些规则，并在准备加强Ceph节点上的端口时将它们替换为适当的规则。

#### 监控IP表

Ceph的监视监听端口`3300`和`6789`默认。此外，Ceph Monitors始终在公共网络上运行。当您使用下面的例子中添加规则，请确保您更换`{iface}`与公共网络接口（例如`eth0`， `eth1`等），`{ip-address}`与公网的IP地址，`{netmask}`网络掩码为公共网络。

```text
sudo iptables -A INPUT -i {iface} -p tcp -s {ip-address}/{netmask} --dport 6789 -j ACCEPT
```

#### MDS和管理器IP表

一个[Ceph的元数据服务器](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-metadata-server)或[头孢管理器](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-manager)监听公共网络上的第一个可用端口在端口6800开始注意，这种行为是不确定的，所以，如果你正在运行在同一台主机上的多个OSD或MDS，或者重新启动守护程序在很短的时间内，它们将绑定到更高的端口。默认情况下，您应该打开整个6800-7300系列。当您使用下面的例子中添加规则，请确保您更换`{iface}`与公共网络接口（例如`eth0`， `eth1`等），`{ip-address}`与公网的IP地址，并`{netmask}`与公众网络的子网掩码。

例如：

```text
sudo iptables -A INPUT -i {iface} -m multiport -p tcp -s {ip-address}/{netmask} --dports 6800:7300 -j ACCEPT
```

#### OSD IP表

默认情况下，Ceph OSD守护程序[绑定](https://docs.ceph.com/docs/nautilus/rados/configuration/network-config-ref/#bind)到Ceph节点上从端口6800开始的第一个可用端口。请注意，此行为不是确定性的，因此，如果您在同一主机上运行多个OSD或MDS，或者重新启动守护程序在很短的时间内，这些守护程序将绑定到更高的端口。一个Ceph节点上的每个Ceph OSD守护程序最多可以使用四个端口：

1. 一种用于与客户和监视器对话。
2. 一种用于将数据发送到其他OSD。
3. 两个用于每个接口上的心跳。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-ec480570ad40651af4ef00fb5ad7735f94a4089c.png)

当守护程序发生故障并在不松开端口的情况下重新启动时，重新启动的守护程序将绑定到新端口。您应该打开整个6800-7300端口范围以应对这种可能性。

如果您设置了单独的公用网络和群集网络，则必须同时为公用网络和群集网络添加规则，因为客户端将使用公用网络进行连接，而其他Ceph OSD守护程序将使用群集网络进行连接。当您使用下面的例子中添加规则，请确保您更换 `{iface}`与网络接口（如`eth0`，`eth1`等） `{ip-address}`的IP地址，并`{netmask}`与公众或群集网络的子网掩码。例如：

```text
sudo iptables -A INPUT -i {iface}  -m multiport -p tcp -s {ip-address}/{netmask} --dports 6800:7300 -j ACCEPT
```

小费 

如果您在与Ceph OSD守护程序相同的Ceph节点上运行Ceph元数据服务器，则可以合并公共网络配置步骤。

### 头孢网络

要配置Ceph网络，您必须将网络配置添加到`[global]`配置文件的 部分。我们5分钟的快速入门提供了一个简单的[Ceph配置文件](https://docs.ceph.com/docs/nautilus/start/quick-ceph-deploy/#create-a-cluster)，该[文件](https://docs.ceph.com/docs/nautilus/start/quick-ceph-deploy/#create-a-cluster)假设一个公共网络，客户端和服务器位于同一网络和子网上。Ceph只能在公共网络上正常运行。但是，Ceph允许您建立更具体的条件，包括公共网络的多个IP网络和子网掩码。您还可以建立一个单独的群集网络来处理OSD心跳，对象复制和恢复流量。不要将您在配置中设置的IP地址与网络客户端可能用来访问您的服务的面向公众的IP地址混淆。典型的内部IP网络通常是`192.168.0.0`或`10.0.0.0`。

小费 

如果为公用网络或群集网络指定了多个IP地址和子网掩码，则网络中的子网必须能够相互路由。此外，请确保在IP表中包括每个IP地址/子网，并在必要时为其打开端口。

注意 

Ceph 对子网（例如）使用[CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)表示法`10.0.0.0/24`。

配置网络后，可以重新启动集群或重新启动每个守护程序。Ceph守护程序是动态绑定的，因此，如果您更改网络配置，则不必立即重新启动整个集群。

#### 公共网络

要配置公共网络，请将以下选项添加到`[global]` Ceph配置文件的部分。

```text
[global]
        # ... elided configuration
        public network = {public-network/netmask}
```

#### 群集网络

如果声明群集网络，则OSD将在群集网络上路由心跳，对象复制和恢复流量。与使用单个网络相比，这可以提高性能。要配置群集网络，请将以下选项添加到`[global]`Ceph配置文件的部分。

```text
[global]
        # ... elided configuration
        cluster network = {cluster-network/netmask}
```

我们宁愿集群网络是**不是**从公共网络或互联网为增加安全性访问。

### CEPH守护程序

每个监视守护程序都配置为绑定到特定的IP地址。这些地址通常由您的部署工具配置。Ceph系统中的其他组件通过通常在文件部分中指定的配置选项来发现监视器。`mon host[global]ceph.conf`

```text
[global]
    mon host = 10.0.0.2, 10.0.0.3, 10.0.0.4
```

该值可以是IP地址列表，也可以是通过DNS查找的名称。对于具有多个A或AAAA记录的DNS名称，将对所有记录进行探测以发现监视器。一旦达到一台监视器，便会发现所有其他当前监视器，因此配置选项仅需要足够最新，这样客户端就可以访问当前在线的一台监视器。`mon hostmon host`

MGR，OSD和MDS守护程序将绑定到任何可用地址，并且不需要任何特殊配置。但是，可以使用（和/或（对于OSD守护程序，是））配置选项为它们指定要绑定的特定IP地址。例如，`public addrcluster addr`

```text
[osd.0]
        public addr = {host-public-ip-address}
        cluster addr = {host-cluster-ip-address}
```

两个网络群集中的一个NIC OSD

通常，我们不建议在具有两个网络的群集中部署具有单个NIC的OSD主机。但是，您可以通过在Ceph配置文件的部分中添加一个条目来强制OSD主机在公共网络上运行来完成此操作，该条目 指的是带有一个NIC的OSD的编号。此外，公共网络和群集网络必须能够相互路由流量，出于安全考虑，我们不建议这样做。`public addr[osd.n]n`

### 网络配置设置

不需要网络配置设置。除非您专门配置群集网络，否则Ceph假定所有主机都在公共网络上运行。

#### 公共网络

公用网络配置允许您专门定义公用网络的IP地址和子网。您可以使用 特定守护程序的设置来专门分配静态IP地址或覆盖设置。`public networkpublic addr`

`public network`描述

公用（前端）网络（例如`192.168.0.0/24`）的IP地址和子网掩码。设置为`[global]`。您可以指定逗号分隔的子网。类型

`{ip-address}/{netmask} [, {ip-address}/{netmask}]`需要

没有默认

不适用

`public addr`描述

公共（前端）网络的IP地址。为每个守护程序设置。类型

IP地址需要

没有默认

不适用

#### 群集网络

群集网络配置允许您声明群集网络，并专门为群集网络定义IP地址和子网。您可以 使用特定OSD守护程序的设置专门分配静态IP地址或覆盖设置。`cluster networkcluster addr`

`cluster network`描述

群集（后端）网络（例如`10.0.0.0/24`）的IP地址和网络掩码。设置为`[global]`。您可以指定逗号分隔的子网。类型

`{ip-address}/{netmask} [, {ip-address}/{netmask}]`需要

没有默认

不适用

`cluster addr`描述

群集（后端）网络的IP地址。为每个守护程序设置。类型

地址需要

没有默认

不适用

#### 绑定

绑定设置可设置Ceph OSD和MDS守护进程使用的默认端口范围。预设范围是`6800:7300`。确保[IP表](https://docs.ceph.com/docs/nautilus/rados/configuration/network-config-ref/#ip-tables)配置允许您使用配置的端口范围。

您还可以使Ceph守护程序绑定到IPv6地址而不是IPv4地址。

`ms bind port min`描述

OSD或MDS守护程序将绑定到的最小端口号。类型

32位整数默认

`6800`需要

没有

`ms bind port max`描述

OSD或MDS守护程序将绑定到的最大端口号。类型

32位整数默认

`7300`需要

没有。

`ms bind ipv6`描述

使Ceph守护程序能够绑定到IPv6地址。目前信使_要么_使用IPv4或IPv6，但不能两者都做。类型

布尔型默认

`false`需要

没有

`public bind addr`描述

在某些动态部署中，Ceph MON守护程序可能会在本地绑定到一个IP地址，该IP地址与 广告给网络中其他对等方的地址不同。环境必须确保正确设置了路由规则。如果设置为，则Ceph MON守护程序将在本地绑定到它，并 在monmap中使用以将其地址通告给对等方。此行为仅限于MON守护程序。`public addrpublic bind addrpublic addr`类型

IP地址需要

没有默认

不适用

#### TCP 

默认情况下，Ceph禁用TCP缓冲。

`ms tcp nodelay`描述

Ceph启用后，每个请求都会立即发送（不进行缓冲）。禁用[Nagle的算法会](https://en.wikipedia.org/wiki/Nagle's_algorithm) 增加网络流量，这可能会导致延迟。如果您遇到大量小数据包，则可以尝试禁用。`ms tcp nodelayms tcp nodelay`类型

布尔型需要

没有默认

`true`

`ms tcp rcvbuf`描述

网络连接的接收端上的套接字缓冲区的大小。默认禁用。类型

32位整数需要

没有默认

`0`

`ms tcp read timeout`描述

如果客户端或守护程序向另一个Ceph守护程序发出请求，并且没有删除未使用的连接，则 在指定的秒数后，该连接会将连接定义为空闲。`ms tcp read timeout`类型

无符号64位整数需要

没有默认

`900` 15分钟。

