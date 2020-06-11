# 监测OSD及PG

## 监测OSD及PG

高可用性和高可靠性要求使用容错方法来管理硬件和软件问题。Ceph没有单一的故障点，可以以“降级”模式处理对数据的请求。Ceph的[数据放置](https://docs.ceph.com/docs/nautilus/rados/operations/data-placement) 引入了一个间接层，以确保数据不直接绑定到特定的OSD地址。这意味着要跟踪系统故障，需要找到问题根源的[放置组](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups)和底层OSD。

提示： 群集某一部分的故障可能会阻止您访问特定对象，但这并不意味着您无法访问其他对象。当您遇到故障时，请不要惊慌。只需按照监视OSD和放置组的步骤进行即可。然后，开始故障排除。

Ceph通常是自我修复。但是，如果问题仍然存在，则监视OSD和放置组将有助于您确定问题。

### 监控屏上显示

OSD的状态位于群集中（`in`）或不在群集中（`out`）。并且，它已启动并正在运行（`up`），或者已关闭且未处于运行（`down`）。如果OSD是`up`，则它可以是`in`群集（可以读取和写入数据），也可以是`out`群集的。如果它是`in`集群并且是 集群的最近移动`out`，则Ceph会将展示位置组迁移到其他OSD。如果OSD属于`out`群集，则CRUSH不会将放置组分配给OSD。如果是OSD `down`，也应该 是OSD `out`。

注意 

如果OSD是`down`和`in`，则存在问题，并且群集将无法处于正常状态。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-700a689bcec45906334cd28500d7f59aa579850a.png)

如果执行的命令，例如，或者，你可能会注意到集群总是不回显。不要惊慌 关于OSD，您应该期望群集在某些预期情况下**不会** 回显 ：`ceph healthceph -sceph -wHEALTH OKHEALTH OK`

1. 您尚未启动集群（它不会响应）。
2. 您刚刚启动或重新启动了群集，但尚未准备好，因为正在创建放置组并且OSD正在对等。
3. 您刚刚添加或删除了OSD。
4. 您刚刚修改了集群图。

监视OSD的一个重要方面是确保在群集启动并运行时，该群集的所有OSD `in`也在`up`运行。要查看所有OSD是否正在运行，请执行：

```text
ceph osd stat
```

结果应该告诉您OSD的总数（x），多少`up`（y），多少`in`（z）和地图时期（eNNNN）。

```text
x osds: y up, z in; epoch: eNNNN
```

如果作为`in`集群的OSD数量大于作为集群的OSD数量`up`，请执行以下命令以标识`ceph-osd` 未运行的守护程序：

```text
ceph osd tree
```

```text
#ID CLASS WEIGHT  TYPE NAME             STATUS REWEIGHT PRI-AFF
 -1       2.00000 pool openstack
 -3       2.00000 rack dell-2950-rack-A
 -2       2.00000 host dell-2950-A1
  0   ssd 1.00000      osd.0                up  1.00000 1.00000
  1   ssd 1.00000      osd.1              down  1.00000 1.00000
```

小费 

通过精心设计的CRUSH层次结构进行搜索的能力可以通过更快地确定物理位置来帮助您对群集进行故障排除。

如果OSD是`down`，请启动它：

```text
sudo systemctl start ceph-osd@1
```

请参阅[OSD未运行](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-osd#osd-not-running)以获取与已停止或不会重新启动的OSD相关的问题。

### PG集

当CRUSH将放置组分配给OSD时，它将查看池的副本数，并将该放置组分配给OSD，以使该放置组的每个副本都分配给一个不同的OSD。例如，如果池需要贴装组的三个副本，CRUSH可以为它们分配 `osd.1`，`osd.2`并`osd.3`分别。实际上，CRUSH会寻找一个伪随机放置，该放置将考虑您在[CRUSH映射中](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map)设置的故障域，因此您很少会看到分配给大型群集中最近邻居OSD的放置组。我们将应包含特定放置组副本的OSD称为“ **代理集”**。在某些情况下，代理集中的OSD为`down`否则将无法为展示位置组中的对象提供服务。当出现这些情况时，请不要惊慌。常见的示例包括：

* 您添加或删除了OSD。然后，CRUSH将布局组重新分配给其他OSD，从而更改了代理集的组成并通过“回填”过程生成了数据迁移。
* OSD是`down`，已经重新启动，现在是`recovering`。
* 代理集中的一个OSD正在`down`或无法为请求提供服务，并且另一个OSD暂时承担了其职责。

Ceph使用**Up Set**处理客户端请求，**Up Set**是实际上将处理请求的OSD集合。在大多数情况下，Up Set和Acting Set实际上是相同的。如果不是，则表明Ceph正在迁移数据，正在恢复OSD或存在问题（即，在这种情况下Ceph通常以“卡死”消息回显“健康警告”状态）。

要检索展示位置组列表，请执行以下操作：

```text
ceph pg dump
```

要查看给定放置组的“动作集”或“上集”中的哪些OSD，请执行以下操作：

```text
ceph pg map {pg-num}
```

结果应该告诉您osdmap时期（eNNN），布局组编号（{pg-num}），上一组（up \[\]）中的OSD和操作集中（acting \[\]）中的OSD。

```text
osdmap eNNN pg {raw-pg-num} ({pg-num}) -> up [0,1,2] acting [0,1,2]
```

注意 

如果Up Set和Acting Set不匹配，则可能表明群集重新平衡自身或群集存在潜在问题。

### 对等

之前，你可以将数据写入到一个放置组，它必须是在`active` 状态，它 **应该**是一个`clean`状态。为了让Ceph确定某个展示位置组的当前状态，该展示位置组的主要OSD（即行为集中的第一个OSD）与第二个和第三个OSD对等以建立关于该展示位置组的当前状态的协议（假设有3个PG副本的池）。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-12ccbc36b952da92b4cb940ca78f39c63f498547.png)

OSD还向监视器报告其状态。有关详细信息，请参见[配置Monitor / OSD交互](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-osd-interaction/)。要解决对等问题，请参阅[对等失败](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-pg#failures-osd-peering)。

### 监视放置组状态

如果执行的命令，例如，或者，你可能会注意到集群总是不回显。在检查OSD是否正在运行之后，还应该检查放置组状态。您应该期望在许多与放置组对等相关的情况下群集**不会**回显：`ceph healthceph -sceph -wHEALTH OKHEALTH OK`

1. 您刚刚创建了一个池，并且尚未对位组。
2. 展示位置组正在恢复。
3. 您刚刚向集群添加了OSD或从集群中删除了OSD。
4. 您刚刚修改了CRUSH地图，并且正在迁移展示位置组。
5. 放置组的不同副本中的数据不一致。
6. Ceph正在清理展示位置组的副本。
7. Ceph没有足够的存储容量来完成回填操作。

如果上述情况之一导致Ceph回显，请不要惊慌。在许多情况下，群集将自行恢复。在某些情况下，您可能需要采取措施。监视放置组的一个重要方面是确保集群启动并运行时，所有放置组均处于 且最好处于该状态。要查看所有展示位置组的状态，请执行以下操作：`HEALTH WARNactiveclean`

```text
ceph pg stat
```

结果将告诉您放置组的总数（x），处于特定状态（例如`active+clean`y）的放置组数和已存储的数据量（z）。

```text
x pgs: y active+clean; z bytes data, aa MB used, bb GB / cc GB avail
```

注意 

Ceph通常会报告展示位置组的多个状态。

除了放置组状态之外，Ceph还将回显已使用的存储容量（aa），剩余存储容量（bb）和该放置组的总存储容量。这些数字在某些情况下可能很重要：

* 您正在达到或。`near full ratiofull ratio`
* 由于CRUSH配置错误，您的数据无法在整个群集中分布。

展示位置组ID

放置组ID由池号（不是池名）后跟句点（。）和放置组ID（十六进制数）组成。您可以从的输出中查看池号及其名称。例如，创建的第一个池对应于池编号。完全限定的展示位置组ID的格式如下：`ceph osd lspools1`

```text
{pool-num}.{pg-id}
```

它通常看起来像这样：

```text
1.1f
```

要检索展示位置组列表，请执行以下操作：

```text
ceph pg dump
```

您还可以将输出格式化为JSON格式并将其保存到文件中：

```text
ceph pg dump -o {filename} --format=json
```

要查询特定的展示位置组，请执行以下操作：

```text
ceph pg {poolnum}.{pg-id} query
```

Ceph将以JSON格式输出查询。

以下小节详细介绍了常见的pg状态。

#### 创建

创建池时，它将创建您指定的展示位置组数。`creating`当Ceph 创建一个或多个展示位置组时，它将回显。创建它们后，属于放置组的“动作集”的OSD将对等。对等完成后，展示位置组状态应为`active+clean`，这意味着Ceph客户端可以开始写入展示位置组。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-48a884d47a318394a948f7e883a89c76115606bc.png)

#### 对等

当Ceph对等放置组时，Ceph会将存储该放置组副本的OSD置于**关于该**放置组中 对象和元数据**状态**的**协议中**。当Ceph完成对等时，这意味着存储放置组的OSD同意放置组的当前状态。但是，对等过程的完成 **并不**意味着每个副本都具有最新的内容。

权威历史

Ceph **不会**确认对客户端的写操作，直到行为集的所有OSD都坚持写操作为止。这种做法可确保至少从上一次成功的对等操作以来，每个动作集的成员都有记录。

有了每个已确认写入操作的准确记录，Ceph可以构造和传播放置组的新权威历史记录-一套完整且有序的操作集，如果执行，将使OSD的放置组副本保持最新状态。

#### 主动

Ceph完成对等过程后，展示位置组可能会变为 `active`。该`active`状态表示放置组中的数据通常在主放置组和副本中可用于读取和写入操作。

#### 清洁

当放置组处于该`clean`状态时，主OSD和副本OSD已成功对等，并且该放置组没有杂散副本。Ceph将展示位置组中的所有对象复制了正确的次数。

#### 降级

当客户端将对象写入主OSD时，主OSD负责将副本写入副本OSD。在主OSD将对象写入存储后，放置组将保持`degraded` 状态，直到主OSD从副本OSD收到确认Ceph成功创建了副本对象的确认为止。

放置组之所以可能`active+degraded`是因为OSD可能 `active`尚未容纳所有对象。如果有OSD `down`，Ceph会将分配给OSD的每个放置组标记为`degraded`。当OSD重新联机时，OSD必须再次对等。但是，客户端仍然可以将新对象写入`degraded`展示位置组`active`。

如果OSD是OSD，`down`并且`degraded`情况仍然存在，则Ceph可能会将`down`OSD 标记 为`out`群集，然后将数据从`down`OSD 重新映射到另一个OSD。被标记`down`和被标记之间的时间由所`out` 控制，默认情况下设置为秒。`mon osd down out interval600`

放置组也可以是`degraded`，因为Ceph找不到Ceph认为应该在该放置组中的一个或多个对象。尽管无法读取或写入未找到的对象，但仍可以访问`degraded`放置组中的所有其他对象。

#### 恢复

Ceph被设计用于在硬件和软件问题持续存在的范围内的容错能力。OSD运行时`down`，其内容可能落在放置组中其他副本的当前状态之后。OSD返回时`up`，必须更新放置组的内容以反映当前状态。在该时间段内，OSD可能会反映一个`recovering` 状态。

恢复并不总是很简单，因为硬件故障可能会导致多个OSD的级联故障。例如，机架或机柜的网络交换机可能发生故障，这可能导致许多主机的OSD落在群集的当前状态之后。解决故障后，每个OSD都必须恢复。

Ceph提供了许多设置来平衡新服务请求之间的资源争用与恢复数据对象并将展示位置组还原到当前状态的需求。该设置允许OSD在开始恢复过程之前重新启动，重新对等甚至处理一些重播请求。该设置一个线程超时，因为多个屏上显示可能会失败，重新启动和重新同行在交错率。该设置限制了OSD将同时接受的恢复请求的数量，以防止OSD服务失败。该设置限制了恢复的数据块的大小，以防止网络拥塞。`osd recovery delay startosd recovery thread timeoutosd recovery max activeosd recovery max chunk`

#### 回填

当新的OSD加入群集时，CRUSH将把放置组从群集中的OSD重新分配给新添加的OSD。强制新OSD立即接受重新分配的放置组会给新OSD带来过多的负担。用放置组回填OSD可使此过程在后台开始。回填完成后，新的OSD将在准备就绪后开始处理请求。

在回填操作期间，您可能会看到以下几种状态之一： `backfill_wait`表示回填操作正在等待执行，但尚未进行；`backfilling`表示正在进行回填操作；并且`backfill_toofull`表示已请求回填操作，但由于存储容量不足而无法完成。如果无法重新填充展示位置组，则可以考虑使用`incomplete`。

该`backfill_toofull`状态可能是瞬态的。随着PG的移动，空间可能变得可用。在`backfill_toofull`类似于`backfill_wait`只要条件改变回填可以继续在这。

Ceph提供了许多设置来管理与将放置组重新分配给OSD（尤其是新OSD）相关的负载峰值。默认情况下， `osd_max_backfills`将往返OSD的并发回填的最大数量设置为1。如果OSD接近其完整比率（默认为90％）并使用命令进行更改，则使OSD可以拒绝回填请求。如果OSD拒绝回填请求，则 OSD使OSD重试该请求（默认为30秒后）。OSD还可以设置和管理扫描间隔（默认情况下为64和512）。`backfill full ratioceph osd set-backfillfull-ratioosd backfill retry intervalosd backfill scan minosd backfill scan max`

#### 重新映射

当为展示位置组提供服务的代理集发生更改时，数据将从旧的代理集迁移到新的代理集。新的主OSD服务请求可能需要一些时间。因此，它可能会要求旧的主要数据库继续为请求提供服务，直到完成放置组迁移为止。数据迁移完成后，映射将使用新行为集的主OSD。

#### 陈旧

当Ceph使用心跳信号确保主机和守护程序正在运行时，这些 `ceph-osd`守护程序也可能进入一种`stuck`状态，即它们没有及时报告统计信息（例如，临时网络故障）。默认情况下，OSD守护程序每隔半秒（即`0.5`）报告启动，失败和启动统计信息的放置组，其频率超过心跳阈值。如果放置组的操作集中的**主OSD**无法向监视器报告，或者如果其他OSD已报告了主OSD `down`，则监视器将标记该放置组`stale`。

启动集群时，通常会看到`stale`状态，直到对等过程完成。在集群运行了一段时间之后，看到处于`stale`状态的放置组，则表明这些放置组的主OSD是否正在`down`向监视器报告放置组统计信息。

### 识别困扰的PG 

如前所述，仅由于放置状态不是，放置组就不一定有问题`active+clean`。通常，当放置组卡住时，Ceph的自我修复功能可能无法正常工作。卡住的状态包括：

* **不干净**：展示位置组中包含未复制所需次数的对象。他们应该正在恢复。
* **不活动**：放置组无法处理读取或写入，因为它们正在等待OSD包含最新数据`up`。
* **Stale**：放置组处于未知状态，因为托管它们的OSD暂时未向监视集群报告（由配置）。`mon osd report timeout`

要确定卡住的放置组，请执行以下操作：

```text
ceph pg dump_stuck [unclean|inactive|stale|undersized|degraded]
```

有关其他详细信息，请参见[放置组子系统](https://docs.ceph.com/docs/nautilus/rados/operations/control#placement-group-subsystem)。要对卡住的放置组进行[故障排除](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-pg#troubleshooting-pg-errors)，请参阅[PG错误故障排除](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-pg#troubleshooting-pg-errors)。

### 查找对象位置

要将对象数据存储在Ceph对象存储中，Ceph客户端必须：

1. 设置对象名称
2. 指定一个[池](https://docs.ceph.com/docs/nautilus/rados/operations/pools)

Ceph客户端检索最新的群集映射，CRUSH算法计算如何将对象映射到[放置组](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups)，然后计算如何动态地将放置组分配给OSD。要找到对象位置，您只需要对象名称和池名称即可。例如：

```text
ceph osd map {poolname} {object-name} [namespace]
```

练习：找到一个对象

作为练习，让我们创建一个对象。使用命令行上的命令指定对象名称，包含某些对象数据的测试文件的路径以及池名称 。例如：`rados put`

```text
rados put {object-name} {file-path} --pool=data
rados put test-object-1 testfile.txt --pool=data
```

要验证Ceph对象存储中是否存储了对象，请执行以下操作：

```text
rados -p data ls
```

现在，确定对象位置：

```text
ceph osd map {pool-name} {object-name}
ceph osd map data test-object-1
```

Ceph应该输出对象的位置。例如：

```text
osdmap e537 pool 'data' (1) object 'test-object-1' -> pg 1.d1743484 (1.4) -> up ([0,1], p0) acting ([0,1], p0)
```

要删除测试对象，只需使用命令将其删除。例如：`rados rm`

```text
rados rm test-object-1 --pool=data
```

随着群集的发展，对象位置可能会动态变化。Ceph动态重新平衡的一个好处是Ceph使您不必手动执行迁移。有关详细信息，请参见 [架构](https://docs.ceph.com/docs/nautilus/architecture)部分。

