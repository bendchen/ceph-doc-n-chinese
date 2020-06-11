# CLAY代码插件

## CLAY代码插件

CLAY（耦合层的缩写）代码是擦除代码，旨在在修复发生故障的节点/ OSD /机架时，显着节省网络带宽和磁盘IO。让：

> d =维修期间接触的OSD数量

如果将_jerasure_配置为_k = 8_且_m = 4_，则丢失一个OSD要求从_d = 8个_其他OSD读取以进行修复。要恢复1GiB，需要下载8 X 1GiB = 8GiB信息。

但是，对于_粘土_插件_d_可以在限制范围内进行配置：

> k + 1 &lt;= d &lt;= k + m-1

默认情况下，粘土代码插件选择_d = k + m-1，_因为它在网络带宽和磁盘IO方面可提供最大的节省。如果_粘土_插件配置为 _k = 8_，_m = 4_且_d = 11，则_当单个OSD发生故障时，将联系d = 11 osds并从每个插件中下载250MiB，导致总下载量为11 X 250MiB = 2.75GiB信息量。下面提供了更多常规参数。当对存储量达到TB级的信息的机架进行维修时，好处是巨大的。

> | 插入 | 磁盘IO总量 |
> | :--- | :--- |
> | 杰拉苏萨 | k \* S |
> | 粘土 | d \* S /（d-k + 1）=（k + m-1）\* S / m |

其中_S_是正在修复的单个OSD上存储的数据量。在上表中，我们使用了_d_的最大可能值，因为这将导致从OSD故障恢复所需的最小数据下载量。

### 擦除代码配置文件示例

可用于观察带宽使用减少的示例配置：

```text
$ ceph osd erasure-code-profile set CLAYprofile \
     plugin=clay \
     k=4 m=2 d=5 \
     crush-failure-domain=host
$ ceph osd pool create claypool 12 12 erasure CLAYprofile
```

### 创建粘土轮廓

要创建新的粘土代码配置文件：

```text
ceph osd erasure-code-profile set {name} \
     plugin=clay \
     k={data-chunks} \
     m={coding-chunks} \
     [d={helper-chunks}] \
     [scalar_mds={plugin-name}] \
     [technique={technique-name}] \
     [crush-failure-domain={bucket-type}] \
     [directory={directory}] \
     [--force]
```

哪里：

`k={data chunks}`描述

每个对象均分为**数据块**部分，每个部分均存储在不同的OSD中。类型

整数需要

是。例

4

`m={coding-chunks}`描述

计算每个对象的**编码块**并将其存储在不同的OSD上。编码块的数量也是可以关闭而不会丢失数据的OSD的数量。类型

整数需要

是。例

2

`d={helper-chunks}`描述

恢复单个块期间请求发送数据的OSD数。_d的_选择必须使k + 1 &lt;= d &lt;= k + m-1。较大_d_，更好的节约。类型

整数需要

没有。默认

k + m-1

`scalar_mds={jerasure|isa|shec}`描述

**scalar\_mds**指定在分层构造中用作构建块的插件。可以是_jerasure_，_isa_，_shec之一_类型

串需要

没有。默认

疯狂

`technique={technique}`描述

**technique**指定将在指定的“ scalar\_mds”插件中采用的技术。支持的技术是'reed\_sol\_van'，'reed\_sol\_r6\_op'，'cauchy\_orig'，'cauchy\_good'，'liber8tion'用于jerasure，'reed\_sol\_van'，'cauchy'用于isa和'single'，'multiple'用于shec。类型

串需要

没有。默认

reed\_sol\_van（用于jerasure，isa），单个（用于shec）

`crush-root={root}`描述

用于CRUSH规则第一步的粉碎桶的名称。对于实例**步骤，请采用默认值**。类型

串需要

没有。默认

默认

`crush-failure-domain={bucket-type}`描述

确保没有两个块位于具有相同故障域的存储桶中。例如，如果故障域是 **主机，则**不会在同一主机上存储两个块。它用于创建CRUSH规则步骤，例如**step choiceleaf host**。类型

串需要

没有。默认

主办

`crush-device-class={device-class}`描述

使用CRUSH映射中的Crush设备类名称，将布局限制为特定类（例如 `ssd`或`hdd`）的设备。类型

串需要

没有。默认

`directory={directory}`描述

设置从中加载擦除代码插件的**目录**名称。类型

串需要

没有。默认

/ usr / lib / ceph / erasure-code

`--force`描述

用相同的名称覆盖现有的配置文件。类型

串需要

没有。

### 子块的概念

Clay代码是矢量代码，因此能够节省磁盘IO和网络带宽，并且能够以称为子块的更精细的粒度查看和操作块中的数据。Clay代码的块中子块的数量由下式给出：

> 子块计数= q （k + m）/ q，其中q = d-k + 1

在修复OSD期间，从可用OSD请求的帮助者信息只是块的一小部分。实际上，修复期间访问的块内子块的数量由下式给出：

> 修复子块计数=子块计数/ q

#### 例子

1. 对于_k = 4_，_m = 2_，_d = 5的配置_，子块计数为8，修复子块计数为4。因此，在修复期间仅读取一半的块。
2. 当_k = 8_，_m = 4_，_d = 11时_，子块计数为64，修复子块计数为16。从可用OSD中读取四分之一的块以修复故障块。

### 如何在给定工作量的情况下选择配置

块中所有子块中只有几个子块被读取。这些子块不必连续存储在块中。为了获得最佳的磁盘IO性能，读取连续的数据很有帮助。因此，建议您选择条带大小，以使子块大小足够大。

对于给定的条带大小（这是基于固定的工作负载），选择`k`，`m`，`d`使得：

```text
sub-chunk size = stripe-size / (k*sub-chunk count) = 4KB, 8KB, 12KB ...
```

1. 对于条带大小较大的大型工作负载，很容易选择k，m，d。例如，考虑大小为64MB的条带大小，选择_k = 16_，_m = 4_和_d = 19_将导致1024的子块计数和4KB的子块大小。
2. 对于较小的工作负载，_k = 4_，_m = 2_是一个很好的配置，可同时带来网络和磁盘IO的好处。

### 与LRC的比较

还设计了本地可恢复代码（LRC），以便在网络带宽方面节省单个OSD恢复期间的磁盘IO。但是，LRC的重点是使维修（d）期间接触的OSD数量保持最少，但这是以存储开销为代价的。的_粘土_代码有一个存储开销米/ K。在_lrc_的情况下，除奇偶校验外，它还存储（k + m）/ d个奇偶`m`校验，从而导致存储开销（m +（k + m）/ d）/ k。两个_粘土_和_LRC_ 可以从任何的故障中恢复`m`的OSD。

> | 参量 | 磁盘IO，存储开销（LRC） | 磁盘IO，存储开销（CLAY） |
> | :--- | :--- | :--- |
> | （k = 10，m = 4） | 7 \* S，0.6（d = 7） | 3.25 \* S，0.4（d = 13） |
> | （k = 16，m = 4） | 4 \* S，0.5625（d = 4） | 4.75 \* S，0.25（d = 19） |

`S`恢复单个OSD的存储数据量在哪里？

