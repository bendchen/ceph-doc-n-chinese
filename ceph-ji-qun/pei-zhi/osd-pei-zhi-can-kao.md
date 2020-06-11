# OSD配置参考

## OSD配置参考

您可以在Ceph配置文件中配置Ceph OSD守护程序，但是Ceph OSD守护程序可以使用默认值和非常小的配置。最小的Ceph OSD守护程序配置设置和，并对几乎所有其他内容使用默认值。`osd journal sizehost`

从`0`使用以下约定开始，以增量方式在数字上标识Ceph OSD守护程序。

```text
osd.0
osd.1
osd.2
```

在配置文件中，您可以通过将配置设置添加到`[osd]`配置文件的部分来为集群中的所有Ceph OSD守护进程指定设置。要将设置直接添加到特定的Ceph OSD守护进程（例如`host`），请在配置文件的OSD特定部分中输入设置。例如：

```text
[osd]
        osd journal size = 5120

[osd.0]
        host = osd-host-a

[osd.1]
        host = osd-host-b
```

### 常规设置

以下设置提供了Ceph OSD守护进程的ID，并确定了数据和日志的路径。Ceph部署脚本通常会自动生成UUID。

警告 

**不要**更改数据或日志的默认路径，因为这会使以后对Ceph进行故障排除更加麻烦。

轴颈的尺寸至少应为预期驱动速度乘以的乘积的两倍。但是，最常见的做法是对日志驱动器（通常是SSD）进行分区，并进行安装，以使Ceph将整个分区用于日志。`filestore max sync interval`

`osd uuid`描述

Ceph OSD守护程序的通用唯一标识符（UUID）。类型

UUID默认

UUID。注意

在适用于单个Ceph的OSD守护进程。将 适用于整个集群。`osd uuidfsid`

`osd data`描述

OSD数据的路径。部署Ceph时必须创建目录。您应该在此安装点安装用于OSD数据的驱动器。我们不建议更改默认值。类型

串默认

`/var/lib/ceph/osd/$cluster-$id`

`osd max write size`描述

写入的最大大小（以兆字节为单位）。类型

32位整数默认

`90`

`osd max object size`描述

RADOS对象的最大大小（以字节为单位）。类型

32位无符号整数默认

128MB

`osd client message size cap`描述

内存中允许的最大客户端数据消息。类型

64位无符号整数默认

默认为500MB。 `500*1024L*1024L`

`osd class dir`描述

RADOS类插件的类路径。类型

串默认

`$libdir/rados-classes`

### 文件系统设置

Ceph建立并挂载用于Ceph OSD的文件系统。

`osd mkfs options {fs-type}`描述

创建{fs-type}类型的新Ceph OSD时使用的选项。类型

串xfs的默认值

`-f -i 2048`其他文件系统的默认设置

{空字符串}例如：：

`osd mkfs options xfs = -f -d agcount=24`

`osd mount options {fs-type}`描述

挂载类型为{fs-type}的Ceph OSD时使用的选项。类型

串xfs的默认值

`rw,noatime,inode64`其他文件系统的默认设置

`rw, noatime`例如：：

`osd mount options xfs = rw, noatime, inode64, logbufs=8`

### 日记设置

默认情况下，Ceph希望您将使用以下路径存储Ceph OSD Daemons日志：

```text
/var/lib/ceph/osd/$cluster-$id/journal
```

当使用单一设备类型（例如，旋转驱动器）时，日志应位于_同一位置_：逻辑卷（或分区）应与`data`逻辑卷位于同一设备中。

当混合使用较快的（SSD，NVMe）设备和较慢的设备（例如旋转驱动器）时，将日志放置在较快的设备上，而`data`完全占用较慢的设备是有意义的 。

默认值为5120（5 GB），但可以更大，在这种情况下，需要在文件中进行设置：`osd journal sizeceph.conf`

```text
osd journal size = 10240
```

`osd journal`描述

OSD日志的路径。这可能是文件或块设备（例如SSD的分区）的路径。如果是文件，则必须创建目录来包含它。我们建议使用与驱动器分开的驱动器。`osd data`类型

串默认

`/var/lib/ceph/osd/$cluster-$id/journal`

`osd journal size`描述

日志的大小（以兆字节为单位）。类型

32位整数默认

`5120`

有关其他详细信息，请参见[日志配置参考](https://docs.ceph.com/docs/nautilus/rados/configuration/journal-ref)。

### 监视OSD交互

Ceph OSD守护程序检查彼此的心跳并定期向监视器报告。在许多情况下，Ceph可以使用默认值。但是，如果您的网络存在延迟问题，则可能需要采用更长的时间间隔。有关心跳的详细讨论，请参阅 [配置Monitor / OSD交互](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-osd-interaction)。

### 数据放置

有关详细信息，请参见“ [池和PG配置参考](https://docs.ceph.com/docs/nautilus/rados/configuration/pool-pg-config-ref) ”。

### 擦洗

除了制作对象的多个副本外，Ceph还通过清理放置组来确保数据完整性。Ceph清理类似于`fsck`对象存储层上的清理。对于每个放置组，Ceph都会生成所有对象的目录，并比较每个主要对象及其副本，以确保没有对象丢失或不匹配。轻擦洗（每天）检查对象的大小和属性。深度清理（每周一次）读取数据并使用校验和以确保数据完整性。

清理对于保持数据完整性很重要，但是会降低性能。您可以调整以下设置以增加或减少擦洗操作。

`osd max scrubs`描述

Ceph OSD守护程序的最大同时清理操作数。类型

32位整数默认

`1`

`osd scrub begin hour`描述

可以执行计划的清理的下限的一天中的时间。类型

整数，范围为0到24默认

`0`

`osd scrub end hour`描述

可以执行计划的清理的上限时间。与一起定义了一个时间窗口，可以在其中进行擦洗。但是只要展示位置组的清理间隔超过，无论时间窗口允许与否，都将执行清理。`osd scrub begin hourosd scrub max interval`类型

整数，范围为0到24默认

`24`

`osd scrub begin week day`描述

这将擦洗限制在一周中的这一天或更晚。0或7 =星期日，1 =星期一，依此类推。类型

整数，范围为0到7默认

`0`

`osd scrub end week day`描述

这样可以将擦洗时间限制在一周的前几天。0或7 =星期日，1 =星期一，依此类推。类型

整数，范围为0到7默认

`7`

`osd scrub during recovery`描述

恢复期间允许擦洗。将此设置为`false`会在活动恢复时禁用调度新的清理（和深度清理）。已经运行的磨砂膏将继续进行。这对于减少繁忙群集上的负载可能很有用。类型

布尔型默认

`true`

`osd scrub thread timeout`描述

超时超时之前（以秒为单位）。类型

32位整数默认

`60`

`osd scrub finalize thread timeout`描述

超时终止清理线程之前的最长时间（以秒为单位）。类型

32位整数默认

`60*10`

`osd scrub load threshold`描述

标准化最大负载。当系统负载（由定义）高于此数字时，Ceph将不会进行清理。默认值为。`getloadavg() / number of online cpus0.5`类型

浮动默认

`0.5`

`osd scrub min interval`描述

当Ceph存储群集负载较低时，清理Ceph OSD守护程序的最小时间间隔（以秒为单位）。类型

浮动默认

每天一次。 `60*60*24`

`osd scrub max interval`描述

清理Ceph OSD守护程序的最大时间间隔（以秒为单位），与群集负载无关。类型

浮动默认

每周一次。 `7*60*60*24`

`osd scrub chunk min`描述

在单个操作期间要清理的最少数量的对象存储块。Ceph块在清理期间写入单个块。类型

32位整数默认

5

`osd scrub chunk max`描述

单个操作期间要清理的对象存储块的最大数量。类型

32位整数默认

25

`osd scrub sleep`描述

在擦洗下一组食物之前需要睡眠。增大此值将减慢整个清理操作，而对客户端操作的影响较小。类型

浮动默认

0

`osd deep scrub interval`描述

“深度”清理的间隔（完全读取所有数据）。在 不影响此设置。`osd scrub load threshold`类型

浮动默认

每周一次。 `60*60*24*7`

`osd scrub interval randomize ratio`描述

在为展示位置组安排下一个清理作业时，添加一个随机延迟。延迟是小于\* 的随机值。因此，默认设置实际上会在\* 的允许时间范围内随机分布磨砂。`osd scrub min intervalosd scrub min intervalosd scrub interval randomized ratio[1, 1.5]osd scrub min interval`类型

浮动默认

`0.5`

`osd deep scrub stride`描述

进行深层清洁时，请阅读尺码。类型

32位整数默认

512 KB。 `524288`

`osd scrub auto repair`描述

`true`当在scrub或deep-scrub中发现错误时，将此选项设置为将启用自动pg修复。但是，如果发现的错误不止一个，那么 将不进行维修。`osd scrub auto repair num errors`类型

布尔型默认

`false`

`osd scrub auto repair num errors`描述

如果发现的错误不止这些，将不会进行自动修复。类型

32位整数默认

`5`

### 操作

`osd op queue`描述

这设置了用于在OSD中确定操作优先级的队列的类型。这两个队列均具有严格的子队列，该子队列在普通队列之前已出队。普通队列在实现之间是不同的。原始的PrioritizedQueue（`prio`）使用令牌桶系统，当有足够的令牌时，它将首先使高优先级队列出队。如果没有足够的令牌，队列将从低优先级出队到高优先级。WeightedPriorityQueue（`wpq`）使所有优先级相对于它们的优先级出队，以防止任何队列饿死。WPQ应该在少数OSD比其他OSD更过载的情况下提供帮助。新的基于mClock的OpClassQueue（`mclock_opclass`）根据操作所属的类（恢复，清理，snaptrim，客户端操作，osd子操作）对操作进行优先级划分。并且，基于mClock的ClientQueue（`mclock_client`）还合并了客户端标识符，以促进客户端之间的公平。请参阅[基于mClock的QoS](https://docs.ceph.com/docs/nautilus/rados/configuration/osd-config-ref/#qos-based-on-mclock)。需要重启。类型

串有效选择

prio，wpq，mclock\_opclass，mclock\_client默认

`prio`

`osd op queue cut off`描述

这将选择将优先级操作发送到严格队列还是普通队列。该`low`设置将所有复制操作和更高版本的操作发送到严格队列，而该`high` 选项仅将复制确认操作和更高版本的操作发送到严格队列。`high`当群集中的一些OSD非常繁忙时`wpq`，将此设置为应该会有所帮助，尤其是与该设置结合使用 时。如果没有这些设置，则忙于处理复制流量的OSD可能会使这些OSD上的主要客户端流量饿死。需要重启。`osd op queue`类型

串有效选择

低高默认

`low`

`osd client op priority`描述

为客户端操作设置的优先级。类型

32位整数默认

`63`有效范围

1-63

`osd recovery op priority`描述

为恢复操作设置的优先级，如果未由池的指定`recovery_op_priority`。类型

32位整数默认

`3`有效范围

1-63

`osd scrub priority`描述

当池未指定值时，为计划的清理工作队列设置的默认优先级`scrub_priority`。可以将其提高到scrub阻止客户端操作的时间。`osd client op priority`类型

32位整数默认

`5`有效范围

1-63

`osd requested scrub priority`描述

为用户请求的清理设置的优先级在工作队列上。如果该值小于该值，则可以将其增大为scrub阻止客户端操作的时间。`osd client op priorityosd client op priority`类型

32位整数默认

`120`

`osd snap trim priority`描述

为对齐修剪工作队列设置的优先级。类型

32位整数默认

`5`有效范围

1-63

`osd snap trim sleep`描述

下一次快速修整操作之前，以秒为单位的睡眠时间。增大此值将减慢快照修剪。此选项将覆盖后端特定的变体。类型

浮动默认

`0`

`osd snap trim sleep hdd`描述

在下一次快速调整HDD之前，睡眠时间（以秒为单位）。类型

浮动默认

`5`

`osd snap trim sleep ssd`描述

下一次对SSD进行快速修整之前，以秒为单位的睡眠时间。类型

浮动默认

`0`

`osd snap trim sleep hybrid`描述

当osd数据位于HDD上且osd日志位于SSD上时，下一次快照修整操作之前的睡眠时间（以秒为单位）。类型

浮动默认

`2`

`osd op thread timeout`描述

Ceph OSD守护程序操作线程超时（以秒为单位）。类型

32位整数默认

`15`

`osd op complaint time`描述

在指定的秒数过去之后，一项操作值得投诉。类型

浮动默认

`30`

`osd op history size`描述

跟踪的已完成操作的最大数量。类型

32位无符号整数默认

`20`

`osd op history duration`描述

要跟踪的最旧的已完成操作。类型

32位无符号整数默认

`600`

`osd op log threshold`描述

一次显示多少个操作日志。类型

32位整数默认

`5`

#### 基于MCLOCK的

Ceph目前在实验阶段使用mClock，应该以探索性的思维方式进行研究。

**核心概念**

使用基于[dmClock算法](https://www.usenix.org/legacy/event/osdi10/tech/full_papers/Gulati.pdf)的排队调度器实现对Ceph的QoS支持。该算法按权重分配Ceph集群的I / O资源，并实施最小预留和最大限制的约束，以便服务可以公平竞争资源。当前， _mclock\_opclass_操作队列将涉及I / O资源的Ceph服务划分为以下存储桶：

* 客户操作：客户发出的iops
* osd subop：主要OSD发出的iops
* 对齐修剪：与对齐修剪有关的请求
* pg recovery：与恢复相关的请求
* pg scrub：与scrub相关的请求

使用以下三组标记对资源进行分区。换句话说，每种服务类型的份额由三个标签控制：

1. 预留：为服务分配的最低IOPS。
2. 限制：为服务分配的最大IOPS。
3. 重量：如果额外容量或系统超额订购，则容量的比例份额。

在Ceph中，操作按“成本”分级。这些“成本”消耗了分配用于服务各种服务的资源。因此，例如，服务保留的次数越多，只要需要，就可以保证拥有更多的资源。假设有两种服务：恢复和客户端操作：

* 恢复：（r：1，l：5，w：1）
* 客户操作：（r：2，l：0，w：9）

上面的设置确保恢复服务每秒收到的请求不会超过5个（即使需要）（请参阅下面的“当前实施说明”），并且没有其他服务与之竞争。但是，如果客户端开始发出大量I / O请求，它们也不会耗尽所有I / O资源。只要有这样的请求，每秒就会始终为恢复作业分配1个请求。因此，即使在高负载集群中，恢复工作也不会饿死。同时，由于其权重为“ 9”，而竞争者为“ 1”，因此客户操作可以享受更大一部分I / O资源。对于客户端操作，它不受限制设置的限制，因此如果没有正在进行的恢复，它可以利用所有资源。

与_mclock\_opclass_一起，_还有_一个名为_mclock\_client的_ mclock操作队列 。它根据类别对操作进行划分，但也根据发出请求的客户端对其进行划分。这不仅有助于管理用于不同类别操作的资源的分配，而且还可以确保客户之间的公平。

当前实施注意：当前的实验实施不强制执行限值。作为第一近似，我们决定不阻止原本会进入操作定序器的操作。

**MCLOCK的微妙之处**

预留和限制值具有每秒的请求单位。然而，重物在技术上没有单位，并且重物彼此相对。因此，如果一类请求的权重为1，另一类的权重为9，则后一类请求应该以9对1的比率获得9，作为第一类。但是，只有在满足保留条件并且这些值包括在保留阶段执行的操作时，才会发生这种情况。

即使权重没有单位，由于算法如何为请求分配权重标签，在选择权重值时也必须小心。如果权重为_W_，则对于给定的请求类别，下一个请求将具有_1 / W_的权重标签加上上一个权重标签或当前时间（以较大者为准）。这意味着如果_W_足够大，因此_1 / W_足够小，则可能永远不会分配计算出的标签，因为它将获得当前时间的值。最终的教训是重量值不应太大。它们应低于人们期望每秒处理的请求数。

**注意事项**

有一些因素可以减少Ceph中的mClock op队列的影响。首先，通过其放置组标识符将对OSD的请求进行分片。每个分片都有自己的mClock队列，这些队列既不交互也不共享信息。碎片的数量可以与所述配置选项来控制 `osd_op_num_shards`，`osd_op_num_shards_hdd`和 `osd_op_num_shards_ssd`。较少的分片数量会增加mClock队列的影响，但可能会产生其他有害影响。

其次，请求从操作队列传输到操作定序器，在操作过程中它们经历执行的各个阶段。操作队列是mClock驻留的位置，并且mClock确定下一个要传输到操作定序器的操作。操作定序器中允许的操作数是一个复杂的问题。通常，我们希望在顺控程序中保留足够的操作，以便在等待磁盘和网络访问在其他操作上完成时，它总是在某些操作上完成工作。另一方面，一旦将操作转移到操作定序器，mClock将不再对其进行控制。因此，为了最大程度地发挥mClock的影响，我们希望在操作定序器中保持尽可能少的操作。因此，我们有着内在的张力。

影响在操作序操作的数量的配置选项`bluestore_throttle_bytes`， `bluestore_throttle_deferred_bytes`， `bluestore_throttle_cost_per_io`， `bluestore_throttle_cost_per_io_hdd`，和 `bluestore_throttle_cost_per_io_ssd`。

影响mClock算法影响的第三个因素是，我们使用的是分布式系统，其中向多个OSD发出请求，每个OSD具有（可以具有）多个分片。但是，我们目前正在使用mClock算法，该算法不是分布式的（请注意：dmClock是mClock的分布式版本）。

目前，各种组织和个人都在尝试使用mClock（因为存在于此代码库中）及其对代码库的修改。我们希望您可以在ceph-devel邮件列表中分享有关mClock和dmClock实验的经验。

`osd push per object cost`描述

服务推送操作的开销类型

无符号整数默认

1000

`osd recovery max chunk`描述

恢复操作可以承载的数据块的最大总大小。类型

无符号整数默认

8 MiB

`osd op queue mclock client op res`描述

客户的保留类型

浮动默认

1000.0

`osd op queue mclock client op wgt`描述

客户的重量op。类型

浮动默认

500.0

`osd op queue mclock client op lim`描述

客户限额类型

浮动默认

1000.0

`osd op queue mclock osd subop res`描述

保留osd subop。类型

浮动默认

1000.0

`osd op queue mclock osd subop wgt`描述

osd subop的重量。类型

浮动默认

500.0

`osd op queue mclock osd subop lim`描述

osd subop的限制。类型

浮动默认

0.0

`osd op queue mclock snap res`描述

保留修剪的保留。类型

浮动默认

0.0

`osd op queue mclock snap wgt`描述

快速修剪的重量。类型

浮动默认

1.0

`osd op queue mclock snap lim`描述

修剪限度。类型

浮动默认

0.001

`osd op queue mclock recov res`描述

恢复的保留。类型

浮动默认

0.0

`osd op queue mclock recov wgt`描述

恢复的重量。类型

浮动默认

1.0

`osd op queue mclock recov lim`描述

恢复极限。类型

浮动默认

0.001

`osd op queue mclock scrub res`描述

保留清理作业。类型

浮动默认

0.0

`osd op queue mclock scrub wgt`描述

擦洗作业的重量。类型

浮动默认

1.0

`osd op queue mclock scrub lim`描述

清理作业的限制。类型

浮动默认

0.001

### 回填

当您将Ceph OSD守护程序添加到集群中或从中删除时，CRUSH算法将要通过将放置组移至Ceph OSD守护程序或从Ceph OSD守护程序移动来重新平衡集群，以恢复平衡。迁移放置组及其包含的对象的过程可能会大大降低群集的运行性能。为了保持操作性能，Ceph通过“回填”执行此迁移，这使Ceph可以将回填操作设置为比读取或写入数据的请求低的优先级。

`osd max backfills`描述

单个OSD允许进出的最大回填数。类型

64位无符号整数默认

`1`

`osd backfill scan min`描述

每次回填扫描的最小对象数。类型

32位整数默认

`64`

`osd backfill scan max`描述

每次回填扫描的最大对象数。类型

32位整数默认

`512`

`osd backfill retry interval`描述

重试回填请求之前要等待的秒数。类型

双默认

`10.0`

### OSD映射

OSD映射反映了集群中运行的OSD守护程序。随着时间的流逝，地图时期的数量增加。Ceph提供了一些设置来确保Ceph在OSD映射变大时表现良好。

`osd map dedup`描述

在OSD映射中启用删除重复项。类型

布尔型默认

`true`

`osd map cache size`描述

要保留的OSD映射数。类型

32位整数默认

`50`

`osd map message max`描述

每个MOSDMap消息允许的最大映射条目。类型

32位整数默认

`40`

### 恢复

当集群启动或Ceph OSD守护进程崩溃并重新启动时，OSD开始与其他Ceph OSD守护进程建立对等关系，然后才能进行写操作。有关详细信息，请参见 [监视OSD和PG](https://docs.ceph.com/docs/nautilus/rados/operations/monitoring-osd-pg#peering)。

如果Ceph OSD守护进程崩溃并重新联机，通常它将与其他Ceph OSD守护进程不同步，而其他Ceph OSD守护进程在放置组中包含对象的最新版本。发生这种情况时，Ceph OSD守护进程将进入恢复模式，并寻求获取数据的最新副本并将其映射恢复到最新状态。根据Ceph OSD守护进程关闭的时间长短，OSD的对象和放置组可能已过时。此外，如果故障域出现故障（例如机架），则可能会同时有多个Ceph OSD守护程序重新联机。这会使恢复过程既耗时又耗费资源。

为了保持操作性能，Ceph执行恢复时会限制恢复请求的数量，线程和对象块大小，这使Ceph在降级状态下表现良好。

`osd recovery delay start`描述

对等完成后，Ceph将延迟指定的秒数，然后开始恢复对象。类型

浮动默认

`0`

`osd recovery max active`描述

一次每个OSD的活动恢复请求数。更多请求将加速恢复，但是这些请求会增加群集的负载。类型

32位整数默认

`3`

`osd recovery max chunk`描述

恢复的要推送数据块的最大大小。类型

64位无符号整数默认

`8 << 20`

`osd recovery max single start`描述

恢复OSD时将重新启动的每个OSD的最大恢复操作数。类型

64位无符号整数默认

`1`

`osd recovery thread timeout`描述

超时恢复线程之前的最长时间（以秒为单位）。类型

32位整数默认

`30`

`osd recover clone overlap`描述

在恢复过程中保留克隆重叠。应该始终设置为`true`。类型

布尔型默认

`true`

`osd recovery sleep`描述

下次恢复或重新填充操作之前入睡的时间（以秒为单位）。增加此值将减慢恢复操作，而客户端操作的影响较小。类型

浮动默认

`0`

`osd recovery sleep hdd`描述

下次恢复或回填HDD之前，以秒为单位的睡眠时间。类型

浮动默认

`0.1`

`osd recovery sleep ssd`描述

下次恢复或重新填充SSD之前，以秒为单位的睡眠时间。类型

浮动默认

`0`

`osd recovery sleep hybrid`描述

当osd数据位于HDD上且osd日志位于SSD上时，下次恢复或回填操作之前进入睡眠状态的时间（以秒为单位）。类型

浮动默认

`0.025`

`osd recovery priority`描述

为恢复工作队列设置的默认优先级。与泳池的无关`recovery_priority`。类型

32位整数默认

`5`

### 分层

`osd agent max ops`描述

高速模式下每个分层代理的最大同时冲洗操作数。类型

32位整数默认

`4`

`osd agent max low ops`描述

低速模式下每个分层代理的最大同时冲洗操作数。类型

32位整数默认

`2`

有关何时分层代理在高速模式下冲洗脏对象的信息，请参见[缓存目标脏高比率](https://docs.ceph.com/docs/nautilus/rados/operations/pools#cache-target-dirty-high-ratio)。

### 杂项

`osd snap trim thread timeout`描述

在超时之前，以秒为单位的最长时间。类型

32位整数默认

`60*60*1`

`osd backlog thread timeout`描述

超时等待积压线程的最长时间（以秒为单位）。类型

32位整数默认

`60*60*1`

`osd default notify timeout`描述

OSD默认通知超时（以秒为单位）。类型

32位无符号整数默认

`30`

`osd check for log corruption`描述

检查日志文件是否损坏。在计算上可能会很昂贵。类型

布尔型默认

`false`

`osd remove thread timeout`描述

超时时间（以秒为单位），超时时间为删除OSD线程超时。类型

32位整数默认

`60*60`

`osd command thread timeout`描述

超时命令线程之前的最长时间（以秒为单位）。类型

32位整数默认

`10*60`

`osd command max records`描述

限制要返回的丢失对象的数量。类型

32位整数默认

`256`

`osd fast fail on connection refused`描述

如果启用此选项，则崩溃的OSD会立即由连接的对等方和MON标记下来（假设崩溃的OSD主机仍然存在）。禁用它以恢复旧的行为，以OSD在I / O操作中间崩溃时可能长时间停顿为代价。类型

布尔型默认

`true`

