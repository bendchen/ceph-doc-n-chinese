# 创建群集

## 创建群集

使用Ceph with的第一步`ceph-deploy`是创建一个新的Ceph集群。一个新的Ceph集群具有：

* 一个Ceph配置文件，以及
* 监视器钥匙圈。

Ceph配置文件至少包含：

* 自己的文件系统ID（`fsid`）
* 初始监视器的主机名，以及
* 初始监控器和IP地址。

有关更多详细信息，请参见《[监视器配置参考》](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-config-ref)。

该`ceph-deploy`工具还会创建一个监视器密钥环，并在其中填充 `[mon.]`密钥。有关更多详细信息，请参阅《[Cephx指南》](https://docs.ceph.com/docs/nautilus/dev/mon-bootstrap#secret-keys)。

### 用法

要使用创建群集`ceph-deploy`，请使用`new`命令并指定将成为监视器仲裁的初始成员的主机。

```text
ceph-deploy new {host [host], ...}
```

例如：

```text
ceph-deploy new mon1.foo.com
ceph-deploy new mon{1,2,3}
```

该`ceph-deploy`实用程序将使用DNS将主机名解析为IP地址。监视器将使用名称的第一个组件来命名（例如，`mon1`上面）。它将指定的主机名添加到Ceph配置文件中。有关其他详细信息，请执行：

```text
ceph-deploy new -h
```

