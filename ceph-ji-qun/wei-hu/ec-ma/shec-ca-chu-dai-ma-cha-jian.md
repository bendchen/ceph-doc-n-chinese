# SHEC擦除代码插件

## SHEC擦除代码插件

该_海希科_插件封装了[多个海希科](http://tracker.ceph.com/projects/ceph/wiki/Shingled_Erasure_Code_%28SHEC%29) 库。与Reed Solomon代码相比，它使ceph可以更有效地恢复数据。

### 创建一个SHEC配置文件

要创建新的_Shec_擦除代码配置文件：

```text
ceph osd erasure-code-profile set {name} \
     plugin=shec \
     [k={data-chunks}] \
     [m={coding-chunks}] \
     [c={durability-estimator}] \
     [crush-root={root}] \
     [crush-failure-domain={bucket-type}] \
     [crush-device-class={device-class}] \
     [directory={directory}] \
     [--force]
```

哪里：

`k={data-chunks}`描述

每个对象均分为**数据块**部分，每个部分存储在不同的OSD中。类型

整数需要

没有。默认

4

`m={coding-chunks}`描述

计算每个对象的**编码块**并将它们存储在不同的OSD上。数**个编码块**不一定等于说可以下，而不会丢失数据的OSD的数量。类型

整数需要

没有。默认

3

`c={durability-estimator}`描述

奇偶校验块的数量，每个奇偶校验块包括其计算范围内的每个数据块。该数字用作**耐久性估算器**。例如，如果c = 2，则2个OSD可以关闭而不会丢失数据。类型

整数需要

没有。默认

2

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

### SHEC布局的简要说明

#### 空间利用率

空间效率是数据块与对象中所有数据块的比率，并表示为k /（k + m）。为了提高空间效率，应增加k或减小m。

```text
space efficiency of SHEC(4,3,2) = 4/(4+3) = 0.57
SHEC(5,3,2) or SHEC(4,2,2) improves SHEC(4,3,2)'s space efficiency
```

#### 耐久性

SHEC的第三个参数（= c）是耐用性估算器，它近似于可以关闭而不会丢失数据的OSD数量。

`durability estimator of SHEC(4,3,2) = 2`

#### 采收率

描述恢复效率的计算超出了本文的范围，但是至少增加m而不增加c可以提高恢复效率。（但是，在这种情况下，我们必须注意牺牲空间效率。）

`SHEC(4,2,2) -> SHEC(4,3,2) : achieves improvement of recovery efficiency`

### 删除代码配置文件示例

```text
$ ceph osd erasure-code-profile set SHECprofile \
     plugin=shec \
     k=8 m=4 c=3 \
     crush-failure-domain=host
$ ceph osd pool create shecpool 256 256 erasure SHECprofile
```

