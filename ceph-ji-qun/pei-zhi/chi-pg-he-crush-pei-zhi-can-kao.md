# 池，PG和CRUSH配置参考

## 池，PG和CRUSH配置参考

当您创建池并设置池的放置组数时，当您没有专门覆盖默认值时，Ceph将使用默认值。**我们建议您**覆盖一些默认设置。具体来说，我们建议设置池的副本大小，并覆盖默认的放置组数。您可以在运行[池](https://docs.ceph.com/docs/nautilus/rados/operations/pools)命令时专门设置这些值。您还可以通过在`[global]`Ceph配置文件的部分中添加新的默认值来覆盖默认值。

```text
[global]

	# By default, Ceph makes 3 replicas of objects. If you want to make four
	# copies of an object the default value--a primary copy and three replica
	# copies--reset the default values as shown in 'osd pool default size'.
	# If you want to allow Ceph to write a lesser number of copies in a degraded
	# state, set 'osd pool default min size' to a number less than the
	# 'osd pool default size' value.

	osd pool default size = 3  # Write an object 3 times.
	osd pool default min size = 2 # Allow writing two copies in a degraded state.

	# Ensure you have a realistic number of placement groups. We recommend
	# approximately 100 per OSD. E.g., total number of OSDs multiplied by 100
	# divided by the number of replicas (i.e., osd pool default size). So for
	# 10 OSDs and osd pool default size = 4, we'd recommend approximately
	# (100 * 10) / 4 = 250.

	osd pool default pg num = 250
	osd pool default pgp num = 250
```

`mon max pool pg num`描述

每个池的最大放置组数。类型

整数默认

`65536`

`mon pg create interval`描述

在同一Ceph OSD守护进程中创建PG之间的秒数。类型

浮动默认

`30.0`

`mon pg stuck threshold`描述

PG被认为卡住的秒数。类型

32位整数默认

`300`

`mon pg min inactive`描述

`HEALTH_ERR`如果PG保持不活动的时间`mon_pg_stuck_threshold`超过此设置的时间，请在群集日志中发出a 。非正数表示已禁用，请勿输入ERR。类型

整数默认

`1`

`mon pg warn min per osd`描述

`HEALTH_WARN`如果每个OSD中的PG的平均数量低于此数量，则在群集日志中发出a 。（非正数将禁用此功能）类型

整数默认

`30`

`mon pg warn min objects`描述

如果群集中的对象总数低于此数目，则不发出警告类型

整数默认

`1000`

`mon pg warn min pool objects`描述

不要警告对象号低于此数字的池类型

整数默认

`1000`

`mon pg check down all threshold`描述

降低OSD百分比的阈值之后，我们将检查所有PG的陈旧状态。类型

浮动默认

`0.5`

`mon pg warn max object skew`描述

`HEALTH_WARN`如果某个池的平均对象数大于整个池的平均对象数，请在群集日志中发出a 。（非正数将禁用此功能）`mon pg warn max object skew`类型

浮动默认

`10`

`mon delta reset interval`描述

在将pg delta重置为0之前，处于非活动状态的秒数。我们跟踪每个池的已用空间的delta，因此，例如，对于我们来说，更容易理解恢复的进度或缓存层的性能。但是，如果没有报告某个池的活动，我们只需重置该池的增量历史记录即可。类型

整数默认

`10`

`mon osd max op age`描述

关注之前的最大操作年龄（使其为2的幂）。`HEALTH_WARN`如果请求被阻止的时间超过此限制，则将发出A。类型

浮动默认

`32.0`

`osd pg bits`描述

每个Ceph OSD守护程序的放置组位。类型

32位整数默认

`6`

`osd pgp bits`描述

PGP的每个Ceph OSD守护程序的位数。类型

32位整数默认

`6`

`osd crush chooseleaf type`描述

`chooseleaf`在CRUSH规则中使用的存储桶类型。使用顺序等级而不是名称。类型

32位整数默认

`1`。通常，一台主机包含一个或多个Ceph OSD守护程序。

`osd crush initial weight`描述

将新添加的osds的初始压缩重量添加到rushmap中。类型

双默认

`the size of newly added osd in TB`。默认情况下，新添加的osd的初始压缩重量设置为以TB为单位的卷大小。有关详细信息，请参见对[存储桶项目](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map#weightingbucketitems)进行[加权](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map#weightingbucketitems)。

`osd pool default crush rule`描述

创建复制池时要使用的默认CRUSH规则。类型

8位整数默认

`-1`，这意味着“选择具有最低数字ID的规则并使用它”。这是为了在没有规则0的情况下创建池。

`osd pool erasure code stripe unit`描述

设置用于擦除编码池的对象条带块的默认大小（以字节为单位）。每个大小为S的对象将存储为N条，每个数据块接收字节。每个字节的条带将分别进行编码/解码。擦除代码配置文件中的设置可以覆盖此选项 。`stripe unitN * stripe unitstripe_unit`类型

无符号32位整数默认

`4096`

`osd pool default size`描述

设置池中对象的副本数。默认值与相同 。`ceph osd pool set {pool-name} size {size}`类型

32位整数默认

`3`

`osd pool default min size`描述

设置池中对象的最小写入副本数，以确认对客户端的写入操作。如果未达到最小值，则Ceph将不会确认对客户端的写入，**这可能会导致数据丢失**。在`degraded`模式下操作时，此设置可确保最少数量的副本。类型

32位整数默认

`0`，表示没有特别的下限。如果是`0`，最小值为。`size - (size / 2)`

`osd pool default pg num`描述

池的默认放置组数。默认值是一样`pg_num`用`mkpool`。类型

32位整数默认

`32`

`osd pool default pgp num`描述

池放置的默认放置组数。默认值是一样`pgp_num`用`mkpool`。PG和PGP应该相等（目前）。类型

32位整数默认

`8`

`osd pool default flags`描述

新池的默认标志。类型

32位整数默认

`0`

`osd max pgls`描述

要列出的展示位置组的最大数量。请求大量请求的客户端可以占用Ceph OSD守护程序。类型

无符号64位整数默认

`1024`注意

默认值可以。

`osd min pg log entries`描述

修剪日志文件时要保留的最小放置组日志数。类型

32位Int Unsigned默认

`1000`

`osd default data pool replay window`描述

OSD等待客户端重播请求的时间（以秒为单位）。类型

32位整数默认

`45`

`osd max pg per osd hard ratio`描述

在OSD拒绝创建新PG之前，集群允许的每个OSD PG数量的比率。如果OSD服务的PG数量超过\*，则OSD停止创建新的PG 。`osd max pg per osd hard ratiomon max pg per osd`类型

浮动默认

`2`

`osd recovery priority`描述

工作队列中恢复的优先级。类型

整数默认

`5`

`osd recovery op priority`描述

如果不覆盖池，则用于恢复操作的默认优先级。类型

整数默认

`3`

