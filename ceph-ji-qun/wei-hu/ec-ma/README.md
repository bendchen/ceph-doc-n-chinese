# EC码

## 删除码

Ceph池与一种类型相关联，以维持OSD（即磁盘，因为大多数情况下每个磁盘上只有一个OSD）的丢失。[创建池](https://docs.ceph.com/docs/nautilus/rados/operations/pools)时的默认选择是_复制_，这意味着每个对象都复制到多个磁盘上。该[擦除码](https://en.wikipedia.org/wiki/Erasure_code)池类型可以用来代替以节省空间。

### 创建一个样本擦除编码池

最简单的擦除编码池等效于[RAID5，](https://en.wikipedia.org/wiki/Standard_RAID_levels#RAID_5)并且至少需要三个主机：

```text
$ ceph osd pool create ecpool 12 12 erasure
pool 'ecpool' created
$ echo ABCDEFGHI | rados --pool ecpool put NYAN -
$ rados --pool ecpool get NYAN -
ABCDEFGHI
```

注意 

_池中_的12个_create_代表 [展示位置组的数量](https://docs.ceph.com/docs/nautilus/rados/operations/pools)。

### 删除码配置文件

默认的擦除代码配置文件会丢失单个OSD。它等效于大小为2的复制池，但需要1.5TB而不是2TB的数据来存储1TB的数据。默认配置文件可以显示为：

```text
$ ceph osd erasure-code-profile get default
k=2
m=1
plugin=jerasure
crush-failure-domain=host
technique=reed_sol_van
```

选择正确的配置文件很重要，因为在创建池后无法对其进行修改：需要创建一个具有不同配置文件的新池，并且将先前池中的所有对象都移到新的池中。

概要文件的最重要参数是_K_，_M_和 _粉碎失败域，_因为它们定义了存储开销和数据持久性。例如，如果所需的体系结构必须承受两个机架的丢失，而存储开销为开销的67％，则可以定义以下配置文件：

```text
$ ceph osd erasure-code-profile set myprofile \
   k=3 \
   m=2 \
   crush-failure-domain=rack
$ ceph osd pool create ecpool 12 12 erasure myprofile
$ echo ABCDEFGHI | rados --pool ecpool put NYAN -
$ rados --pool ecpool get NYAN -
ABCDEFGHI
```

该_NYAN_对象将在三个（被划分_K = 3_）和两个附加 _的块_将被创建（_M = 2_）。_M_的值定义了在不丢失任何数据的情况下可以同时丢失多少个OSD。所述 _压碎故障域=架_将创建一个CRUSH规则，以确保没有两个_组块_被存储在同一个机架。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-96fe8c3c73e5e54cf27fa8a4d64ed08d17679ba3.png)

可以在[擦除代码配置](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-profile)文件文档中找到更多信息。

### 带覆盖的擦除编码

默认情况下，擦除编码池仅适用于执行完整对象写入和附加的RGW之类的用途。

由于是发光的，因此可以通过每个池设置启用对擦除编码池的部分写入。这使RBD和CephFS将其数据存储在擦除编码池中：

```text
ceph osd pool set ec_pool allow_ec_overwrites true
```

这只能在驻留在bluestore OSD上的池上启用，因为bluestore的校验和用于检测深度擦除期间的位腐或其他损坏。除了不安全之外，将文件存储区与ec覆盖一起使用还比bluestore产生较低的性能。

擦除编码池不支持omap，因此要将它们与RBD和CephFS一起使用，必须指示它们将数据存储在ec池中，并将元数据存储在复制池中。对于RBD，这意味着`--data-pool`在图像创建过程中使用擦除编码池：

```text
rbd create --size 1G --data-pool ec_pool replicated_pool/image_name
```

对于CephFS，可以在文件系统创建过程中或通过[文件布局](https://docs.ceph.com/docs/nautilus/cephfs/file-layouts)将擦除编码池设置为默认数据池。

### 擦除编码池和缓存分层

擦除编码池比复制池需要更多的资源，并且缺少某些功能，例如omap。为了克服这些限制，可以 在擦除编码池之前设置一个[缓存层](https://docs.ceph.com/docs/nautilus/rados/operations/cache-tiering)。

例如，如果池_热存储_由快速存储组成：

```text
$ ceph osd tier add ecpool hot-storage
$ ceph osd tier cache-mode hot-storage writeback
$ ceph osd tier set-overlay ecpool hot-storage
```

将放置_热存储_池的层次_ecpool_在_写回_ 模式，让每一个写和读的_ecpool_使用实际的_热存储_从它的灵活性和速度，并从中受益。

可以在[缓存分层](https://docs.ceph.com/docs/nautilus/rados/operations/cache-tiering)文档中找到更多信息。

### 词汇表

_大块_

调用编码函数时，它将返回相同大小的块。可以串联以重建原始对象的数据块和可以用于重建丢失的块的编码块。_ķ_

数据_块_的数量，即原始对象被分割成的_块_的数量。例如，如果_K_ = 2，则将10KB的对象分为每个5KB的_K个_对象。_中号_

编码_块_的数量，即 由编码功能计算出的其他_块_的数量。如果有2个编码_块_，则意味着2个OSD可以输出而不会丢失数据。

### 目录表

* [删除码配置文件](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-profile/)
* [Jerasure擦除代码插件](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-jerasure/)
* [ISA擦除代码插件](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-isa/)
* [可本地修复的擦除代码插件](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-lrc/)
* [SHEC擦除代码插件](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-shec/)
* [CLAY代码插件](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code-clay/)

