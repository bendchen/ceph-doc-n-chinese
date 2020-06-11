# 添加/删除MDS

## 添加/删除元数据服务器

使用`ceph-deploy`，添加和删除元数据服务器是一项简单的任务。您只需使用一个命令在命令行上添加或删除一个或多个元数据服务器。

有关配置元数据服务器的详细信息，请参见[MDS Config Reference](https://docs.ceph.com/docs/nautilus/cephfs/mds-config-ref)。

### 添加元数据服务器

一旦部署了监视器和OSD，就可以部署元数据服务器。

```text
ceph-deploy mds create {host-name}[:{daemon-name}] [{host-name}[:{daemon-name}] ...]
```

如果要在单个服务器上运行多个守护程序，则可以为守护程序实例指定一个名称（可选）。

### 删除元数据服务器

快来了…

