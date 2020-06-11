# LIST

## `LIST`

此子命令将列出可能与Ceph群集关联的所有设备（逻辑和物理设备），只要它们包含足够的元数据以允许该发现即可。

输出按与设备关联的OSD ID分组，与之不同的是， `ceph-disk`它不为与Ceph无关的设备提供任何信息。

命令行选项：

* `--format`允许`json`或`pretty`值。默认值`pretty` ，它将以人类可读的格式对设备信息进行分组。

### 全面的报告

当不使用位置参数时，将显示完整的报告。这意味着将显示系统中找到的所有设备和逻辑卷。

`pretty`两个OSD的完整报告，其中一个带有lv作为日志，另一个带有物理设备，可能类似于：

```text
# ceph-volume lvm list


====== osd.1 =======

  [journal]    /dev/journals/journal1

      journal uuid              C65n7d-B1gy-cqX3-vZKY-ZoE0-IEYM-HnIJzs
      osd id                    1
      cluster fsid              ce454d91-d748-4751-a318-ff7f7aa18ffd
      type                      journal
      osd fsid                  661b24f8-e062-482b-8110-826ffe7f13fa
      data uuid                 SlEgHe-jX1H-QBQk-Sce0-RUls-8KlY-g8HgcZ
      journal device            /dev/journals/journal1
      data device               /dev/test_group/data-lv2
      devices                   /dev/sda

  [data]    /dev/test_group/data-lv2

      journal uuid              C65n7d-B1gy-cqX3-vZKY-ZoE0-IEYM-HnIJzs
      osd id                    1
      cluster fsid              ce454d91-d748-4751-a318-ff7f7aa18ffd
      type                      data
      osd fsid                  661b24f8-e062-482b-8110-826ffe7f13fa
      data uuid                 SlEgHe-jX1H-QBQk-Sce0-RUls-8KlY-g8HgcZ
      journal device            /dev/journals/journal1
      data device               /dev/test_group/data-lv2
      devices                   /dev/sdb

====== osd.0 =======

  [data]    /dev/test_group/data-lv1

      journal uuid              cd72bd28-002a-48da-bdf6-d5b993e84f3f
      osd id                    0
      cluster fsid              ce454d91-d748-4751-a318-ff7f7aa18ffd
      type                      data
      osd fsid                  943949f0-ce37-47ca-a33c-3413d46ee9ec
      data uuid                 TUpfel-Q5ZT-eFph-bdGW-SiNW-l0ag-f5kh00
      journal device            /dev/sdd1
      data device               /dev/test_group/data-lv1
      devices                   /dev/sdc

  [journal]    /dev/sdd1

      PARTUUID                  cd72bd28-002a-48da-bdf6-d5b993e84f3f
```

对于逻辑卷，`devices`键中填充了与逻辑卷关联的物理设备。由于LVM允许多个物理设备成为逻辑卷的一部分，因此使用时该值将以逗号分隔，使用 `pretty`时将使用数组`json`。

注意 

标签以可读格式显示。将密钥存储为一个标签。有关lvm标记约定的更多信息，请参见[标记API。](https://docs.ceph.com/docs/nautilus/dev/ceph-volume/lvm/#ceph-volume-lvm-tag-api)`osd idceph.osd_id`

### 单报告

单个报告可以消耗设备和逻辑卷作为输入（位置参数）。对于逻辑卷，要求使用组名和逻辑卷名。

例如`data-lv2`，`test_group`可以通过以下方式列出卷组中的逻辑卷：

```text
# ceph-volume lvm list test_group/data-lv2


====== osd.1 =======

  [data]    /dev/test_group/data-lv2

      journal uuid              C65n7d-B1gy-cqX3-vZKY-ZoE0-IEYM-HnIJzs
      osd id                    1
      cluster fsid              ce454d91-d748-4751-a318-ff7f7aa18ffd
      type                      data
      osd fsid                  661b24f8-e062-482b-8110-826ffe7f13fa
      data uuid                 SlEgHe-jX1H-QBQk-Sce0-RUls-8KlY-g8HgcZ
      journal device            /dev/journals/journal1
      data device               /dev/test_group/data-lv2
      devices                   /dev/sdc
```

注意 

标签以可读格式显示。将密钥存储为一个标签。有关lvm标记约定的更多信息，请参见[标记API。](https://docs.ceph.com/docs/nautilus/dev/ceph-volume/lvm/#ceph-volume-lvm-tag-api)`osd idceph.osd_id`

对于普通磁盘，需要设备的完整路径。例如，对于类似的设备，`/dev/sdd1`其外观可能如下所示：

```text
# ceph-volume lvm list /dev/sdd1


====== osd.0 =======

  [journal]    /dev/sdd1

      PARTUUID                  cd72bd28-002a-48da-bdf6-d5b993e84f3f
```

### `JSON`输出

使用的所有输出`--format=json`将显示系统已存储为设备元数据的所有内容，包括标签。

`json`报告的可读性未做任何更改，所有信息均按原样显示。可以列出完整的输出以及单个设备。

为简便起见，这是单个逻辑卷与`json` 输出的外观（请注意如何修改标签）：

```text
# ceph-volume lvm list --format=json test_group/data-lv1
{
    "0": [
        {
            "devices": ["/dev/sda"],
            "lv_name": "data-lv1",
            "lv_path": "/dev/test_group/data-lv1",
            "lv_tags": "ceph.cluster_fsid=ce454d91-d748-4751-a318-ff7f7aa18ffd,ceph.data_device=/dev/test_group/data-lv1,ceph.data_uuid=TUpfel-Q5ZT-eFph-bdGW-SiNW-l0ag-f5kh00,ceph.journal_device=/dev/sdd1,ceph.journal_uuid=cd72bd28-002a-48da-bdf6-d5b993e84f3f,ceph.osd_fsid=943949f0-ce37-47ca-a33c-3413d46ee9ec,ceph.osd_id=0,ceph.type=data",
            "lv_uuid": "TUpfel-Q5ZT-eFph-bdGW-SiNW-l0ag-f5kh00",
            "name": "data-lv1",
            "path": "/dev/test_group/data-lv1",
            "tags": {
                "ceph.cluster_fsid": "ce454d91-d748-4751-a318-ff7f7aa18ffd",
                "ceph.data_device": "/dev/test_group/data-lv1",
                "ceph.data_uuid": "TUpfel-Q5ZT-eFph-bdGW-SiNW-l0ag-f5kh00",
                "ceph.journal_device": "/dev/sdd1",
                "ceph.journal_uuid": "cd72bd28-002a-48da-bdf6-d5b993e84f3f",
                "ceph.osd_fsid": "943949f0-ce37-47ca-a33c-3413d46ee9ec",
                "ceph.osd_id": "0",
                "ceph.type": "data"
            },
            "type": "data",
            "vg_name": "test_group"
        }
    ]
}
```

### 同步的信息

在使用任何列表类型之前，都会查询lvm API以确保可能使用的物理设备未更改命名。诸如此类的非永久性设备`/dev/sda1`可能会变为`/dev/sdb1`。

该检测是可能的，因为将`PARTUUID`其作为元数据的一部分存储在数据lv的逻辑卷中。即使在日志是物理设备的情况下，此信息仍然存储在与其关联的数据逻辑卷上。

如果名称不再相同（`blkid`使用时 报告的名称`PARTUUID`），则标记将被更新，并且报告将使用新刷新的信息。

