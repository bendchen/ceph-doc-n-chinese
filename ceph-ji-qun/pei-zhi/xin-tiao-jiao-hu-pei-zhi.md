# 心跳交互配置

## 配置MONITOR / OSD交互

完成初始Ceph配置后，您可以部署并运行Ceph。当您执行诸如或的命令时， [Ceph监视器将](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-monitor)报告[Ceph存储集群](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-storage-cluster)的当前状态。Ceph监控器通过要求每个[Ceph OSD守护程序](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-osd-daemon)提供报告，以及从Ceph OSD守护程序接收有关其相邻Ceph OSD守护程序状态的报告来了解Ceph存储集群。如果Ceph Monitor没有收到报告，或者它收到Ceph Storage Cluster中的更改报告，则Ceph Monitor会更新[Ceph Cluster Map](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-cluster-map)的状态。`ceph healthceph -s`

Ceph为Ceph Monitor / Ceph OSD Daemon交互提供了合理的默认设置。但是，您可以覆盖默认值。以下各节描述了Ceph监视器和Ceph OSD守护程序如何交互以监视Ceph存储群集。

### OSD检查心跳

每个Ceph OSD守护程序都会以少于每6秒一次的随机间隔检查其他Ceph OSD守护程序的心跳。如果相邻的Ceph OSD守护进程在20秒的宽限期内未显示心跳，则Ceph OSD守护进程可以考虑相邻的Ceph OSD守护进程，`down`并将其报告回Ceph监视器，后者将更新Ceph群集映射。您可以通过在Ceph配置文件的 and 或部分下添加设置或在运行时设置值来更改此宽限期。`osd heartbeat grace[mon][osd][global]`

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-2ad4d285aa0fb0ed30f32eb7137638c5d045f92a.png)

### 屏上显示报告下的OSD 

默认情况下，来自不同主机的两个Ceph OSD守护程序必须`down`在Ceph监视器确认所报告的Ceph OSD守护程序为之前，向Ceph监视器报告另一个Ceph OSD守护程序`down`。但是，所有报告故障的OSD都有可能被托管在机架中，而该交换机的开关不良，无法连接到另一个OSD。为了避免这种错误警报，我们将对等的报告故障的节点视为整个集群上类似的落后集群的潜在“子集群”的代理。这显然并非在所有情况下都是正确的，但有时会帮助我们将宽限度校正本地化到不满意的系统子集。`mon osd reporter subtree level`用于将对等方按其在CRUSH映射中的共同祖先类型分组为“子集群”。默认情况下，只需要来自不同子树的两个报告即可报告另一个Ceph OSD Daemon `down`。您可以`down`通过在Ceph配置文件的部分下添加 和设置，或者通过在运行时设置值，来更改唯一子树的报告者数量以及将Ceph OSD守护进程报告给Ceph Monitor 所需的公共祖先类型。`mon osd min down reportersmon osd reporter subtree level[mon]`

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-a658cbf8a1a2182ed8758bff25438d7e4d35e017.png)

### OSD报告对等失败

如果Ceph OSD守护进程无法与其在其Ceph配置文件（或群集映射）中定义的任何Ceph OSD守护进程进行对等，它将每30秒ping Ceph Monitor以获得群集映射的最新副本。您可以通过 在Ceph配置文件的部分下添加设置或在运行时设置值来更改Ceph Monitor心跳间隔。`osd mon heartbeat interval[osd]`

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-db160722d860663ebdb46d66903688b0ee5ab1a8.png)

### OSD报告其状态

如果Ceph OSD守护进程未向Ceph监视器报告，则Ceph监视器将`down`在 经过之后将其视为Ceph OSD守护进程。当可报告事件（例如故障，放置组状态更改，更改或在5秒内启动）发生变化时，Ceph OSD守护进程会将报告发送到Ceph监视器 。您可以通过 在Ceph配置文件的部分下添加设置或在运行时设置值来更改Ceph OSD守护程序的最小报告间隔。Ceph OSD守护程序每120秒将报告发送到Ceph监视器，而不管是否发生任何显着变化。您可以通过在Ceph配置文件的部分下添加设置或在运行时设置值来更改Ceph Monitor报告间隔。`mon osd report timeoutup_thruosd mon report interval[osd]osd mon report interval max[osd]`

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-58a05b0a2682b211eef3e3e7f5c047fcac0e1f3f.png)

### 配置设置

修改心跳设置时，应将它们包括在`[global]` 配置文件的部分中。

#### 监视器设置

`mon osd min up ratio`描述

`up`在Ceph将标记Ceph OSD守护程序之前，Ceph OSD守护程序的最小比率`down`。类型

双默认

`.3`

`mon osd min in ratio`描述

`in`在Ceph将标记Ceph OSD守护程序之前，Ceph OSD守护程序的最小比率`out`。类型

双默认

`.75`

`mon osd laggy halflife`描述

延迟估算的秒数将衰减。类型

整数默认

`60*60`

`mon osd laggy weight`描述

延迟估计中新样本的权重衰减。类型

双默认

`0.3`

`mon osd laggy max interval`描述

`laggy_interval`延迟估算的最大值（以秒为单位）。Monitor使用一种自适应方法来评估`laggy_interval`某个OSD的。该值将用于计算该OSD的宽限时间。类型

整数默认

300

`mon osd adjust heartbeat grace`描述

如果设置为`true`，则Ceph将根据延迟的估计进行缩放。类型

布尔型默认

`true`

`mon osd adjust down out interval`描述

如果设置为`true`，则Ceph将基于延迟的估计进行缩放。类型

布尔型默认

`true`

`mon osd auto mark in`描述

Ceph会将所有正在启动的Ceph OSD守护进程标记为`in` Ceph存储集群。类型

布尔型默认

`false`

`mon osd auto mark auto out in`描述

Ceph将引导启动的自动`out` 将Ceph存储群集的Ceph OSD守护进程标记为`in`群集。类型

布尔型默认

`true`

`mon osd auto mark new in`描述

Ceph将把引导新的Ceph OSD守护进程标记为`in`Ceph存储集群。类型

布尔型默认

`true`

`mon osd down out interval`描述

Ceph在标记Ceph OSD守护程序之前等待的秒数 `down`（`out`如果它没有响应）。类型

32位整数默认

`600`

`mon osd down out subtree limit`描述

Ceph **不会** 自动标出的最小[CRUSH](https://docs.ceph.com/docs/nautilus/glossary/#term-crush)单位类型。例如，如果设置为且主机的所有OSD都关闭，则Ceph不会自动标记出这些OSD。`host`类型

串默认

`rack`

`mon osd report timeout`描述

声明无响应的Ceph OSD守护程序之前的宽限期（以秒为单位）`down`。类型

32位整数默认

`900`

`mon osd min down reporters`描述

报告Ceph OSD守护程序所需的最少数量的Ceph OSD守护 `down`程序。类型

32位整数默认

`2`

`mon osd reporter subtree level`描述

记录在哪个父级存储桶中。OSD发送故障报告以监视它们是否发现对等方没有响应。并在宽限期过后将监视器标记出报告的OSD，然后将其降低。类型

串默认

`host`

#### OSD设置

`osd heartbeat address`描述

用于心跳的Ceph OSD守护程序的网络地址。类型

地址默认

主机地址。

`osd heartbeat interval`描述

Ceph OSD守护程序多久一次对同等对象执行ping操作（以秒为单位）。类型

32位整数默认

`6`

`osd heartbeat grace`描述

Ceph OSD守护进程未显示Ceph存储群集认为的心跳的经过时间`down`。必须在\[mon\]和\[osd\]或\[global\]部分中都设置此设置，以便MON和OSD守护程序都可以读取它。类型

32位整数默认

`20`

`osd mon heartbeat interval`描述

如果Ceph OSD守护进程没有Ceph OSD守护进程对等体，则它多久ping一次Ceph监视器。类型

32位整数默认

`30`

`osd mon heartbeat stat stale`描述

停止报告心跳ping时间，该时间至今未更新。设置为零可禁用此操作。类型

32位整数默认

`3600`

`osd mon report interval`描述

Ceph OSD守护进程从启动或其他可报告事件开始等待之前等待的秒数。类型

32位整数默认

`5`

`osd mon ack timeout`描述

等待Ceph监视器确认统计请求的秒数。类型

32位整数默认

`30`

