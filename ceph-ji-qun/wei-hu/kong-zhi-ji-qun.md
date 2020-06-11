# 控制集群

## 运行群集

### 与SYSTEMD运行CEPH

对于所有支持systemd的发行版（CentOS 7，Fedora，Debian Jessie 8及更高版本，SUSE），现在都使用本地systemd文件而不是旧版sysvinit脚本来管理ceph守护程序。例如：

```text
sudo systemctl start ceph.target       # start all daemons
sudo systemctl status ceph-osd@12      # check status of osd.12
```

要列出节点上的Ceph systemd单元，请执行：

```text
sudo systemctl status ceph\*.service ceph\*.target
```

#### 启动所有守护程序

要在Ceph节点上启动所有守护进程（与类型无关），请执行以下操作：

```text
sudo systemctl start ceph.target
```

#### 停止所有的守护程序

要停止Ceph节点上的所有守护程序（与类型无关），请执行以下操作：

```text
sudo systemctl stop ceph\*.service ceph\*.target
```

#### 开始按类型所有的守护程序

要在Ceph节点上启动特定类型的所有守护程序，请执行以下操作之一：

```text
sudo systemctl start ceph-osd.target
sudo systemctl start ceph-mon.target
sudo systemctl start ceph-mds.target
```

#### 按类型停止所有守护程序

要停止Ceph节点上所有特定类型的守护程序，请执行以下操作之一：

```text
sudo systemctl stop ceph-mon\*.service ceph-mon.target
sudo systemctl stop ceph-osd\*.service ceph-osd.target
sudo systemctl stop ceph-mds\*.service ceph-mds.target
```

#### 启动守护进程

要在Ceph节点上启动特定的守护程序实例，请执行以下操作之一：

```text
sudo systemctl start ceph-osd@{id}
sudo systemctl start ceph-mon@{hostname}
sudo systemctl start ceph-mds@{hostname}
```

例如：

```text
sudo systemctl start ceph-osd@1
sudo systemctl start ceph-mon@ceph-server
sudo systemctl start ceph-mds@ceph-server
```

#### 停止守护程序

要在Ceph节点上停止特定的守护程序实例，请执行以下操作之一：

```text
sudo systemctl stop ceph-osd@{id}
sudo systemctl stop ceph-mon@{hostname}
sudo systemctl stop ceph-mds@{hostname}
```

例如：

```text
sudo systemctl stop ceph-osd@1
sudo systemctl stop ceph-mon@ceph-server
sudo systemctl stop ceph-mds@ceph-server
```

#### 启动所有守护程序

要在Ceph节点上启动所有守护进程（与类型无关），请执行以下操作：

```text
sudo start ceph-all
```

#### 停止所有的守护程序

要停止Ceph节点上的所有守护程序（与类型无关），请执行以下操作：

```text
sudo stop ceph-all
```

#### 开始按类型所有的守护程序

要在Ceph节点上启动特定类型的所有守护程序，请执行以下操作之一：

```text
sudo start ceph-osd-all
sudo start ceph-mon-all
sudo start ceph-mds-all
```

#### 按类型停止所有守护程序

要停止Ceph节点上所有特定类型的守护程序，请执行以下操作之一：

```text
sudo stop ceph-osd-all
sudo stop ceph-mon-all
sudo stop ceph-mds-all
```

#### 启动守护进程

要在Ceph节点上启动特定的守护程序实例，请执行以下操作之一：

```text
sudo start ceph-osd id={id}
sudo start ceph-mon id={hostname}
sudo start ceph-mds id={hostname}
```

例如：

```text
sudo start ceph-osd id=1
sudo start ceph-mon id=ceph-server
sudo start ceph-mds id=ceph-server
```

#### 停止守护程序

要在Ceph节点上停止特定的守护程序实例，请执行以下操作之一：

```text
sudo stop ceph-osd id={id}
sudo stop ceph-mon id={hostname}
sudo stop ceph-mds id={hostname}
```

例如：

```text
sudo stop ceph-osd id=1
sudo start ceph-mon id=ceph-server
sudo start ceph-mds id=ceph-server
```

### 运行CEPH

每次**启动**，**重新启动**和 **停止** Ceph守护程序（或整个集群）时，都必须至少指定一个选项和一个命令。您也可以指定守护程序类型或守护程序实例。

```text
{commandline} [options] [commands] [daemons]
```

该`ceph`选项包括：

| 选项 | 捷径 | 描述 |
| :--- | :--- | :--- |
| `--verbose` | `-v` | 使用详细日志记录。 |
| `--valgrind` | `N/A` | （仅限Dev和QA）使用[Valgrind](http://www.valgrind.org/)调试。 |
| `--allhosts` | `-a` | 在`ceph.conf.` 否则的所有节点上执行，否则仅在上执行`localhost`。 |
| `--restart` | `N/A` | 如果核心转储，则自动重新启动守护程序。 |
| `--norestart` | `N/A` | 如果守护程序发生核心转储，请不要重新启动它。 |
| `--conf` | `-c` | 使用备用配置文件。 |

这些`ceph`命令包括：

| 命令 | 描述 |
| :--- | :--- |
| `start` | 启动守护程序。 |
| `stop` | 停止守护程序。 |
| `forcestop` | 强制守护程序停止。如同`kill -9` |
| `killall` | 杀死所有特定类型的守护程序。 |
| `cleanlogs` | 清除日志目录。 |
| `cleanalllogs` | 清除日志目录中的**所有内容**。 |

对于子系统操作，`ceph`服务可以通过为`[daemons]`选项添加特定的守护程序类型来定位特定的守护程序类型。守护程序类型包括：

* `mon`
* `osd`
* `mds`

