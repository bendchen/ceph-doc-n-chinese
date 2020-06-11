# 日志记录和调试

## 日志记录和调试

通常，将调试添加到Ceph配置时，是在运行时执行的。如果在启动集群时遇到问题，还可以将Ceph调试日志记录添加到Ceph配置文件中。您可以在`/var/log/ceph`（默认位置）下查看Ceph日志文件。

小费 

当调试输出降低系统速度时，延迟会隐藏竞争条件。

日志记录会占用大量资源。如果您在群集的特定区域遇到问题，请启用该群集区域的日志记录。例如，如果您的OSD运行良好，但元数据服务器运行不正常，则应首先为特定的元数据服务器实例启用调试日志记录，这会给您带来麻烦。根据需要为每个子系统启用日志记录。

重要 

详细的日志记录每小时可以生成1GB以上的数据。如果您的操作系统磁盘已满，该节点将停止工作。

如果启用或提高Ceph日志记录速率，请确保OS磁盘上有足够的磁盘空间。有关旋转日志文件的详细信息，请参见[加速日志循环](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/log-and-debug/#accelerating-log-rotation)。当系统运行良好时，请删除不必要的调试设置，以确保集群以最佳状态运行。记录调试输出消息相对较慢，并且在操作集群时浪费资源。

有关可用设置的详细信息，请参见[子系统，日志和调试](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/log-and-debug/#subsystem-log-and-debug-settings)设置。

### 运行时

如果要在运行时查看配置设置，则必须使用正在运行的守护程序登录到主机并执行以下操作：

```text
ceph daemon {daemon-name} config show | less
```

例如，：

```text
ceph daemon osd.0 config show | less
```

要激活Ceph的的调试输出（_即_，`dout()`在运行时），可以使用 命令注入参数到运行时配置：`ceph tell`

```text
ceph tell {daemon-type}.{daemon id or *} config set {name} {value}
```

更换`{daemon-type}`用的一个`osd`，`mon`或`mds`。您可以使用来将运行时设置应用于特定类型的所有守护程序`*`，或指定特定守护程序的ID。例如，要增加`ceph-osd`名为的守护程序的调试日志记录`osd.0`，请执行以下操作：

```text
ceph tell osd.0 config set debug_osd 0/5
```

该命令通过监视器。如果您无法绑定到监视器，则仍然可以通过使用来登录要更改其配置的守护程序的主机，以进行更改。例如：`ceph tellceph daemon`

```text
sudo ceph daemon osd.0 config set debug_osd 0/5
```

有关可用设置的详细信息，请参见[子系统，日志和调试](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/log-and-debug/#subsystem-log-and-debug-settings)设置。

### 启动时间

要激活头孢的调试输出（_即_，`dout()`在启动时），您必须对您的Ceph的配置文件中添加设置。每个守护程序共有的子系统可以`[global]`在配置文件中设置。对于特定的守护程序子系统在配置文件下的守护程序段设置（_例如_，`[mon]`，`[osd]`，`[mds]`）。例如：

```text
[global]
        debug ms = 1/5

[mon]
        debug mon = 20
        debug paxos = 1/5
        debug auth = 2

[osd]
        debug osd = 1/5
        debug filestore = 1/5
        debug journal = 1
        debug monc = 5/20

[mds]
        debug mds = 1
        debug mds balancer = 1
```

有关详细信息[，](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/log-and-debug/#subsystem-log-and-debug-settings)请参见[子系统，日志和调试设置](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/log-and-debug/#subsystem-log-and-debug-settings)。

### 加速日志旋转

如果您的操作系统磁盘相对较满，则可以通过在处修改Ceph日志轮换文件来加速日志轮换`/etc/logrotate.d/ceph`。如果日志超出大小设置，请在旋转频率之后添加大小设置以加速日志旋转（通过cronjob）。例如，默认设置如下所示：

```text
rotate 7
weekly
compress
sharedscripts
```

通过添加`size`设置进行修改。

```text
rotate 7
weekly
size 500M
compress
sharedscripts
```

然后，启动您的用户空间的crontab编辑器。

```text
crontab -e
```

最后，添加一个条目以检查`etc/logrotate.d/ceph`文件。

```text
30 * * * * /usr/sbin/logrotate /etc/logrotate.d/ceph >/dev/null 2>&1
```

前面的示例`etc/logrotate.d/ceph`每30分钟检查一次文件。

### VALGRIND的

调试可能还需要您跟踪内存和线程问题。您可以使用Valgrind运行单个守护程序，一种守护程序或整个集群。您仅应在开发或调试Ceph时使用Valgrind。Valgrind在计算上很昂贵，否则会降低您的系统速度。Valgrind消息已记录到`stderr`。

### 子系统，日志和调试设置

在大多数情况下，您将通过子系统启用调试日志记录输出。

#### CEPH子系统

每个子系统都有其输出日志和内存中日志的日志记录级别。通过为调试日志记录设置日志文件级别和内存级别，可以为每个子系统设置不同的值。Ceph的日志记录级别以`1`to为单位`20`，其中`1`简洁且`20`冗长[1](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/log-and-debug/#id2)。通常，除非以下情况，否则不会将内存中的日志发送到输出日志：

* 发出致命信号或
* 一个`assert`在源代码被触发或
* 根据要求。请参阅[管理套接字上的文档以](http://docs.ceph.com/docs/master/man/8/ceph/#daemon)获取更多详细信息。

调试日志记录设置可以为日志级别和内存级别采用单个值，这会将它们设置为相同的值。例如，如果您指定，Ceph会将其视为日志级别和内存级别。您也可以单独指定它们。第一个设置是日志级别，第二个设置是内存级别。您必须使用正斜杠（/）分隔它们。例如，如果要将子系统的调试日志记录级别设置为，将其内存级别设置为，则可以将其指定为。例如：`debug ms = 55ms15debug ms = 1/5`

```text
debug {subsystem} = {log-level}/{memory-level}
#for example
debug mds balancer = 1/20
```

下表提供了Ceph子系统及其默认日志和内存级别的列表。完成日志记录工作后，将子系统还原到默认级别或适合正常操作的级别。

| 子系统 | 日志级别 | 记忆体水平 |
| :--- | :--- | :--- |
| `default` | 0 | 5 |
| `lockdep` | 0 | 1个 |
| `context` | 0 | 1个 |
| `crush` | 1个 | 1个 |
| `mds` | 1个 | 5 |
| `mds balancer` | 1个 | 5 |
| `mds locker` | 1个 | 5 |
| `mds log` | 1个 | 5 |
| `mds log expire` | 1个 | 5 |
| `mds migrator` | 1个 | 5 |
| `buffer` | 0 | 1个 |
| `timer` | 0 | 1个 |
| `filer` | 0 | 1个 |
| `striper` | 0 | 1个 |
| `objecter` | 0 | 1个 |
| `rados` | 0 | 5 |
| `rbd` | 0 | 5 |
| `rbd mirror` | 0 | 5 |
| `rbd replay` | 0 | 5 |
| `journaler` | 0 | 5 |
| `objectcacher` | 0 | 5 |
| `client` | 0 | 5 |
| `osd` | 1个 | 5 |
| `optracker` | 0 | 5 |
| `objclass` | 0 | 5 |
| `filestore` | 1个 | 3 |
| `journal` | 1个 | 3 |
| `ms` | 0 | 5 |
| `mon` | 1个 | 5 |
| `monc` | 0 | 10 |
| `paxos` | 1个 | 5 |
| `tp` | 0 | 5 |
| `auth` | 1个 | 5 |
| `crypto` | 1个 | 5 |
| `finisher` | 1个 | 1个 |
| `reserver` | 1个 | 1个 |
| `heartbeatmap` | 1个 | 5 |
| `perfcounter` | 1个 | 5 |
| `rgw` | 1个 | 5 |
| `rgw sync` | 1个 | 5 |
| `civetweb` | 1个 | 10 |
| `javaclient` | 1个 | 5 |
| `asok` | 1个 | 5 |
| `throttle` | 1个 | 1个 |
| `refs` | 0 | 0 |
| `compressor` | 1个 | 5 |
| `bluestore` | 1个 | 5 |
| `bluefs` | 1个 | 5 |
| `bdev` | 1个 | 3 |
| `kstore` | 1个 | 5 |
| `rocksdb` | 4 | 5 |
| `leveldb` | 4 | 5 |
| `memdb` | 4 | 5 |
| `fuse` | 1个 | 5 |
| `mgr` | 1个 | 5 |
| `mgrc` | 1个 | 5 |
| `dpdk` | 1个 | 5 |
| `eventtrace` | 1个 | 5 |

#### 记录设置

Ceph配置文件中不需要记录和调试设置，但是您可以根据需要覆盖默认设置。Ceph支持以下设置：

`log file`描述

集群的日志文件的位置。类型

串需要

没有默认

`/var/log/ceph/$cluster-$name.log`

`log max new`描述

新日志文件的最大数量。类型

整数需要

没有默认

`1000`

`log max recent`描述

日志文件中包含的最近事件的最大数量。类型

整数需要

没有默认

`10000`

`log to stderr`描述

确定日志记录消息是否应出现在中`stderr`。类型

布尔型需要

没有默认

`true`

`err to stderr`描述

确定是否在中显示错误消息`stderr`。类型

布尔型需要

没有默认

`true`

`log to syslog`描述

确定日志记录消息是否应出现在中`syslog`。类型

布尔型需要

没有默认

`false`

`err to syslog`描述

确定是否在中显示错误消息`syslog`。类型

布尔型需要

没有默认

`false`

`log flush on exit`描述

确定Ceph在退出后是否应刷新日志文件。类型

布尔型需要

没有默认

`true`

`clog to monitors`描述

确定是否`clog`应将消息发送到监视器。类型

布尔型需要

没有默认

`true`

`clog to syslog`描述

确定是否`clog`应将消息发送到syslog。类型

布尔型需要

没有默认

`false`

`mon cluster log to syslog`描述

确定是否将群集日志输出到系统日志。类型

布尔型需要

没有默认

`false`

`mon cluster log file`描述

群集日志文件的位置。Ceph中有两个渠道：`cluster`和`audit`。此选项表示从通道到日志文件的映射，该通道的日志条目发送到该文件。该`default`条目是未明确指定的通道的后备映射。因此，以下默认设置会将群集日志发送到`$cluster.log`，并将审核日志发送到`$cluster.audit.log`，其中`$cluster`将被替换为实际的群集名称。类型

串需要

没有默认

`default=/var/log/ceph/$cluster.$channel.log,cluster=/var/log/ceph/$cluster.log`

#### OSD 

`osd debug drop ping probability`描述

？类型

双需要

没有默认

0

`osd debug drop ping duration`描述

类型

整数需要

没有默认

0

`osd debug drop pg create probability`描述

类型

整数需要

没有默认

0

`osd debug drop pg create duration`描述

？类型

双需要

没有默认

1个

`osd min pg log entries`描述

放置组的最小日志条目数。类型

32位无符号整数需要

没有默认

1000

`osd op log threshold`描述

一次显示多少个操作日志消息。类型

整数需要

没有默认

5

#### 文件存储

`filestore debug omap check`描述

同步上的调试检查。这是一个昂贵的操作。类型

布尔型需要

没有默认

`false`

#### MDS 

`mds debug scatterstat`描述

Ceph将断言各种递归stat不变量是正确的（仅适用于开发人员）。类型

布尔型需要

没有默认

`false`

`mds debug frag`描述

Ceph会在方便时验证目录碎片不变式（仅适用于开发人员）。类型

布尔型需要

没有默认

`false`

`mds debug auth pins`描述

调试auth引脚不变式（仅适用于开发人员）。类型

布尔型需要

没有默认

`false`

`mds debug subtrees`描述

调试子树不变式（仅适用于开发人员）。类型

布尔型需要

没有默认

`false`

#### RADOS网关

`rgw log nonexistent bucket`描述

我们应该记录一个不存在的存储桶吗？类型

布尔型需要

没有默认

`false`

`rgw log object name`描述

是否应该记录对象的名称。//看日期的人工日期（支持子集）类型

串需要

没有默认

`%Y-%m-%d-%H-%i-%n`

`rgw log object name utc`描述

对象日志名称包含UTC？类型

布尔型需要

没有默认

`false`

`rgw enable ops log`描述

启用每个RGW操作的日志记录。类型

布尔型需要

没有默认

`true`

`rgw enable usage log`描述

启用RGW的带宽使用记录。类型

布尔型需要

没有默认

`false`

`rgw usage log flush threshold`描述

刷新未决日志数据的阈值。类型

整数需要

没有默认

`1024`

`rgw usage log tick interval`描述

每秒钟刷新待处理的日志数据`s`。类型

整数需要

没有默认

30

`rgw intent log object name`描述

类型

串需要

没有默认

`%Y-%m-%d-%i-%n`

`rgw intent log object name utc`描述

在意图日志对象名称中包含UTC时间戳。类型

布尔型需要

没有默认

`false`[1个](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/log-and-debug/#id1)

在极少数情况下，级别&gt; 20且非常冗长。

