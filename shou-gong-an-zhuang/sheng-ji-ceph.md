# 升级CEPH

## 升级CEPH

每个Ceph版本可能都有其他步骤。在使用升级过程之前，请参考[您发行版](https://docs.ceph.com/docs/nautilus/releases)的[发行说明文档，](https://docs.ceph.com/docs/nautilus/releases)以识别集群的发行版特定过程。

### 总结

您可以在群集处于联机状态和服务状态时升级Ceph群集中的守护程序！某些类型的守护程序依赖于其他守护程序。例如，Ceph元数据服务器和Ceph对象网关依赖于Ceph监视器和Ceph OSD守护程序。我们建议按以下顺序升级：

1. [Ceph部署](https://docs.ceph.com/docs/nautilus/install/upgrading-ceph/#ceph-deploy)
2. Ceph监控器
3. Ceph OSD守护程序
4. Ceph元数据服务器
5. Ceph对象网关

通常，我们建议升级特定类型的所有`ceph-mon`守护程序（例如，所有守护程序，所有`ceph-osd`守护程序等），以确保它们都在同一发行版上。我们还建议您在尝试使用发行版中的新功能之前，先升级群集中的所有守护程序。

在[升级过程](https://docs.ceph.com/docs/nautilus/install/upgrading-ceph/#upgrade-procedures)都比较简单，但做看[您发布的发行说明文件](https://docs.ceph.com/docs/nautilus/releases)在升级之前。基本过程包括三个步骤：

1. 使用`ceph-deploy`您的管理节点上的多台主机（使用软件包升级命令），或登录到每个主机和升级包Ceph的[使用你的发行版的包管理器](https://docs.ceph.com/docs/nautilus/install/install-storage-cluster/)。例如，在[升级Monitors时](https://docs.ceph.com/docs/nautilus/install/upgrading-ceph/#upgrading-monitors)，语法可能如下所示：`ceph-deploy installceph-deploy`

   ```text
   ceph-deploy install --release {release-name} ceph-node1[ ceph-node2]
   ceph-deploy install --release firefly mon1 mon2 mon3
   ```

   **注意：**该命令会将指定节点中的软件包从旧版本升级到您指定的版本。没有命令。`ceph-deploy installceph-deploy upgrade`

2. 登录到每个Ceph节点，然后重新启动每个Ceph守护程序。有关详细信息，请参见[操作群集](https://docs.ceph.com/docs/nautilus/rados/operations/operating)。
3. 确保您的群集健康。有关详细信息，请参见[监视群集](https://docs.ceph.com/docs/nautilus/rados/operations/monitoring)。

重要 

升级守护程序后，就无法将其降级。

### CEPH部署

在升级Ceph守护程序之前，请升级该`ceph-deploy`工具。

```text
sudo pip install -U ceph-deploy
```

要么：

```text
sudo apt-get install ceph-deploy
```

要么：

```text
sudo yum install ceph-deploy python-pushy
```

### 升级过程

以下各节描述了升级过程。

重要 

每个Ceph版本可能都有一些其他步骤。**在**开始升级守护程序**之前，**请参阅[发行说明](https://docs.ceph.com/docs/nautilus/releases)的[发行说明文档以](https://docs.ceph.com/docs/nautilus/releases)获取详细信息。

#### 升级显示器

要升级监视器，请执行以下步骤：

1. 为每个守护程序实例升级Ceph软件包。

   您可以用来`ceph-deploy`一次寻址所有监视器节点。例如：

   ```text
   ceph-deploy install --release {release-name} ceph-node1[ ceph-node2]
   ceph-deploy install --release hammer mon1 mon2 mon3
   ```

   您还可以在每个单独的节点上为Linux发行版使用软件包管理器。要在每个Debian / Ubuntu主机上手动升级软件包，请执行以下步骤：

   ```text
   ssh {mon-host}
   sudo apt-get update && sudo apt-get install ceph
   ```

   在CentOS / Red Hat主机上，执行以下步骤：

   ```text
   ssh {mon-host}
   sudo yum update && sudo yum install ceph
   ```

2. 重新启动每个监视器。对于Ubuntu发行版，请使用：

   ```text
   sudo restart ceph-mon id={hostname}
   ```

   对于CentOS / Red Hat / Debian发行版，使用：

   ```text
   sudo /etc/init.d/ceph restart {mon-id}
   ```

   对于使用部署的CentOS / Red Hat发行版`ceph-deploy`，监视器ID通常为`mon.{hostname}`。

3. 确保每个监视器都重新加入仲裁：

   ```text
   ceph mon stat
   ```

确保您已完成所有Ceph Monitor的升级周期。

#### 升级

要升级Ceph OSD守护程序，请执行以下步骤：

1. 升级Ceph OSD守护程序包。

   您可以`ceph-deploy`用来一次寻址所有Ceph OSD Daemon节点。例如：

   ```text
   ceph-deploy install --release {release-name} ceph-node1[ ceph-node2]
   ceph-deploy install --release hammer osd1 osd2 osd3
   ```

   您也可以在每个节点上使用软件包管理器来[使用发行版的软件包管理器](https://docs.ceph.com/docs/nautilus/install/install-storage-cluster/)来升级软件包 。对于Debian / Ubuntu主机，在每个主机上执行以下步骤：

   ```text
   ssh {osd-host}
   sudo apt-get update && sudo apt-get install ceph
   ```

   对于CentOS / Red Hat主机，执行以下步骤：

   ```text
   ssh {osd-host}
   sudo yum update && sudo yum install ceph
   ```

2. 重新启动OSD，这里`N`是OSD号。对于Ubuntu，请使用：

   ```text
   sudo restart ceph-osd id=N
   ```

   对于主机上的多个OSD，可以使用Upstart重新启动所有OSD。

   ```text
   sudo restart ceph-osd-all
   ```

   对于CentOS / Red Hat / Debian发行版，使用：

   ```text
   sudo /etc/init.d/ceph restart N
   ```

3. 确保每个升级的Ceph OSD守护进程都已重新加入集群：

   ```text
   ceph osd stat
   ```

确保已完成所有Ceph OSD守护程序的升级周期。

#### 升级元数据服务器

要升级Ceph Metadata Server，请执行以下步骤：

1. 升级Ceph Metadata Server软件包。您可以`ceph-deploy`用来一次寻址所有Ceph Metadata Server节点，也可以在每个节点上使用程序包管理器。例如：

   ```text
   ceph-deploy install --release {release-name} ceph-node1
   ceph-deploy install --release hammer mds1
   ```

   要手动升级软件包，请在每个Debian / Ubuntu主机上执行以下步骤：

   ```text
   ssh {mon-host}
   sudo apt-get update && sudo apt-get install ceph-mds
   ```

   或在CentOS / Red Hat主机上执行以下步骤：

   ```text
   ssh {mon-host}
   sudo yum update && sudo yum install ceph-mds
   ```

2. 重新启动元数据服务器。对于Ubuntu，请使用：

   ```text
   sudo restart ceph-mds id={hostname}
   ```

   对于CentOS / Red Hat / Debian发行版，使用：

   ```text
   sudo /etc/init.d/ceph restart mds.{hostname}
   ```

   对于使用部署的集群`ceph-deploy`，名称通常是您在创建时指定的名称或主机名。

3. 确保元数据服务器已启动并正在运行：

   ```text
   ceph mds stat
   ```

#### 升级客户端

一旦您在Ceph集群上升级了软件包并重新启动了守护程序，我们也建议您在客户端节点上升级`ceph-common`和客户端库（`librbd1`和`librados2`）。

1. 升级软件包：

   ```text
   ssh {client-host}
   apt-get update && sudo apt-get install ceph-common librados2 librbd1 python-rados python-rbd
   ```

2. 确保您具有最新版本：

   ```text
   ceph --version
   ```

如果没有最新版本，则可能需要卸载，自动删除依赖项并重新安装。

