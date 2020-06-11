# 安装CEPH包

## 包管理

### 安装

要在群集主机上安装Ceph软件包，请在客户端计算机上打开命令行，然后键入以下内容：

```text
ceph-deploy install {hostname [hostname] ...}
```

在没有其他参数的情况下，`ceph-deploy`会将Ceph的最新主要版本安装到群集主机。要指定特定的程序包，您可以从以下选项中进行选择：

* `--release <code-name>`
* `--testing`
* `--dev <branch-or-tag>`

例如：

```text
ceph-deploy install --release cuttlefish hostname1
ceph-deploy install --testing hostname2
ceph-deploy install --dev wip-some-branch hostname{1,2,3,4,5}
```

有关其他用法，请执行：

```text
ceph-deploy install -h
```

### 卸载

要从群集主机上卸载Ceph软件包，请在管理主机上打开一个终端，然后键入以下内容：

```text
ceph-deploy uninstall {hostname [hostname] ...}
```

在Debian或Ubuntu系统上，您还可以：

```text
ceph-deploy purge {hostname [hostname] ...}
```

该工具将从`ceph`指定主机上卸载软件包。清除还会删除配置文件。

