# PROMETHEUS模块

## PROMETHEUS模块

提供一个Prometheus导出器，以从ceph-mgr中的收集点传递Ceph性能计数器。Ceph-mgr从所有MgrClient进程（例如，mons和OSD）接收具有性能计数器架构数据和实际计数器数据的MMgrReport消息，并保留最后N个样本的循环缓冲区。该模块创建一个HTTP端点（像所有Prometheus导出器一样），并在轮询（或在Prometheus术语中“刮掉”）时检索每个计数器的最新样本。HTTP路径和查询参数将被忽略；所有报告实体的所有现有计数器均以文本展示格式返回。（请参阅Prometheus [文档](https://prometheus.io/docs/instrumenting/exposition_formats/#text-format-details)。）

### 启用普罗米修斯输出

该_普罗米修斯_模块启用了：

```text
ceph mgr module enable prometheus
```

#### 配置

默认情况下，模块将`9283`在主机上所有IPv4和IPv6地址上的端口上接受HTTP请求。端口和监听地址都可以使用，键 和进行配置。该端口已在Prometheus的[注册表中注册](https://github.com/prometheus/prometheus/wiki/Default-port-allocations)。`ceph config-key setmgr/prometheus/server_addrmgr/prometheus/server_port`

#### RBD IO统计信息

通过启用动态OSD性能计数器，该模块可以选择收集RBD每映像IO统计信息。收集`mgr/prometheus/rbd_stats_pools` 配置参数中指定的池中所有映像的统计信息。参数是逗号或空格分隔的`pool[/namespace]`条目列表。如果未指定名称空间，则将收集池中所有名称空间的统计信息。

激活启用了RBD的池的示例`pool1`，`pool2`以及`poolN`：

```text
ceph config set mgr mgr/prometheus/rbd_stats_pools "pool1,pool2,poolN"
```

该模块列出扫描指定池和名称空间的所有可用映像的列表，并定期刷新它。该时间段可以通过`mgr/prometheus/rbd_stats_pools_refresh_interval` 参数（以秒为单位）进行配置，默认为300秒（5分钟）。如果模块从先前未知的RBD映像中检测到统计信息，则将强制更早刷新。

将同步间隔设置为10分钟的示例：

```text
ceph config set mgr mgr/prometheus/rbd_stats_pools_refresh_interval 600
```

### 统计名称和标签

统计信息的名称与Ceph命名的名称完全相同，使用非法字符`.`，`-`并`::`转换为`_`，并`ceph_`以所有名称为前缀。

所有_守护程序_统计信息都有一个`ceph_daemon`标签，例如“ osd.123”，用于标识它们来自的守护程序的类型和ID。一些统计信息可能来自不同类型的守护程序，因此在查询OSD的RocksDB统计信息时，您可能希望对以“ osd”开头的ceph\_daemon进行过滤，以免混入监视器的rockdb统计信息。

该_集群_统计（即那些全球性的Ceph的集群）有标签拨出他们汇报。例如，与池有关的度量带有`pool_id`标签。

代表来自核心Ceph的直方图的长期平均值由一对`<name>_sum`和`<name>_count`指标表示。这类似于[Prometheus中](https://prometheus.io/docs/concepts/metric_types/#histogram)直方图的表示方式， 也可以[类似地](https://prometheus.io/docs/practices/histograms/)对其进行处理。

#### 池和OSD元数据系列

输出特殊系列，以允许在某些元数据字段上显示和查询。

池具有如下`ceph_pool_metadata`字段：

```text
ceph_pool_metadata{pool_id="2",name="cephfs_metadata_a"} 1.0
```

OSD具有如下`ceph_osd_metadata`字段：

```text
ceph_osd_metadata{cluster_addr="172.21.9.34:6802/19096",device_class="ssd",ceph_daemon="osd.0",public_addr="172.21.9.34:6801/19096",weight="1.0"} 1.0
```

#### 将驱动器统计信息与NODE\_EXPORTER进行关联

Ceph的prometheus输出旨在与Prometheus node\_exporter的常规主机监视结合使用。

为了使Ceph OSD统计信息与node\_exporter的驱动器统计信息相关联，将输出特殊的序列，如下所示：

```text
ceph_disk_occupation{ceph_daemon="osd.0",device="sdd", exported_instance="myhost"}
```

要使用此方法通过OSD ID获取磁盘统计信息，请在Prometheus查询中使用`and`运算符或`*`运算符。所有的元数据的度量（如\`\`所以它们充当中性ceph\_disk\_occupation\`\`具有值1 `*`。使用`*` 允许使用`group_left`和`group_right`分组改性剂，使得所得到的度量从查询的一侧具有额外的标签。

有关构造查询的更多信息，请参见 [prometheus文档](https://prometheus.io/docs/prometheus/latest/querying/basics)。

目标是运行类似

```text
rate(node_disk_bytes_written[30s]) and on (device,instance) ceph_disk_occupation{ceph_daemon="osd.0"}
```

由于`instance`两个指标的标签都不匹配，因此上述查询开箱即用不会返回任何指标。的`instance`标签`ceph_disk_occupation` 将是当前活动的MGR节点。

> 以下两节概述了两种补救方法。

### 使用LABEL\_REPLACE 

的`label_replace`功能（CP。 [label\_replace文档](https://prometheus.io/docs/prometheus/latest/querying/functions/#label_replace)）可以将标签添加至，或改变的一个标签，一个查询中的度量。

要关联OSD及其磁盘写入速率，可以使用以下查询：

```text
label_replace(rate(node_disk_bytes_written[30s]), "exported_instance", "$1", "instance", "(.*):.*") and on (device,exported_instance) ceph_disk_occupation{ceph_daemon="osd.0"}
```

### 配置PROMETHEUS服务器

#### HONOR\_LABELS 

要使Ceph能够输出与任何主机相关的标签正确的数据，请`honor_labels`在将ceph-mgr端点添加到prometheus配置中时使用该设置。

这允许Ceph导出正确的`instance`标签，而不会被普罗米修斯覆盖。如果没有此设置，Prometheus将应用一个`instance`标签，其中包括该系列来自的端点的主机名和端口。由于Ceph集群具有多个管理器守护程序，因此`instance`，当活动管理器守护程序更改时，该 标签会伪造地更改。

如果不希望这样做，则`instance`可以在Prometheus目标配置中设置自定义标签：您可能希望将其设置为第一个mgr守护程序的主机名，或者完全是任意的，例如“ ceph\_cluster”。

#### NODE\_EXPORTER主机名标签

设置`instance`标签以匹配`instance`字段中Ceph的OSD元数据中显示的标签。通常，这是节点的简短主机名。

仅在要将Ceph统计信息与主机统计信息关联时才需要这样做，但是在以后的情况下，您可能会发现在所有情况下都需要这样做。

#### 配置示例

此示例显示了在名为的服务器上运行ceph-mgr和node\_exporter的单节点配置`senta04`。请注意，这需要向每个`node_exporter`目标分别添加适当的实例标签。

这只是一个例子：还有其他方法可以配置普罗米修斯刮擦目标和标签重写规则。

**PROMETHEUS.YML** 

```text
global:
  scrape_interval:     15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'node'
    file_sd_configs:
      - files:
        - node_targets.yml
  - job_name: 'ceph'
    honor_labels: true
    file_sd_configs:
      - files:
        - ceph_targets.yml
```

**CEPH\_TARGETS.YML** 

```text
[
    {
        "targets": [ "senta04.mydomain.com:9283" ],
        "labels": {}
    }
]
```

**NODE\_TARGETS.YML** 

```text
[
    {
        "targets": [ "senta04.mydomain.com:9100" ],
        "labels": {
            "instance": "senta04"
        }
    }
]
```

### 注释

计数器和量规已出口；目前还没有直方图和长期平均值。可以将Ceph的2D直方图简化为两个单独的1D直方图，并且可以将长期运行的平均值导出为Prometheus的Summary类型。

与许多Prometheus导出器一样，时间戳是由服务器的抓取时间确定的（Prometheus期望它正在同步轮询实际的计数器进程）。可以提供时间戳和统计报告，但是Prometheus团队强烈建议不要这样做。这意味着时间戳将延迟不可预测的时间；尚不清楚这是否会带来问题，但值得了解。

