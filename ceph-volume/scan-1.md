# SCAN

## `SCAN`

扫描允许从已部署的OSD捕获任何重要的详细信息，从而`ceph-volume`无需任何其他启动工作流程或工具（如`udev`或`ceph-disk`）即可对其进行管理。完全支持使用LUKS或PLAIN格式的加密。

通过检查OSD数据的存储目录或使用数据分区，该命令可以检查正在运行的OSD。如果没有提供路径或设备，该命令还可以扫描所有正在运行的OSD。

扫描后，信息（默认情况下）会将元数据作为JSON保留在中的文件中`/etc/ceph/osd`。该`JSON`文件将使用的命名约定。OSD的ID为1，FSID类似于 文件的绝对路径：`{OSD ID}-{OSD FSID}.json86ebd829-1405-43d3-8fd6-4cbc9b6ecf96`

```text
/etc/ceph/osd/1-86ebd829-1405-43d3-8fd6-4cbc9b6ecf96.json
```

该`scan`子命令将拒绝写入该文件，如果它已经存在。如果需要覆盖内容，则`--force`必须使用该标志：

```text
ceph-volume simple scan --force {path}
```

如果不需要保留`JSON`元数据，则支持将内容发送到`stdout`（不会写入任何文件）：

```text
ceph-volume simple scan --stdout {path}
```

### 运行OSD扫描

在不提供OSD目录或设备的情况下使用此命令将扫描任何当前正在运行的OSD的目录。如果不是通过ceph-disk创建正在运行的OSD，它将被忽略并且不会被扫描。

要扫描所有正在运行的ceph磁盘OSD，命令如下所示：

```text
ceph-volume simple scan
```

### 目录扫描

目录扫描将从有趣的文件中捕获OSD文件内容。为了成功扫描，必须存在一些文件：

* `ceph_fsid`
* `fsid`
* `keyring`
* `ready`
* `type`
* `whoami`

如果OSD已加密，它将另外添加以下密钥：

* `encrypted`
* `encryption_type`
* `lockbox_keyring`

对于任何其他文件，只要它不是二进制文件或目录，它也将被捕获并作为JSON对象的一部分持久化。

JSON对象中密钥的约定是，任何文件名都将是密钥，其内容将是其值。如果内容是单行（如的情况`whoami`），则将修剪内容，并删除换行符。例如，对于一个ID为1的OSD，这就是JSON条目的样子：

```text
"whoami": "1",
```

对于可能包含多行的文件，其内容按原样保留，但经过特殊处理并经过分析以提取出密钥环的密钥环除外。例如，一个`keyring`被读取为：

```text
[osd.1]\n\tkey = AQBBJ/dZp57NIBAAtnuQS9WOS0hnLVe0rZnE6Q==\n
```

将存储为：

```text
"keyring": "AQBBJ/dZp57NIBAAtnuQS9WOS0hnLVe0rZnE6Q==",
```

对于目录`/var/lib/ceph/osd/ceph-1`，命令如下所示：

```text
ceph-volume simple scan /var/lib/ceph/osd/ceph1
```

### 设备扫描

当OSD目录不可用（OSD未运行或未安装设备）时，该`scan`命令可以对设备进行自检以捕获所需的数据。就像[运行OSD扫描一样](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/scan/#ceph-volume-simple-scan-directory)，它仍然需要一些文件。这意味着要扫描的设备 **必须是** OSD的数据分区。

只要将OSD的数据分区作为参数传递，子命令就可以扫描其内容。

在已经安装设备的情况下，该工具可以检测到这种情况并从该目录捕获文件内容。

如果未安装设备，则将创建一个临时目录，并且该设备将被临时安装，仅用于扫描内容。扫描内容后，将卸载设备。

对于设备等`/dev/sda1`，其**必须**是数据分区，该命令可能看起来像：

```text
ceph-volume simple scan /dev/sda1
```

### `JSON`内容

JSON对象的内容非常简单。扫描不仅将保留特殊OSD文件及其内容中的信息，还将验证路径和设备UUID。与`ceph-disk`将要执行的操作不同，该工具将它们存储在文件中，并将其作为设备类型密钥的一部分保留。`{device type}_uuid`

例如，`block.db`设备看起来像：

```text
"block.db": {
    "path": "/dev/disk/by-partuuid/6cc43680-4f6e-4feb-92ff-9c7ba204120e",
    "uuid": "6cc43680-4f6e-4feb-92ff-9c7ba204120e"
},
```

但它还将保留`ceph-disk`生成的特殊文件，如下所示：

```text
"block.db_uuid": "6cc43680-4f6e-4feb-92ff-9c7ba204120e",
```

之所以有此重复项，是因为该工具正在尝试确保满足以下条件：

＃支持可能没有ceph磁盘特殊文件的OSD＃通过查询LVM来检查设备上的最新信息和`blkid` ＃支持逻辑卷和GPT设备

这是`JSON`来自使用`bluestore`以下命令的OSD 的示例元数据：

```text
{
    "active": "ok",
    "block": {
        "path": "/dev/disk/by-partuuid/40fd0a64-caa5-43a3-9717-1836ac661a12",
        "uuid": "40fd0a64-caa5-43a3-9717-1836ac661a12"
    },
    "block.db": {
        "path": "/dev/disk/by-partuuid/6cc43680-4f6e-4feb-92ff-9c7ba204120e",
        "uuid": "6cc43680-4f6e-4feb-92ff-9c7ba204120e"
    },
    "block.db_uuid": "6cc43680-4f6e-4feb-92ff-9c7ba204120e",
    "block_uuid": "40fd0a64-caa5-43a3-9717-1836ac661a12",
    "bluefs": "1",
    "ceph_fsid": "c92fc9eb-0610-4363-aafc-81ddf70aaf1b",
    "cluster_name": "ceph",
    "data": {
        "path": "/dev/sdr1",
        "uuid": "86ebd829-1405-43d3-8fd6-4cbc9b6ecf96"
    },
    "fsid": "86ebd829-1405-43d3-8fd6-4cbc9b6ecf96",
    "keyring": "AQBBJ/dZp57NIBAAtnuQS9WOS0hnLVe0rZnE6Q==",
    "kv_backend": "rocksdb",
    "magic": "ceph osd volume v026",
    "mkfs_done": "yes",
    "ready": "ready",
    "systemd": "",
    "type": "bluestore",
    "whoami": "3"
}
```

