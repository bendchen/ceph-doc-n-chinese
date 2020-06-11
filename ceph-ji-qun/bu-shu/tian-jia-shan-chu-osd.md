# 添加/删除OSD

## 添加/删除OSD

与添加和删除其他Ceph守护程序相比，向群集中添加和删除Ceph OSD守护程序可能需要更多步骤。Ceph OSD守护程序将数据写入磁盘和日志。因此，您需要为OSD提供磁盘和日志分区的路径（即，这是最常见的配置，但是您可以根据自己的需要配置系统）。

在Ceph v0.60和更高版本中，Ceph支持`dm-crypt`磁盘加密。您可以`--dmcrypt`在准备OSD时指定参数，以告知 `ceph-deploy`您要使用加密。您也可以指定 `--dmcrypt-key-dir`参数来指定`dm-crypt` 加密密钥的位置。

在构建大型集群之前，应先测试各种驱动器配置以评估其吞吐量。有关其他详细信息，请参见[数据存储](https://docs.ceph.com/docs/nautilus/start/hardware-recommendations#data-storage)。

### 列表磁盘

要列出节点上的磁盘，请执行以下命令：

```text
ceph-deploy disk list {node-name [node-name]...}
```

### ZAP磁盘

要更换磁盘（删除其分区表）以准备与Ceph一起使用，请执行以下操作：

```text
ceph-deploy disk zap {osd-server-name}:{disk-name}
ceph-deploy disk zap osdserver1:sdb
```

重要 

这将删除所有数据。

### 创建屏上显示

创建集群，安装Ceph软件包并收集密钥后，您可以创建OSD并将其部署到OSD节点。如果在准备用作OSD之前需要识别磁盘或对其进行换片，请参阅[列出磁盘](https://docs.ceph.com/docs/nautilus/rados/deployment/ceph-deploy-osd/#list-disks)和[换片磁盘](https://docs.ceph.com/docs/nautilus/rados/deployment/ceph-deploy-osd/#zap-disks)。

```text
ceph-deploy osd create --data {data-disk} {node-name}
```

例如：

```text
ceph-deploy osd create --data /dev/ssd osd-server1
```

对于bluestore（默认设置），该示例假定一个磁盘专用于一个Ceph OSD守护程序。还支持文件存储，在这种情况下`--journal`，除了标志之外，还`--filestore`需要使用一个标志来定义远程主机上的日记设备。

注意 

在单个节点上运行多个Ceph OSD守护程序并与每个OSD守护程序共享分区日志时​​，出于CRUSH的目的，应将整个节点视为最小故障域，因为如果SSD驱动器发生故障，则该日志的所有Ceph OSD守护程序也会失败。

### 清单屏上显示

要列出部署在节点上的OSD，请执行以下命令：

```text
ceph-deploy osd list {node-name}
```

### 销毁的OSD 

注意 

快来了。有关手动步骤，请参阅[卸下OSD](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds#removing-osds-manual)。

