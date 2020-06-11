# MON配置

## 监视器配置参考

了解如何配置[Ceph监视器](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-monitor)是构建可靠的[Ceph存储群集](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-storage-cluster)的重要部分。**所有Ceph存储集群都至少具有一台监视器**。监视器配置通常保持相当一致，但是您可以在群集中添加，删除或替换监视器。有关详细信息，请参见 [添加/删除监视器](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-mons)和[添加/删除监视器（ceph-deploy）](https://docs.ceph.com/docs/nautilus/rados/deployment/ceph-deploy-mon)。

### 背景

Ceph Monitors维护[集群图](https://docs.ceph.com/docs/nautilus/glossary/#term-cluster-map)的“主副本” ，这意味着 [Ceph Client](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-client)只需连接到一个Ceph Monitor并检索当前集群图，即可确定所有Ceph Monitor，Ceph OSD守护程序和Ceph Metadata Server的位置。在Ceph客户端可以读取或写入Ceph OSD守护程序或Ceph元数据服务器之前，它们必须首先连接到Ceph监视器。有了集群图的当前副本和CRUSH算法，Ceph客户端可以计算任何对象的位置。计算对象位置的能力使Ceph客户端可以直接与Ceph OSD守护进程通信，这是Ceph的高可伸缩性和性能的一个非常重要的方面。有关其他详细信息，请参见 [可伸缩性和高可用性](https://docs.ceph.com/docs/nautilus/architecture#scalability-and-high-availability)。

Ceph Monitor的主要作用是维护集群图的主副本。Ceph Monitors还提供身份验证和日志记录服务。Ceph Monitors将Monitor服务中的所有更改都写入单个Paxos实例，并且Paxos将更改写入键/值存储，以实现强一致性。在同步操作期间，Ceph Monitors可以查询集群映射的最新版本。Ceph Monitors利用键/值存储的快照和迭代器（使用leveldb）执行存储范围的同步。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-ae8fc6ae5b4014f064a0bed424507a7a247cd113.png)

自版本版本起已弃用： 0.58

在0.58和更早版本的Ceph中，Ceph Monitors为每个服务使用Paxos实例，并将地图存储为文件。

#### 簇映射

集群地图是地图的组合，包括监视器地图，OSD地图，放置组地图和元数据服务器地图。集群图跟踪了许多重要的事情：哪些进程是`in`Ceph存储集群；哪些进程是Ceph存储集群？作为`in`Ceph存储群集的哪些进程正在`up`运行，或者`down`；展示位置组是否为，`active`或`inactive`和 `clean`或处于其他状态；以及反映集群当前状态的其他详细信息，例如存储空间的总量和使用的存储量。

当集群状态发生重大变化时（例如，Ceph OSD守护进程关闭，放置组进入降级状态等），集群图将更新以反映集群的当前状态。此外，Ceph Monitor还维护集群的先前状态的历史记录。监视器映射，OSD映射，放置组映射和元数​​据服务器映射每个都维护其映射版本的历史记录。我们称每个版本为“时代”。

在操作Ceph Storage Cluster时，跟踪这些状态是系统管理职责的重要组成部分。有关其他详细信息，请参见[监视群集](https://docs.ceph.com/docs/nautilus/rados/operations/monitoring) 以及[监视OSD和PG](https://docs.ceph.com/docs/nautilus/rados/operations/monitoring-osd-pg)。

#### 监控仲裁

我们的配置ceph部分提供了一个简单的[Ceph配置文件](https://docs.ceph.com/docs/nautilus/rados/configuration/ceph-conf/#monitors)，该[文件](https://docs.ceph.com/docs/nautilus/rados/configuration/ceph-conf/#monitors)为测试集群中的一台监视器提供了[配置](https://docs.ceph.com/docs/nautilus/rados/configuration/ceph-conf/#monitors)。群集可以在单个监视器上正常运行；但是，**单个监视器就是一个单故障点**。为了确保生产Ceph存储群集中的高可用性，您应该使用多个监视器运行Ceph，以使单个监视器的故障**不会** 导致整个群集崩溃。

当一个Ceph存储群集运行多个Ceph监视器以实现高可用性时，Ceph监视器使用[Paxos](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)建立有关主群集映射的共识。达成共识需要大多数监视器运行以建立仲裁集群共识的法定人数（例如1； 3中的2； 5中的3； 6中的4；等等）。

`mon force quorum join`描述

即使先前已将其从地图中删除，也强制监视器加入仲裁类型

布尔型默认

`False`

#### 一致性

当您将监视器设置添加到Ceph配置文件时，您需要了解Ceph Monitors的某些架构方面。当在集群中发现另一个Ceph Monitor时，**Ceph对** Ceph Monitor **施加严格的一致性要求**。而Ceph客户端和其他Ceph守护程序使用Ceph配置文件来发现监视器，监视器使用监视器映射（monmap）而不是Ceph配置文件来发现彼此。

当在Ceph存储群集中发现其他Ceph监视器时，Ceph监视器始终引用monmap的本地副本。使用monmap而不是Ceph配置文件可以避免可能破坏群集的错误（例如，在`ceph.conf`指定监视器地址或端口时输入错误）。由于监视器使用monmap进行发现，并且它们与客户端和其他Ceph守护程序共享monmap，**因此monmap为监视器提供了严格的保证，即它们的共识是有效的。**

严格的一致性也适用于对monmap的更新。与Ceph Monitor上的任何其他更新一样，对monmap的更改始终通过称为[Paxos](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)的分布式共识算法运行。Ceph监视器必须同意对monmap的每次更新，例如添加或删除Ceph Monitor，以确保仲裁中的每个监视器都具有相同版本的monmap。对monmap的更新是增量更新，因此Ceph Monitors具有最新的议定版本和一组先前版本。维护历史记录可以使具有旧版本monmap的Ceph Monitor赶上Ceph存储集群的当前状态。

如果Ceph Monitors通过Ceph配置文件而不是通过monmap彼此发现，则将带来其他风险，因为Ceph配置文件不会自动更新和分发。Ceph监视器可能会无意中使用了较旧的Ceph配置文件，无法识别Ceph监视器，无法达到法定人数或出现了 [Paxos](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)无法准确确定系统当前状态的情况。

#### 引导监视器

在大多数配置和部署情况下，部署Ceph的工具可能会通过为您生成监视器映射（例如`ceph-deploy`等）来帮助引导Ceph监视器 。Ceph监视器需要一些明确的设置：

* **文件系统ID**：`fsid`是对象存储库的唯一标识符。由于可以在同一硬件上运行多个集群，因此在引导监视器时必须指定对象存储的唯一ID。部署工具通常会为您执行此操作（例如，`ceph-deploy`可以调用诸如之类的工具`uuidgen`），但是您也可以`fsid`手动指定。
* **监视器ID**：监视器ID是分配给集群中每个监视器的唯一ID。它是字母数字值，并且按照惯例标识符通常遵循一个字母增量（例如，`a`，`b`等等）。这可以在一个Ceph的配置文件中设置（例如，`[mon.a]`，`[mon.b]`使用等），由部署工具，或`ceph`命令行。
* **密钥**：监视器必须具有秘密密钥。诸如此类的部署工具 `ceph-deploy`通常会为您执行此操作，但是您也可以手动执行此步骤。有关详细信息，请参见[监控器钥匙圈](https://docs.ceph.com/docs/nautilus/dev/mon-bootstrap#secret-keys)。

有关引导的更多详细信息，请参见[引导监视器](https://docs.ceph.com/docs/nautilus/dev/mon-bootstrap)。

### 配置监视器

要将配置设置应用于整个集群，请在下输入配置设置`[global]`。要将配置设置应用于群集中的所有监视器，请在下输入配置设置`[mon]`。要将配置设置应用于特定的监视器，请指定监视器实例（例如`[mon.a]`）。按照约定，监视器实例名称使用字母表示法。

```text
[global]

[mon]

[mon.a]

[mon.b]

[mon.c]
```

#### 最低配置

通过Ceph配置文件为Ceph监视器设置的最低监视器设置包括每个监视器的主机名和监视器地址。您可以在`[mon]`特定监视器的条目下或条目下配置它们。

```text
[global]
        mon host = 10.0.0.2,10.0.0.3,10.0.0.4
```

```text
[mon.a]
        host = hostname1
        mon addr = 10.0.0.10:6789
```

有关详细信息，请参见[网络配置参考](https://docs.ceph.com/docs/nautilus/rados/configuration/network-config-ref)。

注意 

监视器的最低配置假定部署工具会为您生成`fsid`和`mon.`密钥。

一旦部署了Ceph集群，就**不应**更改监视器的IP地址。但是，如果决定更改显示器的IP地址，则必须遵循特定的步骤。有关详细信息，请参见[更改显示器的IP地址](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-mons#changing-a-monitor-s-ip-address)。

客户端也可以使用DNS SRV记录找到监视器。有关详细信息，请参见[通过DNS监视查找](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-lookup-dns)。

#### 集群ID 

每个Ceph存储集群都有一个唯一的标识符（`fsid`）。如果指定，它通常出现在`[global]`配置文件的部分下。部署工具通常会生成`fsid`并将其存储在监视器映射中，因此该值可能不会出现在配置文件中。这样`fsid`就可以在同一硬件上为多个集群运行守护程序。

`fsid`描述

集群ID。每个群集一个。类型

UUID需要

是。默认

不适用 如果未指定，则可能由部署工具生成。

注意 

如果使用为您执行此操作的部署工具，请不要设置此值。

#### 初始成员

我们建议使用至少三个Ceph Monitor运行生产的Ceph Storage Cluster，以确保高可用性。运行多个监视器时，可以指定必须是群集成员的初始监视器才能建立仲裁。这可以减少群集联机所需的时间。

```text
[mon]
        mon initial members = a,b,c
```

`mon initial members`描述

启动期间群集中初始监视器的ID。如果指定，则Ceph需要奇数个监视器来形成初始仲裁（例如3个）。类型

串默认

没有

注意 

一个_大多数_集群中的显示器必须能够到达对方，以建立仲裁。您可以减少监视器的初始数量，以使用此设置建立仲裁。

#### 数据

Ceph提供了Ceph Monitors存储数据的默认路径。为了在生产Ceph存储群集中获得最佳性能，我们建议在Ceph OSD守护程序的单独主机和驱动器上运行Ceph Monitor。由于leveldb `mmap()`用于写入数据，因此Ceph监视器经常将其数据从内存刷新到磁盘，如果数据存储与OSD守护程序位于同一位置，则可能会干扰Ceph OSD守护程序的工作负载。

在Ceph 0.58及更早版本中，Ceph监视器将其数据存储在文件中。这种方法允许用户检查与像常用工具监测数据`ls` 和`cat`。但是，它不能提供强大的一致性。

在Ceph 0.59和更高版本中，Ceph监视器将其数据存储为键/值对。Ceph监视器需要[ACID](https://en.wikipedia.org/wiki/ACID)事务。使用数据存储可防止通过Paxos从运行损坏的版本中恢复Ceph Monitor，并且可以在一个原子批处理中进行多个修改操作，还有其他优点。

通常，我们不建议更改默认数据位置。如果您修改默认位置，我们建议您通过`[mon]`在配置文件部分中进行设置，使其在Ceph Monitors中保持一致。

`mon data`描述

监视器的数据位置。类型

串默认

`/var/lib/ceph/mon/$cluster-$id`

`mon data size warn`描述

`HEALTH_WARN`当监视器的数据存储空间超过15GB时，在群集日志中发出a 。类型

整数默认

15 \* 1024 \* 1024 \* 1024 \*

`mon data avail warn`描述

`HEALTH_WARN`当监视器的数据存储的可用磁盘空间小于或等于此百分比时，在群集日志中发出一个。类型

整数默认

30

`mon data avail crit`描述

`HEALTH_ERR`当监视器的数据存储的可用磁盘空间小于或等于此百分比时，在群集日志中发出一个。类型

整数默认

5

`mon warn on cache pools without hit sets`描述

`HEALTH_WARN`如果高速缓存池未`hit_set_type`配置值，则在群集日志中发出a 。有关更多详细信息，请参见[hit\_set\_type](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#hit-set-type)。类型

布尔型默认

真正

`mon warn on crush straw calc version zero`描述

`HEALTH_WARN`如果CRUSH `straw_calc_version`为零，则在群集日志中 发出a 。有关详细信息，请参见 [CRUSH映射可调项](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map/#crush-map-tunables)。类型

布尔型默认

真正

`mon warn on legacy crush tunables`描述

`HEALTH_WARN`如果CRUSH可调参数过旧（早于`mon_min_crush_required_version`），则在集群日志中发出a类型

布尔型默认

真正

`mon crush min required version`描述

集群所需的最低可调配置文件版本。有关详细信息，请参见 [CRUSH映射可调项](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map/#crush-map-tunables)。类型

串默认

`firefly`

`mon warn on osd down out interval zero`描述

`HEALTH_WARN`如果为零，则在群集日志中 发出a 。在领导者上将此选项设置为零的行为很像标志。很难弄清楚没有设置标志但行为相同的群集出了什么问题 ，因此在这种情况下我们会报告警告。`mon osd down out intervalnooutnoout`类型

布尔型默认

真正

`mon warn on slow ping ratio`描述

`HEALTH_WARN`如果OSD之间的心跳超过 ，则在群集日志中发出a 。默认值为5％。`mon warn on slow ping ratioosd heartbeat grace`类型

浮动默认

`0.05`

`mon warn on slow ping time`描述

用特定值覆盖。如果OSD之间的任何心跳超过 毫秒，请在群集日志中发出a 。默认值为0（禁用）。`mon warn on slow ping ratioHEALTH_WARNmon warn on slow ping time`类型

整数默认

`0`

`mon warn on pool no redundancy`描述

`HEALTH_WARN`如果没有配置任何副本池，则在群集日志中发出一个。类型

布尔型默认

`True`

`mon cache target full warn ratio`描述

池`cache_target_full`与 `target_max_object`我们开始警告之间的位置类型

浮动默认

`0.66`

`mon health data update interval`描述

监控中的法定人数（以秒为单位）与对等方共享其健康状态。（负数禁用它）类型

浮动默认

`60`

`mon health to clog`描述

启用定期向集群日志发送运行状况摘要。类型

布尔型默认

真正

`mon health to clog tick interval`描述

监控器将运行状况摘要发送到群集日志的频率（以秒为单位）（非正数会将其禁用）。如果当前运行状况摘要为空或与上次相同，则监控器不会将其发送到集群日志。类型

浮动默认

60.000000

`mon health to clog interval`描述

监控器将运行状况摘要发送到群集日志的频率（以秒为单位）（非正数会将其禁用）。无论摘要是否更改，Monitor都会始终将摘要发送到群集日志。类型

整数默认

3600

#### 存储容量

当Ceph存储群集接近其最大容量（即）时，Ceph会阻止您写入或读取Ceph OSD守护程序，以防止数据丢失。因此，让生产Ceph Storage Cluster达到其完整比率并不是一个好习惯，因为它会牺牲高可用性。默认的完整比率为或容量的95％。对于具有少量OSD的测试群集而言，这是一个非常激进的设置。`mon osd full ratio.95`

小费 

监视群集时，请注意与`nearfull`比率有关的警告 。这意味着，如果一个或多个OSD发生故障，则某些OSD发生故障可能会导致临时服务中断。考虑添加更多OSD以增加存储容量。

测试群集的常见情况包括系统管理员从Ceph存储群集中删除Ceph OSD守护程序以观察群集的重新平衡。然后，删除另一个Ceph OSD守护程序，依此类推，直到Ceph存储群集最终达到完整比率并锁定为止。即使是测试集群，我们也建议您进行一些容量规划。通过规划，您可以估算需要多少备用容量以保持高可用性。理想情况下，您要计划一系列Ceph OSD守护程序故障，使群集可以恢复到某种状态而无需立即替换那些Ceph OSD守护程序。您可以在某种状态下运行集群，但这对于正常操作条件而言并不理想。`active + cleanactive + degraded`

下图描述了一个简单的Ceph存储群集，其中包含33个Ceph节点，每个主机有一个Ceph OSD守护程序，每个Ceph OSD守护程序都读取和写入3TB驱动器。因此，该示例性Ceph存储群集的最大实际容量为99TB。如果设置为，则如果Ceph存储群集的剩余容量降至5TB，则该群集将不允许Ceph客户端读取和写入数据。因此，Ceph存储集群的运行容量为95TB，而不是99TB。`mon osd full ratio0.95`

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-28056cd20c19e97a7617494fbc1cf0d761fb9da5.png)

在这样的群集中，一个或两个OSD发生故障是正常的。不太频繁但合理的情况涉及机架的路由器或电源故障，这会同时关闭多个OSD（例如OSD 7-12）。在这种情况下，您仍应努力争取使群集保持可操作性并达到某种 状态，即使这意味着要在短时间内添加一些带有附加OSD的主机。如果容量利用率过高，则可能不会丢失数据，但是如果群集的容量利用率超过满比例，则在解决故障域内的中断时仍可能会牺牲数据可用性。因此，我们建议至少进行一些粗略的容量规划。`active + clean`

确定集群的两个数字：

1. OSD的数量。
2. 集群总容量

如果将群集的总容量除以群集中的OSD数量，则会发现群集中OSD的平均平均容量。考虑将该数字乘以您期望在正常操作期间会同时发生故障的OSD数目（相对较小的数目）。最后，将集群的容量乘以全部比率，以达到最大运行容量；然后，从您期望无法达到合理的完整比率的OSD中减去数据量。以更高数量的OSD故障（例如OSD机架）重复上述过程，以达到接近满负荷的合理数量。

以下设置仅适用于集群创建，然后存储在OSDMap中。

```text
[global]

        mon osd full ratio = .80
        mon osd backfillfull ratio = .75
        mon osd nearfull ratio = .70
```

`mon osd full ratio`描述

在考虑OSD之前使用的磁盘空间百分比`full`。类型

浮动默认

`.95`

`mon osd backfillfull ratio`描述

在考虑OSD之前也要`full`回填的磁盘空间百分比。类型

浮动默认

`.90`

`mon osd nearfull ratio`描述

在考虑OSD之前使用的磁盘空间百分比`nearfull`。类型

浮动默认

`.85`

小费 

如果某些OSD快要满了，而另一些OSD则有足够的容量，则快满OSD的CRUSH权重可能会出现问题。

小费 

这些设置仅在群集创建期间适用。之后，需要使用和 在OSDMap中对其进行更改。`ceph osd set-nearfull-ratioceph osd set-full-ratio`

#### 心跳

Ceph监视器通过要求每个OSD提供报告以及从OSD接收有关其相邻OSD状态的报告来了解集群。Ceph为监视器/ OSD交互提供了合理的默认设置；但是，您可以根据需要修改它们。有关详细信息，请参见[Monitor / OSD交互](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-osd-interaction)。

#### 监视存储同步

当您运行具有多个监视器的生产集群（推荐）时，每个监视器都会检查相邻监视器是否具有较新版本的集群映射（例如，相邻监视器中的映射具有一个或多个历元数比最高即时监控器地图中的当前纪元）。群集中的一个监视器可能会定期落后于其他监视器，直到必须离开仲裁，进行同步以检索有关群集的最新信息，然后重新加入仲裁。出于同步目的，监视器可以承担以下三个角色之一：

1. **领导者**：领导者是第一个获得最新Paxos版本集群图的监视器。
2. **提供程序**：提供程序是具有最新版本的群集映射的监视器，但不是第一个获得最新版本的监视器。
3. **请求者：**一个请求者是下降落后领先者，并且必须以检索有关集群的最新信息，然后才能重新加入仲裁同步监控。

这些角色使领导者可以将同步职责委派给提供者，从而防止同步请求使领导者超负荷，从而提高性能。在下图中，请求者了解到它已落后于其他监视器。请求者要求领导者进行同步，领导者告诉请求者与提供者进行同步。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-215fab4d12b3f0727a4fbc633b58887918820ca9.png)

当新的监视器加入群集时，总是发生同步。在运行时操作期间，监视器可能会在不同时间接收到群集映射的更新。这意味着领导者和提供者角色可以从一个监视器迁移到另一监视器。如果在同步时发生这种情况（例如，提供者落后于领导者），则提供者可以终止与请求者的同步。

同步完成后，Ceph需要在整个群集中进行修剪。整理要求展示位置组为。`active + clean`

`mon sync trim timeout`描述

类型

双默认

`30.0`

`mon sync heartbeat timeout`描述

类型

双默认

`30.0`

`mon sync heartbeat interval`描述

类型

双默认

`5.0`

`mon sync backoff timeout`描述

类型

双默认

`30.0`

`mon sync timeout`描述

监视器在放弃并重新引导之前将等待其同步提供程序发出的下一个更新消息的秒数。类型

双默认

`60.0`

`mon sync max retries`描述

类型

整数默认

`5`

`mon sync max payload size`描述

同步有效负载的最大大小（以字节为单位）。类型

32位整数默认

`1045676`

`paxos max join drift`描述

必须首先同步监视器数据存储之前的最大Paxos迭代次数。当监视器发现其对等方距离太远时，它将首先与数据存储同步，然后再继续。类型

整数默认

`10`

`paxos stash full interval`描述

存放PaxosService状态的完整副本的频率（以提交为单位）。目前此设置只影响`mds`，`mon`，`auth`和`mgr` PaxosServices。类型

整数默认

25

`paxos propose interval`描述

在建议地图更新之前，收集此时间间隔的更新。类型

双默认

`1.0`

`paxos min`描述

保留的最小Paxos状态数类型

整数默认

500

`paxos min wait`描述

一段时间不活动后收集更新的最短时间。类型

双默认

`0.05`

`paxos trim min`描述

修整前可容忍的额外提案数量类型

整数默认

250

`paxos trim max`描述

一次最多裁切额外提案的数量类型

整数默认

500

`paxos service trim min`描述

触发修剪的最小版本数（0禁用它）类型

整数默认

250

`paxos service trim max`描述

单个投标期间要修剪的最大版本数量（0禁用它）类型

整数默认

500

`mon max log epochs`描述

单个提案期间要修剪的最大日志时期数类型

整数默认

500

`mon max pgmap epochs`描述

单个提案中要裁剪的pgmap历元的最大数量类型

整数默认

500

`mon mds force trim to`描述

强制监视器将mdsmaps修剪到这一点（0禁用它。危险，请谨慎使用）类型

整数默认

0

`mon osd force trim to`描述

即使指定的时间段内没有清洁的PG，也要强制监视器将osdmaps调整为这一点（0禁用它。危险，请谨慎使用）类型

整数默认

0

`mon osd cache size`描述

osdmaps缓存的大小，不依赖底层存储的缓存类型

整数默认

10

`mon election timeout`描述

在选举提议者上，所有ACK的最长等待时间（以秒为单位）。类型

浮动默认

`5`

`mon lease`描述

显示器版本的租约长度（以秒为单位）。类型

浮动默认

`5`

`mon lease renew interval factor`描述

`mon lease`\* 将是领导者续签其他监控者的租约的间隔。该因子应小于。`mon lease renew interval factor1.0`类型

浮动默认

`0.6`

`mon lease ack timeout factor`描述

负责人将等待\* 供方确认租约延期。`mon leasemon lease ack timeout factor`类型

浮动默认

`2.0`

`mon accept timeout factor`描述

领导者将\* 等待 请求者接受Paxos更新。在Paxos恢复阶段也将其用于类似目的。`mon leasemon accept timeout factor`类型

浮动默认

`2.0`

`mon min osdmap epochs`描述

始终保持的最小OSD映射时期数。类型

32位整数默认

`500`

`mon max pgmap epochs`描述

监视器应保留的最大PG映射时期数。类型

32位整数默认

`500`

`mon max log epochs`描述

监视器应保留的最大日志时期数。类型

32位整数默认

`500`

#### 时钟

Ceph守护程序相互传递关键消息，必须在守护程序达到超时阈值之前对其进行处理。如果Ceph监视器中的时钟不同步，则可能导致许多异常。例如：

* 守护程序忽略收到的消息（例如，时间戳已过时）
* 未及时收到消息时，超时触发得太早/太晚了。

有关详细信息，请参见[监控存储同步](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-config-ref/#monitor-store-synchronization)。

小费 

您应该在Ceph监视器主机上安装NTP，以确保监视器群集以同步时钟运行。

即使差异尚无害，但NTP的时钟漂移可能仍然很明显。即使NTP保持合理的同步水平，也可能会触发Ceph的时钟漂移/时钟偏斜警告。在这种情况下，可以允许增加时钟漂移。但是，许多因素，例如工作量，网络延迟，配置对默认超时的替代以及[Monitor Store Synchronization](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-config-ref/#monitor-store-synchronization)设置，可能会影响可接受的时钟漂移水平，而不会影响Paxos的保证。

Ceph提供了以下可调选项，使您可以找到可接受的值。

`clock offset`描述

偏移系统时钟的量。有关`Clock.cc`详细信息，请参见。类型

双默认

`0`

从0.58版开始不推荐使用。

`mon tick interval`描述

监视器的滴答间隔（以秒为单位）。类型

32位整数默认

`5`

`mon clock drift allowed`描述

监视器之间允许的时钟漂移（以秒为单位）。类型

浮动默认

`.050`

`mon clock drift warn backoff`描述

时钟漂移警告的指数补偿类型

浮动默认

`5`

`mon timecheck interval`描述

领导者的时间检查间隔（时钟漂移检查），以秒为单位。类型

浮动默认

`300.0`

`mon timecheck skew interval`描述

领导者以秒为单位的时间偏差（以秒为单位）的时间检查间隔。类型

浮动默认

`30.0`

#### 客户

`mon client hunt interval`描述

客户端将每秒钟尝试一个新的监视器，`N`直到建立连接为止。类型

双默认

`3.0`

`mon client ping interval`描述

客户端将每秒钟对监视器执行一次ping操作`N`。类型

双默认

`10.0`

`mon client max log entries per message`描述

监视器将根据每个客户端消息生成的最大日志条目数。类型

整数默认

`1000`

`mon client bytes`描述

内存中允许的客户端消息数据量（以字节为单位）。类型

64位无符号整数默认

`100ul << 20`

### 池设置

从v0.94版开始，支持池标志，该标志允许或禁止对池进行更改。

如果以此方式配置，监视器也可以禁止删除池。

`mon allow pool delete`描述

监视器是否应允许删除池。不管池标志怎么说。类型

布尔型默认

`false`

`osd pool default ec fast read`描述

是否打开池上的快速读取。如果`fast_read` 在创建时未指定，它将用作新创建的擦除编码池的默认设置。类型

布尔型默认

`false`

`osd pool default flag hashpspool`描述

在新池上设置hashpspool标志类型

布尔型默认

`true`

`osd pool default flag nodelete`描述

在新池上设置nodelete标志。禁止以任何方式使用此标志删除允许的池。类型

布尔型默认

`false`

`osd pool default flag nopgchange`描述

在新池上设置nopgchange标志。不允许更改池的PG数量。类型

布尔型默认

`false`

`osd pool default flag nosizechange`描述

在新池上设置nosizechange标志。不允许更改池的大小。类型

布尔型默认

`false`

有关池标志的更多信息，请参见[池值](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#set-pool-values)。

### 杂项

`mon max osd`描述

群集中允许的最大OSD数。类型

32位整数默认

`10000`

`mon globalid prealloc`描述

要为集群中的客户端和守护程序预分配的全局ID的数量。类型

32位整数默认

`100`

`mon subscribe interval`描述

订阅的刷新间隔（以秒为单位）。订阅机制可以获取群集映射和日志信息。类型

双默认

`86400`

`mon stat smooth intervals`描述

Ceph将对最后的`N`PG地图进行平滑统计。类型

整数默认

`2`

`mon probe timeout`描述

引导程序启动之前，监视器将等待查找对等设备的秒数。类型

双默认

`2.0`

`mon daemon bytes`描述

元数据服务器和OSD消息的消息存储上限（以字节为单位）。类型

64位无符号整数默认

`400ul << 20`

`mon max log entries per event`描述

每个事件的最大日志条目数。类型

整数默认

`4096`

`mon osd prime pg temp`描述

当出站OSD返回群集时，启用或禁用用先前的OSD启动PGMap。通过`true`设置，客户端将继续使用以前的OSD，直到与该PG对等的新进入OSD。类型

布尔型默认

`true`

`mon osd prime pg temp max time`描述

当退出OSD的群集重新进入群集时，监视器应该花多少时间尝试填充PGMap。类型

浮动默认

`0.5`

`mon osd prime pg temp max time estimate`描述

在并行启动所有PG之前，每个PG花费的最大时间估计。类型

浮动默认

`0.25`

`mon osd allow primary affinity`描述

允许`primary_affinity`在osdmap中设置。类型

布尔型默认

假

`mon mds skip sanity`描述

跳过FSMap上的安全性声明（以防我们仍然要继续存在的错误）。如果FSMap完整性检查失败，则Monitor终止，但是我们可以通过启用此选项来禁用它。类型

布尔型默认

假

`mon max mdsmap epochs`描述

在单个提议期间要修剪的mdsmap历元的最大数量。类型

整数默认

500

`mon config key max entry size`描述

配置键条目的最大大小（以字节为单位）类型

整数默认

4096

`mon scrub interval`描述

通过将存储的校验和与所有存储的密钥的计算值进行比较，监视器多久（以秒为单位）清理其存储。类型

整数默认

3600 \* 24

`mon scrub max keys`描述

每次要擦洗的最大按键数。类型

整数默认

100

`mon compact on start`描述

在`ceph-mon`启动时压缩用作Ceph Monitor存储器的数据库 。如果常规压缩无法正常工作，则手动压缩有助于缩小监视器数据库并提高其性能。类型

布尔型默认

假

`mon compact on bootstrap`描述

在引导程序上压缩用作Ceph Monitor存储的数据库。引导程序启动后，Monitor开始互相探测以创建仲裁。如果在加入仲裁之前超时，它将重新启动并重新引导自身。类型

布尔型默认

假

`mon compact on trim`描述

当我们修剪旧的状态时，压缩一个特定的前缀（包括paxos）。类型

布尔型默认

真正

`mon cpu threads`描述

在监视器上执行CPU密集型工作的线程数。类型

布尔型默认

真正

`mon osd mapping pgs per chunk`描述

我们按块计算从放置组到OSD的映射。此选项指定每个块的放置组数。类型

整数默认

4096

`mon session timeout`描述

监视器将终止不活动的会话，并在此时间限制内保持空闲状态。类型

整数默认

300

`mon osd cache size min`描述

osd监视器高速缓存中要保留在内存中的映射的最小字节数。类型

64位整数默认

134217728

`mon memory target`描述

启用高速缓存自动调整后，与osd监视器高速缓存和kv高速缓存有关的字节数将保持映射到内存中。类型

64位整数默认

2147483648

`mon memory autotune`描述

自动调整用于osd监视器和kv数据库的缓存。类型

布尔型默认

真正

