# 高速缓存分层

## 高速缓存分层

缓存层为Ceph客户端提供了更好的I / O性能，用于存储在后备存储层中的部分数据。缓存分层涉及创建配置为充当缓存层的相对较快/昂贵的存储设备（例如，固态驱动器）的池，以及配置为充当经济存储的擦除编码或相对较慢/更便宜的设备的后备池层。Ceph反对者处理放置对象的位置，并且分层代理确定何时将对象从缓存刷新到后备存储层。因此，缓存层和后备存储层对Ceph客户端是完全透明的。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-2982c5ed3031cac4f9e40545139e51fdb0b33897.png)

缓存分层代理自动处理缓存层和后备存储层之间的数据迁移。但是，管理员可以通过设置来配置此迁移的方式`cache-mode`。主要有两种方案：

* **回写**模式：当管理员使用`writeback`模式配置层时，Ceph客户端将数据写入缓存层并从缓存层接收ACK。随着时间的流逝，写入缓存层的数据将迁移到存储层，并从缓存层中清除。从概念上讲，缓存层覆盖在后备存储层的“前面”。当Ceph客户端需要驻留在存储层中的数据时，缓存分层代理在读取时将数据迁移到缓存层，然后将其发送到Ceph客户端。此后，Ceph客户端可以使用缓存层执行I / O，直到数据变为非活动状态为止。这对于可变数据（例如，照片/视频编辑，交易数据等）是理想的。
* **readproxy**模式：此模式将使用高速缓存层中已经存在的任何对象，但是如果高速缓存中不存在对象，则请求将被代理到基本层。这对于从`writeback`模式过渡到禁用的高速缓存很有用，因为它允许在清空高速缓存时工作负载能够正常运行，而无需向高速缓存添加任何新对象。

其他缓存模式包括：

* **readonly**仅在读取操作时将对象提升到缓存；写操作将转发到基本层。此模式适用于不需要一致性的只读工作负载，该负载必须由存储系统强制执行。（**警告**：当在基础层中更新对象时，Ceph **不会**尝试将这些更新同步到缓存中的相应对象。由于该模式被认为是实验性的，因此`--yes-i-really-mean-it`必须传递一个 选项来启用它。）
* **none**用于完全禁用缓存。

### 小心一点

缓存分层将_降低_大多数工作负载的性能。用户在使用此功能之前应格外小心。

* _取决于工作负载_：缓存是否会提高性能在很大程度上取决于工作负载。因为将对象移入或移出缓存会产生一定的成本，所以只有在数据集的访问模式存在_较大的偏斜_（大多数请求都接触少量对象）时，此方法才有效。高速缓存池应足够大，以捕获您的工作负载的工作集，以免发生抖动。
* _难以进行基准测试_：大多数用户用来衡量性能的基准测试将显示缓存分层带来的糟糕性能，部分原因是其中很少有人将请求偏向一小组对象，因此缓存“预热”可能需要很长时间， ”，因为预热费用可能很高。
* _通常较慢_：对于不支持缓存分层的工作负载，性能通常会比未启用缓存分层的普通RADOS池慢。
* _librados对象枚举_：在存在案例的情况下，librados级对象枚举API并不意味着是一致的。如果您的应用程序直接使用librados并依赖于对象枚举，则缓存分层可能无法按预期工作。（对于RGW，RBD或CephFS，这不是问题。）
* _复杂性_：启用缓存分层意味着RADOS集群中正在使用许多其他机制和复杂性。这增加了您在系统中遇到其他用户尚未遇到的错误的可能性，并使您的部署面临更高的风险。

#### 已知的良好工作量

* _RGW时滞性_：如果RGW的工作量使得几乎所有读取操作都针对最近写入的对象，那么一个简单的缓存分层配置可以在可配置的时间段之后将最近写入的对象从缓存降级到基本层。

#### 已知的不良工作量

_已知_以下配置在缓存分层中_无法正常工作_。

* _具有复制缓存和基于擦除编码的RBD_：这是一个常见请求，但通常效果不佳。即使偏斜的工作负载仍然会向冷对象发送一些小写操作，并且由于擦除编码池尚不支持小写操作，因此必须将整个（通常为4 MB）对象迁移到缓存中，才能满足小写操作（通常为4 MB） KB）写入。只有少数用户成功部署了此配置，并且仅对他们有效，因为他们的数据非常冷（备份），并且对性能不敏感。
* _具有复制的缓存和基础的_ RBD：具有复制的基础层的RBD的性能要优于对基础进行擦除编码的情况，但是它仍然高度依赖于工作负载中的偏差量，并且很难验证。用户将需要对他们的工作量有充分的了解，并需要仔细调整缓存分层参数。

### 设置池

要设置缓存分层，您必须有两个池。一个将充当后备存储，另一个将充当缓存。

#### 设置后备存储池

设置后备存储池通常涉及以下两种情况之一：

* **标准存储**：在这种情况下，池将对象的多个副本存储在Ceph存储集群中。
* **擦除编码：**在这种情况下，池使用擦除编码可以以较小的性能折衷来更有效地存储数据。

在标准存储方案中，您可以设置CRUSH规则来建立故障域（例如osd，主机，机箱，机架，行等）。当规则中的所有存储驱动器具有相同的大小，速度（RPM和吞吐量）和类型相同时，Ceph OSD守护程序将以最佳状态运行。有关 创建规则的详细信息，请参见[CRUSH Maps](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map)。创建规则后，请创建一个后备存储池。

在擦除编码方案中，池创建参数将自动生成适当的规则。有关详细信息，请参见[创建池](https://docs.ceph.com/docs/nautilus/rados/operations/pools#create-a-pool)。

在后续示例中，我们将后备存储池称为`cold-storage`。

#### 建立一个缓存池

设置缓存池的过程与标准存储方案相同，但有一个区别：缓存层的驱动器通常是高性能驱动器，它们驻留在自己的服务器中并具有自己的CRUSH规则。设置此类规则时，应考虑具有高性能驱动器的主机，而忽略那些没有高性能的主机。有关详细信息，请参见 [将不同的池放置在不同的OSD](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map-edits/#placing-different-pools-on-different-osds)上。

在后续示例中，我们将缓存池称为`hot-storage`，将后备池称为`cold-storage`。

有关缓存层的配置和默认值，请参阅“ [池-设置池值”](https://docs.ceph.com/docs/nautilus/rados/operations/pools#set-pool-values)。

### 创建一个缓存层

设置缓存层涉及将后备存储池与缓存池相关联

```text
ceph osd tier add {storagepool} {cachepool}
```

例如

```text
ceph osd tier add cold-storage hot-storage
```

要设置缓存模式，请执行以下操作：

```text
ceph osd tier cache-mode {cachepool} {cache-mode}
```

例如：

```text
ceph osd tier cache-mode hot-storage writeback
```

缓存层覆盖了后备存储层，因此它们需要执行另一步骤：必须将所有客户端流量从存储池定向到缓存池。要将客户端流量直接定向到缓存池，请执行以下操作：

```text
ceph osd tier set-overlay {storagepool} {cachepool}
```

例如：

```text
ceph osd tier set-overlay cold-storage hot-storage
```

### 配置高速缓存层级

缓存层具有几个配置选项。您可以使用以下用法来设置缓存层配置选项：

```text
ceph osd pool set {cachepool} {key} {value}
```

有关详细信息，请参见[池-设置池值](https://docs.ceph.com/docs/nautilus/rados/operations/pools#set-pool-values)。

#### 目标尺寸和类型

头孢生产的高速缓存层级使用[布隆过滤器](https://en.wikipedia.org/wiki/Bloom_filter)为`hit_set_type`：

```text
ceph osd pool set {cachepool} hit_set_type bloom
```

例如：

```text
ceph osd pool set hot-storage hit_set_type bloom
```

该`hit_set_count`和`hit_set_period`确定有多少这样的HitSets存储，每个HitSet应该多少时间覆盖。

```text
ceph osd pool set {cachepool} hit_set_count 12
ceph osd pool set {cachepool} hit_set_period 14400
ceph osd pool set {cachepool} target_max_bytes 1000000000000
```

注意 

较大的值会`hit_set_count`导致`ceph-osd`进程消耗更多的RAM 。

随时间推移对访问进行绑定可以使Ceph能够确定Ceph客户端是否在一个时间段（“年龄”与“温度”）中访问对象至少一次或多次。

该`min_read_recency_for_promote`定义多少HitSets处理读操作时，检查对象的存在。检查结果用于决定是否异步升级对象。其值应在0到之间`hit_set_count`。如果将其设置为0，则始终提升对象。如果设置为1，则检查当前的HitSet。如果此对象位于当前的HitSet中，则将其提升。否则不行。对于其他值，将检查存档命中集的确切数量。如果在任何最新的`min_read_recency_for_promote`HitSet中找到该对象，则该对象将被提升。

可以为写操作设置一个类似的参数 `min_write_recency_for_promote`。

```text
ceph osd pool set {cachepool} min_read_recency_for_promote 2
ceph osd pool set {cachepool} min_write_recency_for_promote 2
```

注意 

周期越长，`min_read_recency_for_promote`and 守护程序消耗的时间就越高 。特别是，当代理程序处于活动状态以刷新或逐出缓存对象时，所有HitSet都将加载到RAM中。```min_write_recency_for_promote``values, the more RAM the ``ceph-osdhit_set_count```

#### 高速缓存大小调整

缓存分层代理执行两个主要功能：

* **刷新：**代理识别已修改（或脏）的对象，并将其转发到存储池以进行长期存储。
* **驱逐：**该代理标识尚未修改（或清除）的对象，并从缓存中**逐出**最近使用最少的对象。

**绝对尺寸**

缓存分层代理可以根据字节总数或对象总数刷新或逐出对象。要指定最大字节数，请执行以下操作：

```text
ceph osd pool set {cachepool} target_max_bytes {#bytes}
```

例如，要刷新或逐出1 TB，请执行以下操作：

```text
ceph osd pool set hot-storage target_max_bytes 1099511627776
```

要指定最大对象数，请执行以下操作：

```text
ceph osd pool set {cachepool} target_max_objects {#objects}
```

例如，要刷新或逐出1M对象，请执行以下操作：

```text
ceph osd pool set hot-storage target_max_objects 1000000
```

注意 

Ceph无法自动确定缓存池的大小，因此此处需要对绝对大小进行配置，否则刷新/逐出将不起作用。如果您同时指定两个限制，则在触发任一阈值时，缓存分层代理将开始刷新或逐出。

注意 

所有客户端请求仅在`target_max_bytes`或 `target_max_objects`达到时被阻止

**相对大小**

缓存分层代理可以刷新或逐出相对于缓存池大小的对象（由`target_max_bytes`/ `target_max_objects`在 [Absolute sizing中](https://docs.ceph.com/docs/nautilus/rados/operations/cache-tiering/#absolute-sizing)指定）。当缓存池由一定百分比的已修改（或脏）对象组成时，缓存分层代理会将其刷新到存储池。要设置`cache_target_dirty_ratio`，请执行以下操作：

```text
ceph osd pool set {cachepool} cache_target_dirty_ratio {0.0..1.0}
```

例如，将值设置为`0.4`会在达到缓存池容量的40％时开始刷新已修改的（脏）对象：

```text
ceph osd pool set hot-storage cache_target_dirty_ratio 0.4
```

当脏物达到其容量的一定百分比时，请以较高的速度冲洗脏物。设置`cache_target_dirty_high_ratio`：

```text
ceph osd pool set {cachepool} cache_target_dirty_high_ratio {0.0..1.0}
```

例如，将值设置为`0.6`会在脏对象达到缓存池容量的60％时开始主动清除它们。显然，我们最好在dirty\_ratio和full\_ratio之间设置值：

```text
ceph osd pool set hot-storage cache_target_dirty_high_ratio 0.6
```

当缓存池达到其容量的一定百分比时，缓存分层代理将驱逐对象以保持可用容量。要设置 `cache_target_full_ratio`，请执行以下操作：

```text
ceph osd pool set {cachepool} cache_target_full_ratio {0.0..1.0}
```

例如，将值设置为`0.8`会在未修改（干净）的对象达到缓存池容量的80％时开始刷新它们：

```text
ceph osd pool set hot-storage cache_target_full_ratio 0.8
```

#### 缓存年龄

您可以在缓存分层代理将最近修改的（或脏的）对象刷新到后备存储池之前指定对象的最短使用期限：

```text
ceph osd pool set {cachepool} cache_min_flush_age {#seconds}
```

例如，要在10分钟后刷新修改（或脏）的对象，请执行以下操作：

```text
ceph osd pool set hot-storage cache_min_flush_age 600
```

您可以指定对象的最短使用期限，然后再将其从缓存层中逐出：

```text
ceph osd pool {cache-tier} cache_min_evict_age {#seconds}
```

例如，要在30分钟后驱逐对象，请执行以下操作：

```text
ceph osd pool set hot-storage cache_min_evict_age 1800
```

### 删除高速缓存层级

删除缓存层的不同取决于它是写回缓存还是只读缓存。

#### 删除只读缓存

由于只读缓存未修改数据，因此可以禁用和删除它，而不会丢失对缓存中对象的任何最新更改。

1. 将缓存模式更改`none`为禁用它。

   ```text
   ceph osd tier cache-mode {cachepool} none
   ```

   例如：

   ```text
   ceph osd tier cache-mode hot-storage none
   ```

2. 从备用池中删除缓存池。

   ```text
   ceph osd tier remove {storagepool} {cachepool}
   ```

   例如：

   ```text
   ceph osd tier remove cold-storage hot-storage
   ```

#### 卸下回写式高速缓存

由于回写缓存可能已修改了数据，因此在禁用和删除缓存之前，必须采取步骤以确保不会丢失对缓存中对象的任何最新更改。

1. 将缓存模式更改为，`proxy`以便新对象和修改后的对象将刷新到后备存储池。

   ```text
   ceph osd tier cache-mode {cachepool} proxy
   ```

   例如：

   ```text
   ceph osd tier cache-mode hot-storage proxy
   ```

2. 确保已刷新缓存池。这可能需要几分钟的时间：

   ```text
   rados -p {cachepool} ls
   ```

   如果缓存池中仍然有对象，则可以手动刷新它们。例如：

   ```text
   rados -p {cachepool} cache-flush-evict-all
   ```

3. 删除覆盖，以便客户端不会将流量定向到缓存。

   ```text
   ceph osd tier remove-overlay {storagetier}
   ```

   例如：

   ```text
   ceph osd tier remove-overlay cold-storage
   ```

4. 最后，从后备存储池中删除缓存层池。

   ```text
   ceph osd tier remove {storagepool} {cachepool}
   ```

   例如：

   ```text
   ceph osd tier remove cold-storage hot-storage
   ```

