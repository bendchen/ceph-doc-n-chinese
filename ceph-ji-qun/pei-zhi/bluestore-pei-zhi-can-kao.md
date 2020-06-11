# BLUESTORE配置参考

## BLUESTORE配置参考

### 设备

BlueStore管理一个，两个或（在某些情况下）三个存储设备。

在最简单的情况下，BlueStore使用单个（主）存储设备。存储设备通常作为一个整体使用，占据了直接由BlueStore管理的完整设备。通常，该_主设备_由`block`数据目录中的符号链接标识。

数据目录是一个`tmpfs`装载文件（在引导时或`ceph-volume`激活时），该装载文件中包含所有常见的OSD文件，这些文件包含有关OSD的信息，例如：其标识符，其所属的集群以及其专用密钥环。

还可以在另外两个设备上部署BlueStore：

* 一个_WAL设备_（标识为`block.wal`数据目录），可用于BlueStore的内部刊物或预写日志。仅当WAL设备比主设备快时（例如，当它位于SSD上并且主设备是HDD时），才使用WAL设备。
* 甲_DB设备_（标识为`block.db`在数据目录）可被用于存储BlueStore的内部元数据。BlueStore（或更确切地说，是嵌入式RocksDB）将在数据库设备上放置尽可能多的元数据以提高性能。如果数据库设备已满，则元数据将溢出到主设备上（否则会出现在原设备上）。同样，仅当配置数据库设备的速度比主设备快时，它才有帮助。

如果只有少量快速存储可用（例如，小于1 GB），我们建议将其用作WAL设备。如果还有更多，则配置数据库设备更有意义。BlueStore日志将始终放在可用的最快设备上，因此使用DB设备将提供与WAL设备相同的好处，同时_还_允许将其他元数据存储在该设备中（如果合适的话）。

单设备BlueStore OSD可以配备：

```text
ceph-volume lvm prepare --bluestore --data <device>
```

要指定WAL设备和/或DB设备，

```text
ceph-volume lvm prepare --bluestore --data <device> --block.wal <wal-device> --block.db <db-device>
```

注意 

–data可以是使用vg / lv表示法的逻辑卷。其他设备可以是现有逻辑卷或GPT分区

#### 置备策略

尽管部署Bluestore OSD的方法有多种（不同于Filestore的1），这里有两个常见的用例应有助于阐明初始部署策略：

**仅阻止（数据）**

如果所有设备都是同一类型，例如所有设备都是旋转驱动器，并且没有快速的设备可将它们组合在一起，那么仅使用块部署而不尝试分离`block.db`或便是有意义的`block.wal`。单个设备的 [lvm](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/#ceph-volume-lvm)调用`/dev/sda`如下所示：

```text
ceph-volume lvm create --bluestore --data /dev/sda
```

如果已经为每个设备创建了逻辑卷（使用设备的100％使用1个LV），那么对名为lv的lv 的[lvm](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/#ceph-volume-lvm)调用 `ceph-vg/block-lv`将类似于：

```text
ceph-volume lvm create --bluestore --data ceph-vg/block-lv
```

**阻止和BLOCK.DB** 

如果混合使用快速和慢速设备（旋转和固态），则建议放置`block.db`在速度较快的设备上，而`block` （数据）存放在速度较慢的设备（旋转驱动器）上。调整大小`block.db`应尽可能大，以免造成性能损失。该 `ceph-volume`工具当前无法自动创建这些卷，因此需要手动创建卷组和逻辑卷。

对于下面的示例，假设有4个旋转驱动器（sda，sdb，sdc和sdd）和1个固态驱动器（sdx）。首先创建卷组：

```text
$ vgcreate ceph-block-0 /dev/sda
$ vgcreate ceph-block-1 /dev/sdb
$ vgcreate ceph-block-2 /dev/sdc
$ vgcreate ceph-block-3 /dev/sdd
```

现在为创建逻辑卷`block`：

```text
$ lvcreate -l 100%FREE -n block-0 ceph-block-0
$ lvcreate -l 100%FREE -n block-1 ceph-block-1
$ lvcreate -l 100%FREE -n block-2 ceph-block-2
$ lvcreate -l 100%FREE -n block-3 ceph-block-3
```

我们将为四个慢速旋转的设备创建4个OSD，因此假设其中有200GB的SSD，`/dev/sdx`我们将创建4个逻辑卷，每个逻辑卷为50GB：

```text
$ vgcreate ceph-db-0 /dev/sdx
$ lvcreate -L 50GB -n db-0 ceph-db-0
$ lvcreate -L 50GB -n db-1 ceph-db-0
$ lvcreate -L 50GB -n db-2 ceph-db-0
$ lvcreate -L 50GB -n db-3 ceph-db-0
```

最后，使用以下命令创建4个OSD `ceph-volume`：

```text
$ ceph-volume lvm create --bluestore --data ceph-block-0/block-0 --block.db ceph-db-0/db-0
$ ceph-volume lvm create --bluestore --data ceph-block-1/block-1 --block.db ceph-db-0/db-1
$ ceph-volume lvm create --bluestore --data ceph-block-2/block-2 --block.db ceph-db-0/db-2
$ ceph-volume lvm create --bluestore --data ceph-block-3/block-3 --block.db ceph-db-0/db-3
```

这些操作最终将创建4个OSD，`block`在速度较慢的旋转驱动器上，以及每个50GB逻辑卷（来自固态驱动器）。

### 尺码

使用[混合旋转和固态驱动器设置时](https://docs.ceph.com/docs/nautilus/rados/configuration/bluestore-config-ref/#bluestore-mixed-device-config)，`block.db`为Bluestore 制作足够大的逻辑卷很重要 。通常，`block.db`应具有 _尽可能大的_逻辑卷。

建议`block.db`大小不小于的4％ `block`。例如，如果`block`大小为1TB，则`block.db` 不应小于40GB。

如果_未同时_使用快速和慢速设备，则不需要为`block.db`（或`block.wal`）创建单独的逻辑卷。Bluestore将在的空间内自动管理这些文件`block`。

### 自动缓存大小调整

将tc\_malloc配置为内存分配器并`bluestore_cache_autotune` 启用设置后，可以将Bluestore配置为自动调整其缓存大小。当前默认情况下启用此选项。Bluestore将尝试通过`osd_memory_target`配置选项将OSD堆内存使用量保持在指定的目标大小以下。这是一种尽力而为的算法，缓存的收缩量不会小于所指定的数量 `osd_memory_cache_min`。将根据优先级层次选择缓存比率。如果没有优先级信息，则将 `bluestore_cache_meta_ratio`和`bluestore_cache_kv_ratio`选项用作后备。

`bluestore_cache_autotune`描述

在遵守最小值的同时，自动调整分配给不同的Bluestore缓存的比率。类型

布尔型需要

是默认

`True`

`osd_memory_target`描述

当tcmalloc可用并且启用了高速缓存自动调整功能时，请尝试将这么多字节映射到内存中。注意：这可能与进程的RSS内存使用情况不完全匹配。虽然该进程映射的堆内存总量通常应保持在接近该目标的水平，但不能保证内核会真正回收未映射的内存。在最初的开发过程中，发现某些内核导致OSD的RSS内存超出映射内存达20％。但是，假设存在大量内存压力时，内核通常会更积极地回收未映射的内存。你的旅费可能会改变。类型

无符号整数需要

是默认

`4294967296`

`bluestore_cache_autotune_chunk_size`描述

启用缓存自动调整后，分配给缓存的块大小（以字节为单位）。当自动调谐器将内存分配给不同的缓存时，它将分块分配内存。这样做是为了避免在堆大小或自动调整的缓存比率有较小波动时驱逐。类型

无符号整数需要

没有默认

`33554432`

`bluestore_cache_autotune_interval`描述

启用缓存自动调整后，两次重新平衡之间要等待的秒数。此设置更改差异缓存比率重新计算的速度。注意：将间隔设置得太小会导致CPU使用率升高和性能降低。类型

浮动需要

没有默认

`5`

`osd_memory_base`描述

启用tcmalloc和缓存自动调整后，估计OSD需要的最小内存量（以字节为单位）。这用于帮助自动调谐器估计高速缓存的预期总内存消耗。类型

未签名的整数需要

没有默认

`805306368`

`osd_memory_expected_fragmentation`描述

启用tcmalloc和高速缓存自动调整后，请估计内存碎片的百分比。这用于帮助自动调谐器估计高速缓存的预期总内存消耗。类型

浮动需要

没有默认

`0.15`

`osd_memory_cache_min`描述

启用tcmalloc和高速缓存自动调整后，请设置用于高速缓存的最小内存量。注意：将此值设置得太低会导致严重的高速缓存抖动。类型

无符号整数需要

没有默认

`134217728`

`osd_memory_cache_resize_interval`描述

启用tcmalloc和高速缓存自动调整后，请在调整高速缓存大小之间等待这几秒钟。此设置更改可用于bluestore进行缓存的内存总量。注意：将时间间隔设置得太小会导致内存分配器崩溃并降低性能。类型

浮动需要

没有默认

`1`

### 手动缓存大小调整

每个OSD为BlueStore的缓存占用的内存量由`bluestore_cache_size`配置选项确定。如果未设置该配置选项（即保持为0），则将使用不同的默认值，具体取决于将HDD或SSD用于主要设备（由 `bluestore_cache_size_ssd`和`bluestore_cache_size_hdd`配置选项设置）。

BlueStore和Ceph OSD的其余部分目前在遵守预算的内存方面尽其所能。请注意，除了配置的缓存大小之外，OSD本身还会消耗内存，并且由于内存碎片和其他分配器开销而通常会产生一些开销。

可以以几种不同的方式使用配置的缓存内存预算：

* 键/值元数据（即RocksDB的内部缓存）
* BlueStore元数据
* BlueStore数据（即最近读取或写入的对象数据）

缓存的使用情况由以下选项控制： `bluestore_cache_meta_ratio`和`bluestore_cache_kv_ratio`。专用于数据的缓存部分由有效的bluestore缓存大小（取决于 `bluestore_cache_size[_ssd|_hdd]`设置和主要设备的设备类别）以及元和kv比率控制。数据分数可以通过 `<effective_cache_size> * (1 - bluestore_cache_meta_ratio - bluestore_cache_kv_ratio)`

`bluestore_cache_size`描述

BlueStore将用于其缓存的内存量。如果为零，`bluestore_cache_size_hdd`或`bluestore_cache_size_ssd`将被用来代替。类型

无符号整数需要

是默认

`0`

`bluestore_cache_size_hdd`描述

当有HDD支持时，BlueStore将用于其缓存的默认内存量。类型

无符号整数需要

是默认

`1 * 1024 * 1024 * 1024` （1 GB）

`bluestore_cache_size_ssd`描述

当有SSD备份时，BlueStore的默认内存量将用于其缓存。类型

无符号整数需要

是默认

`3 * 1024 * 1024 * 1024` （3 GB）

`bluestore_cache_meta_ratio`描述

缓存与元数据的比率。类型

浮点需要

是默认

`.4`

`bluestore_cache_kv_ratio`描述

缓存与键/值数据的比率（rocksdb）。类型

浮点需要

是默认

`.4`

`bluestore_cache_kv_max`描述

专用于键/值数据（rocksdb）的最大高速缓存量。类型

无符号整数需要

是默认

`512 * 1024*1024` （512 MB）

### 校验

BlueStore校验和将所有元数据和数据写入磁盘。元数据校验和由RocksDB处理，并使用crc32c。数据校验和由BlueStore完成，可以使用crc32c， xxhash32或xxhash64。默认值为crc32c，应适合大多数用途。

完整的数据校验和确实会增加BlueStore必须存储和管理的元数据量。在可能的情况下（例如，当客户端提示顺序写入和读取数据时），BlueStore将对较大的块进行校验和，但是在许多情况下，它必须为每4 KB的数据块存储一个校验和值（通常为4个字节）。

通过将校验和截断为两个或一个字节，可以使用较小的校验和值，从而减少元数据开销。需要权衡的是，使用较小的校验和，将不会检测到随机错误的概率较高，从32位（4字节）校验和的四十亿分之一到16位的65536的校验（ 2位）校验和或256分之一的8位（1字节）校验和。通过选择crc32c\_16或 crc32c\_8作为校验和算法，可以使用较小的校验和值。

该_校验算法_可以通过每个池设置 `csum_type`属性或全局配置选项。例如，

```text
ceph osd pool set <pool-name> csum_type <algorithm>
```

`bluestore_csum_type`描述

要使用的默认校验和算法。类型

串需要

是有效设定

`none`，`crc32c`，`crc32c_16`，`crc32c_8`，`xxhash32`，`xxhash64`默认

`crc32c`

### 在线压缩

BlueStore支持使用内嵌压缩活泼，ZLIB，或 LZ4。请注意，lz4压缩插件未在官方发行版中分发。

BlueStore中的数据是否被压缩取决于_压缩模式_和与写操作相关的任何提示的组合。这些模式是：

* **none**：从不压缩数据。
* **被动的**：除非写操作具有_可压缩的_提示集，否则不要压缩数据 。
* **积极的**：压缩数据，除非写操作具有 _不可压缩的_提示集。
* **force**：无论如何都尝试压缩数据。

有关_可压缩_和_不可压缩_ IO提示的更多信息，请参见[`rados_set_alloc_hint()`](https://docs.ceph.com/docs/nautilus/rados/api/librados/#c.rados_set_alloc_hint)。

请注意，无论采用哪种模式，如果数据块的大小没有充分减小，则将不会使用它，并且将存储原始（未压缩）数据。例如，如果将设置为，则压缩数据必须是原始数据大小的70％（或更小）。`bluestore compression required ratio.7`

在_压缩模式_，_压缩算法_，_压缩所需比率_，_分钟斑点大小_，和_最大BLOB大小_既可以经由每个缓冲池属性或全局配置选项设置。可以通过以下方式设置池属性：

```text
ceph osd pool set <pool-name> compression_algorithm <algorithm>
ceph osd pool set <pool-name> compression_mode <mode>
ceph osd pool set <pool-name> compression_required_ratio <ratio>
ceph osd pool set <pool-name> compression_min_blob_size <size>
ceph osd pool set <pool-name> compression_max_blob_size <size>
```

`bluestore compression algorithm`描述

如果`compression_algorithm`未设置每池属性，则使用的默认压缩器（如果有） 。请注意， 由于压缩少量数据时会占用大量CPU 资源，因此_不_建议将zstd用于bluestore。类型

串需要

没有有效设定

`lz4`，`snappy`，`zlib`，`zstd`默认

`snappy`

`bluestore compression mode`描述

如果`compression_mode`未设置每池属性，则使用压缩的默认策略 。`none`表示永远不要使用压缩。`passive`表示当数据可压缩时使用 压缩。 表示使用压缩，除非客户端提示数据不可压缩。 意味着即使客户端暗示数据不可压缩，也要在所有情况下都使用压缩。[`clients hint`](https://docs.ceph.com/docs/nautilus/rados/api/librados/#c.rados_set_alloc_hint)`aggressiveforce`类型

串需要

没有有效设定

`none`，`passive`，`aggressive`，`force`默认

`none`

`bluestore compression required ratio`描述

压缩后的数据块大小与原始大小之比必须至少如此之小，以便存储压缩版本。类型

浮点需要

没有默认

.875

`bluestore compression min blob size`描述

小于此大小的块永远不会被压缩。每池属性将`compression_min_blob_size`覆盖此设置。类型

无符号整数需要

没有默认

0

`bluestore compression min blob size hdd`描述

旋转媒体的默认值。`bluestore compression min blob size`类型

无符号整数需要

没有默认

128K

`bluestore compression min blob size ssd`描述

非旋转（固态）媒体的默认值。`bluestore compression min blob size`类型

无符号整数需要

没有默认

8K

`bluestore compression max blob size`描述

大于此的块将在压缩之前分解为较小的斑点大小 。每池属性将覆盖此设置。`bluestore compression max blob sizecompression_max_blob_size`类型

无符号整数需要

没有默认

0

`bluestore compression max blob size hdd`描述

旋转媒体的默认值。`bluestore compression max blob size`类型

无符号整数需要

没有默认

512K

`bluestore compression max blob size ssd`描述

非旋转（固态）媒体的默认值。`bluestore compression max blob size`类型

无符号整数需要

没有默认

64K

### SPDK用法

如果要将SPDK驱动程序用于NVME SSD，则需要准备系统。有关更多详细信息，请参考[SPDK文档](http://www.spdk.io/doc/getting_started.html#getting_started_examples)。

SPDK提供了一个脚本来自动配置设备。用户可以以root用户身份运行脚本：

```text
$ sudo src/spdk/scripts/setup.sh
```

然后，您需要在此处使用前缀“ spdk：”指定NVMe设备的设备选择器 `bluestore_block_path`。

例如，用户可以通过以下方式找到英特尔PCIe SSD的设备选择器：

```text
$ lspci -mm -n -D -d 8086:0953
```

设备选择器的格式始终为`DDDD:BB:DD.FF`或`DDDD.BB.DD.FF`。

然后设置：

```text
bluestore block path = spdk:0000:01:00.0
```

在 上面`0000:01:00.0`的`lspci`命令输出中找到的设备选择器在哪里。

如果要每个节点运行多个SPDK实例，则必须指定每个实例将使用的dpdk内存大小（以MB为单位），以确保每个实例使用自己的dpdk内存

在大多数情况下，我们只需要一台设备即可用作数据，数据库，数据库的目的。我们需要确保进行以下配置，以确保所有SPIO下均发布了IO：

```text
bluestore_block_db_path = ""
bluestore_block_db_size = 0
bluestore_block_wal_path = ""
bluestore_block_wal_size = 0
```

否则，当前实现会将符号文件设置到内核文件系统位置，并使用内核驱动程序发布DB / WAL IO。

