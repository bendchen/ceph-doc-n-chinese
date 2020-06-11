# ceph-run-核心转储重启守护进程

## ceph-run-核心转储重启守护进程

### 概要

**ceph-run** _命令_ …

### 描述

**ceph-run**是一个简单的包装程序，如果守护程序退出并显示表明它已崩溃并可能已转储内核的信号（即信号3、4、5、6、8或11），它将重新启动守护程序。

该命令应在前台运行守护程序。对于Ceph守护程序，这意味着该`-f`选项。

### 选项

没有

### 可用性

**ceph-run**是Ceph的一部分，Ceph是一个可大规模扩展的开源分布式存储系统。有关更多信息，请参阅[http://ceph.com/docs](http://ceph.com/docs)上的Ceph文档。

### 另见

[ceph](https://docs.ceph.com/docs/nautilus/man/8/ceph/)（8）， [ceph-mon](https://docs.ceph.com/docs/nautilus/man/8/ceph-mon/)（8）， [ceph-mds](https://docs.ceph.com/docs/nautilus/man/8/ceph-mds/)（8）， [ceph-osd](https://docs.ceph.com/docs/nautilus/man/8/ceph-osd/)（8）

