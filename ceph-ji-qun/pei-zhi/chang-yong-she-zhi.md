# 常用设置

## 通用设置

该[硬件建议](https://docs.ceph.com/docs/nautilus/start/hardware-recommendations)部分提供了配置Ceph的存储集群一些硬件的指导方针。一个[Ceph节点](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-node)可以运行多个守护程序。例如，具有多个驱动器的单个节点可以`ceph-osd`为每个驱动器运行一个。理想情况下，您将具有用于特定类型流程的节点。例如，某些节点可能会运行`ceph-osd` 守护程序，其他节点可能会运行`ceph-mds`守护程序，而另一些节点可能会运行`ceph-mon`守护程序。

每个节点都有一个由`host`设置标识的名称。监视器还指定该`addr`设置标识的网络地址和端口（即域名或IP地址） 。基本配置文件通常只会为每个监视器守护程序实例指定最少的设置。例如：

```text
[global]
mon_initial_members = ceph1
mon_host = 10.0.0.1
```

重要 

该`host`设置是节点的简称（即不是fqdn）。这是**不是**一个IP地址，无论是。在命令行上输入以检索节点的名称。 除非您手动部署Ceph，否则不要将设置用于初始监视器以外的任何其他设备。使用诸如或的部署工具时，您**不得**在单个守护程序下指定，因为这些工具将在集群图中为您输入适当的值。`hostname -shosthostchefceph-deploy`

## 网络

有关配置与Ceph一起使用的网络的详细讨论，请参见[网络配置参考](https://docs.ceph.com/docs/nautilus/rados/configuration/network-config-ref)。

## 监视器

Ceph生产集群通常至少部署3个[Ceph Monitor](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-monitor) 守护程序，以确保在监视器实例崩溃时具有高可用性。至少三（3）个监视器确保Paxos算法可以从仲裁中的大多数Ceph Monitor中确定哪个版本的[Ceph Cluster Map](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-cluster-map)是最新的。

注意 

您可以使用单个监视器部署Ceph，但是如果实例失败，则缺少其他监视器可能会中断数据服务的可用性。

Ceph监视器通常在端口上侦听`3300`新的v2协议和`6789`旧的v1协议。

默认情况下，Ceph希望您将监视器的数据存储在以下路径下：

```text
/var/lib/ceph/mon/$cluster-$id
```

您或部署工具（例如，`ceph-deploy`）必须创建相应的目录。使用完全表示的元变量和名为“ ceph”的聚类，上述目录将评估为：

```text
/var/lib/ceph/mon/ceph-a
```

有关更多详细信息，请参见《[Monitor Config Reference》](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-config-ref)。

## 认证

新版本的短尾猫： 0.56

对于Bobtail（v 0.56）及更高版本，您应该在`[global]`Ceph配置文件的部分中明确启用或禁用身份验证。

```text
auth cluster required = cephx
auth service required = cephx
auth client required = cephx
```

此外，您应该启用消息签名。有关详细信息，请参见[Cephx Config Reference](https://docs.ceph.com/docs/nautilus/rados/configuration/auth-config-ref)。

重要 

升级时，建议您先明确禁用身份验证，然后再执行升级。升级完成后，请重新启用身份验证。

## 屏上显示

Ceph生产集群通常部署[Ceph OSD守护程序](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-osd-daemons)，其中一个节点有一个OSD守护程序，在一个存储驱动器上运行一个文件存储。典型部署指定日志大小。例如：

```text
[osd]
osd journal size = 10000

[osd.0]
host = {hostname} #manual deployments only.
```

默认情况下，Ceph希望您将使用以下路径存储Ceph OSD守护程序的数据：

```text
/var/lib/ceph/osd/$cluster-$id
```

您或部署工具（例如，`ceph-deploy`）必须创建相应的目录。使用完全表示的元变量和名为“ ceph”的聚类，上述目录将评估为：

```text
/var/lib/ceph/osd/ceph-0
```

您可以使用设置覆盖此路径。我们不建议更改默认位置。在OSD主机上创建默认目录。`osd data`

```text
ssh {osd-host}
sudo mkdir /var/lib/ceph/osd/ceph-{osd-number}
```

理想情况下，该路径导致带有硬盘的挂载点，该挂载点与存储和运行操作系统和守护程序的硬盘分开。如果OSD用于OS磁盘以外的其他磁盘，请准备将其与Ceph一起使用，然后将其安装到刚创建的目录中：`osd data`

```text
ssh {new-osd-host}
sudo mkfs -t {fstype} /dev/{disk}
sudo mount -o user_xattr /dev/{hdd} /var/lib/ceph/osd/ceph-{osd-number}
```

我们建议`xfs`在运行**mkfs**时使用文件系统 。（`btrfs`并且`ext4`不建议使用，并且不再进行测试。）

有关其他配置详细信息，请参见《[OSD配置参考](https://docs.ceph.com/docs/nautilus/rados/configuration/osd-config-ref)》。

## 心跳

在运行时操作期间，Ceph OSD守护程序会检查其他Ceph OSD守护程序，并将其发现结果报告给Ceph Monitor。您不必提供任何设置。但是，如果您遇到网络延迟问题，则可能希望修改设置。

有关其他详细信息，请参阅[配置Monitor / OSD交互](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-osd-interaction)。

## 日志/调试

有时，您可能会遇到Ceph的问题，这些问题需要修改日志记录输出并使用Ceph的调试。有关日志轮换的详细信息，请参见[调试和日志记录](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/log-and-debug)。

## 实施例CEPH.CONF 

```text
[global]
fsid = {cluster-id}
mon initial members = {hostname}[, {hostname}]
mon host = {ip-address}[, {ip-address}]

#All clusters have a front-side public network.
#If you have two NICs, you can configure a back side cluster 
#network for OSD object replication, heart beats, backfilling,
#recovery, etc.
public network = {network}[, {network}]
#cluster network = {network}[, {network}] 

#Clusters require authentication by default.
auth cluster required = cephx
auth service required = cephx
auth client required = cephx

#Choose reasonable numbers for your journals, number of replicas
#and placement groups.
osd journal size = {n}
osd pool default size = {n}  # Write an object n times.
osd pool default min size = {n} # Allow writing n copy in a degraded state.
osd pool default pg num = {n}
osd pool default pgp num = {n}

#Choose a reasonable crush leaf type.
#0 for a 1-node cluster.
#1 for a multi node cluster in a single rack
#2 for a multi node, multi chassis cluster with multiple hosts in a chassis
#3 for a multi node cluster with hosts across racks, etc.
osd crush chooseleaf type = {n}
```

## 运行多个群集

使用Ceph，您可以在同一硬件上运行多个Ceph存储集群。与在具有不同CRUSH规则的同一集群上使用不同的池相比，运行多个集群提供了更高的隔离级别。一个单独的群集将具有单独的监视器，OSD和元数据服务器进程。使用默认设置运行Ceph时，默认集群名称为`ceph`，这意味着您将使用文件名将Ceph配置文件保存 `ceph.conf`在`/etc/ceph`默认目录中。

有关详细信息，请参见[创建集群](https://docs.ceph.com/docs/nautilus/rados/deployment/ceph-deploy-new)。

当您运行多个集群时，必须为集群命名，并使用集群名称保存Ceph配置文件。例如，一个名为的群集 `openstack`将具有一个Ceph配置文件，其文件名 `openstack.conf`位于 `/etc/ceph`默认目录中。

重要 

群集名称只能包含字母az和数字0-9。

单独的群集意味着单独的数据磁盘和日志，它们在群集之间不共享。参考元[变量](https://docs.ceph.com/docs/nautilus/rados/configuration/ceph-conf#Metavariables)，该元`$cluster` 变量求值为群集名称（即，`openstack`在前面的示例中）。各种设置都使用该元`$cluster`变量，包括：

* `keyring`
* `admin socket`
* `log file`
* `pid file`
* `mon data`
* `mon cluster log file`
* `osd data`
* `osd journal`
* `mds data`
* `rgw data`

有关使用元变量的相关路径默认值， 请参见[常规设置](https://docs.ceph.com/docs/nautilus/rados/configuration/general-config-ref)，[OSD设置](https://docs.ceph.com/docs/nautilus/rados/configuration/osd-config-ref)，[监视器设置](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-config-ref)，[MDS设置](https://docs.ceph.com/docs/nautilus/cephfs/mds-config-ref)， [RGW设置](https://docs.ceph.com/docs/nautilus/radosgw/config-ref/)和[日志设置](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/log-and-debug)`$cluster`。

创建默认目录或文件时，应在路径的适当位置使用群集名称。例如：

```text
sudo mkdir /var/lib/ceph/osd/openstack-0
sudo mkdir /var/lib/ceph/mon/openstack-a
```

重要 

在同一主机上运行监视器时，应使用不同的端口。默认情况下，监视器使用端口6789。如果您已经有使用端口6789的监视器，则将其他端口用于其他群集。

要调用默认`ceph`集群以外的集群，请在命令中使用该 选项。例如：`-c {filename}.confceph`

```text
ceph -c {cluster-name}.conf health
ceph -c openstack.conf health
```

