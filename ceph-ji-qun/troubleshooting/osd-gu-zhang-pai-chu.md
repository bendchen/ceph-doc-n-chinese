# OSD故障排除

## OSD故障排除

在对OSD进行故障排除之前，请先检查显示器和网络。如果执行或在命令行上执行并且Ceph返回健康状态，则意味着监视器具有法定人数。如果您没有监视器法定人数或监视器状态有误，请首先[解决监视器问题](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon)。检查网络以确保它们正常运行，因为网络可能会对OSD的运行和性能产生重大影响。`ceph healthceph -s`

### 获取有关OSD的数据

对OSD进行故障排除的一个很好的第一步是获取除[监视OSD时](https://docs.ceph.com/docs/nautilus/rados/operations/monitoring-osd-pg)收集的信息之外的信息 （例如）。`ceph osd tree`

#### CEPH日志

如果您尚未更改默认路径，则可以在`/var/log/ceph`以下位置找到Ceph日志文件 ：

```text
ls /var/log/ceph
```

如果没有足够的日志详细信息，则可以更改日志记录级别。有关详细信息，请参阅 [日志记录和调试](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/log-and-debug)，以确保Ceph在高日志量下能够正常运行。

#### 管理员套接字

使用管理套接字工具检索运行时信息。有关详细信息，请列出您的Ceph进程的套接字：

```text
ls /var/run/ceph
```

然后，执行以下操作，替换`{daemon-name}`为实际的守护程序（例如`osd.0`）：

```text
ceph daemon osd.0 help
```

或者，您可以指定一个`{socket-file}`（例如`/var/run/ceph`）。

```text
ceph daemon {socket-file} help
```

管理员套接字，除其他外，使您能够：

* 在运行时列出您的配置
* 转储历史操作
* 转储操作优先级队列状态
* 飞行中的转储作业
* 转储性能

#### 显示可用空间

文件系统问题可能会出现。要显示文件系统的可用空间，请执行 `df`。

```text
df -h
```

执行其他用途。`df --help`

#### I / O统计信息

使用[iostat](https://en.wikipedia.org/wiki/Iostat)可以识别与I / O相关的问题。

```text
iostat -x
```

#### 诊断消息

要获取诊断信息，使用`dmesg`与`less`，`more`，`grep` 或`tail`。例如：

```text
dmesg | grep scsi
```

### 停止重新平衡

您可能需要定期对群集的子集执行维护，或解决影响故障域（例如机架）的问题。如果您不希望CRUSH在停止OSD进行维护时自动重新平衡群集，请将群集设置为`noout`first：

```text
ceph osd set noout
```

将群集设置为后`noout`，您可以开始在需要维护工作的故障域内停止OSD。

```text
stop ceph-osd id={num}
```

注意 

`degraded` 解决故障域内的问题时，您停止的OSD中的放置组将变为。

完成维护后，请重新启动OSD。

```text
start ceph-osd id={num}
```

最后，您必须从取消设置集群`noout`。

```text
ceph osd unset noout
```

### OSD未运行

在正常情况下，只需重新启动`ceph-osd`守护程序，即可使其重新加入集群并进行恢复。

#### OSD无法启动

如果启动群集，但OSD无法启动，请检查以下内容：

* **配置文件：**如果无法从新安装中运行OSD，请检查您的配置文件以确保其符合要求（例如，`host`不兼容`hostname`，等等）。
* **检查路径：**检查配置中的路径以及数据和日记帐的实际路径本身。如果将OSD数据与日志数据分开，并且配置文件或实际安装中有错误，则可能无法启动OSD。如果要将日志存储在块设备上，则应对日志磁盘进行分区，并为每个OSD分配一个分区。
* **检查最大线程数：**如果您的节点上有很多OSD，则可能达到默认的最大线程数（例如，通常为32k），特别是在恢复期间。您可以使用`sysctl`来增加线程数，以查看将最大线程数增加到允许的最大可能线程数（即4194303）是否有帮助。例如：

  ```text
  sysctl -w kernel.pid_max=4194303
  ```

  如果增加最大线程数可以解决该问题，则可以通过`kernel.pid_max`在`/etc/sysctl.conf`文件中包含一个设置 来使其永久存在。例如：

  ```text
  kernel.pid_max = 4194303
  ```

* **内核版本：**确定您使用的内核版本和发行版。默认情况下，Ceph使用某些第三方工具，这些工具可能有错误，或者可能与某些发行版和/或内核版本（例如Google perftools）冲突。检查[操作系统建议，](https://docs.ceph.com/docs/nautilus/start/os-recommendations) 以确保已解决与内核有关的所有问题。
* **网段故障：**如果存在网段故障，请打开您的日志记录（如果尚未记录），然后重试。如果再次细分故障，请联系ceph-devel电子邮件列表，并提供您的Ceph配置文件，监视器输出以及日志文件的内容。

#### OSD失败

当`ceph-osd`进程终止时，监控器将从存活的`ceph-osd`守护程序中了解故障，并通过以下 命令进行报告：`ceph health`

```text
ceph health
HEALTH_WARN 1/3 in osds are down
```

具体来说，只要有`ceph-osd` 标记为`in`和的进程，您都会收到警告`down`。您可以识别哪些 `ceph-osds`是`down`有：

```text
ceph health detail
HEALTH_WARN 1/3 in osds are down
osd.0 is down since epoch 23, last address 192.168.106.220:6800/11080
```

如果存在磁盘故障或其他故障导致`ceph-osd`无法运行或重新启动，则其日志文件中的错误消息应该出现在中 `/var/log/ceph`。

如果守护程序由于心跳失败而停止，则底层内核文件系统可能无响应。检查`dmesg`输出中是否有磁盘或其他内核错误。

如果问题是软件错误（断言失败或其他意外错误），则应将其报告给[ceph-devel](mailto:ceph-devel%40vger.kernel.org)电子邮件列表。

#### 没有驱动器空间

Ceph阻止您写入完整的OSD，以免丢失数据。在可操作的群集中，当群集接近其满负荷比率时，您应该收到警告。该默认 ，或容量的95％之前停止客户端写入数据。的默认为，或90％的容量时，从它的块回填开始。OSD 生成运行状况警告时，其默认接近满比率为或容量的85％。`mon osd full ratio0.95mon osd backfillfull ratio0.900.85`

更改它可以使用：

```text
ceph osd set-nearfull-ratio <float[0.0-1.0]>
```

当测试Ceph如何处理小型集群上的OSD故障时，通常会出现全集群问题。当一个节点占群集数据的百分比很高时，群集可以轻易地使其接近全满和全满的比例黯然失色。如果要测试Ceph对小型集群上OSD故障的反应，则应保留足够的可用磁盘空间，并考虑使用以下命令暂时降低OSD ，OSD 和OSD ：`full ratiobackfillfull rationearfull ratio`

```text
ceph osd set-nearfull-ratio <float[0.0-1.0]>
ceph osd set-full-ratio <float[0.0-1.0]>
ceph osd set-backfillfull-ratio <float[0.0-1.0]>
```

完整`ceph-osds`报告将由：`ceph health`

```text
ceph health
HEALTH_WARN 1 nearfull osd(s)
```

要么：

```text
ceph health detail
HEALTH_ERR 1 full osd(s); 1 backfillfull osd(s); 1 nearfull osd(s)
osd.3 is full at 97%
osd.4 is backfill full at 91%
osd.2 is near full at 87%
```

处理整个集群的最好方法是添加new `ceph-osds`，使集群可以将数据重新分配到新的可用存储中。

如果由于OSD已满而无法启动OSD，则可以通过删除完整OSD中的某些放置组目录来删除某些数据。

重要 

如果选择删除完整OSD上的放置组目录，请 **不要**删除另一个完整OSD上的相同放置组目录，否则， **您可能会丢失数据**。您**必须**在至少一个OSD上维护至少一个数据副本。

有关其他详细信息，请参见《[Monitor Config Reference](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-config-ref)》。

### OSD缓慢/无响应

经常发生的问题涉及OSD缓慢或无响应。在研究OSD性能问题之前，请确保消除了其他故障排除可能性。例如，确保您的网络正常运行并且OSD正在运行。检查OSD是否限制了恢复流量。

小费 

较新版本的Ceph通过防止恢复OSD消耗系统资源来提供更好的恢复处理，从而使 OSD `up`和`in`OSD不可用或变慢。

#### 网络问题

Ceph是一个分布式存储系统，因此它依赖于网络与OSD对等，复制对象，从故障中恢复并检查心跳。网络问题可能会导致OSD延迟和OSD波动。有关详细信息，请参见[拍打OSD](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-osd/#flapping-osds)。

确保已连接和/或侦听Ceph进程和依赖于Ceph的进程。

```text
netstat -a | grep ceph
netstat -l | grep ceph
sudo netstat -p | grep ceph
```

检查网络统计信息。

```text
netstat -s
```

#### 驱动器配置

一个存储驱动器应仅支持一个OSD。如果其他进程共享驱动器，则顺序读取和顺序写入吞吐量会成为瓶颈，包括日志，操作系统，监视器，其他OSD和非Ceph进程。

Ceph会_在_日志记录_后_确认写入，因此快速的SSD是缩短响应时间的诱人选择，尤其是在使用`XFS`or `ext4`文件系统时。相比之下，`btrfs` 文件系统可以同时写入和记录日志。（但是请注意，我们建议您不要将其`btrfs`用于生产部署。）

注意 

对驱动器进行分区不会更改其总吞吐量或顺序的读/写限制。在单独的分区中运行日志可能会有所帮助，但是您应该首选单独的物理驱动器。

#### 坏扇区/磁盘碎片

检查磁盘是否有坏扇区和碎片。这可能导致总吞吐量大幅下降。

#### 共驻留的显示器/屏上显示

监视器通常是轻量级进程，但是它们会做很多事情`fsync()`，这可能会干扰其他工作负载，尤其是如果监视器与OSD在同一驱动器上运行时。此外，如果在与OSD相同的主机上运行监视器，则可能会引起与以下问题有关的性能问题：

* 运行较旧的内核（3.0之前的版本）
* 运行没有syncfs（2）系统调用的内核。

在这些情况下，运行在同一主机上的多个OSD可以通过进行大量提交而相互拖累。这通常会导致突发写入。

#### 流程

加速共存流程，例如基于云的解决方案，虚拟机和其他将数据写入Ceph，同时在与OSD相同的硬件上运行的应用程序，可能会导致显着的OSD延迟。通常，我们建议优化用于Ceph的主机，以及将其他主机用于其他进程。将Ceph操作与其他应用程序分离的做法可能有助于提高性能，并简化故障排除和维护。

#### 日志记录级别

如果您将日志记录级别调高以跟踪问题，然后又忘记调低日志记录级别，则OSD可能会将大量日志放到磁盘上。如果您打算保持较高的日志记录级别，则可以考虑将驱动器安装到默认的日志记录路径（即`/var/log/ceph/$cluster-$name.log`）。

#### 恢复节流

根据您的配置，Ceph可能会降低恢复率以保持性能，或者可能将恢复率提高到恢复影响OSD性能的程度。检查OSD是否正在恢复。

#### 内核版本

检查您正在运行的内核版本。较早的内核可能不会收到Ceph依赖其以提高性能的新反向端口。

#### SYNCFS的内核问题

尝试每台主机运行一个OSD，以查看性能是否有所提高。旧内核可能没有足够的最新版本`glibc`来支持`syncfs(2)`。

#### 文件系统问题

当前，我们建议使用XFS部署群集。

我们建议不要使用btrfs或ext4。btrfs文件系统具有许多吸引人的功能，但是文件系统中的错误可能会导致性能问题和虚假的ENOSPC错误。我们不建议使用ext4，因为xattr的大小限制会破坏我们对长对象名的支持（RGW需要）。

有关更多信息，请参见[文件系统建议](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/configuration/filesystem-recommendations)。

#### RAM不足

我们建议每个OSD守护程序1GB的RAM。您可能会注意到，在正常操作期间，OSD仅使用该数量的一小部分（例如100-200MB）。未使用的RAM倾向于将多余的RAM用于共同驻留的应用程序，VM等。但是，当OSD进入恢复模式时，其内存利用率会飙升。如果没有可用的RAM，则OSD性能将大大降低。

#### 旧请求或慢请求

如果`ceph-osd`守护程序对请求的响应速度很慢，它将生成日志消息，抱怨请求时间过长。警告阈值默认为30秒，可以通过该 选项进行配置。发生这种情况时，群集日志将接收消息。`osd op complaint time`

旧版Ceph抱怨：`old requests`

```text
osd.0 192.168.106.220:6800/18813 312 : [WRN] old request osd_op(client.5099.0:790 fatty_26485_object789 [write 0~4096] 2.5e54f643) v4 received at 2012-03-06 15:42:56.054801 currently waiting for sub ops
```

新版本的Ceph抱怨：`slow requests`

```text
{date} {osd.num} [WRN] 1 slow requests, 1 included below; oldest blocked for > 30.005692 secs
{date} {osd.num}  [WRN] slow request 30.005692 seconds old, received at {date-time}: osd_op(client.4240.0:8 benchmark_data_ceph-1_39426_object7 [write 0~4194304] 0.69848840) v4 currently waiting for subops from [610]
```

可能的原因包括：

* 驱动器损坏（检查`dmesg`输出）
* 内核文件系统中的错误（检查`dmesg`输出）
* 集群过载（检查系统负载，iostat等）
* `ceph-osd`守护程序中的错误。

可能的解决方案：

* 从Ceph主机中删除VM
* 升级内核
* 升级Ceph
* 重新启动OSD

#### 调试慢请求

如果运行或，您将看到一组操作以及每个操作经历的事件列表。这些将在下面简要描述。`ceph daemon osd.<id> dump_historic_opsceph daemon osd.<id> dump_ops_in_flight`

Messenger层中的事件：

* `header_read`：当Messenger第一次开始从网络上读取消息时。
* `throttled`：当Messenger尝试获取内存限制空间以将消息读入内存时。
* `all_read`：即时通讯程序完成从线下阅读消息后。
* `dispatched`：当Messenger将消息发送给OSD时。
* `initiated`：等同于`header_read`。两者的存在是一个历史古怪。

OSD准备操作时的事件：

* `queued_for_pg`：操作已由其PG放入队列进行处理。
* `reached_pg`：PG已开始执行操作。
* `waiting for \*`：操作在继续进行之前正在等待其他工作完成（例如，新的OSDMap；要清理其对象目标；让PG完成对等；所有消息均已指定）。
* `started`：操作已被OSD接受并且正在执行。
* `waiting for subops from`：操作已发送到副本OSD。

FileStore中的事件：

* `commit_queued_for_journal_write`：操作已分配给FileStore。
* `write_thread_in_journal_buffer`：操作位于日志的缓冲区中并等待被保留（作为下一个磁盘写入操作）。
* `journaled_completion_queued`：操作已记录到磁盘，并且其回调已排队等待调用。

将东西分配给本地磁盘后，来自OSD的事件：

* `op_commit`：操作已由主OSD提交（即写入日志）。
* `op_applied`：操作已[写入主服务器](https://www.freebsd.org/cgi/man.cgi?write%282%29)上的备份FS（即应用于内存，但未刷新到磁盘）。
* `sub_op_applied`：`op_applied`，但适用于副本的“子操作”。
* `sub_op_committed`：`op_commit`，但适用于副本的子操作（仅适用于EC池）。
* `sub_op_commit_rec/sub_op_apply_rec from <X>`：主要服务器在听到上述情况时会对此进行标记，但针对特定副本（即`<X>`）。
* `commit_sent`：我们已将回复发送回客户端（或主OSD，用于子操作）。

这些事件中的许多事件似乎都是多余的，但它们跨越了内部代码中的重要边界（例如，将数据跨锁传递到新线程中）。

### 扑屏上显示

我们建议同时使用公用（前端）网络和群集（后端）网络，以便更好地满足对象复制的容量要求。另一个优点是您可以运行群集网络，使其不连接到Internet，从而防止某些拒绝服务攻击。当OSD对等并检查心跳时，他们会使用群集（后端）网络（如果可用）。有关详细信息，请参见[Monitor / OSD交互](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-osd-interaction)。

但是，如果在公用（前端）网络最佳运行的同时，群集（后端）网络出现故障或出现明显的延迟，则OSD当前无法很好地处理这种情况。发生的事情是OSD `down` 在监视器上互相标记，同时对其进行标记`up`。我们称这种情况为“拍打”。

如果某种原因导致OSD发生“拍打”（反复被标记`down`，然后`up`再次出现），则可以通过以下方法强制显示器停止拍打：

```text
ceph osd set noup      # prevent OSDs from getting marked up
ceph osd set nodown    # prevent OSDs from getting marked down
```

这些标志记录在osdmap结构中：

```text
ceph osd dump | grep flags
flags no-up,no-down
```

您可以使用以下方法清除标志：

```text
ceph osd unset noup
ceph osd unset nodown
```

还支持另外两个标记`noin`和`noout`，这些标记防止引导OSD被标记`in`（分配的数据）或保护OSD最终不被标记`out`（无论当前值是什么 ）。`mon osd down out interval`

注意 

`noup`，`noout`和`nodown`是临时的，因为一旦清除了这些标志，它们阻塞的操作应在不久之后发生。`noin`另一方面，该标志可防止OSD `in`在引导时被标记，并且在设置该标志时启动的所有守护程序都将保持该状态。

