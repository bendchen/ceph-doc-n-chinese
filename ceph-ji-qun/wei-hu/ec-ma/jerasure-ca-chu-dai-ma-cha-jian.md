# JERASURE擦除代码插件

## JERASURE擦除代码插件

该_jerasure_插件是最通用的和灵活的插件，它也是Ceph的擦除编码池的默认。

该_jerasure_插件封装[Jerasure](http://jerasure.org/)库。建议阅读_jerasure_文档，以更好地了解这些参数。

### 创建一个快照配置文件

要创建新的_jerasure_擦除代码配置文件：

```text
ceph osd erasure-code-profile set {name} \
     plugin=jerasure \
     k={data-chunks} \
     m={coding-chunks} \
     technique={reed_sol_van|reed_sol_r6_op|cauchy_orig|cauchy_good|liberation|blaum_roth|liber8tion} \
     [crush-root={root}] \
     [crush-failure-domain={bucket-type}] \
     [crush-device-class={device-class}] \
     [directory={directory}] \
     [--force]
```

哪里：

`k={data chunks}`描述

每个对象均分为**数据块**部分，每个部分存储在不同的OSD中。类型

整数需要

是。例

4

`m={coding-chunks}`描述

计算每个对象的**编码块**并将其存储在不同的OSD上。编码块的数量也是可以关闭而不会丢失数据的OSD的数量。类型

整数需要

是。例

2

`technique={reed_sol_van|reed_sol_r6_op|cauchy_orig|cauchy_good|liberation|blaum_roth|liber8tion}`描述

更灵活的技术是_reed\_sol\_van_：足以设置_k_和_m_。该_cauchy\_good_技术可以更快，但你需要选择的_PACKETSIZE_ 小心。从只能使用_m = 2_进行配置的意义上来说，所有_reed\_sol\_r6\_op_，_liberation_， _blaum\_roth_，_liber8tion_都是_RAID6_等效项。类型

串需要

没有。默认

reed\_sol\_van

`packetsize={bytes}`描述

一次将对_字节_大小的数据包进行编码。选择正确的数据包大小很困难。该 _jerasure_文档包含有关这一主题的大量信息。类型

整数需要

没有。默认

2048

`crush-root={root}`描述

用于CRUSH规则第一步的粉碎桶的名称。例如**step为default**。类型

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

没有。

`directory={directory}`描述

设置从中加载擦除代码插件的**目录**名称。类型

串需要

没有。默认

/ usr / lib / ceph / erasure-code

`--force`描述

用相同的名称覆盖现有的配置文件。类型

串需要

没有。

