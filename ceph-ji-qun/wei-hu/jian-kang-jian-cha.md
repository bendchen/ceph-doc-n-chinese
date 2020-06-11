# 健康检查

## 健康检查

### 概述

Ceph集群可以发出有限的一组可能的健康消息-这些被定义为具有唯一标识符的_健康检查_。

标识符是一个简短的伪人类可读（如变量名）字符串。旨在使工具（例如UI）能够进行健康检查，并以反映其含义的方式进行显示。

此页面列出了监视器和管理器守护程序引发的运行状况检查。除此之外，您还可能会看到源自MDS守护程序的运行状况检查（请参阅[CephFS运行状况消息](https://docs.ceph.com/docs/nautilus/cephfs/health-messages/#cephfs-health-messages)）以及[ceph](https://docs.ceph.com/docs/nautilus/cephfs/health-messages/#cephfs-health-messages) -mgr python模块定义的运行状况检查。

### 定义

#### 监控器

**MON\_DOWN** 

当前有一个或多个监视守护程序已关闭。该群集需要大多数（超过1/2）的监视器才能正常运行。当一个或多个监视器关闭时，客户端可能很难与群集建立初始连接，因为它们可能需要在尝试运行监视器之前尝试更多地址。

通常，应该尽快重新启动关闭监视器的守护程序，以减少子监视器失败导致服务中断的风险。

**MON\_CLOCK\_SKEW** 

运行ceph-mon监视器守护程序的主机上的时钟同步不够充分。如果群集检测到时钟偏差大于，则会发出此健康警报`mon_clock_drift_allowed`。

最好使用诸如`ntpd`或的工具对时钟进行同步来解决 `chrony`。

如果使时钟保持紧密同步是不切实际的， `mon_clock_drift_allowed`也可以增加阈值，但是此值必须保持在明显低于该`mon_lease`间隔的水平，才能使监视器群集正常运行。

**MON\_MSGR2\_NOT\_ENABLED** 

`ms_bind_msgr2`启用了该选项，但未将一个或多个监视器配置为绑定到群集monmap中的v2端口。这意味着特定于msgr2协议的功能（例如，加密）在某些或所有连接上不可用。

在大多数情况下，可以通过发出以下命令来纠正此问题：

```text
ceph mon enable-msgr2
```

该命令将更改为旧的默认端口6789配置的任何监视器，以继续侦听6789上的v1连接，并继续侦听新的默认3300端口上的v2连接。

如果将监视器配置为侦听非标准端口（不是6789）上的v1连接，则需要手动修改monmap。

#### 经理

**MGR\_MODULE\_DEPENDENCY** 

已启用的管理器模块未能通过依赖关系检查。此运行状况检查应随附来自模块的有关该问题的解释性消息。

例如，某个模块可能报告未安装所需的软件包：安装所需的软件包并重新启动管理器守护程序。

此运行状况检查仅适用于已启用的模块。如果未启用模块，则可以在ceph模块ls的输出中查看它是否正在报告依赖性问题。

**MGR\_MODULE\_ERROR** 

管理器模块遇到意外错误。通常，这意味着从模块的服务 功能引发了未处理的异常。如果异常未提供其自身的有用描述，则可能会用措辞含糊的措辞来使人可读的错误描述。

此运行状况检查可能表明存在错误：如果您认为自己遇到错误，请打开Ceph错误报告。

如果您认为错误是暂时的，则可以重新启动管理器守护程序，或在活动守护程序上使用ceph mgr fail来提示故障转移到另一个守护程序。

#### 屏上显示

**OSD\_DOWN** 

标记了一个或多个OSD。ceph-osd守护程序可能已停止，或者对等OSD可能无法通过网络访问OSD。常见原因包括守护程序停止或崩溃，主机关闭或网络中断。

验证主机是否正常，守护程序已启动以及网络是否正常运行。如果守护程序已崩溃，则守护程序日志文件（`/var/log/ceph/ceph-osd.*`）可能包含调试信息。

**OSD\_ &lt;压碎类型&gt; \_DOWN** 

（例如OSD\_HOST\_DOWN，OSD\_ROOT\_DOWN）

特定CRUSH子树中的所有OSD都被标记下来，例如主机上的所有OSD。

**OSD\_ORPHAN** 

OSD在CRUSH映射层次结构中被引用，但不存在。

可以使用以下方法从CRUSH层次结构中删除OSD：

```text
ceph osd crush rm osd.<id>
```

**OSD\_OUT\_OF\_ORDER\_FULL** 

backfillfull，nearfull，full和/或failsafe\_full的利用率阈值未提高。特别是，我们期望 backfillfull &lt;nearfull，nearfull &lt;full和full &lt;failsafe\_full。

阈值可以通过以下方式调整：

```text
ceph osd set-backfillfull-ratio <ratio>
ceph osd set-nearfull-ratio <ratio>
ceph osd set-full-ratio <ratio>
```

**OSD\_FULL** 

一个或多个OSD超出了整个阈值，并正在阻止群集为写入提供服务。

可以通过以下方式检查池的利用率：

```text
ceph df
```

当前定义的满比率可以通过以下方式看到：

```text
ceph osd dump | grep full_ratio
```

恢复写可用性的一种短期解决方法是将整个阈值提高一点：

```text
ceph osd set-full-ratio <ratio>
```

应该通过部署更多OSD将新存储添加到群集，或者应该删除现有数据以释放空间。

**OSD\_BACKFILLFULL** 

一个或多个OSD超过了回填阈值，这将阻止数据重新平衡到该设备。这是一个早期警告，即重新平衡可能无法完成，并且群集即将用尽。

可以通过以下方式检查池的利用率：

```text
ceph df
```

**OSD\_NEARFULL** 

一个或多个OSD已超过接近满阈值。这是群集即将满的预警。

可以通过以下方式检查池的利用率：

```text
ceph df
```

**OSDMAP\_FLAGS** 

已设置一个或多个感兴趣的群集标志。这些标志包括：

* _已满_ -群集被标记为已满，无法处理写入
* _pauserd_，_pausewr-_暂停的读取或写入
* _noup-_不允许启动OSD
* _nodown_ -OSD故障报告被忽略，因此监视器不会将OSD标记为向下
* _诺茵_ -以前标记屏上显示出来，不会标示后面的，当他们开始
* _noout-_在配置的间隔后，向下的OSD不会自动标记出来
* _nobackfill_，_norecover_，_norebalance-_恢复或数据重新平衡已暂停
* _noscrub_，_nodeep\_scrub-_禁用清理
* _notieragent-_缓存分层活动已暂停

除_full之外_，可以使用以下方式设置或清除这些标志：

```text
ceph osd set <flag>
ceph osd unset <flag>
```

**OSD\_FLAGS** 

一个或多个OSD或CRUSH {nodes，device classes}已设置了关注标记。这些标志包括：

* _noup_：这些OSD不允许启动
* _nodown_：这些OSD的故障报告将被忽略
* _noin_：如果这些OSD 在故障后先前已自动标记出来，则它们在启动时将不会被标记
* _noout_：如果这些OSD处于关闭状态，则在配置的间隔后它们不会自动被标记 出来

可以通过以下方式批量设置和清除这些标志：

```text
ceph osd set-group <flags> <who>
ceph osd unset-group <flags> <who>
```

例如，

```text
ceph osd set-group noup,noout osd.0 osd.1
ceph osd unset-group noup,noout osd.0 osd.1
ceph osd set-group noup,noout host-foo
ceph osd unset-group noup,noout host-foo
ceph osd set-group noup,noout class-hdd
ceph osd unset-group noup,noout class-hdd
```

**OLD\_CRUSH\_TUNABLES** 

CRUSH地图使用的设置非常旧，应该进行更新。可以使用的最旧的可调参数（即可以连接到群集的最旧的客户端版本）在不触发此运行状况警告的情况下由`mon_crush_min_required_version`config选项确定。有关更多信息，请参见[可调参数](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map/#crush-map-tunables)。

**OLD\_CRUSH\_STRAW\_CALC\_VERSION** 

CRUSH映射正在使用较旧的非最佳方法来计算`straw`铲斗的中间重量值。

CRUSH图应更新为使用较新的方法（`straw_calc_version=1`）。有关更多信息，请参见 [可调参数](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map/#crush-map-tunables)。

**CACHE\_POOL\_NO\_HIT\_SET** 

一个或多个高速缓存池未配置有_命中集_来跟踪利用率，这将阻止分层代理识别要从高速缓存中清除和退出的冷对象。

命中集可以在缓存池上通过以下方式配置：

```text
ceph osd pool set <poolname> hit_set_type <type>
ceph osd pool set <poolname> hit_set_period <period-in-seconds>
ceph osd pool set <poolname> hit_set_count <number-of-hitsets>
ceph osd pool set <poolname> hit_set_fpp <target-false-positive-rate>
```

**OSD\_NO\_SORTBITWISE** 

没有发光的v12.yz OSD在运行，但`sortbitwise`尚未设置该标志。

`sortbitwise`必须先设置该标志，然后才能启动发光的v12.yz或更新的OSD。您可以使用以下方法安全地设置标志：

```text
ceph osd set sortbitwise
```

**POOL\_FULL** 

一个或多个池已达到其配额，并且不再允许写入。

可以通过以下方式查看池配额和利用率：

```text
ceph df detail
```

您可以通过以下方法提高池配额：

```text
ceph osd pool set-quota <poolname> max_objects <num-objects>
ceph osd pool set-quota <poolname> max_bytes <num-bytes>
```

或删除一些现有数据以降低利用率。

**BLUEFS\_SPILLOVER** 

已为使用BlueStore后端的一个或多个OSD分配了 db分区（用于元数据的存储空间，通常在较快的设备上），但该空间已满，因此元数据已“溢出”到正常的慢速设备上。这不一定是错误情况，甚至不一定是意外情况，但是如果管理员期望所有元数据都适合更快的设备，则表明没有提供足够的空间。

可以使用以下命令在所有OSD上禁用此警告：

```text
ceph config set osd bluestore_warn_on_bluefs_spillover false
```

或者，可以使用以下命令在特定的OSD上禁用它：

```text
ceph config set osd.123 bluestore_warn_on_bluefs_spillover false
```

为了提供更多的元数据空间，可以销毁和重新配置所讨论的OSD。这将涉及数据迁移和恢复。

也可以扩展支持数据库存储的LVM逻辑卷 。如果基础LV已扩展，则需要停止OSD守护程序，并通过以下命令将设备大小更改通知BlueFS：

```text
ceph-bluestore-tool bluefs-bdev-expand --path /var/lib/ceph/osd/ceph-$ID
```

**BLUEFS\_AVAILABLE\_SPACE** 

要检查BlueFS有多少可用空间，请执行以下操作：

```text
ceph daemon osd.123 bluestore bluefs available
```

这将最多输出3个值：BDEV\_DB free，BDEV\_SLOW free和 available\_from\_bluestore。BDEV\_DB和BDEV\_SLOW报告BlueFS已获取的空间量，并认为这是可用空间。值available\_from\_bluestore 表示BlueStore释放更多空间给BlueFS的能力。通常，此值与BlueStore可用空间量不同，因为BlueFS分配单位通常大于BlueStore分配单位。这意味着BlueFS仅可接受部分BlueStore可用空间。

**BLUEFS\_LOW\_SPACE** 

如果BlueFS上的可用空间不足，并没有什么 available\_from\_bluestore一个可以考虑减少BlueFS分配单元的大小。要模拟分配单位不同时的可用空间，请执行以下操作：

```text
ceph daemon osd.123 bluestore bluefs available <alloc-unit-size>
```

**BLUESTORE\_FRAGMENTATION** 

随着BlueStore的工作，底层存储上的可用空间将变得零散。这是正常且不可避免的，但过多的碎片会导致速度降低。要检查BlueStore碎片，可以执行以下操作：

```text
ceph daemon osd.123 bluestore allocator score block
```

分数在\[0-1\]范围内。\[0.0 .. 0.4\]小碎片\[0.4 .. 0.7\]小，可接受的碎片\[0.7 .. 0.9\]相当大，但是安全碎片\[0.9 .. 1.0\]严重碎片，可能会影响BlueFS从BlueStore获取空间的能力

如果需要详细的免费碎片报告，请执行以下操作：

```text
ceph daemon osd.123 bluestore allocator dump block
```

如果要处理未运行的OSD进程，则可以使用ceph-bluestore-tool检查。获取碎片分数：

```text
ceph-bluestore-tool --path /var/lib/ceph/osd/ceph-123 --allocator block free-score
```

并转储详细的免费块：

```text
ceph-bluestore-tool --path /var/lib/ceph/osd/ceph-123 --allocator block free-dump
```

**BLUESTORE\_LEGACY\_STATFS** 

在Nautilus版本中，BlueStore逐个跟踪其内部使用情况统计信息，并且一个或多个OSD具有在Nautilus之前创建的BlueStore卷。如果_所有_ OSD都早于Nautilus，则仅表示每个池指标不可用。但是，如果Nautilus之前和Nautilus之后的OSD混合使用，则报告的群集使用情况统计信息将不准确。`ceph df`

通过停止每个OSD，运行修复操作并重新启动它，可以更新旧的OSD以使用新的使用情况跟踪方案。例如，如果`osd.123`需要更新，则：

```text
systemctl stop ceph-osd@123
ceph-bluestore-tool repair --path /var/lib/ceph/osd/ceph-123
systemctl start ceph-osd@123
```

可以通过以下方式禁用此警告：

```text
ceph config set global bluestore_warn_on_legacy_statfs false
```

**BLUESTORE\_DISK\_SIZE\_MISMATCH** 

使用BlueStore的一个或多个OSD在物理设备的大小和跟踪其大小的元数据之间存在内部不一致。将来可能会导致OSD崩溃。

相关的OSD应该销毁并重新配置。应当小心地一次执行一个OSD，并且这种方式不会使任何数据受到威胁。例如，如果osd `$N`出现错误，则：

```text
ceph osd out osd.$N
while ! ceph osd safe-to-destroy osd.$N ; do sleep 1m ; done
ceph osd destroy osd.$N
ceph-volume lvm zap /path/to/device
ceph-volume lvm create --osd-id $N --data /path/to/device
```

#### 设备运行状况

**DEVICE\_HEALTH** 

预期一台或多台设备很快就会发生故障，其中警告阈值由`mgr/devicehealth/warn_threshold` config选项控制。

此警告仅适用于当前标记为“输入”的OSD，因此对此故障的预期响应是将设备标记为“输出”，以便将数据从设备中迁移出来，然后从系统中删除硬件。请注意，如果`mgr/devicehealth/self_heal`根据启用了标记，通常会自动进行标记`mgr/devicehealth/mark_out_threshold`。

可以使用以下方法检查设备的运行状况：

```text
ceph device info <device-id>
```

设备预期寿命由mgr或外部工具通过以下命令运行的预测模型设置：

```text
ceph device set-life-expectancy <device-id> <from> <to>
```

您可以手动更改存储的预期寿命，但是通常不会完成任何操作，因为最初设置的任何工具都可能会再次设置它，并且更改存储的值不会影响硬件设备的实际运行状况。

**DEVICE\_HEALTH\_IN\_USE** 

预期一台或多台设备很快就会发生故障，并且已根据标记为“不在集群中” `mgr/devicehealth/mark_out_threshold`，但该设备仍在参与另外一个PG。这可能是因为它最近才被标记为“出”，并且数据仍在迁移中，或者由于某种原因（例如，集群几乎已满，或者CRUSH层次结构使得没有其他合适的数据）而无法迁移数据。 OSD也可以迁移数据）。

通过禁用自我修复行为（设置`mgr/devicehealth/self_heal`为false），调整 `mgr/devicehealth/mark_out_threshold`或解决阻止数据从故障设备中迁移的问题，可以使此消息静音。

**DEVICE\_HEALTH\_TOOMANY** 

预计很快会有太多设备发生故障并`mgr/devicehealth/self_heal`启用了该 行为，因此，标记出所有不正常的设备将超出群集 `mon_osd_min_in_ratio`比率，这将阻止过多的OSD被自动标记为“ out”。

通常，这表明群集中的太多设备将很快出现故障，因此您应采取措施添加更多（更健康）的设备，然后再出现太多设备出现故障和数据丢失的情况。

也可以通过调整参数（例如`mon_osd_min_in_ratio`或）来使运行状况消息静音 `mgr/devicehealth/mark_out_threshold`，但是要警告该消息会增加群集中不可恢复的数据丢失的可能性。

#### 数据健康（池和放置组）

**PG\_AVAILABILITY** 

数据可用性降低，这意味着群集无法满足群集中某些数据的潜在读取或写入请求。具体而言，一个或多个PG处于不允许为IO请求提供服务的状态。有问题的PG状态包括_对等_， _陈旧_，_不完整_和缺乏_活动_（如果这些条件不能很快消除）。

有关受影响的PG的详细信息可从以下网站获得：

```text
ceph health detail
```

在大多数情况下，根本原因是一个或多个OSD当前关闭。请参阅`OSD_DOWN`上面的讨论。

特定问题PG的状态可以通过以下方式查询：

```text
ceph tell <pgid> query
```

**PG\_DEGRADED** 

对于某些数据，减少了数据冗余，这意味着群集没有为所有数据（对于复制池）或擦除代码片段（对于擦除编码池）具有所需数量的副本。具体来说，一个或多个PG：

* 设置了_降级_或大小_不足的_标志，这意味着集群中没有足够的该展示位置组实例；
* 一段时间未设置_清洁_标志。

有关受影响的PG的详细信息可从以下网站获得：

```text
ceph health detail
```

在大多数情况下，根本原因是一个或多个OSD当前关闭。参见`OSD_DOWN`上面的讨论。

特定问题PG的状态可以通过以下方式查询：

```text
ceph tell <pgid> query
```

**PG\_RECOVERY\_FULL** 

由于群集中没有可用空间，因此可能会减少数据冗余或对某些数据有风险。具体来说，一个或多个PG 设置了 _recovery\_toofull_标志，这意味着群集无法迁移或恢复数据，因为一个或多个OSD高于_满_阈值。

请参阅上面有关_OSD\_FULL_的讨论，以获取解决此问题的步骤。

**PG\_BACKFILL\_FULL** 

由于群集中没有可用空间，因此可能会减少数据冗余或对某些数据有风险。具体来说，一个或多个PG 设置了 _backfill\_toofull_标志，这意味着群集无法迁移或恢复数据，因为一个或多个OSD高于_backfillfull_阈值。

有关解决此情况的步骤，请参见上面有关_OSD\_BACKFILLFULL_的讨论。

**PG\_DAMAGED** 

数据清理发现了集群中数据一致性的一些问题。具体而言，一个或多个PG设置了_不一致_或 _snaptrim\_error_标志，表明较早的_清理_操作发现了问题，或者设置了_修复_标志，这意味着当前正在进行这种不一致的修复。

有关更多信息，请参见[修复PG不一致](https://docs.ceph.com/docs/nautilus/rados/operations/pg-repair/)。

**OSD\_SCRUB\_ERRORS** 

最近的OSD清理发现了不一致的地方。此错误通常与_PG\_DAMAGED_配对（请参见上文）。

有关更多信息，请参见[修复PG不一致](https://docs.ceph.com/docs/nautilus/rados/operations/pg-repair/)。

**LARGE\_OMAP\_OBJECTS** 

一个或多个池包含较大的omap对象，这些对象由 `osd_deep_scrub_large_omap_object_key_threshold`（确定大型omap对象的键数阈值）或`osd_deep_scrub_large_omap_object_value_sum_threshold`（确定大型omap对象 的所有键值的总大小（字节）的阈值）或两者确定。通过在群集日志中搜索“找到的大型omap对象”，可以找到有关对象名称，键数和字节大小的更多信息。大型omap对象可能是由未启用自动重新分片的RGW存储桶索引对象引起的。有关重新分[片](https://docs.ceph.com/docs/nautilus/radosgw/dynamicresharding/#rgw-dynamic-bucket-index-resharding)的更多信息，请参见[RGW动态存储桶索引重新](https://docs.ceph.com/docs/nautilus/radosgw/dynamicresharding/#rgw-dynamic-bucket-index-resharding)分片。

阈值可以通过以下方式调整：

```text
ceph config set osd osd_deep_scrub_large_omap_object_key_threshold <keys>
ceph config set osd osd_deep_scrub_large_omap_object_value_sum_threshold <bytes>
```

**CACHE\_POOL\_NEAR\_FULL** 

缓存层池几乎已满。在这种情况下，已满由缓存池上的`target_max_bytes`和`target_max_objects`属性确定。一旦池达到目标阈值，在刷新数据并从高速缓存中逐出数据时，可能会阻止对池的写入请求，这种状态通常会导致很高的延迟和较差的性能。

可以通过以下方式调整缓存池目标大小：

```text
ceph osd pool set <cache-pool-name> target_max_bytes <bytes>
ceph osd pool set <cache-pool-name> target_max_objects <objects>
```

由于降低了基本层的可用性或性能，或降低了总体群集负载，因此也可能限制了正常的缓存刷新和逐出活动。

**TOO\_FEW\_PGS** 

集群中正在使用的PG数量低于`mon_pg_warn_min_per_osd`每个OSD PG 的可配置阈值。这会导致群集中OSD上的数据分布不理想和平衡，并且同样会降低整体性能。

如果尚未创建数据池，则可能是预期的情况。

可以增加现有池的PG数量，也可以创建新的池。有关更多信息，请参考[选择放置组的数量](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups/#choosing-number-of-placement-groups)。

**POOL\_PG\_NUM\_NOT\_POWER\_OF\_TWO** 

一个或多个池的`pg_num`值不是2的幂。尽管这并非严格错误，但确实会导致数据分配不平衡，因为某些PG的数据大约是其他PG的两倍。

通过将`pg_num`受影响的池的值设置为附近的2的幂可以很容易地纠正此问题：

```text
ceph osd pool set <pool-name> pg_num <value>
```

可以通过以下方式禁用此健康警告：

```text
ceph config set global mon_warn_on_pool_pg_num_not_power_of_two false
```

**POOL\_TOO\_FEW\_PGS** 

根据池中当前存储的数据量，一个或多个池可能应该具有更多的PG。这会导致群集中OSD上的数据分布不理想和平衡，并且同样会降低整体性能。如果`pg_autoscale_mode`池上的属性设置为， 则会生成此警告`warn`。

要禁用该警告，可以完全使用以下方式禁用该池的PG的自动缩放：

```text
ceph osd pool set <pool-name> pg_autoscale_mode off
```

要允许集群自动调整PG的数量，请执行以下操作：

```text
ceph osd pool set <pool-name> pg_autoscale_mode on
```

您还可以使用以下方法将池的PG数量手动设置为建议数量：

```text
ceph osd pool set <pool-name> pg_num <new-pg-num>
```

有关更多信息，请参考[选择放置组](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups/#choosing-number-of-placement-groups)和 [自动缩放放置组](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups/#pg-autoscaler)的数量。

**TOO\_MANY\_PGS** 

群集中正在使用的PG数量超过了`mon_max_pg_per_osd`每个OSD PG 的可配置阈值。如果超过此阈值，集群将不允许创建新池，增加池pg\_num或增加池复制（其中任何一个都会导致集群中有更多PG）。大量PG可能导致OSD守护程序使用更高的内存，在集群状态更改（例如OSD重新启动，添加或删除）后，对等的速度变慢，以及Manager和Monitor守护程序上的负载增加。

解决问题的最简单方法是通过添加更多硬件来增加群集中OSD的数量。请注意，用于此运行状况检查的OSD计数是“内” OSD的数量，因此将“外” OSD标记为“内”（如果有）也有帮助：

```text
ceph osd in <osd id(s)>
```

有关更多信息，请参考[选择放置组的数量](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups/#choosing-number-of-placement-groups)。

**POOL\_TOO\_MANY\_PGS** 

根据池中当前存储的数据量，一个或多个池可能应该具有更多的PG。这可能会导致OSD守护程序的内存利用率更高，集群状态更改（例如OSD重新启动，添加或删除）后的对等速度变慢，以及Manager和Monitor守护程序的负载更高。如果`pg_autoscale_mode`池上的属性设置为，则会生成此警告 `warn`。

要禁用该警告，可以完全使用以下方式禁用该池的PG的自动缩放：

```text
ceph osd pool set <pool-name> pg_autoscale_mode off
```

要允许集群自动调整PG的数量，请执行以下操作：

```text
ceph osd pool set <pool-name> pg_autoscale_mode on
```

您还可以使用以下方法将池的PG数量手动设置为建议数量：

```text
ceph osd pool set <pool-name> pg_num <new-pg-num>
```

有关更多信息，请参考[选择放置组](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups/#choosing-number-of-placement-groups)和 [自动缩放放置组](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups/#pg-autoscaler)的数量。

**POOL\_TARGET\_SIZE\_BYTES\_OVERCOMMITTED** 

一个或多个池具有`target_size_bytes`设置为估计池的预期大小的属性，但该值超过了总可用存储量（无论是单独使用还是与其他池的实际使用量结合使用）。

这通常表明`target_size_bytes`池的值太大，应使用以下方法将其减小或设置为零：

```text
ceph osd pool set <pool-name> target_size_bytes 0
```

有关更多信息，请参阅[指定期望的池大小](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups/#specifying-pool-target-size)。

**POOL\_HAS\_TARGET\_SIZE\_BYTES\_AND\_RATIO** 

一个或多个池同时具有这两个池，`target_size_bytes`并 `target_size_ratio`设置为估计池的预期大小。这些属性中只有一个应为非零。如果两者都设置， `target_size_ratio`则优先并且将`target_size_bytes`被忽略。

重置`target_size_bytes`为零：

```text
ceph osd pool set <pool-name> target_size_bytes 0
```

有关更多信息，请参阅[指定期望的池大小](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups/#specifying-pool-target-size)。

**TOO\_FEW\_OSDS** 

群集中的OSD数量低于的可配置阈值`osd_pool_default_size`。

**SMALLER\_PGP\_NUM** 

一个或多个池的`pgp_num`值小于`pg_num`。这通常表明PG计数增加而放置行为也不增加。

有时会故意进行此操作，以在调整PG计数时将拆分步骤与`pgp_num`更改时所需的数据迁移分开。

通常可以通过设置`pgp_num`match `pg_num`，触发数据迁移来解决此问题，方法是：

```text
ceph osd pool set <pool> pgp_num <pg-num-value>
```

**MANY\_OBJECTS\_PER\_PG** 

每个PG一个或多个池的平均对象数明显高于整个群集的平均值。特定阈值由`mon_pg_warn_max_object_skew` 配置值控制。

这通常表明群集中包含大多数数据的一个或多个池中的PG太少，和/或其他不包含太多数据的池中的PG过多。请参阅上面对_TOO\_MANY\_PGS_的讨论 。

可以通过调整`mon_pg_warn_max_object_skew`监视器上的config选项来提高阈值以使运行状况警告静音。

**POOL\_APP\_NOT\_ENABLED** 

存在一个池，其中包含一个或多个对象，但没有被标记为由特定应用程序使用。

通过标记池供应用程序使用来解决此警告。例如，如果RBD使用该池，则：

```text
rbd pool init <poolname>
```

如果自定义应用程序'foo'正在使用该池，则还可以通过低级命令进行标记：

```text
ceph osd pool application enable foo
```

有关更多信息，请参见将[池关联到应用程序](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#associate-pool-to-application)。

**POOL\_FULL** 

一个或多个池已达到（或非常接近达到）其配额。触发此错误情况的阈值由`mon_pool_quota_crit_threshold`配置选项控制。

可以通过以下方式向上或向下（或删除）调整池配额：

```text
ceph osd pool set-quota <pool> max_bytes <bytes>
ceph osd pool set-quota <pool> max_objects <objects>
```

将配额值设置为0将禁用配额。

**POOL\_NEAR\_FULL** 

配额正在接近一个或多个池。触发此警告条件的阈值由`mon_pool_quota_warn_threshold`配置选项控制 。

可以通过以下方式向上或向下（或删除）调整池配额：

```text
ceph osd pool set-quota <pool> max_bytes <bytes>
ceph osd pool set-quota <pool> max_objects <objects>
```

将配额值设置为0将禁用配额。

**OBJECT\_MISPLACED** 

集群中的一个或多个对象未存储在集群希望存储在其上的节点上。这表明由于某些最近的群集更改而导致的数据迁移尚未完成。

错放的数据本身并不是危险的情况；数据一致性永远不会受到威胁，并且在存在所需数量的新副本（位于所需位置）之前，绝不会删除对象的旧副本。

**OBJECT\_UNFOUND** 

找不到群集中的一个或多个对象。具体地说，OSD知道应该存在对象的新副本或更新副本，但是在当前在线的OSD上找不到该对象版本的副本。

对未找到对象的读取或写入请求将被阻止。

理想情况下，关闭的OSD可以重新联机，该OSD具有未找到对象的最新副本。可以从对等状态中标识负责未找到对象的PG的候选OSD：

```text
ceph tell <pgid> query
```

如果该对象的最新副本不可用，则可以告知群集回滚到该对象的先前版本。有关更多信息，请参见未 [找到对象](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-pg/#failures-osd-unfound)。

**SLOW\_OPS** 

一个或多个OSD请求需要很长时间才能处理。这可能表示负载过大，存储设备缓慢或软件错误。

可以使用从OSD主机执行的以下命令来查询有问题的OSD上的请求队列：

```text
ceph daemon osd.<id> ops
```

可以通过以下方式查看最近请求最慢的摘要：

```text
ceph daemon osd.<id> dump_historic_ops
```

OSD的位置可以通过以下方式找到：

```text
ceph osd find osd.<id>
```

**PG\_NOT\_SCRUBBED** 

最近尚未擦除一台或多台PG。PG通常每秒钟清理一次`mon_scrub_interval`，如果`mon_warn_pg_not_scrubbed_ratio`间隔的百分比已到期，则该警告会在间隔百分比过去后触发。

PG如果未标记为_clean_，则不会进行_清理_，如果它们放错位置或降级，则可能会发生（请参见上面的_PG\_AVAILABILITY_和 _PG\_DEGRADED_）。

您可以使用以下方法手动启动清洁PG的清理：

```text
ceph pg scrub <pgid>
```

**PG\_NOT\_DEEP\_SCRUBBED** 

最近尚未对一台或多台PG进行深度清理。PG通常每秒钟清理一次`osd_deep_scrub_interval`，如果`mon_warn_pg_not_deep_scrubbed_ratio`间隔的百分比已到期，则该警告会在间隔百分比过去后触发。

如果PG未被标记为_clean_，它们将不会（深度）_清理_，如果它们放错位置或降级，则可能会发生（请参见上面的_PG\_AVAILABILITY_和 _PG\_DEGRADED_）。

您可以使用以下方法手动启动清洁PG的清理：

```text
ceph pg deep-scrub <pgid>
```

#### 杂项

**RECENT\_CRASH** 

一个或多个Ceph守护进程最近已崩溃，并且该崩溃尚未由管理员存档（确认）。这可能表示软件错误，硬件问题（例如磁盘故障）或其他问题。

新的崩溃可以通过以下方式列出：

```text
ceph crash ls-new
```

有关特定崩溃的信息可以通过以下方法检查：

```text
ceph crash info <crash-id>
```

可以通过“存档”崩溃（可能是在管理员检查之后）来消除此警告，从而不会生成此警告：

```text
ceph crash archive <crash-id>
```

同样，所有新的崩溃都可以通过以下方式存档：

```text
ceph crash archive-all
```

已归档的崩溃仍然可以通过看到，但看不到 。`ceph crash lsceph crash ls-new`

“最近”所指的时间段由选项控制 `mgr/crash/warn_recent_interval`（默认值：两周）。

可以通过以下方式完全禁用这些警告：

```text
ceph config set mgr/crash/warn_recent_interval 0
```

**TELEMETRY\_CHANGED** 

遥测功能已启用，但此后遥测报告的内容已更改，因此将不发送遥测报告。

Ceph开发人员会定期修改遥测功能，以包括新的有用信息，或删除发现无用或敏感的信息。如果报告中包含任何新信息，Ceph将要求管理员重新启用遥测功能，以确保他们有机会（重新）查看将共享的信息。

要查看遥测报告的内容，请执行以下操作：

```text
ceph telemetry show
```

请注意，遥测报告由几个可选通道组成，这些通道可以独立启用或禁用。有关更多信息，请参阅 [遥测模块](https://docs.ceph.com/docs/nautilus/mgr/telemetry/#telemetry)。

要重新启用遥测（并使此警告消失），请执行以下操作：

```text
ceph telemetry on
```

要禁用遥测（并使该警告消失），请执行以下操作：

```text
ceph telemetry off
```

