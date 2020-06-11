# CEPH-VOLUME-SYSTEMD – SYSTEMD CEPH-VOLUME HELPER TOOL

### 概要

**ceph-volume-systemd** _systemd实例名称_

### 描述

**ceph-volume-systemd**是一个systemd助手工具，该工具从（动态创建的）systemd单元接收输入，以便可以继续激活OSD。

它将输入转换为对ceph-volume的系统调用，仅用于激活目的。

### 例子

它的输入为（以systemd单位表示），并且应采用以下格式：`systemd instance name%i`

```text
<ceph-volume subcommand>-<extra metadata>
```

在`lvm`通话的情况下可能看起来像：

```text
/usr/bin/ceph-volume-systemd lvm-0-8715BEB4-15C5-49DE-BA6F-401086EC7B41
```

依次将`ceph-volume`通过以下方式调用：

```text
ceph-volume lvm trigger  0-8715BEB4-15C5-49DE-BA6F-401086EC7B41
```

任何其他子命令都需要实现一个`trigger`可以使用此格式使用额外元数据的命令。

### 可用性

**ceph-volume-systemd**是Ceph的一部分，Ceph是一个可大规模扩展的开源分布式存储系统。有关更多信息，请参考[http://docs.ceph.com/](http://docs.ceph.com/)上的文档 。

### 另见

[ceph-osd](https://docs.ceph.com/docs/nautilus/man/8/ceph-osd/)（8），

