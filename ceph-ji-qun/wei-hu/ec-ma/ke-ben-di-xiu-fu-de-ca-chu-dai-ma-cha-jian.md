# 可本地修复的擦除代码插件

## 可本地修复的擦除代码插件

使用_jerasure_插件，当一个擦除编码的对象存储在多个OSD上时，要从丢失一个OSD中恢复，就需要从所有其他OSD中读取。例如，如果将_jerasure_配置为 _k = 8_且_m = 4_，则丢失一个OSD要求从其他11个中读取以进行修复。

该_LRC_纠错编码插件创建本地校验块，以便能够使用较少的OSD恢复。例如，如果_lrc_配置为 _k = 8_，_m = 4_和_l = 4_，它将为每四个OSD创建一个额外的奇偶校验块。当单个OSD丢失时，只能使用四个OSD（而不是十一个）来恢复它。

### 删除代码配置文件示例

#### 降低主机之间的恢复带宽

尽管将所有主机都连接到同一交换机可能不是一个有趣的用例，但实际上可以观察到带宽使用减少。

```text
$ ceph osd erasure-code-profile set LRCprofile \
     plugin=lrc \
     k=4 m=2 l=3 \
     crush-failure-domain=host
$ ceph osd pool create lrcpool 12 12 erasure LRCprofile
```

#### 减少机架之间恢复带宽

在Firefly中，只有在主OSD与丢失的块位于同一机架中时，才会观察到带宽减少的情况：

```text
$ ceph osd erasure-code-profile set LRCprofile \
     plugin=lrc \
     k=4 m=2 l=3 \
     crush-locality=rack \
     crush-failure-domain=host
$ ceph osd pool create lrcpool 12 12 erasure LRCprofile
```

### 创建一个LRC配置文件

要创建新的lrc擦除代码配置文件：

```text
ceph osd erasure-code-profile set {name} \
     plugin=lrc \
     k={data-chunks} \
     m={coding-chunks} \
     l={locality} \
     [crush-root={root}] \
     [crush-locality={bucket-type}] \
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

`l={locality}`描述

将编码和数据块分组为大小**局部性**集 。例如，对于**k = 4**和**m = 2**，当**locality = 3**时，将创建两组，每组三个。每个集合都可以恢复，而无需从另一个集合读取块。类型

整数需要

是。例

3

`crush-root={root}`描述

用于CRUSH规则第一步的粉碎桶的名称。例如**step为default**。类型

串需要

没有。默认

默认

`crush-locality={bucket-type}`描述

粉碎桶的类型，其中存储由**l**定义的每组块。例如，如果将其设置为**rack**，则每组**l个**块将放置在不同的机架中。它用于创建CRUSH规则步骤，例如**step select rack**。如果未设置，则不会进行此类分组。类型

串需要

没有。

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

### 低级插件配置

**k**和**m**的总和必须是**l**参数的倍数。低级配置参数没有施加这样的限制，将其用于特定目的可能会更方便。例如，可以定义两个组，一个具有4个块，另一个具有3个块。还可以递归定义位置集，例如数据中心和机架到数据中心。的**K / M / L**通过产生一个低级别的配置来实现。

该_LRC_纠错编码插件递归地应用纠错编码技术，使一些大块的损失中恢复只需要使用大块，大部分时间的一个子集。

例如，当三个编码步骤描述为：

```text
chunk nr    01234567
step 1      _cDD_cDD
step 2      cDDD____
step 3      ____cDDD
```

其中_c_是根据数据块_D_计算出的编码块，则可以使用最后四个块来恢复块_7_的丢失。而第_2个_块的丢失可以通过前四个块来恢复。

### 使用低级配置的擦除代码配置文件示例

#### 最小测试

它严格等同于使用默认的擦除代码配置文件。的_DD_ 意味着_K = 2_时，_C ^_意味着_M = 1_和_jerasure_插件缺省使用：

```text
$ ceph osd erasure-code-profile set LRCprofile \
     plugin=lrc \
     mapping=DD_ \
     layers='[ [ "DDc", "" ] ]'
$ ceph osd pool create lrcpool 12 12 erasure LRCprofile
```

#### 降低主机之间的恢复带宽

当所有主机都连接到同一台交换机时，虽然这可能不是有趣的用例，但实际上可以观察到带宽使用减少。尽管块的布局不同，但它等效于**k = 4**，**m = 2**和**l = 3**：

```text
$ ceph osd erasure-code-profile set LRCprofile \
     plugin=lrc \
     mapping=__DD__DD \
     layers='[
               [ "_cDD_cDD", "" ],
               [ "cDDD____", "" ],
               [ "____cDDD", "" ],
             ]'
$ ceph osd pool create lrcpool 12 12 erasure LRCprofile
```

#### 减少机架之间恢复带宽

在Firefly中，只有在主OSD与丢失的块位于同一机架中时，才会观察到带宽减少的情况：

```text
$ ceph osd erasure-code-profile set LRCprofile \
     plugin=lrc \
     mapping=__DD__DD \
     layers='[
               [ "_cDD_cDD", "" ],
               [ "cDDD____", "" ],
               [ "____cDDD", "" ],
             ]' \
     crush-steps='[
                     [ "choose", "rack", 2 ],
                     [ "chooseleaf", "host", 4 ],
                    ]'
$ ceph osd pool create lrcpool 12 12 erasure LRCprofile
```

#### 使用不同的擦除代码后端进行测试

LRC现在使用jerasure作为默认EC后端。可以使用低级别配置在每个层的基础上指定EC后端/算法。层中的第二个参数='\[\[\[“ DDc”，“”\]\]'实际上是要用于此级别的擦除代码配置文件。下面的示例使用lrcpool中使用的柯西技术指定ISA后端：

```text
$ ceph osd erasure-code-profile set LRCprofile \
     plugin=lrc \
     mapping=DD_ \
     layers='[ [ "DDc", "plugin=isa technique=cauchy" ] ]'
$ ceph osd pool create lrcpool 12 12 erasure LRCprofile
```

您还可以为每个层使用不同的擦除代码配置文件：

```text
$ ceph osd erasure-code-profile set LRCprofile \
     plugin=lrc \
     mapping=__DD__DD \
     layers='[
               [ "_cDD_cDD", "plugin=isa technique=cauchy" ],
               [ "cDDD____", "plugin=isa" ],
               [ "____cDDD", "plugin=jerasure" ],
             ]'
$ ceph osd pool create lrcpool 12 12 erasure LRCprofile
```

### 擦除编码和解码算法

在图层描述中找到的步骤：

```text
chunk nr    01234567

step 1      _cDD_cDD
step 2      cDDD____
step 3      ____cDDD
```

按顺序应用。例如，如果对4K对象进行了编码，则它将首先经过_步骤1，_然后分为四个1K块（四个大写字母D）。它们按顺序存储在块2、3、6和7中。根据这些，计算出两个编码块（两个小写字母c）。编码块分别存储在块1和5中。

在_步骤2中_重新使用由创建的内容_的步骤1_中类似的方式，并存储在单个编码块_Ç_为0的最后四个数据块的位置，标有下划线（_\__为了可读性），将被忽略。

在_步骤3_中存储有单个编码块_Ç_在位置4.创建的三个组块_步骤1_被用于计算这种编码块，即，从编码块_步骤1_变成在一个数据块_的步骤3_。

如果块_2_丢失：

```text
chunk nr    01234567

step 1      _c D_cDD
step 2      cD D____
step 3      __ _cDDD
```

解码将尝试通过以相反的顺序执行以下操作来恢复它：_步骤3_然后是_步骤2_，最后是_步骤1_。

在_步骤3中_知道块没什么_2_（即它是一个下划线）和被跳过。

来自_步骤2_的编码块（存储在块_0中_）允许其恢复块_2_的内容。没有更多块可恢复，并且该过程停止，无需考虑_步骤1_。

恢复块_2_需要读取的块_0,1，3_和写回块_2_。

如果组块_2，3，6_丢失：

```text
chunk nr    01234567

step 1      _c  _c D
step 2      cD  __ _
step 3      __  cD D
```

在_步骤3中_可以恢复组块的内容_6_：

```text
chunk nr    01234567

step 1      _c  _cDD
step 2      cD  ____
step 3      __  cDDD
```

在_步骤2中_未能恢复和被跳过，因为存在两个组块缺失（_2，3_），它只能从一个缺少的块恢复。

存储在块_1、5中的步骤1_的编码块允许其恢复块_2、3_的内容：

```text
chunk nr    01234567

step 1      _cDD_cDD
step 2      cDDD____
step 3      ____cDDD
```

### 控制CRUSH位置

默认的CRUSH规则提供位于不同主机上的OSD。例如：

```text
chunk nr    01234567

step 1      _cDD_cDD
step 2      cDDD____
step 3      ____cDDD
```

恰好需要_8个_ OSD，每个块一个。如果主机位于两个相邻的机架中，则可以将前四个块放在第一个机架中，将最后四个放在第二个机架中。因此，从丢失单个OSD中恢复就不需要在两个机架之间使用带宽。

例如：

```text
crush-steps='[ [ "choose", "rack", 2 ], [ "chooseleaf", "host", 4 ] ]'
```

将创建一个规则，该规则将选择两个_机架式_破碎桶，并为每个破碎桶 选择四个OSD，每个OSD位于不同类型的_host_桶中。

也可以手动制作CRUSH规则以进行更精细的控制。

