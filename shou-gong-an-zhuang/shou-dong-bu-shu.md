# 手动部署

## 手动部署

所有Ceph集群都需要至少一个监视器，以及至少与集群中存储的对象副本一样多的OSD。引导初始监视器是部署Ceph存储群集的第一步。监控器部署还为整个群集设置了重要条件，例如池的副本数，每个OSD的放置组数，心跳间隔，是否需要身份验证等。这些值大多数都是默认设置，因此在设置集群以进行生产时了解它们很有用。

按照同样的配置[安装（快速）](https://docs.ceph.com/docs/nautilus/start)，我们将建立一个集群`node1`作为监控节点，`node2`并`node3`为OSD节点。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-b67d58275cae03a5573d36907b437e36df685600.png)

### 监控启动

引导监视器（理论上是Ceph存储集群）需要做很多事情：

* **唯一标识符：**该`fsid`是为集群的唯一标识符，并从天代表文件系统ID当Ceph的存储集群是主要的Ceph的文件系统。Ceph现在也支持本机接口，块设备和对象存储网关接口，`fsid`这有点用词不当。
* **群集名称：** Ceph群集具有群集名称，它是一个简单的字符串，没有空格。默认群集名称为`ceph`，但是您可以指定其他群集名称。当您使用多个群集并且需要清楚地了解正在使用哪个群集时，覆盖默认群集名称特别有用。

  例如，当您在[多站点配置中](https://docs.ceph.com/docs/nautilus/radosgw/multisite/#multisite)运行多个集群时，集群名称（例如`us-west`，`us-east`）标识当前CLI会话的集群。**注意：**为了识别命令行界面上群集名称，指定与群集名称Ceph的配置文件（例如，`ceph.conf`，`us-west.conf`，`us-east.conf`，等等）。另请参阅CLI用法（）。`ceph --cluster {cluster-name}`

* **监视器名称：**集群中的每个监视器实例都有一个唯一的名称。通常，Ceph Monitor名称是主机名（我们建议每个主机一个Ceph Monitor，并且不要将Ceph OSD守护程序与Ceph Monitors混合使用）。您可以使用检索简短的主机名。`hostname -s`
* **监视器映射：**启动初始监视器需要您生成监视器映射。监视器映射需要`fsid`，群集名称（或使用默认名称），以及至少一个主机名及其IP地址。
* **监视器密钥环**：监视器之间通过秘密密钥进行通信。您必须生成一个带有监视器机密的密钥环，并在引导初始监视器时提供它。
* **管理员密钥环**：要使用`ceph`CLI工具，您必须有一个`client.admin`用户。因此，您必须生成admin用户和密钥环，并且还必须将`client.admin`用户添加到监视器密钥环。

前述要求并不意味着创建Ceph配置文件。但是，作为最佳实践，我们建议创建一个Ceph配置文件，并使用`fsid`，和 设置进行填充。`mon initial membersmon host`

您还可以在运行时获取并设置所有监视器设置。但是，Ceph配置文件只能包含那些覆盖默认值的设置。将设置添加到Ceph配置文件时，这些设置将覆盖默认设置。通过在Ceph配置文件中维护这些设置，可以更轻松地维护集群。

步骤如下：

1. 登录到初始监视节点：

   ```text
   ssh {hostname}
   ```

   例如：

   ```text
   ssh node1
   ```

2. 确保您具有Ceph配置文件的目录。默认情况下，Ceph使用`/etc/ceph`。安装时`ceph`，安装程序将`/etc/ceph`自动创建目录。

   ```text
   ls /etc/ceph
   ```

   **注意：**清除集群时，部署工具可能会删除此目录（例如，）。`ceph-deploy purgedata {node-name}ceph-deploy purge {node-name}`

3. 创建一个Ceph配置文件。默认情况下，Ceph使用 `ceph.conf`，其中`ceph`反映了集群名称。

   ```text
   sudo vim /etc/ceph/ceph.conf
   ```

4. `fsid`为您的集群生成一个唯一的ID（即）。

   ```text
   uuidgen
   ```

5. 将唯一ID添加到您的Ceph配置文件中。

   ```text
   fsid = {UUID}
   ```

   例如：

   ```text
   fsid = a7f64266-0894-4f1e-a635-d0aeaca0e993
   ```

6. 将初始监视器添加到您的Ceph配置文件中。

   ```text
   mon initial members = {hostname}[,{hostname}]
   ```

   例如：

   ```text
   mon initial members = node1
   ```

7. 将初始监视器的IP地址添加到您的Ceph配置文件中并保存该文件。

   ```text
   mon host = {ip-address}[,{ip-address}]
   ```

   例如：

   ```text
   mon host = 192.168.0.1
   ```

   **注意：**您可以使用IPv6地址而不是IPv4地址，但是必须设置为。有关[网络配置](https://docs.ceph.com/docs/nautilus/rados/configuration/network-config-ref)的详细信息，请参见[网络配置参考](https://docs.ceph.com/docs/nautilus/rados/configuration/network-config-ref)。`ms bind ipv6true`

8. 为您的集群创建一个密钥环，并生成一个监视器秘密密钥。

   ```text
   ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'
   ```

9. 生成管理员密钥环，生成`client.admin`用户并将用户添加到密钥环。

   ```text
   sudo ceph-authtool --create-keyring /etc/ceph/ceph.client.admin.keyring --gen-key -n client.admin --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *' --cap mgr 'allow *'
   ```

10. 生成bootstrap-osd密钥环，生成`client.bootstrap-osd`用户并将用户添加到密钥环。

    ```text
    sudo ceph-authtool --create-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring --gen-key -n client.bootstrap-osd --cap mon 'profile bootstrap-osd'
    ```

11. 将生成的密钥添加到中`ceph.mon.keyring`。

    ```text
    sudo ceph-authtool /tmp/ceph.mon.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring
    sudo ceph-authtool /tmp/ceph.mon.keyring --import-keyring /var/lib/ceph/bootstrap-osd/ceph.keyring
    ```

12. 使用主机名，主机IP地址和FSID生成监视器映射。另存为`/tmp/monmap`：

    ```text
    monmaptool --create --add {hostname} {ip-address} --fsid {uuid} /tmp/monmap
    ```

    例如：

    ```text
    monmaptool --create --add node1 192.168.0.1 --fsid a7f64266-0894-4f1e-a635-d0aeaca0e993 /tmp/monmap
    ```

13. 在监视器主机上创建一个或多个默认数据目录。

    ```text
    sudo mkdir /var/lib/ceph/mon/{cluster-name}-{hostname}
    ```

    例如：

    ```text
    sudo -u ceph mkdir /var/lib/ceph/mon/ceph-node1
    ```

    有关详细信息，请参见[Monitor Config Reference-Data](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-config-ref#data)。

14. 用监视器映射和密钥环填充监视器守护程序。

    ```text
    sudo -u ceph ceph-mon [--cluster {cluster-name}] --mkfs -i {hostname} --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring
    ```

    例如：

    ```text
    sudo -u ceph ceph-mon --mkfs -i node1 --monmap /tmp/monmap --keyring /tmp/ceph.mon.keyring
    ```

15. 考虑Ceph配置文件的设置。常用设置包括：

    ```text
    [global]
    fsid = {cluster-id}
    mon initial members = {hostname}[, {hostname}]
    mon host = {ip-address}[, {ip-address}]
    public network = {network}[, {network}]
    cluster network = {network}[, {network}]
    auth cluster required = cephx
    auth service required = cephx
    auth client required = cephx
    osd journal size = {n}
    osd pool default size = {n}  # Write an object n times.
    osd pool default min size = {n} # Allow writing n copies in a degraded state.
    osd pool default pg num = {n}
    osd pool default pgp num = {n}
    osd crush chooseleaf type = {n}
    ```

    在前面的示例中，`[global]`配置部分可能如下所示：

    ```text
    [global]
    fsid = a7f64266-0894-4f1e-a635-d0aeaca0e993
    mon initial members = node1
    mon host = 192.168.0.1
    public network = 192.168.0.0/24
    auth cluster required = cephx
    auth service required = cephx
    auth client required = cephx
    osd journal size = 1024
    osd pool default size = 3
    osd pool default min size = 2
    osd pool default pg num = 333
    osd pool default pgp num = 333
    osd crush chooseleaf type = 1
    ```

16. 启动监视器。

    对于大多数发行版，服务现在通过systemd启动：

    ```text
    sudo systemctl start ceph-mon@node1
    ```

    对于较旧的Debian / CentOS / RHEL，请使用sysvinit：

    ```text
    sudo /etc/init.d/ceph start mon.node1
    ```

17. 验证监视器是否正在运行。

    ```text
    ceph -s
    ```

    您应该看到输出，表明您启动的监视器已启动并且正在运行，并且应该看到运行状况错误，指示放置组处于非活动状态。它看起来应该像这样：

    ```text
    cluster:
      id:     a7f64266-0894-4f1e-a635-d0aeaca0e993
      health: HEALTH_OK

    services:
      mon: 1 daemons, quorum node1
      mgr: node1(active)
      osd: 0 osds: 0 up, 0 in

    data:
      pools:   0 pools, 0 pgs
      objects: 0 objects, 0 bytes
      usage:   0 kB used, 0 kB / 0 kB avail
      pgs:
    ```

    **注意：**添加OSD并启动它们后，放置组运行状况错误应消失。有关详细信息，请参见[添加OSD](https://docs.ceph.com/docs/nautilus/install/manual-deployment/#adding-osds)。

### 管理器守护程序配置

在运行ceph-mon守护程序的每个节点上，还应该设置一个ceph-mgr守护程序。

请参阅[ceph-mgr管理员指南](https://docs.ceph.com/docs/nautilus/mgr/administrator/#mgr-administrator-guide)

### 添加的OSD 

初始监视器运行后，应添加OSD。您必须有足够的OSD来处理对象的副本数（例如，至少需要两个OSD），才能使群集达到某种状态。自举监视器后，您的群集具有默认的CRUSH映射；但是，CRUSH映射没有任何映射到Ceph节点的Ceph OSD守护程序。`active + cleanosd pool default size = 2`

#### 短表

Ceph提供了该`ceph-volume`实用程序，可以准备逻辑卷，磁盘或分区以供Ceph使用。该`ceph-volume`实用程序通过增加索引来创建OSD ID。此外，`ceph-volume`还会为您将新的OSD添加到主机下的CRUSH映射中。执行以获取CLI详细信息。该实用程序将自动执行下面“ [长格式”的](https://docs.ceph.com/docs/nautilus/install/manual-deployment/#long-form)步骤。要使用简短过程创建前两个OSD，请在和上执行以下操作 ：`ceph-volume -hceph-volumenode2node3`

**BLUESTORE** 

1. 创建OSD。

   ```text
   ssh {node-name}
   sudo ceph-volume lvm create --data {data-path}
   ```

   例如：

   ```text
   ssh node1
   sudo ceph-volume lvm create --data /dev/hdd1
   ```

另外，创建过程可以分为两个阶段（准备和激活）：

1. 准备OSD。

   ```text
   ssh {node-name}
   sudo ceph-volume lvm prepare --data {data-path} {data-path}
   ```

   例如：

   ```text
   ssh node1
   sudo ceph-volume lvm prepare --data /dev/hdd1
   ```

   一旦准备好，就需要激活OSD中的`ID`和`FSID`。这些可以通过列出当前服务器中的OSD来获得：

   ```text
   sudo ceph-volume lvm list
   ```

2. 激活OSD：

   ```text
   sudo ceph-volume lvm activate {ID} {FSID}
   ```

   例如：

   ```text
   sudo ceph-volume lvm activate 0 a7f64266-0894-4f1e-a635-d0aeaca0e993
   ```

**文件存储**

1. 创建OSD。

   ```text
   ssh {node-name}
   sudo ceph-volume lvm create --filestore --data {data-path} --journal {journal-path}
   ```

   例如：

   ```text
   ssh node1
   sudo ceph-volume lvm create --filestore --data /dev/hdd1 --journal /dev/hdd2
   ```

另外，创建过程可以分为两个阶段（准备和激活）：

1. 准备OSD。

   ```text
   ssh {node-name}
   sudo ceph-volume lvm prepare --filestore --data {data-path} --journal {journal-path}
   ```

   例如：

   ```text
   ssh node1
   sudo ceph-volume lvm prepare --filestore --data /dev/hdd1 --journal /dev/hdd2
   ```

   一旦准备好，就需要激活OSD中的`ID`和`FSID`。这些可以通过列出当前服务器中的OSD来获得：

   ```text
   sudo ceph-volume lvm list
   ```

2. 激活OSD：

   ```text
   sudo ceph-volume lvm activate --filestore {ID} {FSID}
   ```

   例如：

   ```text
   sudo ceph-volume lvm activate --filestore 0 a7f64266-0894-4f1e-a635-d0aeaca0e993
   ```

#### 长格式

没有任何帮助程序实用程序的好处，请按照以下步骤创建OSD并将其添加到群集和CRUSH映射中。要使用长格式过程创建前两个OSD，请对每个OSD执行以下步骤。

注意 

此过程未描述通过使用dm-crypt'lockbox'在dm-crypt之上进行部署。

1. 连接到OSD主机并成为root用户。

   ```text
   ssh {node-name}
   sudo bash
   ```

2. 为OSD生成一个UUID。

   ```text
   UUID=$(uuidgen)
   ```

3. 为OSD生成一个cephx密钥。

   ```text
   OSD_SECRET=$(ceph-authtool --gen-print-key)
   ```

4. 创建OSD。请注意，如果需要重用以前销毁的OSD ID，则可以将OSD ID作为附加参数提供。我们假定 密钥存在于机器上。您也可以在存在该密钥的其他主机上执行该命令：`ceph osd newclient.bootstrap-osdclient.admin`

   ```text
   ID=$(echo "{\"cephx_secret\": \"$OSD_SECRET\"}" | \
      ceph osd new $UUID -i - \
      -n client.bootstrap-osd -k /var/lib/ceph/bootstrap-osd/ceph.keyring)
   ```

   还可以`crush_device_class`在JSON中包含一个属性以设置默认值以外的初始类（`ssd`或`hdd`基于自动检测到的设备类型）。

5. 在新的OSD上创建默认目录。

   ```text
   mkdir /var/lib/ceph/osd/ceph-$ID
   ```

6. 如果OSD用于OS驱动器以外的驱动器，请准备将其与Ceph一起使用，并将其安装到刚创建的目录中。

   ```text
   mkfs.xfs /dev/{DEV}
   mount /dev/{DEV} /var/lib/ceph/osd/ceph-$ID
   ```

7. 将机密写入OSD密钥环文件。

   ```text
   ceph-authtool --create-keyring /var/lib/ceph/osd/ceph-$ID/keyring \
        --name osd.$ID --add-key $OSD_SECRET
   ```

8. 初始化OSD数据目录。

   ```text
   ceph-osd -i $ID --mkfs --osd-uuid $UUID
   ```

9. 修正所有权。

   ```text
   chown -R ceph:ceph /var/lib/ceph/osd/ceph-$ID
   ```

10. 将OSD添加到Ceph之后，该OSD已在您的配置中。但是，它尚未运行。您必须先启动新的OSD，然后才能开始接收数据。

    对于现代系统发行版：

    ```text
    systemctl enable ceph-osd@$ID
    systemctl start ceph-osd@$ID
    ```

    例如：

    ```text
    systemctl enable ceph-osd@12
    systemctl start ceph-osd@12
    ```

### 添加

在以下说明中，`{id}`是一个任意名称，例如计算机的主机名。

1. 创建mds数据目录。

   ```text
   mkdir -p /var/lib/ceph/mds/{cluster-name}-{id}
   ```

2. 创建一个密钥环：

   ```text
   ceph-authtool --create-keyring /var/lib/ceph/mds/{cluster-name}-{id}/keyring --gen-key -n mds.{id}
   ```

3. 导入密钥环并设置上限：

   ```text
   ceph auth add mds.{id} osd "allow rwx" mds "allow" mon "allow profile mds" -i /var/lib/ceph/mds/{cluster}-{id}/keyring
   ```

4. 添加到ceph.conf .:

   ```text
   [mds.{id}]
   host = {id}
   ```

5. 手动启动守护程序。

   ```text
   ceph-mds --cluster {cluster-name} -i {id} -m {mon-hostname}:{mon-port} [-f]
   ```

6. 以正确的方式启动守护程序（使用ceph.conf条目）：

   ```text
   service ceph start
   ```

7. 如果启动守护程序失败并显示以下错误：

   ```text
   mds.-1.0 ERROR: failed to authenticate: (22) Invalid argument
   ```

   然后确保在global区域的ceph.conf中没有设置密钥环。将其移至客户部分；或添加特定于此mds守护程序的密钥环设置。并确认您在mds数据目录和输出中看到相同的密钥。`ceph auth get mds.{id}`

8. 现在您可以[创建一个Ceph文件系统了](https://docs.ceph.com/docs/nautilus/cephfs/createfs)。

### 总结

监视器和两个OSD启动并运行后，可以通过执行以下操作来观察放置组的对等情况：

```text
ceph -w
```

要查看树，请执行以下操作：

```text
ceph osd tree
```

您应该看到如下所示的输出：

```text
# id    weight  type name       up/down reweight
-1      2       root default
-2      2               host node1
0       1                       osd.0   up      1
-3      1               host node2
1       1                       osd.1   up      1
```

要添加（或删除）其他监视器，请参阅“ [添加/删除监视器”](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-mons)。要添加（或删除）其他Ceph OSD守护程序，请参阅“ [添加/删除OSD”](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds)。

