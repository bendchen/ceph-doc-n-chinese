# 文件存储配置参考

## 文件存储配置参考

`filestore debug omap check`描述

同步上的调试检查。昂贵。仅用于调试。类型

布尔型需要

没有默认

`false`

### 扩展属性

扩展属性（XATTR）是配置中的重要方面。某些文件系统对XATTRS中存储的字节数有限制。此外，在某些情况下，文件系统可能不如存储XATTR的另一种方法快。以下设置可以通过使用存储底层文件系统外部的XATTR的方法来帮助提高性能。

如果Ceph XATTR 没有施加大小限制，则使用基础文件系统提供的XATTR将其存储为。如果存在大小限制（例如，ext4上总共为4KB），则在达到或阈值时，某些Ceph XATTR将存储在键/值数据库中 。`inline xattrfilestore max inline xattr sizefilestore max inline xattrs`

`filestore max inline xattr size`描述

每个对象存储在文件系统中的XATTR的最大大小（即XFS，btrfs，ext4等）。不应大于文件系统可以处理的大小。默认值0表示使用特定于基础文件系统的值。类型

无符号32位整数需要

没有默认

`0`

`filestore max inline xattr size xfs`描述

XFS文件系统中存储的XATTR的最大大小。仅在== 0时使用。`filestore max inline xattr size`类型

无符号32位整数需要

没有默认

`65536`

`filestore max inline xattr size btrfs`描述

存储在btrfs文件系统中的XATTR的最大大小。仅在== 0时使用。`filestore max inline xattr size`类型

无符号32位整数需要

没有默认

`2048`

`filestore max inline xattr size other`描述

存储在其他文件系统中的XATTR的最大大小。仅在== 0时使用。`filestore max inline xattr size`类型

无符号32位整数需要

没有默认

`512`

`filestore max inline xattrs`描述

每个对象存储在文件系统中的XATTR的最大数量。默认值0表示使用特定于基础文件系统的值。类型

32位整数需要

没有默认

`0`

`filestore max inline xattrs xfs`描述

每个对象存储在XFS文件系统中的XATTR的最大数量。仅在== 0时使用。`filestore max inline xattrs`类型

32位整数需要

没有默认

`10`

`filestore max inline xattrs btrfs`描述

每个对象存储在btrfs文件系统中的XATTR的最大数量。仅在== 0时使用。`filestore max inline xattrs`类型

32位整数需要

没有默认

`10`

`filestore max inline xattrs other`描述

每个对象存储在其他文件系统中的XATTR的最大数量。仅在== 0时使用。`filestore max inline xattrs`类型

32位整数需要

没有默认

`2`

### 同步间隔

文件存储区需要定期停止写入并同步文件系统，这会创建一致的提交点。然后，它可以释放日记帐分录直到提交点。更频繁地进行同步往往会减少执行同步所需的时间，并减少需要保留在日志中的数据量。不太频繁的同步使后备文件系统可以更优化地合并小写操作和元数据更新，从而有可能导致更有效的同步。

`filestore max sync interval`描述

同步文件存储的最大间隔（以秒为单位）。类型

双需要

没有默认

`5`

`filestore min sync interval`描述

同步文件存储的最小间隔（以秒为单位）。类型

双需要

没有默认

`.01`

### 冲洗器

文件存储刷新程序会强制在同步之前使用大写操作来写出数据， 以（希望）降低最终同步的成本。实际上，在某些情况下，禁用“文件存储刷新器”似乎可以提高性能。`sync file range`

`filestore flusher`描述

启用文件存储刷新器。类型

布尔型需要

没有默认

`false`

从v.65版本开始不推荐使用。

`filestore flusher max fds`描述

设置刷新程序的最大文件描述符数。类型

整数需要

没有默认

`512`

从v.65版本开始不推荐使用。

`filestore sync flush`描述

启用同步刷新程序。类型

布尔型需要

没有默认

`false`

从v.65版本开始不推荐使用。

`filestore fsync flushes journal data`描述

在文件系统同步期间刷新日志数据。类型

布尔型需要

没有默认

`false`

### 队列

以下设置提供了对文件存储队列大小的限制。

`filestore queue max ops`描述

定义在阻止新操作排队之前文件存储接受的正在进行的操作的最大数量。类型

整数需要

否。对性能的影响最小。默认

`50`

`filestore queue max bytes`描述

一个操作的最大字节数。类型

整数需要

没有默认

`100 << 20`

### 超时

`filestore op threads`描述

并行执行的文件系统操作线程的数量。类型

整数需要

没有默认

`2`

`filestore op thread timeout`描述

文件系统操作线程的超时（以秒为单位）。类型

整数需要

没有默认

`60`

`filestore op thread suicide timeout`描述

取消提交之前提交操作的超时（以秒为单位）。类型

整数需要

没有默认

`180`

### B树文件系统

`filestore btrfs snap`描述

为`btrfs`文件存储启用快照。类型

布尔型需要

否。仅用于`btrfs`。默认

`true`

`filestore btrfs clone range`描述

启用`btrfs`文件存储区的克隆范围。类型

布尔型需要

否。仅用于`btrfs`。默认

`true`

### 日记

`filestore journal parallel`描述

启用并行日志记录，默认为btrfs。类型

布尔型需要

没有默认

`false`

`filestore journal writeahead`描述

启用预写日记功能，默认为xfs。类型

布尔型需要

没有默认

`false`

`filestore journal trailing`描述

已弃用，请勿使用。类型

布尔型需要

没有默认

`false`

### 杂项

`filestore merge threshold`描述

合并到父目录之前，子目录中文件的最小数目注意：负值表示禁用子目录合并类型

整数需要

没有默认

`-10`

`filestore split multiple`描述

`(filestore_split_multiple * abs(filestore_merge_threshold) + (rand() % filestore_split_rand_factor)) * 16` 是在拆分为子目录之前子目录中文件的最大数量。类型

整数需要

没有默认

`2`

`filestore split rand factor`描述

将随机因素添加到拆分阈值，以避免一次发生太多文件存储拆分。有关详细信息，请参见 。只能通过ceph-objectstore-tool的apply-layout-settings命令对现有的osd脱机进行更改。`filestore split multiple`类型

无符号32位整数需要

没有默认

`20`

`filestore update to`描述

限制文件存储自动升级到指定版本。类型

整数需要

没有默认

`1000`

`filestore blackhole`描述

将所有新交易放在地板上。类型

布尔型需要

没有默认

`false`

`filestore dump file`描述

存储事务转储到的文件。类型

布尔型需要

没有默认

`false`

`filestore kill at`描述

在第n个机会注入失败类型

串需要

没有默认

`false`

`filestore fail eio`描述

eio失败/崩溃。类型

布尔型需要

没有默认

`true`

