# BATCH

## `BATCH`

给定设备输入，此子命令允许同时创建多个OSD。根据设备类型（旋转驱动器或固态驱动器），内部引擎将决定创建OSD的最佳方法。

这个决定在创建OSD时抽象出许多细微差别：应该多大`block.db`？如何有效地将固态设备与旋转设备混合？

该过程与[create](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/create/#ceph-volume-lvm-create)相似，并且将按照每个OSD的相同工作流程立即进行准备和激活。但是，如果该`--prepare`标志通过，则仅执行准备步骤，并且OSD未激活。

支持所有支持的功能，例如，避免启动单元，定义bluestore或filestore。不支持任何可能影响单个OSD的细粒度选项，例如：指定应在何处放置日志。`ceph-volume lvm createdmcryptsystemd`

### `BLUESTORE`

所述[bluestore](https://docs.ceph.com/docs/nautilus/glossary/#term-bluestore)对象存储（默认值）创建与所述多个屏上显示当使用`batch`子命令。根据设备的输入，它允许几种不同的方案：

1. 设备都是旋转硬盘：每个设备创建1个OSD
2. 设备都是固态硬盘：每个设备创建2个OSD
3. 设备是HDD和SSD的混合体：数据放置在旋转的设备上，数据`block.db`创建在SSD上，并尽可能大。

注意 

尽管子命令不支持允许使用的 操作`ceph-volume lvm createblock.walbatch`

### `FILESTORE`

的[文件存储](https://docs.ceph.com/docs/nautilus/glossary/#term-filestore)对象存储可以创建与多个屏上显示时，可以使用`batch`的子命令。根据设备的输入，它允许两种不同的方案：

1. 设备都是同一类型（例如，所有旋转的HDD或所有SSD）：每个设备创建1个OSD，并将日志放置在同一HDD中。
2. 设备是HDD和SSD的混合体：数据放置在旋转设备上，而日记则使用ceph.conf中的大小选项在SSD上创建，并回退到默认的日记大小5GB。

当混合使用固态设备和旋转设备时，`ceph-volume`将尝试检测固态设备上现有的卷组。如果找到了VG，它将尝试从那里创建逻辑卷，否则，如果空间不足，则会引发错误。

如果将原始固态设备与具有某些旋转设备的卷组的设备一起使用，`ceph-volume`则将尝试扩展现有的卷组，然后创建逻辑卷。

## 报告

收到创建OSD的调用时，如果可接受预先计算的输出，该工具将提示用户继续。此输出对于了解接收到的设备的结果很有用。接受确认后，该过程将继续。

尽管提示很容易理解结果，但是尝试不同的输入以找到可能的最佳产品非常有用。使用该`--report` 标志，可以阻止任何实际操作，而只需验证输入的结果即可。

**漂亮的报告** 对于两个旋转的设备，这是`pretty`报告的外观（默认）：

```text
$ ceph-volume lvm batch --report /dev/sdb /dev/sdc

Total OSDs: 2

  Type            Path                      LV Size         % of device
--------------------------------------------------------------------------------
  [data]          /dev/sdb                  10.74 GB        100%
--------------------------------------------------------------------------------
  [data]          /dev/sdc                  10.74 GB        100%
```

**JSON报告** Reporting可以使用产生更丰富的输出`JSON`，这为调整大小提供了更多提示。对于其他工具来说，使用此功能可能需要更好的功能，此功能可能会更好。

对于两个旋转设备，`JSON`报告如下所示：

```text
$ ceph-volume lvm batch --report --format=json /dev/sdb /dev/sdc
{
    "osds": [
        {
            "block.db": {},
            "data": {
                "human_readable_size": "10.74 GB",
                "parts": 1,
                "path": "/dev/sdb",
                "percentage": 100,
                "size": 11534336000.0
            }
        },
        {
            "block.db": {},
            "data": {
                "human_readable_size": "10.74 GB",
                "parts": 1,
                "path": "/dev/sdc",
                "percentage": 100,
                "size": 11534336000.0
            }
        }
    ],
    "vgs": [
        {
            "devices": [
                "/dev/sdb"
            ],
            "parts": 1
        },
        {
            "devices": [
                "/dev/sdc"
            ],
            "parts": 1
        }
    ]
}
```

