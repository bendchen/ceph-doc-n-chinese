# 认证配置

## CEPHX配置参考

`cephx`默认情况下启用该协议。密码认证有一些计算成本，尽管它们通常应该很低。如果连接客户端和服务器主机的网络环境非常安全，并且您无法承担身份验证的费用，则可以将其关闭。**通常不建议这样做**。

注意 

如果禁用身份验证，则可能会受到中间人攻击，从而更改您的客户端/服务器消息，这可能会导致灾难性的安全影响。

要创建用户，请参阅[用户管理](https://docs.ceph.com/docs/nautilus/rados/operations/user-management)。有关Cephx架构的详细信息，请参阅《[架构-高可用性身份验证》](https://docs.ceph.com/docs/nautilus/architecture#high-availability-authentication)。

### 部署方案

部署Ceph集群有两种主要方案，它们会影响您最初配置Cephx的方式。大多数Ceph用户第一次使用 `ceph-deploy`创建集群（最简单）。对于使用其他部署工具（例如Chef，Juju，Puppet等）的群集，您将需要使用手动过程或配置您的部署工具来引导您的监视器。

#### CEPH- 

当使用部署集群时`ceph-deploy`，不必手动引导监视器或创建`client.admin`用户或密钥环。您在执行的步骤[存储集群快速入门](https://docs.ceph.com/docs/nautilus/start/quick-ceph-deploy/)将调用`ceph-deploy`来为你做的。

当您执行时，Ceph将为您创建一个监视器密钥环（仅用于引导监视器），它将为您生成一个初始Ceph配置文件，其中包含以下身份验证设置，表明Ceph默认情况下启用身份验证：`ceph-deploy new {initial-monitor(s)}`

```text
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
```

当您执行时，Ceph将引导初始监视器，并检索包含该用户密钥的文件 。此外，它也将检索钥匙扣，让和事业编制的能力和激活OSD及元数据服务器。`ceph-deploy mon create-initialceph.client.admin.keyringclient.adminceph-deployceph-volume`

执行时（**注意：**必须先安装Ceph），这会将Ceph配置文件和推 送到 节点的目录中。您将能够在该节点的命令行上执行Ceph管理功能。`ceph-deploy admin {node-name}ceph.client.admin.keyring/etc/cephroot`

#### 手动部署

手动部署群集时，必须手动引导监视器并创建`client.admin`用户和密钥环。要引导监视器，请遵循[Monitor Bootstrapping中](https://docs.ceph.com/docs/nautilus/install/manual-deployment#monitor-bootstrapping)的步骤。监控程序引导步骤是使用诸如Chef，Puppet，Juju等第三方部署工具时必须执行的逻辑步骤。

### 启用/禁用CEPHX 

启用Cephx要求您为监视器，OSD和元数据服务器部署密钥。如果您只是在打开/关闭Cephx上切换，则不必重复执行自举程序。

#### 启用CEPHX 

当`cephx`启用时，头孢将寻找在默认的搜索路径，其中包括钥匙圈`/etc/ceph/$cluster.$name.keyring`。您可以通过`keyring`在[Ceph配置](https://docs.ceph.com/docs/nautilus/rados/configuration/ceph-conf)文件的`[global]`部分中添加一个选项来覆盖此位置，但是不建议这样做。

执行以下过程以`cephx`在禁用身份验证的群集上启用。如果您（或您的部署实用程序）已经生成了密钥，则可以跳过与生成密钥有关的步骤。

1. 创建一个`client.admin`密钥，并为您的客户端主机保存密钥的副本：

   ```text
   ceph auth get-or-create client.admin mon 'allow *' mds 'allow *' mgr 'allow *' osd 'allow *' -o /etc/ceph/ceph.client.admin.keyring
   ```

   **警告：**这会破坏现有 `/etc/ceph/client.admin.keyring`文件。如果部署工具已经为您完成了此步骤，则不要执行此步骤。小心！

2. 为您的监视器群集创建一个密钥环，并生成一个监视器密钥。

   ```text
   ceph-authtool --create-keyring /tmp/ceph.mon.keyring --gen-key -n mon. --cap mon 'allow *'
   ```

3. 将监视器密钥环复制到`ceph.mon.keyring`每个监视器目录中的文件中 。例如，要将其复制到cluster中，请使用以下命令：`mon datamon.aceph`

   ```text
   cp /tmp/ceph.mon.keyring /var/lib/ceph/mon/ceph-a/keyring
   ```

4. 为每个MGR生成一个密钥，`{$id}`MGR字母在哪里：

   ```text
   ceph auth get-or-create mgr.{$id} mon 'allow profile mgr' mds 'allow *' osd 'allow *' -o /var/lib/ceph/mgr/ceph-{$id}/keyring
   ```

5. 为每个OSD生成一个密钥，其中`{$id}`OSD编号为：

   ```text
   ceph auth get-or-create osd.{$id} mon 'allow rwx' osd 'allow *' -o /var/lib/ceph/osd/ceph-{$id}/keyring
   ```

6. 为每个MDS生成一个密钥，`{$id}`MDS字母在哪里：

   ```text
   ceph auth get-or-create mds.{$id} mon 'allow rwx' osd 'allow *' mds 'allow *' mgr 'allow profile mds' -o /var/lib/ceph/mds/ceph-{$id}/keyring
   ```

7. `cephx`通过在[Ceph配置](https://docs.ceph.com/docs/nautilus/rados/configuration/ceph-conf)文件的`[global]`部分中设置以下选项来启用身份验证 ：

   ```text
   auth cluster required = cephx
   auth service required = cephx
   auth client required = cephx
   ```

8. 启动或重启Ceph集群。有关详细信息，请参见[操作群集](https://docs.ceph.com/docs/nautilus/rados/operations/operating)。

有关手动引导监视器的详细信息，请参见《[手动部署》](https://docs.ceph.com/docs/nautilus/install/manual-deployment)。

#### 禁用CEPHX 

以下过程介绍了如何禁用Cephx。如果您的集群环境相对安全，则可以抵消运行身份验证的计算费用。**我们不建议这样做。**但是，在设置和/或故障排除过程中暂时禁用身份验证可能会更容易。

1. `cephx`通过在[Ceph配置](https://docs.ceph.com/docs/nautilus/rados/configuration/ceph-conf)文件的`[global]`部分中设置以下选项来禁用身份验证 ：

   ```text
   auth cluster required = none
   auth service required = none
   auth client required = none
   ```

2. 启动或重启Ceph集群。有关详细信息，请参见[操作群集](https://docs.ceph.com/docs/nautilus/rados/operations/operating)。

### 配置设置

#### 启用

`auth cluster required`描述

如果启用，Ceph的存储集群守护进程（即`ceph-mon`， `ceph-osd`，`ceph-mds`和`ceph-mgr`）必须相互进行身份验证。有效设置为`cephx`或`none`。类型

串需要

没有默认

`cephx`。

`auth service required`描述

如果启用，则Ceph Storage Cluster守护进程需要Ceph客户端向Ceph Storage Cluster进行身份验证才能访问Ceph服务。有效设置为`cephx`或`none`。类型

串需要

没有默认

`cephx`。

`auth client required`描述

如果启用，则Ceph客户端需要Ceph存储群集向Ceph客户端进行身份验证。有效设置为`cephx` 或`none`。类型

串需要

没有默认

`cephx`。

#### 按键

在启用身份验证的情况下运行Ceph时，`ceph`管理命令和Ceph客户端需要身份验证密钥才能访问Ceph存储群集。

向`ceph`管理命令和客户端提供这些密钥的最常见方法是在`/etc/ceph` 目录下包含Ceph密钥环。对于使用的墨鱼和更高版本`ceph-deploy`，文件名通常为`ceph.client.admin.keyring`（或`$cluster.client.admin.keyring`）。如果在`/etc/ceph`目录下包含密钥环，则无需`keyring`在Ceph配置文件中指定条目。

我们建议将Ceph Storage Cluster的密钥环文件复制到要运行管理命令的节点，因为它包含`client.admin`密钥。

您可以用来执行此任务。有关详细信息，请参见[创建管理主机](https://docs.ceph.com/docs/nautilus/rados/deployment/ceph-deploy-admin)。要手动执行此步骤，请执行以下操作：`ceph-deploy admin`

```text
sudo scp {user}@{ceph-cluster-host}:/etc/ceph/ceph.client.admin.keyring /etc/ceph/ceph.client.admin.keyring
```

小费 

确保`ceph.keyring`文件在客户端计算机上具有适当的权限集（例如）。`chmod 644`

您可以使用`key` 设置（不推荐）在Ceph配置文件中指定密钥本身，或者使用设置来指定密钥文件的路径`keyfile`。

`keyring`描述

密钥环文件的路径。类型

串需要

没有默认

`/etc/ceph/$cluster.$name.keyring,/etc/ceph/$cluster.keyring,/etc/ceph/keyring,/etc/ceph/keyring.bin`

`keyfile`描述

密钥文件（即仅包含密钥的文件）的路径。类型

串需要

没有默认

没有

`key`描述

密钥（即密钥本身的文本字符串）。不建议。类型

串需要

没有默认

没有

#### 守护程序密钥环

管理用户或部署工具（例如`ceph-deploy`）可以按照与生成用户密钥环相同的方式来生成守护程序密钥环。默认情况下，Ceph将守护程序密钥环存储在其数据目录中。默认密钥环位置以及守护程序运行所需的功能如下所示。

`ceph-mon`位置

`$mon_data/keyring`能力

`mon 'allow *'`

`ceph-osd`位置

`$osd_data/keyring`能力

`mgr 'allow profile osd' mon 'allow profile osd' osd 'allow *'`

`ceph-mds`位置

`$mds_data/keyring`能力

`mds 'allow' mgr 'allow profile mds' mon 'allow profile mds' osd 'allow rwx'`

`ceph-mgr`位置

`$mgr_data/keyring`能力

`mon 'allow profile mgr' mds 'allow *' osd 'allow *'`

`radosgw`位置

`$rgw_data/keyring`能力

`mon 'allow rwx' osd 'allow rwx'`

注意 

监视器密钥环（即`mon.`）包含密钥但不具有功能，并且不属于集群`auth`数据库。

守护程序数据目录位置默认为以下形式的目录：

```text
/var/lib/ceph/$type/$cluster-$id
```

例如，`osd.12`将是：

```text
/var/lib/ceph/osd/ceph-12
```

您可以覆盖这些位置，但不建议这样做。

#### 签名

Ceph执行签名检查，以提供一些有限的保护，以防止消息在飞行中被篡改（例如，“中间人”攻击）。

与Ceph身份验证的其他部分一样，Ceph提供了细粒度的控制，因此您可以为客户端和Ceph之间的服务消息启用/禁用签名，并且可以为Ceph守护程序之间的消息启用/禁用签名。

请注意，即使启用了签名，也不会在飞行中对数据进行加密。

`cephx require signatures`描述

如果设置为`true`，则Ceph要求在Ceph客户端和Ceph存储群集之间以及组成Ceph存储群集的守护程序之间的所有消息通信上签名。

Ceph Argonaut和3.19之前的Linux内核版本不支持签名。如果正在使用此类客户端，则可以关闭此选项以允许它们连接。类型

布尔型需要

没有默认

`false`

`cephx cluster require signatures`描述

如果设置为`true`，则Ceph要求在组成Ceph存储群集的Ceph守护程序之间的所有消息流量上签名。类型

布尔型需要

没有默认

`false`

`cephx service require signatures`描述

如果设置为`true`，则Ceph要求在Ceph客户端和Ceph存储集群之间的所有消息流量上签名。类型

布尔型需要

没有默认

`false`

`cephx sign messages`描述

如果Ceph版本支持消息签名，则Ceph将对所有消息进行签名，因此更难以欺骗。类型

布尔型默认

`true`

#### 生存时间

`auth service ticket ttl`描述

当Ceph存储群集向Ceph客户端发送用于身份验证的票证时，Ceph存储群集会为该票证分配生存时间。类型

双默认

`60*60`

