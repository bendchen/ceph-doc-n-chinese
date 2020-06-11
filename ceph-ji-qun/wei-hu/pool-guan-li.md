# POOL管理

## 池

当您首次部署群集而不创建池时，Ceph使用默认池来存储数据。游泳池为您提供：

* **弹性**：您可以设置允许多少OSD发生故障而不丢失数据。对于复制池，它是对象的所需副本数/副本数。典型的配置存储一个对象和一个附加副本（即），但是您可以确定副本/副本的数量。对于[擦除编码池](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code)，它是编码块的数量（即在**擦除代码配置文件中**）`size = 2m=2`
* **放置组**：您可以设置池的放置组数。一个典型的配置每个OSD使用大约100个放置组，以提供最佳的平衡，而不会消耗太多的计算资源。设置多个池时，请确保为池和整个集群设置合理数量的放置组。
* **CRUSH规则**：将数据存储在池中时，对象及其副本（或用于擦除编码池的块）在群集中的位置由CRUSH规则控制。如果默认规则不适用于您的用例，则可以为您的池创建自定义的CRUSH规则。
* **快照**：使用创建快照时，可以有效地为特定池拍摄快照。`ceph osd pool mksnap`

要将数据组织到池中，可以列出，创建和删除池。您还可以查看每个池的利用率统计信息。

### 列表池

要列出集群的池，请执行：

```text
ceph osd lspools
```

### 创建池

在创建池之前，请参考“ [池，PG和CRUSH配置参考”](https://docs.ceph.com/docs/nautilus/rados/configuration/pool-pg-config-ref)。理想情况下，您应该覆盖Ceph配置文件中的展示位置组数的默认值，因为默认值不理想。有关放置组编号的详细信息，请参阅[设置放置组的数量](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups#set-the-number-of-placement-groups)

注意 

从Luminous开始，所有池都需要使用该池与应用程序相关联。有关更多信息，请参见下面的将[池关联到应用程序](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#id1)。

例如：

```text
osd pool default pg num = 100
osd pool default pgp num = 100
```

要创建一个池，执行：

```text
ceph osd pool create {pool-name} {pg-num} [{pgp-num}] [replicated] \
     [crush-rule-name] [expected-num-objects]
ceph osd pool create {pool-name} {pg-num}  {pgp-num}   erasure \
     [erasure-code-profile] [crush-rule-name] [expected_num_objects]
```

哪里：

`{pool-name}`描述

池的名称。它必须是唯一的。类型

串需要

是。

`{pg-num}`描述

池的放置组总数。有关 计算合适数字的详细信息，请参见[展示位置组](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups)。默认值`8`不适用于大多数系统。类型

整数需要

是。默认

8

`{pgp-num}`描述

用于放置目的的放置组总数。除展示位置组拆分方案外，这 **应等于展示位置组的总数**。类型

整数需要

是。如果未指定，则选择默认值或Ceph配置值。默认

8

`{replicated|erasure}`描述

池类型可以通过保留对象的多个副本来**复制**以从丢失的OSD中恢复，也可以通过**擦除**来获得某种 [通用的RAID5](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code)功能。该**复制**池需要更多的原料存储，但实现所有头孢操作。该 **擦除**池需要较少的原料存储，但仅实现了可用操作的一个子集。类型

串需要

没有。默认

复制的

`[crush-rule-name]`描述

用于此池的CRUSH规则的名称。指定的规则必须存在。类型

串需要

没有。默认

对于**复制**池，这是config变量指定的规则。此规则必须存在。对于**擦除**池，它如果[纠删码的个人资料](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-profile)被使用或以其他方式。如果该规则尚不存在，则会隐式创建。`osd pool default crush ruleerasure-codedefault` `{pool-name}`

`[erasure-code-profile=profile]`描述

仅用于**擦除**池。使用[擦除代码配置文件](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-profile)。它必须是由**osd erasure-code-profile set**定义的现有配置 **文件**。类型

串需要

没有。

创建池时，将展示位置组的数量设置为合理的值（例如`100`）。还应考虑每个OSD的放置组总数。放置组在计算上很昂贵，因此，当您有多个包含许多放置组的池时（例如，每个池有100个放置组的50个池），性能将下降。收益递减的点取决于OSD主机的功能。

有关为您的池计算适当数量的展示位置组的详细信息，请参见[展示](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups)位置组。

`[expected-num-objects]`描述

该池的预期对象数。通过设置此值（以及负的**文件存储合并阈值**），PG文件夹拆分将在池创建时发生，以避免对运行时文件夹拆分产生延迟的影响。类型

整数需要

没有。默认

0，在池创建时不拆分。

### 将池关联到应用程序

池在使用前需要与应用程序关联。将与CephFS一起使用的池或RGW自动创建的池自动关联。打算与RBD一起使用的池应使用该`rbd`工具进行初始化（有关更多信息，请参见[块设备命令](https://docs.ceph.com/docs/nautilus/rbd/rados-rbd-cmds/#create-a-block-device-pool)）。

对于其他情况，您可以手动将自由格式的应用程序名称与池关联。

```text
ceph osd pool application enable {pool-name} {application-name}
```

注意 

CephFS使用应用程序名称`cephfs`，RBD使用应用程序名称`rbd`，而RGW使用应用程序名称`rgw`。

### 设置池配额

您可以将池配额设置为每个池的最大字节数和/或最大对象数。

```text
ceph osd pool set-quota {pool-name} [max_objects {obj-count}] [max_bytes {bytes}]
```

例如：

```text
ceph osd pool set-quota data max_objects 10000
```

要删除配额，请将其值设置为`0`。

### 删除池

要删除池，请执行：

```text
ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]
```

要删除池，必须在Monitor的配置中将mon\_allow\_pool\_delete标志设置为true。否则，他们将拒绝删除池。

有关更多信息，请参见[Monitor Configuration](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-config-ref)。

如果您为自己创建的池创建了自己的规则，则在不再需要池时应考虑将其删除：

```text
ceph osd pool get {pool-name} crush_rule
```

例如，如果规则是“ 123”，则可以像这样检查其他池：

```text
ceph osd dump | grep "^pool" | grep "crush_rule 123"
```

如果没有其他池使用该自定义规则，则可以从群集中删除该规则。

如果您创建的用户严格地具有不再存在的池的权限，则也应该考虑删除这些用户：

```text
ceph auth ls | grep -C 5 {pool-name}
ceph auth del {user}
```

### 重命名池

要重命名池，请执行：

```text
ceph osd pool rename {current-pool-name} {new-pool-name}
```

如果重命名池，并且具有已通过身份验证的用户的每个池功能，则必须使用新的池名称更新用户的功能（即上限）。

### 显示池统计信息

要显示池的利用率统计信息，请执行：

```text
rados df
```

此外，要获取特定池或全部池的I / O信息，请执行以下操作：

```text
ceph osd pool stats [{pool-name}]
```

### 制作池快照

要制作池的快照，请执行：

```text
ceph osd pool mksnap {pool-name} {snap-name}
```

### 删除池快照

要删除池的快照，请执行：

```text
ceph osd pool rmsnap {pool-name} {snap-name}
```

### 集池值

要将值设置为池，请执行以下操作：

```text
ceph osd pool set {pool-name} {key} {value}
```

您可以为以下键设置值：

`compression_algorithm`描述

设置用于基础BlueStore的内联压缩算法。此设置将覆盖[全局设置](http://docs.ceph.com/docs/master/rados/configuration/bluestore-config-ref/#inline-compression)的。`bluestore compression algorithm`类型

串有效设定

`lz4`，`snappy`，`zlib`，`zstd`

`compression_mode`描述

为基础BlueStore的嵌入式压缩算法设置策略。此设置将覆盖[全局设置](http://docs.ceph.com/docs/master/rados/configuration/bluestore-config-ref/#inline-compression)的。`bluestore compression mode`类型

串有效设定

`none`，`passive`，`aggressive`，`force`

`compression_min_blob_size`描述

小于此大小的块永远不会被压缩。此设置将覆盖[全局设置](http://docs.ceph.com/docs/master/rados/configuration/bluestore-config-ref/#inline-compression)的。`bluestore compression min blob *`类型

无符号整数

`compression_max_blob_size`描述

大于此的块将`compression_max_blob_size`在压缩之前分解为较小的斑点大小 。类型

无符号整数

`size`描述

设置池中对象的副本数。有关更多详细信息，请参见[设置对象副本数](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#set-the-number-of-object-replicas)。仅复制池。类型

整数

`min_size`描述

设置I / O所需的最小副本数。有关更多详细信息，请参见[设置对象副本数](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#set-the-number-of-object-replicas)。仅复制池。类型

整数版

`0.54` 以上

`pg_num`描述

计算数据放置时要使用的放置组的有效数量。类型

整数有效范围

优于`pg_num`当前值。

`pgp_num`描述

计算数据放置时要使用的放置的有效放置组数。类型

整数有效范围

等于或小于`pg_num`。

`crush_rule`描述

用于在集群中映射对象放置的规则。类型

串

`allow_ec_overwrites`描述

是否写入擦除代码池可以更新对象的一部分，因此cephfs和rbd可以使用它。有关更多详细信息，请参见 [带覆盖的擦除编码](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code#erasure-coding-with-overwrites)。类型

布尔型版

`12.2.0` 以上

`hashpspool`描述

在给定的池上设置/取消设置HASHPSPOOL标志。类型

整数有效范围

1个设置标志，0个未设置标志

`nodelete`描述

在给定的池上设置/取消设置NODELETE标志。类型

整数有效范围

1个设置标志，0个未设置标志版

版 `FIXME`

`nopgchange`描述

在给定的池上设置/取消设置NOPGCHANGE标志。类型

整数有效范围

1个设置标志，0个未设置标志版

版 `FIXME`

`nosizechange`描述

在给定的池上设置/取消设置NOSIZECHANGE标志。类型

整数有效范围

1个设置标志，0个未设置标志版

版 `FIXME`

`write_fadvise_dontneed`描述

在给定的池上设置/取消设置WRITE\_FADVISE\_DONTNEED标志。类型

整数有效范围

1个设置标志，0个未设置标志

`noscrub`描述

在给定的池上设置/取消设置NOSCRUB标志。类型

整数有效范围

1个设置标志，0个未设置标志

`nodeep-scrub`描述

在给定的池上设置/取消设置NODEEP\_SCRUB标志。类型

整数有效范围

1个设置标志，0个未设置标志

`hit_set_type`描述

对高速缓存池启用命中集跟踪。有关其他信息，请参见[Bloom Filter](https://en.wikipedia.org/wiki/Bloom_filter)。类型

串有效设定

`bloom`，`explicit_hash`，`explicit_object`默认

`bloom`。其他值用于测试。

`hit_set_count`描述

要为缓存池存储的命中集数量。该数字越高，`ceph-osd`守护程序消耗的RAM就越多。类型

整数有效范围

`1`。代理尚未处理&gt; 1。

`hit_set_period`描述

高速缓存池的命中设置周期的持续时间（以秒为单位）。该数字越高，`ceph-osd`守护程序消耗的RAM就越多 。类型

整数例

`3600` 1小时

`hit_set_fpp`描述

`bloom`匹配集类型的误报概率。有关其他信息，请参见[Bloom Filter](https://en.wikipedia.org/wiki/Bloom_filter)。类型

双有效范围

0.0-1.0默认

`0.05`

`cache_target_dirty_ratio`描述

在缓存分层代理将其刷新到后备存储池之前，包含修改后的（脏）对象的缓存池的百分比。类型

双默认

`.4`

`cache_target_dirty_high_ratio`描述

在缓存分层代理将它们以较高速度刷新到后备存储池之前，包含修改后的（脏）对象的缓存池的百分比。类型

双默认

`.6`

`cache_target_full_ratio`描述

在缓存分层代理将其从缓存池中驱逐之前，包含未修改（干净）对象的缓存池的百分比。类型

双默认

`.8`

`target_max_bytes`描述

当`max_bytes`触发阈值时，Ceph将开始刷新或逐出对象 。类型

整数例

`1000000000000` ＃1-TB

`target_max_objects`描述

当`max_objects`触发阈值时，Ceph将开始刷新或逐出对象 。类型

整数例

`1000000` ＃1M个对象

`hit_set_grade_decay_rate`描述

两个连续命中点之间的温度衰减率类型

整数有效范围

0-100默认

`20`

`hit_set_search_last_n`描述

计算hit\_sets中最多N个出现以进行温度计算类型

整数有效范围

0-hit\_set\_count默认

`1`

`cache_min_flush_age`描述

缓存分层代理将对象从缓存池刷新到存储池之前的时间（以秒为单位）。类型

整数例

`600` 10分钟

`cache_min_evict_age`描述

缓存分层代理将对象从缓存池中逐出之前的时间（以秒为单位）。类型

整数例

`1800` 30分钟

`fast_read`描述

在擦除编码池上，如果打开此标志，则读取请求将对所有分片发出子读取，并等待直到接收到足够的分片以解码以服务于客户端。对于jerasure和isaerasure插件，一旦返回第一个K答复，就会使用从这些答复中解码的数据立即满足客户的请求。这有助于权衡一些资源以获得更好的性能。当前，仅擦除编码池支持此标志。类型

布尔型默认值

`0`

`scrub_min_interval`描述

负载低时用于清理池的最小时间间隔（以秒为单位）。如果为0，则使用config中的osd\_scrub\_min\_interval值。类型

双默认

`0`

`scrub_max_interval`描述

池清理的最大时间间隔（以秒为单位），与群集负载无关。如果为0，则使用config中的osd\_scrub\_max\_interval值。类型

双默认

`0`

`deep_scrub_interval`描述

池“深度”清理的时间间隔（以秒为单位）。如果为0，则使用config中的osd\_deep\_scrub\_interval值。类型

双默认

`0`

`recovery_priority`描述

设置值后，它将增加或减少计算出的预留优先级。此值的范围必须在-10到10之间。对于不太重要的池，请使用负优先级，以使它们的优先级低于任何新池。类型

整数默认

`0`

`recovery_op_priority`描述

指定该池的恢复操作优先级，而不是`osd_recovery_op_priority`。类型

整数默认

`0`

### 获取池值

要从池中获取值，请执行以下操作：

```text
ceph osd pool get {pool-name} {key}
```

您可能会获得以下键的值：

`size`描述

看[大小](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#size)类型

整数

`min_size`描述

见[min\_size](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#min-size)类型

整数版

`0.54` 以上

`pg_num`描述

见[pg\_num](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#pg-num)类型

整数

`pgp_num`描述

见[pgp\_num](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#pgp-num)类型

整数有效范围

等于或小于`pg_num`。

`crush_rule`描述

见[rush\_rule](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#crush-rule)

`hit_set_type`描述

见[hit\_set\_type](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#hit-set-type)类型

串有效设定

`bloom`，`explicit_hash`，`explicit_object`

`hit_set_count`描述

见[hit\_set\_count](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#hit-set-count)类型

整数

`hit_set_period`描述

参见[hit\_set\_period](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#hit-set-period)类型

整数

`hit_set_fpp`描述

见[hit\_set\_fpp](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#hit-set-fpp)类型

双

`cache_target_dirty_ratio`描述

参见[cache\_target\_dirty\_ratio](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#cache-target-dirty-ratio)类型

双

`cache_target_dirty_high_ratio`描述

参见[cache\_target\_dirty\_high\_ratio](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#cache-target-dirty-high-ratio)类型

双

`cache_target_full_ratio`描述

请参阅[cache\_target\_full\_ratio](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#cache-target-full-ratio)类型

双

`target_max_bytes`描述

参见[target\_max\_bytes](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#target-max-bytes)类型

整数

`target_max_objects`描述

参见[target\_max\_objects](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#target-max-objects)类型

整数

`cache_min_flush_age`描述

参见[cache\_min\_flush\_age](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#cache-min-flush-age)类型

整数

`cache_min_evict_age`描述

参见[cache\_min\_evict\_age](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#cache-min-evict-age)类型

整数

`fast_read`描述

见[fast\_read](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#fast-read)类型

布尔型

`scrub_min_interval`描述

参见[scrub\_min\_interval](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#scrub-min-interval)类型

双

`scrub_max_interval`描述

参见[scrub\_max\_interval](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#scrub-max-interval)类型

双

`deep_scrub_interval`描述

见[deep\_scrub\_interval](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#deep-scrub-interval)类型

双

`allow_ec_overwrites`描述

参见[allow\_ec\_overwrites](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#allow-ec-overwrites)类型

布尔型

`recovery_priority`描述

请参阅[recovery\_priority](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#recovery-priority)类型

整数

`recovery_op_priority`描述

见[recovery\_op\_priority](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#recovery-op-priority)类型

整数

### 设置对象副本数

要设置复制池上对象副本的数量，请执行以下操作：

```text
ceph osd pool set {poolname} size {num-replicas}
```

重要 

的`{num-replicas}`包括所述对象本身。如果要使用该对象和该对象的两个副本，以总共三个对象的实例，请指定`3`。

例如：

```text
ceph osd pool set data size 3
```

您可以为每个池执行此命令。**注意：**对象可能以降级模式接受的I / O数量少于副本的数量。要设置I / O所需的最小副本数，应使用该设置。例如：`pool sizemin_size`

```text
ceph osd pool set data min_size 2
```

这样可以确保数据池中的任何对象都不会收到少于`min_size`副本的I / O。

### 获取对象副本数

要获取对象副本的数量，请执行以下操作：

```text
ceph osd dump | grep 'replicated size'
```

Ceph将列出池，并突出显示该属性。默认情况下，ceph创建一个对象的两个副本（共三个副本，或3个大小）。`replicated size`

