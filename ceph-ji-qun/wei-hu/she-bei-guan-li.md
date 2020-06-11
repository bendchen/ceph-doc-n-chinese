# 设备管理

## 设备管理

Ceph跟踪哪些守护程序消耗了哪些硬件存储设备（例如，HDD，SSD），并收集有关这些设备的运行状况度量，以便提供预测和/或自动响应硬件故障的工具。

### 设备跟踪

您可以查询哪些存储设备正在使用：

```text
ceph device ls
```

您还可以按守护程序或主机列出设备：

```text
ceph device ls-by-daemon <daemon>
ceph device ls-by-host <host>
```

对于任何单个设备，您可以通过以下方式查询有关其位置以及如何使用它的信息：

```text
ceph device info <devid>
```

### 启用监控

Ceph还可以监视与您的设备关联的健康指标。例如，SATA硬盘实现了一个称为SMART的标准，该标准提供了有关设备使用情况和运行状况的广泛内部指标，例如开机小时数，电源循环次数或不可恢复的读取错误。其他设备类型（例如SAS和NVMe）实现了一组相似的指标（通过略有不同的标准）。所有这些都可以由Ceph通过该`smartctl`工具收集。

您可以通过以下方式启用或禁用运行状况监视：

```text
ceph device monitoring on
```

要么：

```text
ceph device monitoring off
```

### 抓取

如果启用了监视，则将定期自动刮除指标。该间隔可以配置为：

```text
ceph config set mgr mgr/devicehealth/scrape_frequency <seconds>
```

默认值为每24小时刮一次。

您可以使用以下方法手动触发所有设备的刮擦：

```text
ceph device scrape-health-metrics
```

单个设备可以用以下方式刮取：

```text
ceph device scrape-health-metrics <device-id>
```

或单个守护进程的设备可以通过以下方式进行刮取：

```text
ceph device scrape-daemon-health-metrics <who>
```

可以使用以下命令检索设备的存储健康指标（可选地，用于特定时间戳）：

```text
ceph device get-health-metrics <devid> [sample-timestamp]
```

### 故障预测

Ceph可以根据其收集的健康指标预测预期寿命和设备故障。共有三种模式：

* _none_：禁用设备故障预测。
* _本地_：使用来自ceph-mgr守护程序的预训练预测模型
* _cloud_：使用ProphetStor的免费服务或付费服务（具有更准确的预测）与ProphetStor运行的外部云服务共享设备运行状况和性能指标

预测模式可以配置为：

```text
ceph config set global device_failure_prediction_mode <mode>
```

预测通常在后台定期进行，因此填充预期寿命值可能需要一些时间。您可以从以下输出中看到所有设备的预期寿命：

```text
ceph device ls
```

您还可以使用以下方法查询特定设备的元数据：

```text
ceph device info <devid>
```

您可以使用以下命令显式强制预测设备的预期寿命：

```text
ceph device predict-life-expectancy <devid>
```

如果您没有使用Ceph的内部设备故障预测，但是拥有一些有关设备故障的外部信息源，则可以通过以下方式告知Ceph设备的预期寿命：

```text
ceph device set-life-expectancy <devid> <from> [<to>]
```

预期寿命以时间间隔表示，因此不确定性可以以宽间隔的形式表示。间隔结束也可以不指定。

### 健康警报

在`mgr/devicehealth/warn_threshold`控制预期的设备故障必须在多久之前，我们生成一个健康警告。

可以使用以下方法检查所有设备的存储预期寿命，并生成任何适当的健康警报：

```text
ceph device check-health
```

### 自动缓解

如果`mgr/devicehealth/self_heal`启用了该选项（默认情况下是默认设置），则对于预计将很快发生故障的设备，模块将通过将设备标记为“出”来自动将数据从它们中迁移出来。

在`mgr/devicehealth/mark_out_threshold`控制预期的设备故障必须在多久之前，我们自动标记OSD“走出去”。

