# ACTIVATE

## `ACTIVATE`

一旦[准备工作](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/prepare/#ceph-volume-lvm-prepare)完成，并且完成了所有各个步骤，便可以开始激活该卷了。

此激活过程使systemd单元可以保留OSD ID及其UUID（`fsid`在Ceph CLI工具中也称为UUID ），以便在启动时可以了解启用了什么OSD以及需要安装什么OSD。

注意 

此调用的执行是完全幂等的，并且多次运行时没有副作用

### 新的OSD 

要激活新准备的[OSD](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-id)， 需要同时提供[OSD ID](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-id)和[OSD UUID](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-uuid)。例如：

```text
ceph-volume lvm activate --bluestore 0 0263644D-0BF1-4D6D-BC34-28BD98AE3BC8
```

注意 

UUID存储在`fsid`OSD路径中的文件中，该文件在使用[prepare](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/prepare/#ceph-volume-lvm-prepare)时生成。

### 激活所有的OSD 

使用该`--all` 标志可以一次激活所有现有的OSD 。例如：

```text
ceph-volume lvm activate --all
```

此调用将检查由ceph-volume创建的所有非活动OSD，并将逐个激活它们。如果任何OSD已经在运行，它将在命令输出中报告它们并跳过它们，从而使其可以安全地重新运行（幂等）。

#### 需要的UUID 

该[UUID OSD](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-uuid)被要求作为一个额外的步骤，以确保正确的OSD被激活。先前存在具有相同ID的OSD很有可能会导致激活错误的OSD。

#### DMCRYPT 

如果OSD是使用ceph-volume的dmcrypt准备的，则无需`--dmcrypt`再次在命令行上指定（该标志不可用于`activate`子命令）。将会自动检测到加密的OSD。

### 发现

对于先前由创建的OSD `ceph-volume`，使用[LVM标签](https://docs.ceph.com/docs/nautilus/glossary/#term-lvm-tags)执行_发现_过程以启用系统单元。

systemd单元将捕获[OSD id](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-id)和[OSD uuid](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-uuid)并将其保留。在内部，激活将启用它，例如：

```text
systemctl enable ceph-volume@lvm-$id-$uuid
```

例如：

```text
systemctl enable ceph-volume@lvm-0-8715BEB4-15C5-49DE-BA6F-401086EC7B41
```

将以ID为`0`且UUID 为的OSD启动发现过程`8715BEB4-15C5-49DE-BA6F-401086EC7B41`。

注意 

有关systemd工作流程的更多详细信息，请参见[systemd](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/systemd/#ceph-volume-lvm-systemd)

systemd单元将查找匹配的OSD设备，并通过查看其 [LVM标签](https://docs.ceph.com/docs/nautilus/glossary/#term-lvm-tags)将继续执行以下操作：＃将设备安装在相应的位置（按照惯例，这是

`/var/lib/ceph/osd/<cluster name>-<osd id>/`）

＃确保所有必需的设备都已准备好用于该OSD。对于日志（`--filestore`选择时），将查询该设备（`blkid`对于分区，查询为，对于逻辑卷，查询 为lvm），以确保链接了正确的设备。符号链接将_始终_被重做以确保链接了正确的设备。

＃启动`ceph-osd@0`系统单元

注意 

系统通过检查应用于OSD设备的LVM标签来推断对象存储类型（文件存储或bluestore）

### 现有的OSD 

对于已经部署了现有OSD，`ceph-disk`需要[使用简单的sub命令](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/#ceph-volume-simple)对其进行扫描和激活。如果使用了不同的工具，那么将它们移植到新机制的唯一方法是再次准备它们（丢失数据）。有关如何进行操作的详细信息，请参阅 [现有的OSD](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/prepare/#ceph-volume-lvm-existing-osds)。

### 总结

回顾一下[bluestore](https://docs.ceph.com/docs/nautilus/glossary/#term-bluestore)的`activate`过程：

1. 同时要求[OSD ID](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-id)和[OSD UUID](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-uuid)
2. 使系统单元具有匹配的id和uuid
3. `tmpfs`在以下位置的OSD目录中创建安装 `/var/lib/ceph/osd/$cluster-$id/`
4. 通过将其指向OSD 设备来重新创建所需的所有文件。`ceph-bluestore-tool prime-osd-dirblock`
5. 系统单元将确保所有设备准备就绪并已链接
6. 匹配的`ceph-osd`系统单元将开始

对于[文件存储](https://docs.ceph.com/docs/nautilus/glossary/#term-filestore)：

1. 同时要求[OSD ID](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-id)和[OSD UUID](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-uuid)
2. 使系统单元具有匹配的id和uuid
3. 系统单元将确保所有设备准备就绪并已安装（如果需要）
4. 匹配的`ceph-osd`系统单元将开始

