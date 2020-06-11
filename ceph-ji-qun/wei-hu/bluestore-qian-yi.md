# BLUESTORE迁移

## BLUESTORE迁移

每个OSD都可以运行BlueStore或FileStore，并且单个Ceph集群可以包含两者的混合。先前已部署FileStore的用户可能希望过渡到BlueStore，以利用改进的性能和健壮性。有几种实现这种过渡的策略。

单个OSD不能单独进行原地转换，但是：BlueStore和FileStore根本不同，以致于无法实用。“转换”将依靠群集的正常复制和修复支持，或者依靠将OSD内容从旧的（FileStore）设备复制到新的（BlueStore）设备的工具和策略。

### 部署新的OSD与BLUESTORE 

可以使用BlueStore部署任何新的OSD（例如，在扩展集群时）。这是默认行为，因此不需要进行特定更改。

同样，更换故障驱动器后重新配置的任何OSD都可以使用BlueStore。

### 将现有的OSD 

#### 标记并替换

最简单的方法是依次标记每个设备，等待数据在群集中复制，重新配置OSD，然后再次将其标记回。它简单且易于自动化。但是，它需要的数据迁移量超出了必要，因此不是最佳选择。

1. 确定要替换的FileStore OSD：

   ```text
   ID=<osd-id-number>
   DEVICE=<disk-device>
   ```

   您可以使用以下命令判断给定的OSD是FileStore还是BlueStore：

   ```text
   ceph osd metadata $ID | grep osd_objectstore
   ```

   您可以使用以下命令获取文件存储与bluestore的当前计数：

   ```text
   ceph osd count-metadata osd_objectstore
   ```

2. 将文件存储OSD标记为：

   ```text
   ceph osd out $ID
   ```

3. 等待数据从有问题的OSD迁移：

   ```text
   while ! ceph osd safe-to-destroy $ID ; do sleep 60 ; done
   ```

4. 停止OSD：

   ```text
   systemctl kill ceph-osd@$ID
   ```

5. 记下此OSD使用的设备：

   ```text
   mount | grep /var/lib/ceph/osd/ceph-$ID
   ```

6. 卸载OSD：

   ```text
   umount /var/lib/ceph/osd/ceph-$ID
   ```

7. 销毁OSD数据。请务必_小心，_因为这会破坏设备的内容；在继续操作之前，请确保不需要设备上的数据（即，群集运行状况良好）。

   ```text
   ceph-volume lvm zap $DEVICE
   ```

8. 告诉集群OSD已被破坏（并且可以使用相同的ID重新配置新的OSD）：

   ```text
   ceph osd destroy $ID --yes-i-really-mean-it
   ```

9. 用相同的OSD ID在其位置重新配置BlueStore OSD。这要求您确实根据上面看到的内容确定要擦除的设备。小心！

   ```text
   ceph-volume lvm create --bluestore --data $DEVICE --osd-id $ID
   ```

10. 重复。

您可以允许替换OSD的重新填充与下一个OSD的排空同时进行，或者对多个OSD并行执行相同的过程，只要确保在销毁集群之前，集群是完全干净的（所有数据具有所有副本）即可。任何OSD。否则，将减少数据的冗余，并增加（甚至可能导致）数据丢失的风险。

优点：

* 简单。
* 可以逐个设备完成。
* 不需要备用设备或主机。

缺点：

* 数据通过网络复制了两次：一次复制到集群中的某个其他OSD（以保持所需的副本数），然后再次返回到重新配置的BlueStore OSD。

#### 整个主机更换

如果群集中有一个备用主机，或者有足够的可用空间来疏散整个主机以用作备用主机，则可以在每个主机的基础上使用存储的每个数据副本进行转换仅迁移一次。

首先，您需要有一个没有数据的空主机。有两种方法可以执行此操作：从尚未包含在集群中的新的空主机开始，或者从集群中现有主机上卸载数据。

**使用新的，空的主机**

理想情况下，主机应与要转换的其他主机具有大致相同的容量（尽管这并不严格）。

```text
NEWHOST=<empty-host-name>
```

将主机添加到CRUSH层次结构，但不要将其附加到根目录：

```text
ceph osd crush add-bucket $NEWHOST host
```

确保已安装ceph软件包。

**使用现有的主机**

如果要使用已经是集群一部分的现有主机，并且该主机上有足够的可用空间，以便可以迁移其所有数据，则可以执行以下操作：

```text
OLDHOST=<existing-cluster-host-to-offload>
ceph osd crush unlink $OLDHOST default
```

其中“默认”是CRUSH地图中的直接祖先。（对于具有未修改配置的较小集群，通常将为“默认”，但也可能是机架名称。）现在，您应该在OSD树输出的顶部看到没有父节点的主机：

```text
$ bin/ceph osd tree
ID CLASS WEIGHT  TYPE NAME     STATUS REWEIGHT PRI-AFF
-5             0 host oldhost
10   ssd 1.00000     osd.10        up  1.00000 1.00000
11   ssd 1.00000     osd.11        up  1.00000 1.00000
12   ssd 1.00000     osd.12        up  1.00000 1.00000
-1       3.00000 root default
-2       3.00000     host foo
 0   ssd 1.00000         osd.0     up  1.00000 1.00000
 1   ssd 1.00000         osd.1     up  1.00000 1.00000
 2   ssd 1.00000         osd.2     up  1.00000 1.00000
...
```

如果一切正常，请直接跳至下面的“等待数据迁移完成”步骤，然后从那里继续进行操作以清理旧的OSD。

**迁移过程**

如果您使用的是新主机，请从步骤1开始。对于现有主机，请跳至下面的步骤5。

1. 为所有设备配置新的BlueStore OSD：

   ```text
   ceph-volume lvm create --bluestore --data /dev/$DEVICE
   ```

2. 验证OSD通过以下方式加入集群：

   ```text
   ceph osd tree
   ```

   您应该看到新主机`$NEWHOST`与它下面的所有的OSD的，但主机应该_不_被嵌套任何其他节点下的层次结构（像）。例如，如果是空主机，则可能会看到以下内容：`root defaultnewhost`

   ```text
   $ bin/ceph osd tree
   ID CLASS WEIGHT  TYPE NAME     STATUS REWEIGHT PRI-AFF
   -5             0 host newhost
   10   ssd 1.00000     osd.10        up  1.00000 1.00000
   11   ssd 1.00000     osd.11        up  1.00000 1.00000
   12   ssd 1.00000     osd.12        up  1.00000 1.00000
   -1       3.00000 root default
   -2       3.00000     host oldhost1
    0   ssd 1.00000         osd.0     up  1.00000 1.00000
    1   ssd 1.00000         osd.1     up  1.00000 1.00000
    2   ssd 1.00000         osd.2     up  1.00000 1.00000
   ...
   ```

3. 确定要转换的第一个目标主机

   ```text
   OLDHOST=<existing-cluster-host-to-convert>
   ```

4. 将新主机交换到群集中旧主机的位置：

   ```text
   ceph osd crush swap-bucket $NEWHOST $OLDHOST
   ```

   此时，所有数据`$OLDHOST`将开始迁移到上的OSD `$NEWHOST`。如果新旧主机的总容量存在差异，您可能还会看到一些数据迁移到群集中的其他节点或从群集中的其他节点迁移，但是只要这些主机的大小相同，这将是相对少量的数据。

5. 等待数据迁移完成：

   ```text
   while ! ceph osd safe-to-destroy $(ceph osd ls-tree $OLDHOST); do sleep 60 ; done
   ```

6. 停止所有空的旧OSD `$OLDHOST`：

   ```text
   ssh $OLDHOST
   systemctl kill ceph-osd.target
   umount /var/lib/ceph/osd/ceph-*
   ```

7. 销毁并清除旧的OSD：

   ```text
   for osd in `ceph osd ls-tree $OLDHOST`; do
       ceph osd purge $osd --yes-i-really-mean-it
   done
   ```

8. 擦拭旧的OSD设备。这要求您确定要手动擦除哪些设备（请小心！）。对于每个设备：

   ```text
   ceph-volume lvm zap $DEVICE
   ```

9. 将现在为空的主机用作新主机，然后重复：

   ```text
   NEWHOST=$OLDHOST
   ```

优点：

* 数据只能通过网络复制一次。
* 一次转换整个主机的OSD。
* 可以并行转换为一次转换多个主机。
* 每个主机上都不需要备用设备。

缺点：

* 需要备用主机。
* 整个主机的OSD值将同时迁移数据。这很可能会影响整个群集的性能。
* 所有迁移的数据仍会在网络上进行一整跳。

#### 每OSD设备副本

可以使用的`copy`功能转换单个逻辑OSD `ceph-objectstore-tool`。这要求主机具有一个或多个空闲设备来供应新的空BlueStore OSD。例如，如果群集中的每个主机都有12个OSD，则需要第13个可用设备，以便可以依次转换每个OSD，然后再收回旧设备以转换下一个OSD。

注意事项：

* 此策略要求准备一个空白的BlueStore OSD，而不分配新的OSD ID，该`ceph-volume` 工具不支持此功能。更重要的是，_dmcrypt_的设置与OSD身份紧密相关，这意味着该方法不适用于加密的OSD。
* 设备必须手动分区。
* 工具未实现！
* 没有记录！

优点：

* 转换期间，很少或没有数据在网络上迁移。

缺点：

* 工具尚未完全实现。
* 流程未记录。
* 每个主机必须具有备用或空设备。
* OSD在转换过程中处于脱机状态，这意味着新的写入操作将仅写入OSD的一部分。这会增加由于后续故障而导致数据丢失的风险。（但是，如果在转换完成之前出现故障，则可以启动原始FileStore OSD来提供对其原始数据的访问。）

