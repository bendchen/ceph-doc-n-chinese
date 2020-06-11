# 添加/删除OSD

## 添加/删除OSD

群集启动并运行后，可以在运行时添加OSD或从群集中删除OSD。

### 添加的OSD 

当您要扩展群集时，可以在运行时添加OSD。对于Ceph，OSD通常`ceph-osd`是主机中一个存储驱动器的一个Ceph 守护程序。如果主机有多个存储驱动器，则可`ceph-osd`以为每个驱动器映射一个 守护程序。

通常，最好检查群集的容量以查看是否达到其容量的上限。当群集达到其比例时，应添加一个或多个OSD以扩展群集的容量。`near full`

警告 

在添加OSD之前，请勿让您的群集到达其群集。群集达到其比率后发生的OSD故障可能会导致群集超出其限制 。`full rationear fullfull ratio`

#### 部署您的硬件

如果要在添加新OSD时添加新主机，请参阅[硬件建议，](https://docs.ceph.com/docs/nautilus/start/hardware-recommendations)以获取有关OSD硬件最低建议的详细信息。要将OSD主机添加到群集中，请首先确保已安装最新版本的Linux，并且已为存储驱动器进行了一些初步准备。有关详细信息，请参见[文件系统建议](https://docs.ceph.com/docs/nautilus/rados/configuration/filesystem-recommendations)。

将OSD主机添加到群集中的机架，将其连接到网络并确保其具有网络连接性。有关详细信息，请参见[网络配置参考](https://docs.ceph.com/docs/nautilus/rados/configuration/network-config-ref)。

#### 安装所需的软件

对于手动部署的群集，必须手动安装Ceph软件包。有关详细信息，请参见[安装Ceph（手册）](https://docs.ceph.com/docs/nautilus/install)。您应该为具有无密码身份验证和root权限的用户配置SSH。

#### 添加OSD（手动）

此过程将设置`ceph-osd`守护程序，将其配置为使用一个驱动器，并配置集群以将数据分发到OSD。如果主机有多个驱动器，则可以通过重复此过程为每个驱动器添加OSD。

要添加OSD，请为其创建数据目录，将驱动器安装到该目录，将OSD添加到群集，然后将其添加到CRUSH映射。

将OSD添加到CRUSH映射时，请考虑赋予新OSD的权重。硬盘驱动器容量每年增长40％，因此，较新的OSD主机可能具有比群集中较旧的主机更大的硬盘驱动器（即，它们的重量可能更大）。

小费 

Ceph倾向于跨池使用统一的硬件。如果要添加大小不同的驱动器，则可以调整其重量。但是，为了获得最佳性能，请考虑使用具有相同类型/大小的驱动器的CRUSH层次结构。

1. 创建OSD。如果未给出UUID，则它将在OSD启动时自动设置。以下命令将输出OSD号，您将在后续步骤中使用该OSD号。

   ```text
   ceph osd create [{uuid} [{id}]]
   ```

   如果给出了可选参数{id}，它将用作OSD ID。注意，在这种情况下，如果该号码已被使用，则该命令可能会失败。

   警告 

   通常，不建议显式指定{id}。ID作为数组分配，跳过条目会占用一些额外的内存。如果存在很大的差距和/或集群很大，这可能变得很重要。如果未指定{id}，则使用最小的可用值。

2. 在新的OSD上创建默认目录。

   ```text
   ssh {new-osd-host}
   sudo mkdir /var/lib/ceph/osd/ceph-{osd-number}
   ```

3. 如果OSD用于OS驱动器以外的驱动器，请准备将其与Ceph一起使用，并将其安装到刚创建的目录中：

   ```text
   ssh {new-osd-host}
   sudo mkfs -t {fstype} /dev/{drive}
   sudo mount -o user_xattr /dev/{hdd} /var/lib/ceph/osd/ceph-{osd-number}
   ```

4. 初始化OSD数据目录。

   ```text
   ssh {new-osd-host}
   ceph-osd -i {osd-num} --mkfs --mkkey
   ```

   该目录必须为空才能运行`ceph-osd`。

5. 注册OSD验证密钥。路径中`ceph`for 的值`ceph-{osd-num}`是`$cluster-$id`。如果您的集群名称不同于`ceph`，请改用您的集群名称。：

   ```text
   ceph auth add osd.{osd-num} osd 'allow *' mon 'allow rwx' -i /var/lib/ceph/osd/ceph-{osd-num}/keyring
   ```

6. 将OSD添加到CRUSH映射中，以便OSD可以开始接收数据。该 命令允许您将OSD添加到CRUSH层次结构中的任何位置。如果您至少指定一个存储桶，该命令会将OSD放入您指定的最特定的存储桶中，_并将_该存储桶移动到您指定的任何其他存储桶下方。**重要：**如果仅指定根存储桶，该命令会将OSD直接附加到根，但是CRUSH规则要求OSD位于主机内部。`ceph osd crush add`

   执行以下命令：

   ```text
   ceph osd crush add {id-or-name} {weight}  [{bucket-type}={bucket-name} ...]
   ```

   您还可以反编译CRUSH映射，将OSD添加到设备列表中，将主机添加为存储桶（如果尚未在CRUSH映射中添加），将设备添加为主机中的一项，为其分配权重，然后重新编译并设置它。有关详细信息，请参见 [添加/移动OSD](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map#addosd)。

#### 更换

当磁盘出现故障时，或者如果管理员想用新的后端重新配置OSD（例如，从FileStore切换到BlueStore），则需要更换OSD。与[删除OSD](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#removing-the-osd)不同，在销毁OSD进行替换后，必须保持替换后的OSD的ID和CRUSH映射条目不变。

1. 首先销毁OSD：

   ```text
   ceph osd destroy {id} --yes-i-really-mean-it
   ```

2. 如果以前将该磁盘用于其他目的，请为新OSD更换磁盘。不需要新磁盘：

   ```text
   ceph-volume lvm zap /dev/sdX
   ```

3. 使用以前销毁的OSD ID准备要更换的磁盘：

   ```text
   ceph-volume lvm  prepare --osd-id {id} --data /dev/sdX
   ```

4. 并激活OSD：

   ```text
   ceph-volume lvm activate {id} {fsid}
   ```

或者，除了准备和激活外，还可以在一个呼叫中重新创建设备，例如：

```text
ceph-volume lvm create --osd-id {id} --data /dev/sdX
```

#### 启动

将OSD添加到Ceph之后，该OSD已在您的配置中。但是，它尚未运行。OSD是`down`和`in`。您必须先启动新的OSD，然后才能开始接收数据。您可以从管理主机使用，也可以 从其主机启动OSD。`service ceph`

对于Ubuntu Trusty，请使用Upstart。

```text
sudo start ceph-osd id={osd-num}
```

对于所有其他发行版，请使用systemd。

```text
sudo systemctl start ceph-osd@{osd-num}
```

一旦启动OSD，它就是`up`和`in`。

#### 观察数据迁移

将新的OSD添加到CRUSH映射后，Ceph将通过将展示位置组迁移到新的OSD开始重新平衡服务器。您可以使用[ceph](https://docs.ceph.com/docs/nautilus/rados/operations/monitoring)工具观察此过程。

```text
ceph -w
```

您应该看到展示位置组的状态从`active+clean`变为，最后在迁移完成时变为 。（Ctrl-c退出。）`active, some degraded objectsactive+clean`

### 删除OSD（手动）

当您要减小群集的大小或更换硬件时，可以在运行时删除OSD。对于Ceph，OSD通常`ceph-osd` 是主机中一个存储驱动器的一个Ceph 守护程序。如果主机有多个存储驱动器，则可能需要`ceph-osd`为每个驱动器删除一个守护程序。通常，最好检查群集的容量以查看是否达到其容量的上限。确保在删除OSD时，群集不处于集群状态。`near full`

警告 

删除OSD时，不要让您的群集到达其群集。删除OSD可能导致群集达到或超过其群集。`full ratiofull ratio`

#### 将OSD移出群集

删除OSD之前，通常为`up`和`in`。您需要将其从群集中删除，以便Ceph可以开始重新平衡并将其数据复制到其他OSD。

```text
ceph osd out {osd-num}
```

#### 观察数据迁移

在获取`out`了群集的OSD 之后，Ceph将通过从您删除的OSD中迁移放置组来开始重新平衡群集。您可以使用[ceph](https://docs.ceph.com/docs/nautilus/rados/operations/monitoring)工具观察此过程。

```text
ceph -w
```

您应该看到展示位置组的状态从`active+clean`变为，最后在迁移完成时变为 。（Ctrl-c退出。）`active, some degraded objectsactive+clean`

注意 

有时，通常在主机很少的“小型”群集中（例如，小型测试群集），采用`out`OSD 的事实可能会导致CRUSH极端情况，其中某些PG仍停留在该 `active+remapped`状态。如果是这种情况，则应将OSD标记为`in`：

> `ceph osd in {osd-num}`

返回初始状态，然后代替标记`out` OSD，通过以下操作将其权重设置为0：

> `ceph osd crush reweight osd.{osd-num} 0`

之后，您可以观察到应该完成的数据迁移。标记`out`OSD和将其重新加权为0 之间的区别在于，在第一种情况下，包含OSD的铲斗的重量不会改变，而在第二种情况下，铲斗的重量会更新（并且OSD重量会减少）。在“小型”群集的情况下，有时会倾向于使用reweight命令。

#### 停止

从群集中取出OSD之后，它可能仍在运行。即，OSD可以是`up`和`out`。从配置中删除OSD之前，必须先停止它。

```text
ssh {osd-host}
sudo systemctl stop ceph-osd@{osd-num}
```

一旦停止OSD，它就是`down`。

#### 删除

此过程从群集映射中删除OSD，删除其身份验证密钥，从OSD映射中删除OSD，然后从`ceph.conf`文件中删除OSD 。如果主机有多个驱动器，则可能需要通过重复此过程为每个驱动器删除OSD。

1. 让群集首先忘记OSD。此步骤从CRUSH映射中删除OSD，并删除其身份验证密钥。并且它也从OSD映射中删除。请注意，在Luminous中引入了[purge子命令](https://docs.ceph.com/docs/nautilus/man/8/ceph/#ceph-admin-osd)，对于较旧的版本，请参见下文

   ```text
   ceph osd purge {id} --yes-i-really-mean-it
   ```

2. 导航到保留群集`ceph.conf`文件主副本的主机 。

   ```text
   ssh {admin-host}
   cd /etc/ceph
   vim ceph.conf
   ```

3. 从`ceph.conf`文件中删除OSD条目（如果存在）。

   ```text
   [osd.1]
           host = {hostname}
   ```

4. 从保留群集`ceph.conf`文件主副本的主机上，将更新的文件复制`ceph.conf`到`/etc/ceph`群集中其他主机的目录中。

如果您的Ceph集群比Luminous早，请使用手动执行以下步骤，而不要使用：`ceph osd purge`

1. 从CRUSH映射中删除OSD，使其不再接收数据。您还可以反编译CRUSH映射，从设备列表中删除OSD，将设备作为主机存储桶中的一项删除，或删除主机存储桶（如果它在CRUSH映射中，并且您打算删除主机），重新编译该映射并设置它。有关详细信息，请参阅[卸下OSD](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map#removeosd)。

   ```text
   ceph osd crush remove {name}
   ```

2. 删除OSD身份验证密钥。

   ```text
   ceph auth del osd.{osd-num}
   ```

   路径中`ceph`for 的值`ceph-{osd-num}`是`$cluster-$id`。如果您的群集名称不同于`ceph`，请改用您的群集名称。

3. 卸下OSD。

   ```text
   ceph osd rm {osd-num}
   #for example
   ceph osd rm 1
   ```

[![&#x5546;&#x6807;](https://docs.ceph.com/docs/nautilus/_static/logo.png)](https://docs.ceph.com/docs/nautilus/)

#### [目录](https://docs.ceph.com/docs/nautilus/)

* [Ceph简介](https://docs.ceph.com/docs/nautilus/start/intro/)
* [安装（ceph部署）](https://docs.ceph.com/docs/nautilus/start/)
* [安装手册）](https://docs.ceph.com/docs/nautilus/install/)
* [安装（Kubernetes + Helm）](https://docs.ceph.com/docs/nautilus/start/kube-helm/)
* [Ceph存储集群](https://docs.ceph.com/docs/nautilus/rados/)
  * [组态](https://docs.ceph.com/docs/nautilus/rados/configuration/)
  * [部署方式](https://docs.ceph.com/docs/nautilus/rados/deployment/)
  * [运作方式](https://docs.ceph.com/docs/nautilus/rados/operations/)
    * [操作集群](https://docs.ceph.com/docs/nautilus/rados/operations/operating/)
    * [健康检查](https://docs.ceph.com/docs/nautilus/rados/operations/health-checks/)
    * [监控集群](https://docs.ceph.com/docs/nautilus/rados/operations/monitoring/)
    * [监视OSD和PG](https://docs.ceph.com/docs/nautilus/rados/operations/monitoring-osd-pg/)
    * [用户管理](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/)
    * [修复PG不一致](https://docs.ceph.com/docs/nautilus/rados/operations/pg-repair/)
    * [数据放置概述](https://docs.ceph.com/docs/nautilus/rados/operations/data-placement/)
    * [泳池](https://docs.ceph.com/docs/nautilus/rados/operations/pools/)
    * [清除码](https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code/)
    * [缓存分层](https://docs.ceph.com/docs/nautilus/rados/operations/cache-tiering/)
    * [展示位置组](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups/)
    * [平衡器](https://docs.ceph.com/docs/nautilus/rados/operations/balancer/)
    * [使用pg-upmap](https://docs.ceph.com/docs/nautilus/rados/operations/upmap/)
    * [粉碎地图](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map/)
    * [手动编辑CRUSH地图](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map-edits/)
    * [添加/删除OSD](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#)
      * [添加OSD](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#adding-osds)
        * [部署您的硬件](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#deploy-your-hardware)
        * [安装所需的软件](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#install-the-required-software)
        * [添加OSD（手动）](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#adding-an-osd-manual)
        * [更换OSD](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#replacing-an-osd)
        * [启动OSD](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#starting-the-osd)
        * [观察数据迁移](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#observe-the-data-migration)
      * [删除OSD（手动）](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#removing-osds-manual)
        * [将OSD移出群集](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#take-the-osd-out-of-the-cluster)
        * [观察数据迁移](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#id1)
        * [停止OSD](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#stopping-the-osd)
        * [卸下OSD](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#removing-the-osd)
    * [添加/删除监视器](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-mons/)
    * [设备管理](https://docs.ceph.com/docs/nautilus/rados/operations/devices/)
    * [BlueStore迁移](https://docs.ceph.com/docs/nautilus/rados/operations/bluestore-migration/)
    * [命令参考](https://docs.ceph.com/docs/nautilus/rados/operations/control/)
    * [Ceph社区](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/community/)
    * [监控显示器故障](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon/)
    * [OSD故障排除](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-osd/)
    * [PG故障排除](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-pg/)
    * [记录和调试](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/log-and-debug/)
    * [CPU分析](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/cpu-profiling/)
    * [内存分析](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/memory-profiling/)
  * [手册页](https://docs.ceph.com/docs/nautilus/rados/man/)
  * [故障排除](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/)
  * [蜜蜂](https://docs.ceph.com/docs/nautilus/rados/api/)
* [Ceph文件系统](https://docs.ceph.com/docs/nautilus/cephfs/)
* [Ceph块设备](https://docs.ceph.com/docs/nautilus/rbd/)
* [Ceph对象网关](https://docs.ceph.com/docs/nautilus/radosgw/)
* [Ceph Manager守护程序](https://docs.ceph.com/docs/nautilus/mgr/)
* [Ceph仪表板](https://docs.ceph.com/docs/nautilus/mgr/dashboard/)
* [API文档](https://docs.ceph.com/docs/nautilus/api/)
* [建筑](https://docs.ceph.com/docs/nautilus/architecture/)
* [开发人员指南](https://docs.ceph.com/docs/nautilus/dev/)
* [Ceph内部](https://docs.ceph.com/docs/nautilus/dev/internals/)
* [管治](https://docs.ceph.com/docs/nautilus/governance/)
* [头颅容积](https://docs.ceph.com/docs/nautilus/ceph-volume/)
* [Ceph发布](https://docs.ceph.com/docs/nautilus/releases/)
* [词汇表](https://docs.ceph.com/docs/nautilus/glossary/)
* [指数](https://docs.ceph.com/docs/nautilus/genindex/)

#### 快速搜索 <a id="searchlabel"></a>

* [指数](https://docs.ceph.com/docs/nautilus/genindex/)
* [模块](https://docs.ceph.com/docs/nautilus/py-modindex/) \|
* [下一个](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-mons/) \|
* [上一个](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map-edits/) \|
* [Ceph文档](https://docs.ceph.com/docs/nautilus/) »
* * [Ceph存储集群](https://docs.ceph.com/docs/nautilus/rados/) »
* * [集群操作](https://docs.ceph.com/docs/nautilus/rados/operations/) »

©版权所有2016，Red Hat，Inc和贡献者。根据知识共享署名相同方式共享3.0（CC-

