# 手工安装

## 安装（手动）

### 获取软件

有几种获取Ceph软件的方法。最简单，最常见的方法是通过添加存储库来[获取软件包](https://docs.ceph.com/docs/nautilus/install/get-packages)，以与软件包管理工具（如高级软件包工具（APT）或经过修改的Yellowdog Updater（YUM））一起使用。您也可以从Ceph存储库中检索预编译的软件包。最后，您可以检索tarball或克隆Ceph源代码存储库并自己构建Ceph。

* [取得包裹](https://docs.ceph.com/docs/nautilus/install/get-packages/)
* [获取tarballs](https://docs.ceph.com/docs/nautilus/install/get-tarballs/)
* [克隆源](https://docs.ceph.com/docs/nautilus/install/clone-source/)
* [建立Ceph](https://docs.ceph.com/docs/nautilus/install/build-ceph/)
* [西弗镜](https://docs.ceph.com/docs/nautilus/install/mirrors/)

### 安装软件

有了Ceph软件（或添加的存储库）后，安装该软件很容易。在群集中的每个[Ceph节点](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-node)上安装软件包。您可以用来 `ceph-deploy`为存储集群安装Ceph或使用程序包管理工具。如果要安装Ceph对象网关或QEMU，则应为使用Rum的RHEL / CentOS和其他发行版安装Yum Priorities。

* [安装ceph-deploy](https://docs.ceph.com/docs/nautilus/install/install-ceph-deploy/)
* [安装Ceph存储集群](https://docs.ceph.com/docs/nautilus/install/install-storage-cluster/)
* [安装Ceph对象网关](https://docs.ceph.com/docs/nautilus/install/install-ceph-gateway/)
* [安装虚拟块](https://docs.ceph.com/docs/nautilus/install/install-vm-cloud/)

### 部署一个集群手动

一旦在节点上安装了Ceph，就可以手动部署集群。手动过程主要是为那些使用Chef，Juju，Puppet等开发部署脚本的人提供示例性目的。

* [手动部署](https://docs.ceph.com/docs/nautilus/install/manual-deployment/)
  * [监控启动](https://docs.ceph.com/docs/nautilus/install/manual-deployment/#monitor-bootstrapping)
  * [管理器守护程序配置](https://docs.ceph.com/docs/nautilus/install/manual-deployment/#manager-daemon-configuration)
  * [添加OSD](https://docs.ceph.com/docs/nautilus/install/manual-deployment/#adding-osds)
    * [简写](https://docs.ceph.com/docs/nautilus/install/manual-deployment/#short-form)
      * [蓝店](https://docs.ceph.com/docs/nautilus/install/manual-deployment/#bluestore)
      * [文件存储](https://docs.ceph.com/docs/nautilus/install/manual-deployment/#filestore)
    * [长表](https://docs.ceph.com/docs/nautilus/install/manual-deployment/#long-form)
  * [添加MDS](https://docs.ceph.com/docs/nautilus/install/manual-deployment/#adding-mds)
  * [摘要](https://docs.ceph.com/docs/nautilus/install/manual-deployment/#summary)
* [在FreeBSD上手动部署](https://docs.ceph.com/docs/nautilus/install/manual-freebsd-deployment/)
  * [FreeBSD上的磁盘布局](https://docs.ceph.com/docs/nautilus/install/manual-freebsd-deployment/#disklayout-on-freebsd)
    * [组态](https://docs.ceph.com/docs/nautilus/install/manual-freebsd-deployment/#configuration)
  * [监控启动](https://docs.ceph.com/docs/nautilus/install/manual-freebsd-deployment/#monitor-bootstrapping)
  * [添加OSD](https://docs.ceph.com/docs/nautilus/install/manual-freebsd-deployment/#adding-osds)
    * [长表](https://docs.ceph.com/docs/nautilus/install/manual-freebsd-deployment/#long-form)
  * [添加MDS](https://docs.ceph.com/docs/nautilus/install/manual-freebsd-deployment/#adding-mds)
  * [摘要](https://docs.ceph.com/docs/nautilus/install/manual-freebsd-deployment/#summary)

### 升级软件

随着新版本的Ceph可用，您可以升级群集以利用新功能。在升级群集之前，请阅读升级文档。有时，升级Ceph要求您遵循升级顺序。

* [升级Ceph](https://docs.ceph.com/docs/nautilus/install/upgrading-ceph/)
  * [摘要](https://docs.ceph.com/docs/nautilus/install/upgrading-ceph/#summary)
  * [Ceph部署](https://docs.ceph.com/docs/nautilus/install/upgrading-ceph/#ceph-deploy)
  * [升级程序](https://docs.ceph.com/docs/nautilus/install/upgrading-ceph/#upgrade-procedures)

