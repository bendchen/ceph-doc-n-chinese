# 配置CEPH

## 配置CEPH

当您启动Ceph服务时，初始化过程将激活一系列在后台运行的守护程序。一个[Ceph的存储集群](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-storage-cluster)上运行三种类型的守护程序：

* [Ceph监控器](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-monitor)（`ceph-mon`）
* [Ceph经理](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-manager)（`ceph-mgr`）
* [Ceph OSD守护程序](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-osd-daemon)（`ceph-osd`）

支持[Ceph文件系统的](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-filesystem) Ceph存储集群至少运行一台[Ceph元数据服务器](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-metadata-server)（`ceph-mds`）。支持[Ceph对象存储的](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-object-storage)集群运行Ceph Gateway守护程序（`radosgw`）。

每个守护程序都有一系列配置选项，每个配置选项都有一个默认值。您可以通过更改这些配置选项来调整系统的行为。

### 选项名称

所有Ceph配置选项都有一个唯一的名称，该名称由用小写字母组成的单词和下划线（`_`）字符组成。

在命令行上指定选项名称时，下划线（`_`）或破折号（`-`）字符可以互换使用（例如， `--mon-host`等效于`--mon_host`）。

当选项名称出现在配置文件中时，也可以使用空格代替下划线或破折号。

### 配置源

每个Ceph守护进程，进程和库都将从以下列出的几个来源中提取其配置。如果同时存在，则列表后面的源将覆盖列表前面的源。

* 编译的默认值
* 监视器集群的集中式配置数据库
* 存储在本地主机上的配置文件
* 环境变量
* 命令行参数
* 管理员设置的运行时替代

Ceph进程在启动时要做的第一件事就是解析通过命令行，环境和本地配置文件提供的配置选项。然后，该过程将与监视器群集联系，以检索整个群集集中存储的配置。一旦可获得完整的配置视图，则将继续执行守护程序或进程。

#### 引导选项

由于某些配置选项会影响进程与监视器联系，进行身份验证和检索集群存储的配置的能力，因此可能需要将它们本地存储在节点上并在本地配置文件中进行设置。这些选项包括：

> * `mon_host`，集群的监视器列表
> * `mon_dns_serv_name`（默认值：ceph-mon），DNS SRV记录的名称，该记录用于检查以通过DNS识别集群监视器
> * `mon_data`，`osd_data`，`mds_data`，`mgr_data`，以及定义其本地目录中的守护程序存储在其数据类似的选项。
> * `keyring`，`keyfile`和和/或`key`可以用来指定用于与监视器进行身份验证的身份验证凭据。请注意，在大多数情况下，默认密钥环位置在上面指定的数据目录中。

在大多数情况下，这些默认值都是合适的，但`mon_host`可以识别群集监视器地址的选项除外。使用DNS标识监视器时，可以完全避免使用本地ceph配置文件。

#### 跳过监视器配置

可以向任何进程传递选项，`--no-mon-config`以跳过从群集监视器检索配置的步骤。在完全通过配置文件管理配置或监视器群集当前处于关闭状态但需要完成一些维护活动的情况下，这很有用。

### 配置部分

任何给定的进程或守护程序的每个配置选项都有一个值。但是，选项的值可能在不同的守护程序类型之间变化，甚至是同一类型的守护程序也可能不同。存储在监视器配置数据库或本地配置文件中的Ceph选项分为几部分，以指示它们适用于哪些守护程序或客户端。

这些部分包括：

`global`描述

下的设置`global`会影响Ceph存储群集中的所有守护程序和客户端。例

`log_file = /var/log/ceph/$cluster-$type.$id.log`

`mon`描述

下的设置`mon`会影响`ceph-mon`Ceph Storage Cluster中的所有守护程序，并覆盖中的相同设置 `global`。例

`mon_cluster_log_to_syslog = true`

`mgr`描述

本`mgr`节中的设置会影响`ceph-mgr`Ceph Storage Cluster中的所有守护程序，并覆盖中的相同设置 `global`。例

`mgr_stats_period = 10`

`osd`描述

下的设置`osd`会影响`ceph-osd`Ceph Storage Cluster中的所有守护程序，并覆盖中的相同设置 `global`。例

`osd_op_queue = wpq`

`mds`描述

本`mds`节中的设置会影响`ceph-mds`Ceph Storage Cluster中的所有守护程序，并覆盖中的相同设置 `global`。例

`mds_cache_size = 10G`

`client`描述

下的设置`client`会影响所有Ceph客户端（例如，已安装的Ceph文件系统，已安装的Ceph块设备等）以及Rados Gateway（RGW）守护程序。例

`objecter_inflight_ops = 512`

部分还可以指定单个守护程序或客户端名称。例如 `mon.foo`，`osd.123`和`client.smith`都是有效的节名。

任何给定的守护程序都将从全局部分，守护程序或客户机类型部分以及共享其名称的部分中获取其设置。在最特定部分占先的设置，因此，例如，如果相同的选项在同时指定`global`，`mon`和 `mon.foo`相同的源（即，在相同的ConfigurationFile）上时，`mon.foo`将被使用的值。

请注意，本地配置文件中的值始终优先于监视器配置数据库中的值，而不管它们出现在哪个部分中。

### 元变量

元变量极大地简化了Ceph存储集群的配置。在配置值中设置了元变量后，Ceph会在使用配置值时将元变量扩展为具体值。Ceph元变量类似于Bash shell中的变量扩展。

Ceph支持以下元变量：

`$cluster`描述

扩展为Ceph存储群集名称。在同一硬件上运行多个Ceph存储群集时很有用。例

`/etc/ceph/$cluster.keyring`默认

`ceph`

`$type`描述

膨胀到守护程序或处理类型（例如，`mds`，`osd`，或`mon`）例

`/var/lib/ceph/$type`

`$id`描述

扩展为守护程序或客户端标识符。因为 `osd.0`这是`0`; 因为`mds.a`是这样`a`。例

`/var/lib/ceph/$type/$cluster-$id`

`$host`描述

扩展为运行进程的主机名。

`$name`描述

扩展到`$type.$id`。例

`/var/run/ceph/$cluster-$name.asok`

`$pid`描述

扩展为守护进程pid。例

`/var/run/ceph/$cluster-$name-$pid.asok`

### 配置文件

启动时，Ceph进程在以下位置搜索配置文件：

1. `$CEPH_CONF`（_即，_`$CEPH_CONF` 环境变量之后的路径）
2. `-c path/path` （_即，_该`-c`命令行参数）
3. `/etc/ceph/$cluster.conf`
4. `~/.ceph/$cluster.conf`
5. `./$cluster.conf`（_即_在当前工作目录中）
6. 仅在FreeBSD系统上， `/usr/local/etc/ceph/$cluster.conf`

`$cluster`集群的名称在哪里（默认`ceph`）。

Ceph配置文件使用_ini_样式的语法。您可以在注释之前添加井号（＃）或分号（;）。例如：

```text
# <--A number (#) sign precedes a comment.
; A comment may be anything.
# Comments always follow a semi-colon (;) or a pound (#) on each line.
# The end of the line terminates a comment.
# We recommend that you provide comments in your configuration file(s).
```

#### 配置文件节名

配置文件分为几部分。每个部分都必须以有效的配置部分名称开头（请参见上方的[配置部分）](https://docs.ceph.com/docs/nautilus/rados/configuration/ceph-conf/#configuration-sections)，并用方括号括起来。例如，

```text
[global]
debug ms = 0

[osd]
debug ms = 1

[osd.1]
debug ms = 10

[osd.2]
debug ms = 10
```

### 监视器配置数据库

监视群集管理整个群集可以使用的配置选项数据库，从而可以简化整个系统的中央配置管理。可以并且应该将绝大多数配置选项存储在此处，以简化管理和提高透明度。

少数设置可能仍需要存储在本地配置文件中，因为它们会影响连接到监视器，验证和获取配置信息的能力。在大多数情况下，这仅限于该`mon_host`选项，尽管也可以通过使用DNS SRV记录来避免这种情况。

#### 部分和遮罩

监视器存储的配置选项可以位于全局部分，守护程序类型部分或特定守护程序部分中，就像配置文件中的选项可以一样。

此外，选项还可能具有与之关联的_掩码_，以进一步限制该选项适用于哪些守护程序或客户端。口罩有两种形式：

1. `type:location`其中_type_是CRUSH属性，例如rack或 host，而_location_是该属性的值。例如， `host:foo`将选项仅限制为在特定主机上运行的守护程序或客户端。
2. `class:device-class`其中_device-class_是CRUSH设备类的名称（例如`hdd`或`ssd`）。例如， `class:ssd`将选项仅限于由SSD支持的OSD。（此掩码对非OSD守护程序或客户端无效。）

设置配置选项时，谁可以是节名，掩码或两者的组合，并用斜杠（`/`）字符分隔。例如，`osd/rack:foo`这意味着`foo`机架中的所有OSD守护程序。

在查看配置选项时，通常将节名称和掩码分隔为单独的字段或列，以简化可读性。

#### 命令

以下CLI命令用于配置群集：

* `ceph config dump` 将转储群集的整个配置数据库。
* `ceph config get <who>`将转储特定的守护程序或客户端（例如`mds.a`）的配置，该配置存储在监视器的配置数据库中。
* `ceph config set <who> <option> <value>` 将在监视器的配置数据库中设置一个配置选项。
* `ceph config show <who>`将显示报告的正在运行的守护程序的运行配置。如果还使用本地配置文件，或者在命令行或运行时覆盖了选项，则这些设置可能与监视器存储的设置不同。选项值的来源报告为输出的一部分。
* `ceph config assimilate-conf -i <input file> -o <output file>` 将从_输入文件中_提取配置文件，并将所有有效选项移至监视器的配置数据库。监视器无法识别，无效或无法控制的所有设置都将在_输出文件中_存储的简短配置文件中返回 。此命令对于从旧版配置文件过渡到基于集中式监视器的配置很有用。

### 帮助

您可以通过以下方式获得有关特定选项的帮助：

```text
ceph config help <option>
```

请注意，这将使用编译到正在运行的监视器中的配置架构。如果您具有混合版本的集群（例如，在升级过程中），则可能还需要从特定的正在运行的守护程序中查询选项架构：

```text
ceph daemon <name> config help [option]
```

例如，：

```text
$ ceph config help log_file
log_file - path to log file
  (std::string, basic)
  Default (non-daemon):
  Default (daemon): /var/log/ceph/$cluster-$name.log
  Can update at runtime: false
  See also: [log_to_stderr,err_to_stderr,log_to_syslog,err_to_syslog]
```

要么：

```text
$ ceph config help log_file -f json-pretty
{
    "name": "log_file",
    "type": "std::string",
    "level": "basic",
    "desc": "path to log file",
    "long_desc": "",
    "default": "",
    "daemon_default": "/var/log/ceph/$cluster-$name.log",
    "tags": [],
    "services": [],
    "see_also": [
        "log_to_stderr",
        "err_to_stderr",
        "log_to_syslog",
        "err_to_syslog"
    ],
    "enum_values": [],
    "min": "",
    "max": "",
    "can_update_at_runtime": false
}
```

该`level`属性可以是basic，advanced或dev之一。在开发选项供开发者使用，一般用于测试目的，而不是运营商推荐使用。

### 运行时更改

在大多数情况下，Ceph允许您在运行时更改守护程序的配置。此功能对于增加/减少日志记录输出，启用/禁用调试设置，甚至用于运行时优化非常有用。

一般而言，可以通过命令以常规方式更新配置选项。例如，在特定的OSD上启用调试日志级别，请执行以下操作：`ceph config set`

```text
ceph config set osd.123 debug_ms 20
```

请注意，如果在本地配置文件中也自定义了相同的选项，则将忽略监视器设置（其优先级低于本地配置文件）。

#### 覆盖值

您还可以使用 Ceph CLI上的tell或daemon界面临时设置一个选项。这些_替代_值是短暂的，因为它们仅影响正在运行的进程，并且在守护程序或进程重新启动时将被丢弃/丢失。

覆盖值可以通过两种方式设置：

1. 在任何主机上，我们都可以使用以下命令通过网络向守护进程发送消息：

   ```text
   ceph tell <name> config set <option> <value>
   ```

   例如，：

   ```text
   ceph tell osd.123 config set debug_osd 20
   ```

   该告诉命令也可以接受守护进程标识符通配符。例如，要调整所有OSD守护程序的调试级别，请执行以下操作：

   ```text
   ceph tell osd.* config set debug_osd 20
   ```

2. 从运行过程的主机，我们可以通过一个套接字直接连接到该过程`/var/run/ceph`：

   ```text
   ceph daemon <name> config set <option> <value>
   ```

   例如，：

   ```text
   ceph daemon osd.4 config set debug_osd 20
   ```

请注意，在命令输出中，这些临时值将显示为。`ceph config showoverride`

### 查看运行时设置

您可以使用以下命令查看为正在运行的守护程序设置的当前选项。例如，：`ceph config show`

```text
ceph config show osd.0
```

将显示该守护程序的（非默认）选项。您还可以通过以下方式查看特定选项：

```text
ceph config show osd.0 debug_osd
```

或使用以下命令查看所有选项（甚至是具有默认值的选项）：

```text
ceph config show-with-defaults osd.0
```

您也可以通过管理套接字从本地主机连接到正在运行的守护程序来观察其设置。例如，：

```text
ceph daemon osd.0 config show
```

将转储所有当前设置：

```text
ceph daemon osd.0 config diff
```

将仅显示非默认设置（以及值的来源：配置文件，监视器，替代等），以及：

```text
ceph daemon osd.0 config get debug_osd
```

将报告单个期权的价值。

