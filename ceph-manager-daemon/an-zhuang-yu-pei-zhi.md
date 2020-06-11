# 安装与配置

## CEPH-MGR管理员指南

### 手动设置

通常，您将使用诸如ceph-ansible之类的工具来设置ceph-mgr守护程序。这些说明描述了如何手动设置ceph-mgr守护程序。

首先，为守护程序创建身份验证密钥：

```text
ceph auth get-or-create mgr.$name mon 'allow profile mgr' osd 'allow *' mds 'allow *'
```

将该密钥放入路径，对于集群“ ceph”和mgr $ name“ foo”将为。`mgr data/var/lib/ceph/mgr/ceph-foo`

启动ceph-mgr守护程序：

```text
ceph-mgr -i $name
```

通过查看的输出来检查mgr是否出现，现在该输出应包括mgr状态行：`ceph status`

```text
mgr active: $name
```

### 客户端认证

管理器是一个新的守护进程，它需要新的CephX功能。如果您从旧版本的Ceph升级群集，或使用默认的安装/部署工具，则管理客户端应自动获得此功能。如果您从其他地方使用工具，则在调用某些ceph集群命令时可能会出现EACCES错误。要解决此问题，请通过[修改用户功能](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#modify-user-capabilities)在客户端的cephx功能中添加“ mgr allow \*”节 。

### 高可用性

通常，您应该在每个运行ceph-mon守护程序的主机上设置ceph-mgr，以实现相同级别的可用性。

默认情况下，无论哪个先出现的ceph-mgr实例都将由监视器激活，而其他实例将成为备用实例。在ceph-mgr守护程序之间不需要仲裁。

如果活动守护程序未能将信标发送到监视器超过（默认30秒），则它将替换为备用。`mon mgr beacon grace`

如果要抢占故障转移，可以使用来将ceph-mgr守护程序明确标记为失败。`ceph mgr fail <mgr name>`

### 使用模块

使用命令查看哪些模块可用，哪些模块当前已启用。分别使用命令和 启用或禁用模块。`ceph mgr module lsceph mgr module enable <module>ceph mgr module disable <module>`

如果_启用_了模块，则活动的ceph-mgr守护程序将加载并执行它。在提供服务的模块（例如HTTP服务器）的情况下，模块可以在加载时发布其地址。要查看此类模块的地址，请使用命令 。`ceph mgr services`

一些模块还可以实现特殊的待机模式，该模式可以在待机ceph-mgr守护程序以及活动守护程序上运行。如果客户端尝试连接到备用服务器，这将使提供服务的模块将其客户端重定向到活动守护程序。

有关各个模块提供的功能的更多信息，请查阅各个管理器模块的文档页面。

这是启用[仪表板](https://docs.ceph.com/docs/nautilus/glossary/#term-dashboard)模块的示例：

```text
$ ceph mgr module ls
{
        "enabled_modules": [
                "restful",
                "status"
        ],
        "disabled_modules": [
                "dashboard"
        ]
}

$ ceph mgr module enable dashboard
$ ceph mgr module ls
{
        "enabled_modules": [
                "restful",
                "status",
                "dashboard"
        ],
        "disabled_modules": [
        ]
}

$ ceph mgr services
{
        "dashboard": "http://myserver.com:7789/",
        "restful": "https://myserver.com:8789/"
}
```

群集第一次启动时，它将使用该`mgr_initial_modules` 设置覆盖要启用的模块。但是，此设置在群集的整个生命周期中都将被忽略：仅将其用于引导。例如，在第一次启动监视守护程序之前，您可以在您的中添加如下所示的部分`ceph.conf`：

```text
[mon]
    mgr initial modules = dashboard balancer
```

### 调用模块的命令

在模块实现命令行挂钩的情况下，这些命令将像普通的Ceph命令一样可访问：

```text
ceph <command | help>
```

如果您想查看管理器处理的命令列表（正常情况下将显示所有mon和mgr命令），则可以将命令直接发送到管理器守护程序：`ceph help`

```text
ceph tell mgr help
```

请注意，不必寻址特定的mgr实例，只需`mgr`选择当前的活动守护程序即可。

### 配置

`mgr module path`描述

从中加载模块的路径类型

串默认

`"<library dir>/mgr"`

`mgr data`描述

加载守护程序数据的路径（例如密钥环）类型

串默认

`"/var/lib/ceph/mgr/$cluster-$id"`

`mgr tick period`描述

监视mgr信标与其他定期检查之间需要间隔多少秒。类型

整数默认

`5`

`mon mgr beacon grace`描述

最后一个信标应在多长时间后才被视为管理失败类型

整数默认

`30`

