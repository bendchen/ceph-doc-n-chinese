# CEPH-VOLUME – CEPH OSD DEPLOYMENT AND INSPECTION TOOL

## CEPH-VOLUME – CEPH OSD DEPLOYMENT AND INSPECTION TOOL

### SYNOPSIS

**ceph-volume** \[-h\] \[–cluster CLUSTER\] \[–log-level LOG\_LEVEL\]\[–log-path LOG\_PATH\]**ceph-volume** **inventoryceph-volume** **lvm** \[ _trigger_ \| _create_ \| _activate_ \| _preparezap_ \| _list_ \| _batch_\]**ceph-volume** **simple** \[ _trigger_ \| _scan_ \| _activate_ \]

### DESCRIPTION

**ceph-volume**是用于将逻辑卷部署为OSD的单一目的命令行工具，试图维护与`ceph-disk`准备，激活和创建OSD时类似的API 。

它`ceph-disk`通过不交互或不依赖于为Ceph安装的udev规则而偏离。这些规则允许自动检测先前设置的设备，这些设备又被送入`ceph-disk`以激活它们。

### 命令

#### 库存

此子命令提供有关主机的物理磁盘清单的信息，并报告有关这些磁盘的元数据。在此元数据中，可以找到特定于光盘的数据项（例如型号，大小，旋转或固态），以及使用设备特定于ceph的数据项，例如它是否可用于ceph或存在逻辑卷。

例子：

```text
ceph-volume inventory
ceph-volume inventory /dev/sda
ceph-volume inventory --format json-pretty
```

可选参数：

* \[-h，–help\]显示帮助消息并退出
* \[–format\]报告格式，有效值为`plain`（默认），

  `json` 和 `json-pretty`

#### LVM 

通过使用LVM标签，该`lvm`子命令能够存储和以后重新发现和查询与OSD相关的设备，以便以后可以激活它们。

子命令：

**批处理** 使用`filestore` 或`bluestore`（默认）设置从设备列表中创建OSD 。它将创建拥有正常工作的OSD所需的所有必要的卷组和逻辑卷。

三种设备的用法示例：

```text
ceph-volume lvm batch --bluestore /dev/sda /dev/sdb /dev/sdc
```

可选参数：

* \[-h，–help\]显示帮助消息并退出
* \[–bluestore\]使用bluestore对象库（默认）
* \[–filestore\]使用文件存储对象库
* \[–是\]跳过报告并提示继续设置
* \[–prepare\]仅准备OSD，不激活
* \[–dmcrypt\]为基础OSD设备启用加密
* \[–crush-device-class\]定义一个CRUSH设备类以将OSD分配给
* \[–no-systemd\]不要启用或创建任何systemd单元
* \[–报告\]报告潜在的结果是

  当前输入（需要传入的设备）

* \[–format\]报告时的输出格式（与

  –report），可以是“ pretty”（默认）或“ json”之一

* \[–block-db-size\]设置（或覆盖）“ bluestore\_block\_db\_size”值，

  以字节为单位

* \[–journal-size\]覆盖“ osd\_journal\_size”值，以兆字节为单位

必需的位置参数：

* &lt;DEVICE&gt;原始设备的完整路径，例如`/dev/sda`。多

  `<DEVICE>` 可以传递路径。

**activate** 启用一个可保留OSD ID及其UUID（`fsid`在Ceph CLI工具中也称为UUID）的systemd单元 ，以便在启动时它可以了解已启用和需要安装的OSD。

用法：

```text
ceph-volume lvm activate --bluestore <osd id> <osd fsid>
```

可选参数：

* \[-h，–help\]显示帮助消息并退出
* \[–auto-detect-objectstore\]通过检查OSD自动检测对象库
* \[–bluestore\] bluestore对象库（默认）
* \[–filestore\]文件存储对象存储
* \[–all\]激活系统中找到的所有OSD
* \[–no-systemd\]跳过创建和启用systemd单元以及启动OSD服务的操作

使用（幂等）`--all`标志可以一次激活多个OSD ：

```text
ceph-volume lvm activate --all
```

**prepare** 使用`filestore` 或`bluestore`（默认）设置准备要用作OSD和日志的逻辑卷。除了添加额外的元数据外，它不会创建或修改逻辑卷。

用法：

```text
ceph-volume lvm prepare --filestore --data <data lv> --journal <journal device>
```

可选参数：

* \[-h，–help\]显示帮助消息并退出
* \[–journal Journal\]逻辑组名称，逻辑卷的路径或设备的路径
* \[–bluestore\]使用bluestore对象库（默认）
* \[–block.wal\] bluestore block.wal逻辑卷或分区的路径
* \[–block.db\] bluestore block.db逻辑卷或分区的路径
* \[–filestore\]使用文件存储对象库
* \[–dmcrypt\]为基础OSD设备启用加密
* \[–osd-id OSD\_ID\]重用现有的OSD ID
* \[–osd-fsid OSD\_FSID\]重用现有的OSD fsid
* \[–crush-device-class\]定义一个CRUSH设备类以将OSD分配给

必选参数：

* --data

  逻辑组名称或逻辑卷的路径

为了对OSD进行加密，`--dmcrypt`必须在准备时添加标志（`create`子命令也支持该标志）。

**create** 将两步过程包装起来，将一个新的osd（`prepare`首先调用，然后调用`activate`）置为一个。首选`prepare`然后再`activate`逐步将新的OSD引入群集的原因是，避免大量数据被重新平衡。

单次呼叫流程可以准确地统一做什么`prepare`和`activate`做什么，并且方便一次完成所有操作。标志和常规用法与`prepare`and `activate`子命令相同。

**触发器** 此子命令并不旨在直接使用，它由systemd使用，以便它通过解析来自systemd的输入，检测与OSD关联的UUID和ID 来代理输入。`ceph-volume lvm activate`

用法：

```text
ceph-volume lvm trigger <SYSTEMD-DATA>
```

系统化的“数据”应采用以下格式：

```text
<OSD ID>-<OSD UUID>
```

与OSD相关联的lvs必须事先准备好，以便所有必需的标签和元数据都存在。

位置参数：

* &lt;SYSTEMD\_DATA&gt;来自包含OSD的ID和UUID的systemd单元的数据。

**list** 列出与Ceph关联的设备或逻辑卷。确定关联是否设备具有与OSD相关的信息。通过查询LVM的元数据并将其与设备关联，可以对此进行验证。

与OSD相关联的lvs必须事先通过ceph-volume进行准备，以便存在所有需要的标签和元数据。

用法：

```text
ceph-volume lvm list
```

列出特定设备，报告有关该设备的所有元数据：

```text
ceph-volume lvm list /dev/sda1
```

列出逻辑卷及其所有元数据（vg是卷组，lv逻辑卷名）：

```text
ceph-volume lvm list {vg/lv}
```

位置参数：

* &lt;DEVICE&gt; `vg/lv`逻辑卷 `/path/to/sda1`或`/path/to/sda`常规设备的形式。

**zap切换** 给定的逻辑卷或分区。如果指定了逻辑卷的路径，则必须采用vg / lv的格式。给定lv或分区上存在的任何文件系统都将被删除，所有数据将被清除。

但是，lv或分区将保持不变。

用法，对于逻辑卷：

```text
ceph-volume lvm zap {vg/lv}
```

用法，对于逻辑分区：

```text
ceph-volume lvm zap /dev/sdc1
```

要完全移除设备，请使用`--destroy`标志（适用于所有设备类型）：

```text
ceph-volume lvm zap --destroy /dev/sdc1
```

通过指定OSD ID和/或OSD FSID可以删除多个设备：

```text
ceph-volume lvm zap --destroy --osd-id 1
ceph-volume lvm zap --destroy --osd-id 1 --osd-fsid C9605912-8395-4D76-AFC0-7DFDAC315D59
```

位置参数：

* &lt;DEVICE&gt; `vg/lv`逻辑卷 `/path/to/sda1`或`/path/to/sda`常规设备的形式。

#### 简单

扫描可能由ceph-disk创建或手动创建的旧版OSD目录或数据设备。

子命令：

**activate** 启用一个可保留OSD ID及其UUID（`fsid`在Ceph CLI工具中也称为UUID）的systemd单元 ，以便在启动时它可以了解启用了哪些OSD以及需要安装哪些OSD，同时读取先前创建并保留在其中的信息。`/etc/ceph/osd/`JSON格式。

用法：

```text
ceph-volume simple activate --bluestore <osd id> <osd fsid>
```

可选参数：

* \[-h，–help\]显示帮助消息并退出
* \[–bluestore\] bluestore对象库（默认）
* \[–filestore\]文件存储对象存储

注意：它需要一个具有以下格式的匹配JSON文件：

```text
/etc/ceph/osd/<osd id>-<osd fsid>.json
```

**扫描** 在运行的OSD或数据设备上扫描OSD以获得元数据，该元数据以后可用于通过ceph-volume激活和管理OSD。scan方法将创建一个JSON文件，其中包含必需的信息以及OSD目录中的所有内容。

可以选择将JSON blob发送到stdout进行进一步检查。

在所有运行的OSD上的用法：

```text
ceph-voume simple scan
```

在数据设备上的用法：

```text
ceph-volume simple scan <data device>
```

运行OSD目录：

```text
ceph-volume simple scan <path to osd dir>
```

可选参数：

* \[-h，–help\]显示帮助消息并退出
* \[–stdout\]将JSON Blob发送到stdout
* \[–force\]如果JSON文件位于目标位置，则将其覆盖

可选的位置参数：

* &lt;DATA DEVICE或OSD DIR&gt;实际数据分区或运行OSD的路径

**触发器** 此子命令并不旨在直接使用，它由systemd使用，以便它通过解析来自systemd的输入，检测与OSD关联的UUID和ID 来代理输入。`ceph-volume simple activate`

用法：

```text
ceph-volume simple trigger <SYSTEMD-DATA>
```

系统化的“数据”应采用以下格式：

```text
<OSD ID>-<OSD UUID>
```

与OSD相关联的JSON文件需要事先通过扫描（或手动）进行保存，以便可以使用所有需要的元数据。

位置参数：

* &lt;SYSTEMD\_DATA&gt;来自包含OSD的ID和UUID的systemd单元的数据。

### 可用性

**ceph-volume**是Ceph的一部分，Ceph是一个可大规模扩展的开源分布式存储系统。有关更多信息，请参考[http://docs.ceph.com/](http://docs.ceph.com/)上的文档。

### 另见

[ceph-osd](https://docs.ceph.com/docs/nautilus/man/8/ceph-osd/)（8），

