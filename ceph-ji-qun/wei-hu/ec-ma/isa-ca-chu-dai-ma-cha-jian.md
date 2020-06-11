# ISA擦除代码插件

## ISA擦除代码插件

在_ISA_插件封装[ISA](https://01.org/intel%C2%AE-storage-acceleration-library-open-source-version/) 库。它仅在Intel处理器上运行。

### 创建一个ISA配置文件

要创建新的_isa_擦除代码配置文件：

```text
ceph osd erasure-code-profile set {name} \
     plugin=isa \
     technique={reed_sol_van|cauchy} \
     [k={data-chunks}] \
     [m={coding-chunks}] \
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

没有。默认

7

`m={coding-chunks}`描述

计算每个对象的**编码块**并将其存储在不同的OSD上。编码块的数量也是可以关闭而不会丢失数据的OSD的数量。类型

整数需要

没有。默认

3

`technique={reed_sol_van|cauchy}`描述

ISA插件有两种[Reed Solomon](https://en.wikipedia.org/wiki/Reed%E2%80%93Solomon_error_correction) 形式。如果设置了_reed\_sol\_van_，则为[Vandermonde](https://en.wikipedia.org/wiki/Vandermonde_matrix)；如果 设置了_cauchy_，则为[Cauchy](https://en.wikipedia.org/wiki/Cauchy_matrix)。类型

串需要

没有。默认

reed\_sol\_van

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

