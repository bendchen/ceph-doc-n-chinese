# 概述

## 概述

该`ceph-volume`工具旨在成为一种用于将逻辑卷部署为OSD的单一用途的命令行工具，以尝试与`ceph-disk`准备，激活和创建OSD时维护类似的API 。

它`ceph-disk`通过不交互或不依赖于为Ceph安装的udev规则而偏离。这些规则允许自动检测先前设置的设备，这些设备又被送入`ceph-disk`以激活它们。

## 替换`CEPH-DISK`

该`ceph-disk`工具是一次创建的，当时需要该项目支持许多不同类型的初始化系统（启动，sysvinit等），同时能够发现设备。这导致该工具最初（以及之后仅）集中在GPT分区上。特别是在GPT GUID上，这些GUID用于以独特的方式标记设备以回答以下问题：

* 这是日记本吗？
* 加密的数据分区？
* 设备是否部分准备好了？

为了解决这些问题，它使用`UDEV`规则来匹配GUID，该GUID会调用 `ceph-disk`，并最终在`ceph-disk`systemd单元和`ceph-disk`可执行文件之间来回移动。该过程非常不可靠且耗时（**每个OSD**必须设置将近3个小时的超时），并且会导致OSD在节点的引导过程中根本无法启动。

鉴于的异步行为，很难调试甚至很难复制这些问题`UDEV`。

由于的世界观`ceph-disk`必须仅是GPT分区，这意味着它不能与其他技术（例如LVM或类似的设备映射器设备）一起使用。最终决定从LVM支持以及根据需要扩展其他技术的能力来创建模块化的东西。

## GPT分区很简单吗？

尽管一般来说分区很容易推论，但是`ceph-disk` 分区无论如何都不是简单的。为了使它们与设备发现工作流程正确配合，需要大量特殊标志。这是创建数据分区的示例调用：

```text
/sbin/sgdisk --largest-new=1 --change-name=1:ceph data --partition-guid=1:f0fc39fd-eeb2-49f1-b922-a11939cf8a0f --typecode=1:89c57f98-2fe5-4dc0-89c1-f3ad0ceff2be --mbrtogpt -- /dev/sdb
```

创建这些不仅困难，而且这些分区要求设备仅由Ceph拥有。例如，在某些情况下，对设备进行加密时会创建一个特殊的分区，其中将包含未加密的密钥。这是`ceph-disk`领域知识，不会转化为对“ GPT分区很简单”的理解。这是创建特殊分区的示例：

```text
/sbin/sgdisk --new=5:0:+10M --change-name=5:ceph lockbox --partition-guid=5:None --typecode=5:fb3aabf9-d25f-47cc-bf5e-721d181642be --mbrtogpt -- /dev/sdad
```

## 模块化

`ceph-volume`之所以将其设计为模块化工具，是因为我们预计人们会以多种方式提供我们需要考虑的硬件设备。已经有两种：仍在使用并具有GPT分区（由[simple进行](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/#ceph-volume-simple)处理）的旧式ceph磁盘设备，以及lvm。我们直接从用户空间管理NVMe设备的SPDK设备即将出现，由于根本不涉及内核，因此LVM无法在其中运行。

## `CEPH-VOLUME LVM`

通过使用[LVM标记](https://docs.ceph.com/docs/nautilus/glossary/#term-lvm-tags)，[lvm](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/#ceph-volume-lvm)子命令能够存储并稍后重新发现和查询与OSD相关的设备，以便以后可以激活它们。这也包括对基于vm的技术（例如dm-cache）的支持。

对于`ceph-volume`，dm-cache的使用是透明的，该工具没有区别，并且将dm-cache视为普通逻辑卷。

## LVM性能损失

简而言之：我们尚未注意到与LVM更改相关的任何重大性能损失。通过能够与LVM紧密`dmcache`协作，可以与其他设备映射器技术（例如）一起使用：可以处理逻辑卷以下的任何内容都没有技术上的困难。

