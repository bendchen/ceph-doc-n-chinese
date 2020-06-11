# 手动编辑CRUSH MAP

## 手动编辑CRUSH映射

注意 

手动编辑CRUSH映射被认为是高级管理员操作。绝大多数安装所需的所有CRUSH更改都可以通过标准ceph CLI进行，并且不需要手动进行CRUSH映射编辑。如果你已经确定了手动编辑的使用情况_是_必要的，考虑接触Ceph的开发人员，以便头孢的未来版本可以使这种不必要的。

要编辑现有的CRUSH映射，请执行以下操作：

1. [获取CRUSH地图](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map-edits/#getcrushmap)。
2. [反编译](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map-edits/#decompilecrushmap) CRUSH映射。
3. 编辑[设备](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map-edits/#crushmapdevices)，存储[桶](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map-edits/#crushmapbuckets)和[规则](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map-edits/#crushmaprules)中的至少一项。
4. [重新编译](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map-edits/#compilecrushmap) CRUSH映射。
5. [设置CRUSH图](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map-edits/#setcrushmap)。

有关为特定池设置CRUSH映射规则的详细信息，请参阅“ [设置池值”](https://docs.ceph.com/docs/nautilus/rados/operations/pools#setpoolvalues)。

### 获取一个CRUSH 

要获取群集的CRUSH映射，请执行以下操作：

```text
ceph osd getcrushmap -o {compiled-crushmap-filename}
```

Ceph将输出（-o）已编译的CRUSH映射到您指定的文件名。由于CRUSH映射为已编译形式，因此必须首先对其进行反编译，然后才能对其进行编辑。

### 反编译CRUSH映射

要反编译CRUSH映射，请执行以下操作：

```text
crushtool -d {compiled-crushmap-filename} -o {decompiled-crushmap-filename}
```

### 章节

粉碎地图有六个主要部分。

1. **可调项：**地图顶部的序言描述 了CRUSH行为的任何_可调项_，与历史/旧版CRUSH行为不同。这些可以纠正多年来为改善CRUSH行为所做的旧错误，优化或其他行为更改。
2. **设备：**设备是`ceph-osd`可以存储数据的单个守护程序。
3. **types**：存储桶`types`定义在CRUSH层次结构中使用的存储桶的类型。存储桶由存储位置（例如，行，机架，机箱，主机等）及其分配的权重的分层聚合组成。
4. **存储桶：**定义存储桶类型后，必须定义层次结构中的每个节点，其类型以及它包含的设备或其他节点。
5. **规则：**规则定义有关数据如何在层次结构中的各个设备之间分配的策略。
6. choice\_args **：** Choose\_args是与层次结构关联的替代权重，这些权重已进行调整以优化数据放置。单个choice\_args映射可以用于整个集群，也可以为每个单独的池创建一个映射。

### 粉碎地图设备

设备是`ceph-osd`可以存储数据的单个守护程序。通常，您将为集群中的每个OSD守护程序定义一个。设备由ID（非负整数）和名称标识，通常`osd.N`在哪里`N`是设备ID。

设备还可以具有与它们关联的_设备类_（例如 `hdd`或`ssd`），从而可以通过挤压规则方便地将它们作为目标。

```text
# devices
device {num} {osd.name} [class {class}]
```

例如：

```text
# devices
device 0 osd.0 class ssd
device 1 osd.1 class hdd
device 2 osd.2
device 3 osd.3
```

在大多数情况下，每个设备都映射到一个`ceph-osd`守护程序。通常，它是单个存储设备，一对设备（例如，一个用于数据，一个用于日志或元数据），或者在某些情况下是小型RAID设备。

### CRUSH映射桶类型

CRUSH映射中的第二个列表定义了“存储桶”类型。桶有助于节点和叶子的层次结构。节点（或非叶子）存储桶通常表示层次结构中的物理位置。节点聚合其他节点或叶子。叶存储桶代表`ceph-osd`守护程序及其相应的存储介质。

小费 

在CRUSH上下文中使用的术语“存储桶”是指层次结构中的节点，即位置或物理硬件。在RADOS网关API的上下文中使用时，它与术语“存储桶”不同。

要将存储桶类型添加到CRUSH映射中，请在存储桶类型列表下创建新行。输入，`type`后跟唯一的数字ID和存储桶名称。按照惯例，只有一个叶片桶，它是；但是，您可以使用任何喜欢的名称（例如osd，磁盘，驱动器，存储等）：`type 0`

```text
#types
type {num} {bucket-name}
```

例如：

```text
# types
type 0 osd
type 1 host
type 2 chassis
type 3 rack
type 4 row
type 5 pdu
type 6 pod
type 7 room
type 8 datacenter
type 9 region
type 10 root
```

### 粉碎地图桶层次结构

CRUSH算法根据每个设备的权重值在存储设备之间分配数据对象，从而近似于均匀的概率分布。CRUSH根据您定义的层次集群图来分发对象及其副本。您的CRUSH映射表示可用的存储设备以及包含它们的逻辑元素。

为了跨故障域将放置组映射到OSD，CRUSH映射定义了存储桶类型的分层列表（即，`#types`在生成的CRUSH映射中的）。创建存储桶层次结构的目的是通过叶子节点的故障域（例如主机，机箱，机架，配电单元，窗格，行，房间和数据中心）隔离叶子节点。除代表OSD的叶节点外，其余层次结构是任意的，您可以根据自己的需要进行定义。

我们建议您将CRUSH映射调整为适用于公司的硬件命名约定，并使用反映物理硬件的实例名称。当OSD和/或其他硬件出现故障并且管理员需要访问物理硬件时，您的命名实践可以使管理群集和解决问题的工作变得更加容易。

在以下示例中，存储桶层次结构具有一个名为的叶存储桶`osd`和两个分别名为`host`和的节点存储桶`rack`。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-91dff8176c752894890e24c5e8844d0fdfb8a890.png)

注意 

编号较高的`rack`存储桶类型会聚合编号较低的`host`存储桶类型。

由于叶节点反映`#devices`了在CRUSH映射的开头在列表下声明的存储设备，因此您无需将它们声明为存储桶实例。层次结构中第二低的存储桶类型通常会聚合设备（即，通常是包含存储介质的计算机，并使用您喜欢描述的任何术语，例如“节点”，“计算机”，“服务器”，“主机” ”，“机器”等）。在高密度环境中，每个机箱看到多个主机/节点越来越普遍。您还应该考虑机箱故障-例如，如果节点发生故障，则需要拉动机箱，这可能会导致大量主机/节点及其OSD停机。

在声明存储桶实例时，您必须指定其类型，为其指定唯一的名称（字符串），为其分配唯一的ID（以负整数表示）（可选），并指定相对于其项的总容量/容量的权重），指定存储桶算法（通常为`straw`）和哈希（通常为`0`反映哈希算法`rjenkins1`）。一个存储桶可能有一个或多个项目。这些项目可能包含节点存储桶或叶子。物品的重量可能反映出物品的相对重量。

您可以使用以下语法声明节点存储桶：

```text
[bucket-type] [bucket-name] {
        id [a unique negative numeric ID]
        weight [the relative capacity/capability of the item(s)]
        alg [the bucket type: uniform | list | tree | straw ]
        hash [the hash type: 0 by default]
        item [item-name] weight [weight]
}
```

例如，使用上图，我们将定义两个主机存储桶和一个机架存储桶。OSD被声明为主机存储桶中的项目：

```text
host node1 {
        id -1
        alg straw
        hash 0
        item osd.0 weight 1.00
        item osd.1 weight 1.00
}

host node2 {
        id -2
        alg straw
        hash 0
        item osd.2 weight 1.00
        item osd.3 weight 1.00
}

rack rack1 {
        id -3
        alg straw
        hash 0
        item node1 weight 2.00
        item node2 weight 2.00
}
```

注意 

在上述示例中，请注意，机架式存储区不包含任何OSD。而是包含较低级别的主机存储桶，并在项目条目中包括其重量的总和。

铲斗类型

Ceph支持四种存储桶类型，每种类型代表性能与重组效率之间的折衷。如果不确定要使用哪种存储桶类型，建议您使用`straw`存储桶。有关存储桶类型的详细讨论，请参阅 [CRUSH控制，可伸缩，去中心化复制数据的放置，](https://ceph.com/wp-content/uploads/2016/08/weil-crush-sc06.pdf)尤其是**第3.4节**。值区类型为：

> 1. **制服**：制服的水桶会聚集重量**完全相同**的设备。例如，当公司调试或退役硬件时，通常会使用许多物理配置完全相同的机器（例如，批量购买）进行调试。当存储设备的重量完全相同时，可以使用`uniform`存储桶类型，这使CRUSH可以在恒定时间内将副本映射到统一存储桶中。如果权重不一致，则应使用其他存储桶算法。
> 2. **列表**：列表存储桶将其内容汇总为链接列表。基于RUSH P算法，列表是**扩展集群**的自然而直观的选择：将某个对象以适当的概率重新放置到最新的设备上，或者像以前一样将其保留在旧设备上。将项目添加到存储桶时，结果是最佳的数据迁移。但是，从列表中间或结尾删除的项目可能会导致大量不必要的移动，从而使列表存储桶最适合**从未（或很少）收缩的情况**。
> 3. **树**：树桶使用二进制搜索树。当存储桶包含更多项目集时，它们比列表存储桶更有效。基于RUSH R算法，树桶将放置时间减少到O（log n），使其适合管理更大的设备集或嵌套桶。
> 4. **秸秆**：列表桶和树桶使用分而治之的策略可以使某些项具有优先级（例如，列表开头的项），或者根本不需要考虑整个项子树。这样可以提高副本放置过程的性能，但是当存储桶的内容由于项目的增加，删除或重新加权而发生更改时，也会引入次优的重组行为。吸管类型允许所有物品相互竞争，从而通过复制吸管的过程进行复制品放置。
> 5. **Straw2**：Straw2存储桶改进了Straw，以在邻居权重改变时正确避免项目之间的任何数据移动。
>
>    例如，项目A的重量（包括重新添加或完全删除项目），仅数据往返于项目A。

杂凑

每个存储桶均使用哈希算法。目前，Ceph支持`rjenkins1`。输入`0`作为您的哈希设置以选择`rjenkins1`。

称重斗项目

Ceph将铲斗重量表示为两倍，从而可以进行精细称重。重量是设备容量之间的相对差。我们建议使用`1.00`1TB存储设备的相对重量。在这种情况下，的权重`0.5`将代表大约500GB，而的权重`3.00`将代表大约3TB。较高级别的存储桶的权重是该存储桶聚合的所有叶子项的总和。

桶项目的重量是一维的，但是您也可以计算项目的重量以反映存储驱动器的性能。例如，如果您有许多1TB驱动器，其中一些具有较低的数据传输速率，而另一些具有较高的数据传输速率，则即使它们具有相同的容量（例如，硬盘的权重为0.80），也可以对其进行不同的加权。第一组总吞吐量较低的驱动器，以及1.20第二组总吞吐量较高的驱动器。

### CRUSH映射规则

CRUSH映射支持“ CRUSH规则”的概念，“ CRUSH规则”是确定池数据放置的规则。默认的CRUSH映射对每个池都有一个规则。对于大型群集，您可能会创建许多池，其中每个池可能都有自己的非默认CRUSH规则。

注意 

在大多数情况下，您将不需要修改默认规则。创建新池时，默认情况下该规则将设置为`0`。

CRUSH规则定义了放置和复制策略或分发策略，使您可以精确指定CRUSH如何放置对象副本。例如，您可能创建一条规则，该规则选择一对目标以进行2路镜像，另一条规则在两个不同的数据中心中选择三个目标以进行3路镜像，再创建一条规则以擦除六个存储设备上的编码。有关CRUSH规则的详细讨论，请参阅 [CRUSH控制的，可伸缩的，去中心化的复制数据放置，](https://ceph.com/wp-content/uploads/2016/08/weil-crush-sc06.pdf)尤其是**第3.2节**。

规则采用以下形式：

```text
rule <rulename> {

        id [a unique whole numeric ID]
        type [ replicated | erasure ]
        min_size <min-size>
        max_size <max-size>
        step take <bucket-name> [class <device-class>]
        step [choose|chooseleaf] [firstn|indep] <N> type <bucket-type>
        step emit
}
```

`id`描述

用于标识规则的唯一整数。目的

规则掩码的组成部分。类型

整数需要

是默认

0

`type`描述

描述存储驱动器（复制）或RAID的规则。目的

规则掩码的组成部分。类型

串需要

是默认

`replicated`有效值

目前仅`replicated`和`erasure`

`min_size`描述

如果池中的副本数少于此数量，则CRUSH将 **不会**选择此规则。类型

整数目的

规则掩码的组成部分。需要

是默认

`1`

`max_size`描述

如果池中的副本数量超过此数量，则CRUSH将 **不会**选择此规则。类型

整数目的

规则掩码的组成部分。需要

是默认

10

`step take <bucket-name> [class <device-class>]`描述

取一个存储桶名称，然后开始遍历树。如果`device-class`指定，则它必须与定义设备时先前使用的类匹配。排除所有不属于该类别的设备。目的

规则的组成部分。需要

是例

`step take data`

`step choose firstn {num} type {bucket-type}`描述

从当前存储桶中选择给定类型的存储桶数。该数量通常是池中副本的数量（即池大小）。

* 如果为，则选择存储桶（全部可用）。`{num} == 0pool-num-replicas`
* 如果为，则选择那么多桶。`{num} > 0 && < pool-num-replicas`
* 如果是的话。`{num} < 0pool-num-replicas - {num}`

目的

规则的组成部分。先决条件

跟随或。`step takestep choose`例

`step choose firstn 1 type row`

`step chooseleaf firstn {num} type {bucket-type}`描述

选择的一组存储桶，`{bucket-type}`并从该组存储桶中每个存储桶的子树中选择一个叶节点（即OSD）。集合中的存储桶数通常是池中的副本数（即池大小）。

* 如果为，则选择存储桶（全部可用）。`{num} == 0pool-num-replicas`
* 如果为，则选择那么多桶。`{num} > 0 && < pool-num-replicas`
* 如果是的话。`{num} < 0pool-num-replicas - {num}`

目的

规则的组成部分。使用无需使用两个步骤来选择设备。先决条件

跟随或。`step takestep choose`例

`step chooseleaf firstn 0 type row`

`step emit`描述

输出当前值并清空堆栈。通常在规则末尾使用，但也可以用于从同一规则中的不同树中选取。目的

规则的组成部分。先决条件

跟随。`step choose`例

`step emit`

重要 

给定的CRUSH规则可以分配给多个池，但是单个池不可能有多个CRUSH规则。

`firstn` 与 `indep`描述

控制在CRUSH映射中标记了项目（OSD）时CRUSH使用的替换策略。如果此规则与复制池一起使用，则应为`firstn`；对于擦除编码池，应使用`indep`。

原因与它们在先前选择的设备发生故障时的行为方式有关。假设您有一个PG存储在OSD 1、2、3、4、5上。然后3下降。

在“ firstn”模式下，CRUSH只需将其计算调整为选择1和2，然后选择3，但是发现它已关闭，因此它重试并选择4和5，然后继续选择一个新的OSD6。因此最终的CRUSH映射更改为1、2、3、4、5-&gt; 1、2、4、5、6。

但是，如果要存储EC池，则意味着您只需更改映射到OSD 4、5和6的数据！因此，“独立”模式试图不这样做。相反，您可以期望它在选择失败的OSD 3时再次尝试并选择6，以进行以下最终转换：1，2，3，4，5-&gt; 1，2，6，4，5

### 从旧版SSD规则迁移到设备类

以前必须手动编辑CRUSH映射并为每种专用设备类型（例如SSD）维护并行层次结构，以便编写适用于那些设备的规则。自发光版本以来，_设备类_功能已透明启用了此功能。

但是，以微不足道的方式从现有的，手动定制的每个设备映射迁移到新的设备类规则，将导致系统中的所有数据重新组合。

的`crushtool`一些命令可以转换旧规则和层次结构，以便您可以开始使用基于新类的规则。可能有三种类型的转换：

1. `--reclassify-root <root-name> <device-class>`

   这将需要在根名称下的层次的一切，任何调整规则的引用，通过根来代替。它使用以下方式对存储区进行重新编号：将旧ID代替用于指定类的“影子树”，这样就不会发生数据移动。`take <root-name>take <root-name> class <device-class>`

   例如，假设您有一个现有规则，例如：

   ```text
   rule replicated_ruleset {
      id 0
      type replicated
      min_size 1
      max_size 10
      step take default
      step chooseleaf firstn 0 type rack
      step emit
   }
   ```

   如果将根默认目录重新分类为hdd类，则规则将变为：

   ```text
   rule replicated_ruleset {
      id 0
      type replicated
      min_size 1
      max_size 10
      step take default class hdd
      step chooseleaf firstn 0 type rack
      step emit
   }
   ```

2. `--set-subtree-class <bucket-name> <device-class>`

   这将以 指定的设备类标记以根_名称_为根的子树中的每个设备。

   通常将此`--reclassify-root` 选项与选项结合使用，以确保该根目录中的所有设备都标记有正确的类。但是，在某些情况下，其中某些设备（正确）具有不同的类，我们不希望对其进行重新标记。在这种情况下，可以排除该`--set-subtree-class` 选项。这意味着重新映射过程将不是完美的，因为先前的规则分布在多个类的设备上，但是调整后的规则将仅映射到指定_设备类的设备_，但是当数字键被接受时，这通常是可接受的数据移动级别离群设备的数量很小。

3. `--reclassify-bucket <match-pattern> <device-class> <default-parent>`

   这将允许您将并行的特定于类型的层次结构与正常层次结构合并。例如，许多用户拥有如下地图：

   ```text
   host node1 {
      id -2           # do not change unnecessarily
      # weight 109.152
      alg straw
      hash 0  # rjenkins1
      item osd.0 weight 9.096
      item osd.1 weight 9.096
      item osd.2 weight 9.096
      item osd.3 weight 9.096
      item osd.4 weight 9.096
      item osd.5 weight 9.096
      ...
   }

   host node1-ssd {
      id -10          # do not change unnecessarily
      # weight 2.000
      alg straw
      hash 0  # rjenkins1
      item osd.80 weight 2.000
      ...
   }

   root default {
      id -1           # do not change unnecessarily
      alg straw
      hash 0  # rjenkins1
      item node1 weight 110.967
      ...
   }

   root ssd {
      id -18          # do not change unnecessarily
      # weight 16.000
      alg straw
      hash 0  # rjenkins1
      item node1-ssd weight 2.000
      ...
   }
   ```

   此功能将对与模式匹配的每个存储桶进行重新分类。模式可以看起来像`%suffix`或`prefix%`。例如，在上面的示例中，我们将使用pattern `%-ssd`。对于每个匹配的存储桶，名称的其余部分（与`%`通配符匹配）指定_基本存储桶_。匹配的存储桶中的所有设备都标记有指定的设备类，然后移至基本存储桶。如果基本存储桶不存在（例如，`node12-ssd`存在但`node12`不存在），则将在指定的_默认父_存储桶下创建该基本存储桶并将其链接 。在每种情况下，我们都会小心保留新影子桶的旧桶ID，以防止数据移动。任何规则`take` 调整参照旧桶的步骤。

4. `--reclassify-bucket <bucket-name> <device-class> <base-bucket>`

   也可以在不使用通配符的情况下使用同一命令来映射单个存储桶。例如，在前面的示例中，我们希望将 `ssd`存储桶映射到`default`存储桶。

转换由以上片段组成的地图的最终命令将类似于：

```text
$ ceph osd getcrushmap -o original
$ crushtool -i original --reclassify \
    --set-subtree-class default hdd \
    --reclassify-root default hdd \
    --reclassify-bucket %-ssd ssd default \
    --reclassify-bucket ssd ssd default \
    -o adjusted
```

为了确保转换正确，有一条`--compare`命令将测试CRUSH映射的大量输入样本，并确保返回相同的结果。这些输入由适用于该`--test`命令的相同选项控制。对于上面的示例，：

```text
$ crushtool -i original --compare adjusted
rule 0 had 0/10240 mismatched mappings (0)
rule 1 had 0/10240 mismatched mappings (0)
maps appear equivalent
```

如果存在差异，您会在括号中看到重新映射的输入比例。

如果您对调整后的地图满意，可以使用以下方法将其应用于集群：

```text
ceph osd setcrushmap -i adjusted
```

### 调整CRUSH的困难方法

如果可以确保所有客户端都在运行最新代码，则可以通过提取CRUSH映射，修改值并将其重新注入到集群中来调整可调参数。

* 提取最新的CRUSH地图：

  ```text
  ceph osd getcrushmap -o /tmp/crush
  ```

* 调整可调项。对于我们测试过的大型集群和小型集群，这些值似乎都提供了最佳性能。您将需要另外指定`--enable-unsafe-tunables`参数以 `crushtool`使其起作用。请谨慎使用此选项：

  ```text
  crushtool -i /tmp/crush --set-choose-local-tries 0 --set-choose-local-fallback-tries 0 --set-choose-total-tries 50 -o /tmp/crush.new
  ```

* 重新注入修改后的地图：

  ```text
  ceph osd setcrushmap -i /tmp/crush.new
  ```

### 旧版值

作为参考，可以使用以下命令设置CRUSH可调参数的旧值：

```text
crushtool -i /tmp/crush --set-choose-local-tries 2 --set-choose-local-fallback-tries 5 --set-choose-total-tries 19 --set-chooseleaf-descend-once 0 --set-chooseleaf-vary-r 0 -o /tmp/crush.legacy
```

同样，`--enable-unsafe-tunables`需要特殊选项。此外，如上所述，`ceph-osd`由于功能位没有得到很好的执行，因此在恢复到旧值后请小心运行守护程序的旧版本 。

