# PREPARE

## `PREPARE`

此子命令允许[文件存储](https://docs.ceph.com/docs/nautilus/glossary/#term-filestore)或[bluestore](https://docs.ceph.com/docs/nautilus/glossary/#term-bluestore)设置。建议在与一起使用之前预先配置一个逻辑卷 。`ceph-volume lvm`

除了添加额外的元数据外，逻辑卷不会更改。

注意 

这是部署OSD的两步过程的一部分。如果要使用单次通话方式，请参见[创建](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/create/#ceph-volume-lvm-create)

为了帮助识别卷，在准备一个或多个卷以与Ceph一起使用的过程中，该工具将使用LVM tags 分配一些元数据信息 。

LVM tags 使卷在以后易于发现，并有助于将它们识别为Ceph系统的一部分，以及它们所扮演的角色（新闻，文件存储，bluestore等）。

尽管最初支持（默认情况下也支持）[文件存储](https://docs.ceph.com/docs/nautilus/glossary/#term-filestore)，但可以通过以下方式指定后端：

* [–文件存储](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/prepare/#ceph-volume-lvm-prepare-filestore)
* [–bluestore](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/prepare/#ceph-volume-lvm-prepare-bluestore)

### `BLUESTORE`

该[bluestore](https://docs.ceph.com/docs/nautilus/glossary/#term-bluestore)对象存储是新的OSD默认。与[文件存储](https://docs.ceph.com/docs/nautilus/glossary/#term-filestore)相比，它为设备提供了更多的灵活性。Bluestore支持以下配置：

* 块设备，block.wal和block.db设备
* 块设备和块沃尔玛设备
* 块设备和block.db设备
* 单块设备

bluestore子命令接受物理块设备，物理块设备上的分区或逻辑卷作为各种设备参数的参数。如果提供了物理设备，则将创建逻辑卷。将创建或重新使用卷组，其名称以开头`ceph`。这允许在使用LVM时使用更简单的方法，但是会以灵活性为代价：没有选择或配置可以更改LV的创建方式。

将`block`与指定的`--data`标志，并在其最简单的使用情况下，它看起来像：

```text
ceph-volume lvm prepare --bluestore --data vg/lv
```

原始设备可以用相同的方式指定：

```text
ceph-volume lvm prepare --bluestore --data /path/to/device
```

要启用[加密](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/encryption/#ceph-volume-lvm-encryption)，该`--dmcrypt`标志是必需的：

```text
ceph-volume lvm prepare --bluestore --dmcrypt --data vg/lv
```

如果需要a `block.db`或a `block.wal`（它们对于bluestore是可选的），则可以使用`--block.db`和进行指定`--block.wal` 。这些可以是物理设备，分区或逻辑卷。

对于`block.db`和`block.wal`分区，由于它们可以按原样使用，因此不构成逻辑卷。

在创建OSD目录时，该过程将使用`tmpfs`挂载放置OSD所需的所有文件。这些文件最初由临时创建， 并且完全是临时的。`ceph-osd --mkfs`

总是为`block`设备创建一个符号链接，并为 `block.db`和创建一个符号链接`block.wal`。对于具有默认名称且OSD ID为0的集群，目录可能类似于：

```text
# ls -l /var/lib/ceph/osd/ceph-0
lrwxrwxrwx. 1 ceph ceph 93 Oct 20 13:05 block -> /dev/ceph-be2b6fbd-bcf2-4c51-b35d-a35a162a02f0/osd-block-25cf0a05-2bc6-44ef-9137-79d65bd7ad62
lrwxrwxrwx. 1 ceph ceph 93 Oct 20 13:05 block.db -> /dev/sda1
lrwxrwxrwx. 1 ceph ceph 93 Oct 20 13:05 block.wal -> /dev/ceph/osd-wal-0
-rw-------. 1 ceph ceph 37 Oct 20 13:05 ceph_fsid
-rw-------. 1 ceph ceph 37 Oct 20 13:05 fsid
-rw-------. 1 ceph ceph 55 Oct 20 13:05 keyring
-rw-------. 1 ceph ceph  6 Oct 20 13:05 ready
-rw-------. 1 ceph ceph 10 Oct 20 13:05 type
-rw-------. 1 ceph ceph  2 Oct 20 13:05 whoami
```

在上述情况下，使用了设备，`block`因此`ceph-volume`使用以下约定创建卷组和逻辑卷：

* 卷组名称：或者如果vg已经存在 `ceph-{cluster fsid}ceph-{random uuid}`
* 逻辑卷名称： `osd-block-{osd_fsid}`

### `FILESTORE`

这是OSD后端，允许为[文件存储](https://docs.ceph.com/docs/nautilus/glossary/#term-filestore)对象库OSD 准备逻辑卷。

它可以将逻辑卷用于OSD数据，并将物理设备，分区或逻辑卷用于日志。物理设备将在其上创建逻辑卷。将创建或重新使用卷组，其名称以开头`ceph`。除了遵循数据和日志的最小大小要求外，无需为这些卷进行任何特殊准备。

CLI调用类似于基本的独立文件存储OSD：

```text
ceph-volume lvm prepare --filestore --data <data block device>
```

要将文件存储与外部日记一起部署：

```text
ceph-volume lvm prepare --filestore --data <data block device> --journal <journal block device>
```

要启用[加密](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/encryption/#ceph-volume-lvm-encryption)，该`--dmcrypt`标志是必需的：

```text
ceph-volume lvm prepare --filestore --dmcrypt --data <data block device> --journal <journal block device>
```

日志和数据块设备都可以采用三种形式：

* 物理块设备
* 物理块设备上的分区
* 逻辑卷

使用逻辑卷时，该值_必须_为格式 `volume_group/logical_volume`。由于没有为唯一性强加逻辑卷名称，因此可以防止意外选择错误的卷。

使用分区时，该分区_必须_包含`PARTUUID`，可以通过找到`blkid`。这样可以确保以后无论设备名称（或路径）如何都能正确识别它。

例如：传递数据的逻辑卷和`/dev/sdc1`日志的分区：

```text
ceph-volume lvm prepare --filestore --data volume_group/lv_name --journal /dev/sdc1
```

向裸机传递数据和逻辑卷的裸设备即表示日志：

```text
ceph-volume lvm prepare --filestore --data /dev/sdc --journal volume_group/journal_lv
```

生成的uuid用于向群集请求新的OSD。这两部分对于识别OSD至关重要，以后将在整个 [激活](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/activate/#ceph-volume-lvm-activate)过程中使用。

使用以下约定创建OSD数据目录：

```text
/var/lib/ceph/osd/<cluster name>-<osd id>
```

此时，数据卷已安装在此位置，并且日记卷已链接：

```text
ln -s /path/to/journal /var/lib/ceph/osd/<cluster_name>-<osd-id>/journal
```

使用OSD中的引导程序密钥获取monmap：

```text
/usr/bin/ceph --cluster ceph --name client.bootstrap-osd
--keyring /var/lib/ceph/bootstrap-osd/ceph.keyring
mon getmap -o /var/lib/ceph/osd/<cluster name>-<osd id>/activate.monmap
```

`ceph-osd` 将被调用以填充已经安装的OSD目录，并重新使用初始步骤中的所有信息：

```text
ceph-osd --cluster ceph --mkfs --mkkey -i <osd id> \
--monmap /var/lib/ceph/osd/<cluster name>-<osd id>/activate.monmap --osd-data \
/var/lib/ceph/osd/<cluster name>-<osd id> --osd-journal /var/lib/ceph/osd/<cluster name>-<osd id>/journal \
--osd-uuid <osd uuid> --keyring /var/lib/ceph/osd/<cluster name>-<osd id>/keyring \
--setuser ceph --setgroup ceph
```

### 分区

`ceph-volume lvm`当前不从整个设备创建分区。如果使用设备分区，则唯一的要求是它们包含 `PARTUUID`和，并且可以通过进行发现`blkid`。双方`fdisk`并 `parted`会自动创建一个新的分区。

例如，使用新的未格式化驱动器（`/dev/sdd`在这种情况下），我们可以`parted`用来创建新分区。首先我们列出设备信息：

```text
$ parted --script /dev/sdd print
Model: VBOX HARDDISK (scsi)
Disk /dev/sdd: 11.5GB
Sector size (logical/physical): 512B/512B
Disk Flags:
```

该设备甚至不标注呢，所以我们可以用`parted`创建一个`gpt`标签，我们创建一个分区之前，与再次验证：`parted print`

```text
$ parted --script /dev/sdd mklabel gpt
$ parted --script /dev/sdd print
Model: VBOX HARDDISK (scsi)
Disk /dev/sdd: 11.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:
```

现在让我们创建一个分区，稍后再验证是否`blkid`可以找到`PARTUUID`所需的分区`ceph-volume`：

```text
$ parted --script /dev/sdd mkpart primary 1 100%
$ blkid /dev/sdd1
/dev/sdd1: PARTLABEL="primary" PARTUUID="16399d72-1e1f-467d-96ee-6fe371a7d0d4"
```

### 现有的OSD 

对于要使用此新系统并已在运行OSD的现有集群，需要考虑以下几点：

警告 

此过程将强制格式化数据设备，破坏现有数据（如果有）。

* OSD路径应遵循以下约定：

  ```text
  /var/lib/ceph/osd/<cluster name>-<osd id>
  ```

* 最好，不应该存在任何其他安装卷的机制，并且应该将其删除（例如fstab挂载点）

ID为0且使用`"ceph"`群集名称的现有OSD的一次性过程看起来像（以下命令将**破坏** OSD中的**所有数据**）：

```text
ceph-volume lvm prepare --filestore --osd-id 0 --osd-fsid E3D291C1-E7BF-4984-9794-B60D9FA139CB
```

命令行工具将不与监视器联系以生成OSD ID，并且除了将元数据存储在其上之外还将格式化LVM设备，以便以后可以启动它（有关详细的元数据描述，请参见 [Metadata](https://docs.ceph.com/docs/nautilus/dev/ceph-volume/lvm/#ceph-volume-lvm-tags)）。

### 粉碎设备类

要为OSD设置粉碎设备类，请使用该`--crush-device-class`标志。这将同时适用于bluestore和文件存储OSD：

```text
ceph-volume lvm prepare --bluestore --data vg/lv --crush-device-class foo
```

### `MULTIPATH`支持

原始设备`multipath`不受支持。该工具将拒绝使用原始的多路径设备，并报告如下消息：

```text
-->  RuntimeError: Cannot use device (/dev/mapper/<name>). A vg/lv path or an existing device is needed
```

不支持多路径的原因是，根据多路径设置的类型，如果使用主动/被动阵列作为基础物理设备，则需要使用过滤器`lvm.conf`以排除属于那些基础设备的磁盘。

对于ceph-volume来说，要使LVM能够在各种不同的多路径方案中工作，了解哪种配置类型是不可行的。为您创建LV的功能仅仅是（天真的）便利，任何涉及不同设置或配置的内容都必须由配置管理系统提供，然后配置管理系统可以提供VG和LV供ceph-volume使用。

仅当尝试使用从设备创建卷组和逻辑卷的ceph-volume功能时，才会出现这种情况。如果多路径设备已经是逻辑卷，_则应_正确运行LVM配置以避免出现问题，它_应该可以_工作。

### 存储元数据

无论卷的类型（期刊或数据）或OSD对象库如何，以下标记将在准备过程中应用：

* `cluster_fsid`
* `encrypted`
* `osd_fsid`
* `osd_id`
* `crush_device_class`

对于[文件存储，](https://docs.ceph.com/docs/nautilus/glossary/#term-filestore)将添加以下标记：

* `journal_device`
* `journal_uuid`

对于[bluestore，](https://docs.ceph.com/docs/nautilus/glossary/#term-bluestore)将添加以下标签：

* `block_device`
* `block_uuid`
* `db_device`
* `db_uuid`
* `wal_device`
* `wal_uuid`

注意 

有关完整的lvm标签约定，请参见[Tag API](https://docs.ceph.com/docs/nautilus/dev/ceph-volume/lvm/#ceph-volume-lvm-tag-api)

### 总结

回顾一下[bluestore](https://docs.ceph.com/docs/nautilus/glossary/#term-bluestore)的`prepare`过程：

1. 接受原始物理设备，物理设备上的分区或逻辑卷作为参数。
2. 在任何原始物理设备上创建逻辑卷。
3. 为OSD生成UUID
4. 要求监视器获取OSD ID以重新使用生成的UUID
5. OSD数据目录是在tmpfs挂载上创建的。
6. `block`，`block.wal`和`block.db`被定义为符号链接。
7. 提取monmap进行激活
8. 数据目录由 `ceph-osd`
9. 使用lvm标记为逻辑卷分配了所有Ceph元数据

以及[文件存储](https://docs.ceph.com/docs/nautilus/glossary/#term-filestore)的`prepare`过程：

1. 接受原始物理设备，物理设备上的分区或逻辑卷作为参数。
2. 为OSD生成UUID
3. 要求监视器获取OSD ID以重新使用生成的UUID
4. OSD数据目录已创建并已安装数据卷
5. 日记是从数据量到日记位置的符号链接
6. 提取monmap进行激活
7. 安装设备并通过以下命令填充数据目录 `ceph-osd`
8. 数据和日志卷使用lvm标签分配了所有Ceph元数据

