# 删除码配置文件

## 删除码配置文件

擦除代码由**配置文件**定义，并在创建擦除代码池和关联的CRUSH规则时使用。

该**缺省**擦除码简档（其当Ceph的群集初始化创建）提供冗余的相同水平的两个副本，但是需要25％的磁盘空间更少。它被描述为**k = 2**和**m = 1**的配置文件，这意味着信息分布在三个OSD（k + m == 3）上，其中之一可能会丢失。

为了在不增加原始存储需求的情况下提高冗余度，可以创建一个新的配置文件。例如，通过将对象分布在十四个（k + m = 14）OSD上，具有**k = 10**和 **m = 4**的配置文件可以承受四个（**m = 4**）OSD 的损失。首先将对象分为 **10个**块（如果对象为10MB，则每个块为1MB），并 计算**4个**编码块以进行恢复（每个编码块与数据块的大小相同，即1MB）。原始空间开销仅为40％，即使四个OSD同时中断，该对象也不会丢失。

* [Jerasure擦除代码插件](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-jerasure/)
* [ISA擦除代码插件](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-isa/)
* [可本地修复的擦除代码插件](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-lrc/)
* [SHEC擦除代码插件](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-shec/)
* [CLAY代码插件](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-clay/)

### OSD擦除代码配置文件集

要创建新的擦除代码配置文件：

```text
ceph osd erasure-code-profile set {name} \
     [{directory=directory}] \
     [{plugin=plugin}] \
     [{stripe_unit=stripe_unit}] \
     [{key=value} ...] \
     [--force]
```

哪里：

`{directory=directory}`描述

设置从中加载擦除代码插件的**目录**名称。类型

串需要

没有。默认

/ usr / lib / ceph / erasure-code

`{plugin=plugin}`描述

使用擦除代码**插件**来计算编码块并恢复丢失的块。有关更多信息，请参见[可用插件列表](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-profile/#list-of-available-plugins)。类型

串需要

没有。默认

疯狂

`{stripe_unit=stripe_unit}`描述

每个条带中的数据块中的数据量。例如，具有2个数据块且stripe\_unit = 4K的配置文件会将范围0-4K放入块0，将4K-8K放入块1，然后将8K-12K再次放入块0。为了获得最佳性能，这应该是4K的倍数。`osd_pool_erasure_code_stripe_unit`创建池时，默认值是从monitor config选项获取的 。使用此配置文件的池的stripe\_width将是数据块的数量乘以该stripe\_unit。类型

串需要

没有。

`{key=value}`描述

其余键/值对的语义由擦除代码插件定义。类型

串需要

没有。

`--force`描述

用相同的名称覆盖现有配置文件，并允许设置非4K对齐的stripe\_unit。类型

串需要

没有。

### OSD擦除代码配置文件

删除删除码配置文件：

```text
ceph osd erasure-code-profile rm {name}
```

如果池引用了配置文件，则删除将失败。

### OSD擦除代码配置文件获取

要显示擦除代码配置文件：

```text
ceph osd erasure-code-profile get {name}
```

### OSD擦除代码配置文件

列出所有擦除代码配置文件的名称：

```text
ceph osd erasure-code-profile ls
```

