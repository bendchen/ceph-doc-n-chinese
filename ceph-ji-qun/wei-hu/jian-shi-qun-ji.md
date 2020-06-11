# 监视群集

## 监视群集

拥有正在运行的群集后，可以使用该`ceph`工具监视群集。监视群集通常涉及检查OSD状态，监视器状态，放置组状态和元数据服务器状态。

### 使用命令行

#### 交互模式

要`ceph`以交互方式运行该工具，请`ceph`在命令行中键入，不带参数。例如：

```text
ceph
ceph> health
ceph> status
ceph> quorum_status
ceph> mon_status
```

#### 非默认路径

如果您为配置或密钥环指定了非默认位置，则可以指定它们的位置：

```text
ceph -c /path/to/conf -k /path/to/keyring health
```

### 检查集群状态

在启动集群之后，以及在开始读取和/或写入数据之前，请首先检查集群的状态。

要检查集群的状态，请执行以下操作：

```text
ceph status
```

要么：

```text
ceph -s
```

在交互模式下，键入`status`，然后按**Enter**。

```text
ceph> status
```

Ceph将打印集群状态。例如，一个带有每个服务之一的微型Ceph示范集群可能会打印以下内容：

```text
cluster:
  id:     477e46f1-ae41-4e43-9c8f-72c918ab0a20
  health: HEALTH_OK

services:
  mon: 3 daemons, quorum a,b,c
  mgr: x(active)
  mds: cephfs_a-1/1/1 up  {0=a=up:active}, 2 up:standby
  osd: 3 osds: 3 up, 3 in

data:
  pools:   2 pools, 16 pgs
  objects: 21 objects, 2.19K
  usage:   546 GB used, 384 GB / 931 GB avail
  pgs:     16 active+clean
```

Ceph如何计算数据使用量

该`usage`值反映_实际_使用的原始存储量。该 值表示群集的整体存储容量的可用量（较少的数量）。名义数字反映了复制，克隆或快照之前存储的数据的大小。因此，实际存储的数据量通常超过名义存储量，因为Ceph会创建数据的副本，并且还可能使用存储容量进行克隆和快照。`xxx GB / xxx GB`

### 看着群集

除了每个守护程序的本地日志记录之外，Ceph集群还维护一个_集群日志_，该_日志_记录有关整个系统的高级事件。它被记录到监视服务器上的磁盘（`/var/log/ceph/ceph.log`默认情况下），但是也可以通过命令行监视。

要跟踪群集日志，请使用以下命令

```text
ceph -w
```

Ceph将打印系统状态，并在发出每条日志消息后打印。例如：

```text
cluster:
  id:     477e46f1-ae41-4e43-9c8f-72c918ab0a20
  health: HEALTH_OK

services:
  mon: 3 daemons, quorum a,b,c
  mgr: x(active)
  mds: cephfs_a-1/1/1 up  {0=a=up:active}, 2 up:standby
  osd: 3 osds: 3 up, 3 in

data:
  pools:   2 pools, 16 pgs
  objects: 21 objects, 2.19K
  usage:   546 GB used, 384 GB / 931 GB avail
  pgs:     16 active+clean


2017-07-24 08:15:11.329298 mon.a mon.0 172.21.9.34:6789/0 23 : cluster [INF] osd.0 172.21.9.34:6806/20527 boot
2017-07-24 08:15:14.258143 mon.a mon.0 172.21.9.34:6789/0 39 : cluster [INF] Activating manager daemon x
2017-07-24 08:15:15.446025 mon.a mon.0 172.21.9.34:6789/0 47 : cluster [INF] Manager daemon x is now available
```

除了用于打印发出的日志行之外，还用于查看集群日志中的最新行。`ceph -wceph log last [n]n`

### 监视健康检查

Ceph会根据自己的状态不断运行各种_健康检查_。当运行状况检查失败时，这将反映在（或 ）的输出中。此外，还会将消息发送到集群日志，以指示检查何时失败以及集群何时恢复。`ceph statusceph health`

例如，当OSD发生故障时，`health`状态输出的部分可能会更新如下：

```text
health: HEALTH_WARN
        1 osds down
        Degraded data redundancy: 21/63 objects degraded (33.333%), 16 pgs unclean, 16 pgs degraded
```

这时，还会发出集群日志消息以记录运行状况检查失败：

```text
2017-07-25 10:08:58.265945 mon.a mon.0 172.21.9.34:6789/0 91 : cluster [WRN] Health check failed: 1 osds down (OSD_DOWN)
2017-07-25 10:09:01.302624 mon.a mon.0 172.21.9.34:6789/0 94 : cluster [WRN] Health check failed: Degraded data redundancy: 21/63 objects degraded (33.333%), 16 pgs unclean, 16 pgs degraded (PG_DEGRADED)
```

当OSD重新联机时，群集日志记录群集返回到运行状况的状态：

```text
2017-07-25 10:11:11.526841 mon.a mon.0 172.21.9.34:6789/0 109 : cluster [WRN] Health check update: Degraded data redundancy: 2 pgs unclean, 2 pgs degraded, 2 pgs undersized (PG_DEGRADED)
2017-07-25 10:11:13.535493 mon.a mon.0 172.21.9.34:6789/0 110 : cluster [INF] Health check cleared: PG_DEGRADED (was: Degraded data redundancy: 2 pgs unclean, 2 pgs degraded, 2 pgs undersized)
2017-07-25 10:11:13.535577 mon.a mon.0 172.21.9.34:6789/0 111 : cluster [INF] Cluster is now healthy
```

#### 网络性能检查

Ceph OSD在它们之间发送心跳ping消息以监视守护程序的可用性。我们还使用响应时间来监视网络性能。尽管繁忙的OSD可能会延迟ping响应，但我们可以假设，如果网络交换机失败，则会在不同的OSD对之间检测到多个延迟。

默认情况下，我们将警告ping时间超过1秒（1000毫秒）。

```text
HEALTH_WARN Long heartbeat ping times on back interface seen, longest is 1118.001 msec
```

运行状况详细信息将添加OSD的组合，这些组合将看到延迟以及延迟的程度。详情订单项数量上限为10个。

```text
[WRN] OSD_SLOW_PING_TIME_BACK: Long heartbeat ping times on back interface seen, longest is 1118.001 msec
    Slow heartbeat ping on back interface from osd.0 to osd.1 1118.001 msec
    Slow heartbeat ping on back interface from osd.0 to osd.2 1030.123 msec
    Slow heartbeat ping on back interface from osd.2 to osd.1 1015.321 msec
    Slow heartbeat ping on back interface from osd.1 to osd.0 1010.456 msec
```

要查看更多详细信息和完整的网络性能信息转储，`dump_osd_network`可以使用该命令。通常，这将被发送到mgr，但是可以通过将其发布到任何OSD来限制于特定OSD的交互。缺省值1秒（1000毫秒）的当前阈值可以作为参数（以毫秒为单位）被覆盖。

以下命令将通过指定阈值0并将其发送到mgr，显示所有收集的网络性能数据。

```text
$ ceph daemon /var/run/ceph/ceph-mgr.x.asok dump_osd_network 0
{
    "threshold": 0,
    "entries": [
        {
            "last update": "Wed Sep  4 17:04:49 2019",
            "stale": false,
            "from osd": 2,
            "to osd": 0,
            "interface": "front",
            "average": {
                "1min": 1.023,
                "5min": 0.860,
                "15min": 0.883
            },
            "min": {
                "1min": 0.818,
                "5min": 0.607,
                "15min": 0.607
            },
            "max": {
                "1min": 1.164,
                "5min": 1.173,
                "15min": 1.544
            },
            "last": 0.924
        },
        {
            "last update": "Wed Sep  4 17:04:49 2019",
            "stale": false,
            "from osd": 2,
            "to osd": 0,
            "interface": "back",
            "average": {
                "1min": 0.968,
                "5min": 0.897,
                "15min": 0.830
            },
            "min": {
                "1min": 0.860,
                "5min": 0.563,
                "15min": 0.502
            },
            "max": {
                "1min": 1.171,
                "5min": 1.216,
                "15min": 1.456
            },
            "last": 0.845
        },
        {
            "last update": "Wed Sep  4 17:04:48 2019",
            "stale": false,
            "from osd": 0,
            "to osd": 1,
            "interface": "front",
            "average": {
                "1min": 0.965,
                "5min": 0.811,
                "15min": 0.850
            },
            "min": {
                "1min": 0.650,
                "5min": 0.488,
                "15min": 0.466
            },
            "max": {
                "1min": 1.252,
                "5min": 1.252,
                "15min": 1.362
            },
        "last": 0.791
    },
    ...
```

### 检测配置问题

除了Ceph可以持续以自己的状态运行的运行状况检查之外，还有一些配置问题只能由外部工具检测到。

使用[ceph-medic](http://docs.ceph.com/ceph-medic/master/)工具在Ceph集群的配置上运行这些附加检查。

### 检查集群的使用情况统计

要检查群集在池中的数据使用情况和数据分布，可以使用该`df`选项。它类似于Linux `df`。执行以下命令：

```text
ceph df
```

输出的**RAW STORAGE**部分提供了群集管理的存储量的概述。

* **类别：** OSD设备的类别（或群集的总数）
* **大小：**集群管理的存储容量。
* **可用：**自由空间的集群中使用的量。
* **已用：**用户数据消耗的原始存储量。
* **未使用**的原始资源**：**用户数据，内部开销或保留的容量消耗的原始存储量。
* **已用％RAW：**已用原始存储空间的百分比。将此数字与和结合使用，以确保未达到群集的容量。有关更多详细信息，请参见[存储容量](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-config-ref#storage-capacity)。`full rationear full ratio`

输出的**POOLS**部分提供了池的列表以及每个池的概念用法。本节的输出**不**反映副本，克隆或快照。例如，如果存储的对象具有1MB的数据，则名义使用量将为1MB，但实际使用量可能为2MB或更多，具体取决于副本，克隆和快照的数量。

* **NAME：**池的名称。
* **ID：**池ID。
* **USED​​：**存储在千字节，除非有数字数据附加的名义量**中号**为兆字节或**ģ**千兆字节。
* **％USED：**每个池**使用**的名义存储百分比。
* **MAX AVAIL：**可以写入此池的名义数据量的估计值。
* **对象：**每个池中存储的对象的名义数量。

注意 

**POOLS**部分中的数字是名义上的。它们不包括副本，快照或克隆的数量。其结果是，在总和**USED**和**％USED**金额不会加起来 **USED**和**％USED**的数量**RAW**输出的部分。

注意 

该**MAX AVAIL**值是用于复制或纠删码的复杂函数，映射到存储设备上的CRUSH规则，这些设备的利用率，以及配置的mon\_osd\_full\_ratio。

### 检查OSD状态

您可以检查的OSD，以确保它们`up`和`in`通过执行：

```text
ceph osd stat
```

要么：

```text
ceph osd dump
```

您还可以根据其在CRUSH映射中的位置检查视图OSD。

```text
ceph osd tree
```

Ceph将打印出带有主机，其OSD，是否启动以及其重量的CRUSH树。

```text
#ID CLASS WEIGHT  TYPE NAME             STATUS REWEIGHT PRI-AFF
 -1       3.00000 pool default
 -3       3.00000 rack mainrack
 -2       3.00000 host osd-host
  0   ssd 1.00000         osd.0             up  1.00000 1.00000
  1   ssd 1.00000         osd.1             up  1.00000 1.00000
  2   ssd 1.00000         osd.2             up  1.00000 1.00000
```

有关详细讨论，请参阅[监视OSD和放置组](https://docs.ceph.com/docs/nautilus/rados/operations/monitoring-osd-pg)。

### 检查监视器状态

如果您的群集有多个监视器（可能），则应在启动群集之后以及读取和/或写入数据之前检查监视器仲裁状态。当多个监视器正在运行时，必须有一个仲裁。您还应该定期检查监视器状态，以确保它们正在运行。

要查看显示的监视器地图，请执行以下操作：

```text
ceph mon stat
```

要么：

```text
ceph mon dump
```

要检查监视器群集的仲裁状态，请执行以下操作：

```text
ceph quorum_status
```

Ceph将返回仲裁状态。例如，由三个监视器组成的Ceph群集可能返回以下内容：

```text
{ "election_epoch": 10,
  "quorum": [
        0,
        1,
        2],
  "quorum_names": [
        "a",
        "b",
        "c"],
  "quorum_leader_name": "a",
  "monmap": { "epoch": 1,
      "fsid": "444b489c-4f16-4b75-83f0-cb8097468898",
      "modified": "2011-12-12 13:28:27.505520",
      "created": "2011-12-12 13:28:27.505520",
      "features": {"persistent": [
                        "kraken",
                        "luminous",
                        "mimic"],
        "optional": []
      },
      "mons": [
            { "rank": 0,
              "name": "a",
              "addr": "127.0.0.1:6789/0",
              "public_addr": "127.0.0.1:6789/0"},
            { "rank": 1,
              "name": "b",
              "addr": "127.0.0.1:6790/0",
              "public_addr": "127.0.0.1:6790/0"},
            { "rank": 2,
              "name": "c",
              "addr": "127.0.0.1:6791/0",
              "public_addr": "127.0.0.1:6791/0"}
           ]
  }
}
```

### 检查MDS状态

元数据服务器为CephFS提供元数据服务。元数据服务器具有两种状态：和。为确保您的元数据服务器为和，请执行以下操作：`up | downactive | inactiveupactive`

```text
ceph mds stat
```

要显示元数据集群的详细信息，请执行以下操作：

```text
ceph fs dump
```

### 检查放置组状态

放置组将对象映射到OSD。监视展示位置组时，您希望它们成为`active`和`clean`。有关详细讨论，请参阅[监视OSD和放置组](https://docs.ceph.com/docs/nautilus/rados/operations/monitoring-osd-pg)。

### 使用管理插槽

Ceph管理员套接字允许您通过套接字接口查询守护程序。默认情况下，Ceph套接字位于下`/var/run/ceph`。要通过管理套接字访问守护程序，请登录到运行该守护程序的主机，并使用以下命令：

```text
ceph daemon {daemon-name}
ceph daemon {path-to-socket-file}
```

例如，以下等同：

```text
ceph daemon osd.0 foo
ceph daemon /var/run/ceph/ceph-osd.0.asok foo
```

要查看可用的管理套接字命令，请执行以下命令：

```text
ceph daemon {daemon-name} help
```

admin socket命令使您能够在运行时显示和设置配置。有关详细信息，请参见[在运行时查看配置](https://docs.ceph.com/docs/nautilus/rados/configuration/ceph-conf#viewing-a-configuration-at-runtime)。

另外，您可以直接在运行时设置配置值（即，admin套接字绕过监视器，这与依赖于监视器但不需要您直接登录所涉及的主机不同，）。`ceph tell {daemon-type}.{id} config set`

