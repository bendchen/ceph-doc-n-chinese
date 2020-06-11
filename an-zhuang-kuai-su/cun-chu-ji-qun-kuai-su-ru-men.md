# 存储集群快速入门

## 存储集群快速入门

如果尚未完成起飞前检查清单，请先执行此操作。此 **快速入门** 在您的管理节点上使用设置了一个Ceph存储集群`ceph-deploy`。创建三个Ceph节点集群，以便您探索Ceph功能。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-b490c5d9d3bb6984503b59681d08337aff62e992.png)

作为第一个练习，使用一个Ceph Monitor和三个Ceph OSD守护进程创建一个Ceph存储集群。集群达到状态后，通过添加第四个Ceph OSD守护程序，一个元数据服务器和两个以上的Ceph Monitors对其进行扩展。为了获得最佳结果，请在管理节点上创建一个目录，以维护为集群生成的配置文件和密钥。`active + cleanceph-deploy`

```text
mkdir my-cluster
cd my-cluster
```

该`ceph-deploy`实用程序会将文件输出到当前目录。执行时请确保您位于此目录中`ceph-deploy`。

**重要 ：不要用`ceph-deploy`用`sudo`或运行它`root` ，如果你已经登录为不同的用户，因为它不会发出`sudo` 需要在远程主机上的命令。**

### 从头开始

如果您在任何时候遇到麻烦并且想要重新开始，请执行以下操作清除Ceph软件包，并清除其所有数据和配置：

```text
ceph-deploy purge {ceph-node} [{ceph-node}]
ceph-deploy purgedata {ceph-node} [{ceph-node}]
ceph-deploy forgetkeys
rm ceph.*
```

如果执行`purge`，则必须重新安装Ceph。最后一个`rm` 命令删除在先前安装过程中由ceph-deploy在本地写出的所有文件。

### 创建群集

在您创建的用于保存配置详细信息的目录的管理节点上，使用进行以下步骤`ceph-deploy`。

1. 创建集群。

   ```text
   ceph-deploy new {initial-monitor-node(s)}
   ```

   将节点指定为主机名，fqdn或主机名：fqdn。例如：

   ```text
   ceph-deploy new node1
   ```

   检查`ceph-deploy`with `ls`和`cat`当前目录中的输出。您应该看到一个Ceph配置文件（`ceph.conf`），一个监视器秘密密钥环（`ceph.mon.keyring`）和新集群的日志文件。有关更多详细信息，请参见[ceph-deploy new -h](https://docs.ceph.com/docs/nautilus/rados/deployment/ceph-deploy-new)。

2. 如果您有多个网络接口，请 在您的Ceph配置文件的部分下添加设置。有关详细信息，请参见[网络配置参考](https://docs.ceph.com/docs/nautilus/rados/configuration/network-config-ref)。`public network[global]`

   ```text
   public network = {ip-address}/{bits}
   ```

   例如，：

   ```text
   public network = 10.1.2.0/24
   ```

   在10.1.2.0/24（或10.1.2.0/255.255.255.0）网络中使用IP。

3. 如果要在IPv6环境中进行部署，请`ceph.conf`在本地目录中添加以下内容 ：

   ```text
   echo ms bind ipv6 = true >> ceph.conf
   ```

4. 安装Ceph软件包：

   ```text
   ceph-deploy install {ceph-node} [...]
   ```

   例如：

   ```text
   ceph-deploy install node1 node2 node3
   ```

   该`ceph-deploy`实用程序将在每个节点上安装Ceph。

5. 部署初始监视器并收集密钥：

   ```text
   ceph-deploy mon create-initial
   ```

   完成此过程后，您的本地目录应具有以下密钥环：

   * `ceph.client.admin.keyring`
   * `ceph.bootstrap-mgr.keyring`
   * `ceph.bootstrap-osd.keyring`
   * `ceph.bootstrap-mds.keyring`
   * `ceph.bootstrap-rgw.keyring`
   * `ceph.bootstrap-rbd.keyring`
   * `ceph.bootstrap-rbd-mirror.keyring`

   注意 

   如果此过程失败并显示类似“无法找到/etc/ceph/ceph.client.admin.keyring”的消息，请确保在ceph.conf中为监视节点列出的IP是公用IP，而不是专用IP。 。

6. 使用`ceph-deploy`配置文件和管理密钥复制到您的管理节点和你的Ceph的节点，以便您可以使用`ceph` CLI，而无需指定监视地址和 `ceph.client.admin.keyring`每次执行命令。

   ```text
   ceph-deploy admin {ceph-node(s)}
   ```

   例如：

   ```text
   ceph-deploy admin node1 node2 node3
   ```

7. 部署管理器守护程序。（仅对于发光+构建需要）：

   ```text
   ceph-deploy mgr create node1  *Required only for luminous+ builds, i.e >= 12.x builds*
   ```

8. 添加三个OSD。出于这些说明的目的，我们假设您在名为的每个节点中都有一个未使用的磁盘`/dev/vdb`。 _确保该设备当前未使用并且不包含任何重要数据。_

   ```text
   ceph-deploy osd create --data {device} {ceph-node}
   ```

   例如：

   ```text
   ceph-deploy osd create --data /dev/vdb node1
   ceph-deploy osd create --data /dev/vdb node2
   ceph-deploy osd create --data /dev/vdb node3
   ```

   注意 

   如果要在LVM卷上创建OSD，则参数to `--data` _必须_为`volume_group/lv_name`，而不是卷的块设备的路径。

9. 检查集群的运行状况。

   ```text
   ssh node1 sudo ceph health
   ```

   您的集群应报告`HEALTH_OK`。您可以通过以下方式查看更完整的集群状态：

   ```text
   ssh node1 sudo ceph -s
   ```

### 扩展群集

基本集群启动并运行后，下一步就是扩展集群。将Ceph元数据服务器添加到中`node1`。然后将Ceph Monitor和Ceph Manager添加到`node2`，`node3`以提高可靠性和可用性。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-fc1f16db6801b202a4cc57c98bc87943031d9f80.png)

#### 添加元数据服务器

要使用CephFS，您至少需要一个元数据服务器。执行以下操作以创建元数据服务器：

```text
ceph-deploy mds create {ceph-node}
```

例如：

```text
ceph-deploy mds create node1
```

#### 添加监视器

一个Ceph存储集群至少需要一个Ceph Monitor和Ceph Manager才能运行。为了获得高可用性，Ceph存储群集通常运行多个Ceph监视器，因此单个Ceph监视器的故障不会使Ceph存储群集停机。Ceph使用Paxos算法，该算法需要大多数监视器（即，大于_N / 2_，其中_N_是监视器的数量）才能形成仲裁。监视器的奇数往往会更好，尽管这不是必需的。

将两个Ceph监视器添加到您的集群中：

```text
ceph-deploy mon add {ceph-nodes}
```

例如：

```text
ceph-deploy mon add node2 node3
```

一旦添加了新的Ceph监视器，Ceph将开始同步监视器并形成仲裁。您可以通过执行以下命令检查仲裁状态：

```text
ceph quorum_status --format json-pretty
```

小费 

当您使用多台监视器运行Ceph时，应在每台监视器主机上安装和配置NTP。确保监视器是NTP对等方。

#### 添加经理

Ceph Manager守护程序以活动/备用模式运行。部署其他管理器守护程序可确保如果一个守护程序或主机发生故障，另一守护程序或主机可以接管而不会中断服务。

要部署其他管理器守护程序：

```text
ceph-deploy mgr create node2 node3
```

您应该在以下输出中看到备用管理器：

```text
ssh node1 sudo ceph -s
```

#### 添加一个RGW实例

要使用[Ceph的Ceph对象网关](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-object-gateway)组件，您必须部署[RGW](https://docs.ceph.com/docs/nautilus/glossary/#term-rgw)的实例。执行以下操作以创建RGW的新实例：

```text
ceph-deploy rgw create {gateway-node}
```

例如：

```text
ceph-deploy rgw create node1
```

默认情况下，[RGW](https://docs.ceph.com/docs/nautilus/glossary/#term-rgw)实例将侦听端口7480。可以通过在运行[RGW](https://docs.ceph.com/docs/nautilus/glossary/#term-rgw)的节点上编辑ceph.conf来更改此端口，如下所示：

```text
[client]
rgw frontends = civetweb port=80
```

要使用IPv6地址，请使用：

```text
[client]
rgw frontends = civetweb port=[::]:80
```

### 存储/检索对象数据

要将对象数据存储在Ceph存储集群中，Ceph客户端必须：

1. 设置对象名称
2. 指定一个[池](https://docs.ceph.com/docs/nautilus/rados/operations/pools)

Ceph客户端检索最新的群集映射，CRUSH算法计算如何将对象映射到[放置组](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups)，然后计算如何将放置组动态分配给Ceph OSD守护程序。要找到对象位置，您只需要对象名称和池名称即可。例如：

```text
ceph osd map {poolname} {object-name}
```

练习：找到一个对象

作为练习，让我们创建一个对象。使用命令行上的命令指定对象名称，包含某些对象数据的测试文件的路径以及池名称 。例如：`rados put`

```text
echo {Test-data} > testfile.txt
ceph osd pool create mytest 8
rados put {object-name} {file-path} --pool=mytest
rados put test-object-1 testfile.txt --pool=mytest
```

要验证Ceph存储集群是否存储了该对象，请执行以下操作：

```text
rados -p mytest ls
```

现在，确定对象位置：

```text
ceph osd map {pool-name} {object-name}
ceph osd map mytest test-object-1
```

Ceph应该输出对象的位置。例如：

```text
osdmap e537 pool 'mytest' (1) object 'test-object-1' -> pg 1.d1743484 (1.4) -> up [1,0] acting [1,0]
```

要删除测试对象，只需使用 命令将其删除。`rados rm`

例如：

```text
rados rm test-object-1 --pool=mytest
```

删除`mytest`池：

```text
ceph osd pool rm mytest
```

（出于安全原因，您将需要根据提示提供其他参数；删除池会破坏数据。）

随着群集的发展，对象位置可能会动态变化。Ceph动态重新平衡的一个好处是，Ceph使您不必手动执行数据迁移或平衡。

