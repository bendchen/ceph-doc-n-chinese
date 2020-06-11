# CRUSH MAP

## CRUSH地图

所述CRUSH算法确定如何存储和通过计算数据存储位置检索数据。CRUSH使Ceph客户端能够直接与OSD通信，而不是通过集中式服务器或代理进行通信。通过算法确定的存储和检索数据的方法，Ceph避免了单点故障，性能瓶颈以及对其可扩展性的物理限制。

CRUSH需要您的群集的映射，并使用CRUSH映射伪随机地存储和检索OSD中的数据，并使数据在整个群集中均匀分布。有关CRUSH的详细讨论，请参见 [CRUSH-复制数据的受控，可伸缩，分散式放置](https://ceph.com/wp-content/uploads/2016/08/weil-crush-sc06.pdf)

CRUSH映射包含OSD列表，用于将设备聚合到物理位置的“存储桶”列表以及告诉CRUSH如何在Ceph集群池中复制数据的规则列表。通过反映安装的底层物理组织，CRUSH可以对相关设备故障的潜在来源进行建模（从而解决）。典型的来源包括物理邻近度，共享的电源和共享的网络。通过将此信息编码到群集图中，CRUSH放置策略可以将对象副本复制到不同的故障域中，同时仍保持所需的分布。例如，为了解决并发故障的可能性，可能需要确保数据副本位于使用不同架子，机架，电源，控制器和/或物理位置的设备上。

部署OSD时，它们会自动放置在CRUSH映射中的`host`节点下，该 节点的名称是运行它们的主机的主机名。这与默认的CRUSH故障域结合在一起，可确保副本或擦除代码碎片在主机之间是分开的，并且单个主机故障不会影响可用性。但是，对于较大的群集，管理员应仔细考虑他们对故障域的选择。例如，在中大型集群中，跨机架分隔副本很常见。

### 粉碎位置

OSD在CRUSH地图层次结构方面的位置称为。该位置说明符采用描述位置的键和值对列表的形式。例如，如果一个OSD位于特定的行，机架，机箱和主机中，并且是“默认” CRUSH树的一部分（绝大多数集群就是这种情况），则其压缩位置可以描述为：`crush location`

```text
root=default row=a rack=a2 chassis=a2a host=a2a1
```

注意：

1. 请注意，键的顺序无关紧要。
2. 键名（的左侧`=`）必须是有效的CRUSH `type`。默认情况下，它们包括根目录，数据中心，房间，行，窗格，pdu，机架，机箱和主机，但是可以通过修改CRUSH映射将这些类型自定义为任何合适的类型。
3. 并非所有键都需要指定。例如，默认情况下，Ceph会自动将`ceph-osd`守护进程的位置设置为 （基于的输出）。`root=default host=HOSTNAMEhostname -s`

OSD的压缩位置通常通过 在文件中设置的config选项表示。每次OSD启动时，它都会验证它在CRUSH映射中的正确位置，如果不在，则它会自行移动。要禁用此自动CRUSH映射管理，请在部分中将以下内容添加到您的配置文件中：`crush locationceph.conf[osd]`

```text
osd crush update on start = false
```

#### 自定义位置钩子

定制的位置挂钩可用于在启动时生成更完整的挤压位置。暗恋位置是根据以下优先顺序：

1. ceph.conf中的一个选项。`crush location`
2. 使用命令生成主机名的默认位置。`root=default host=HOSTNAMEhostname -s`

它本身没有用，因为OSD本身具有完全相同的行为。但是，可以编写脚本来提供其他位置字段（例如机架或数据中心），然后通过config选项启用挂钩：

```text
crush location hook = /path/to/customized-ceph-crush-location
```

这个钩子传递了几个参数（在下面），并且应该使用CRUSH位置描述将一行输出到stdout：

```text
--cluster CLUSTER --id ID --type TYPE
```

在此集群名称通常是“CEPH”中，id是守护进程标识符（例如，OSD数或守护进程标识符），并且后台程序类型是`osd`，`mds`，或类似的。

例如，另外一个基于假设文件指定机架位置的简单挂钩`/etc/rack`可能是：

```text
#!/bin/sh
echo "host=$(hostname -s) rack=$(cat /etc/rack) root=default"
```

### CRUSH结构

粗略地说，CRUSH映射由描述群集物理拓扑的层次结构和一组规则定义，这些规则定义了有关如何将数据放置在这些设备上的策略。层次结构`ceph-osd`在叶子上具有设备（守护程序），以及与其他物理功能或分组相对应的内部节点：主机，机架，行，数据中心等。规则描述了如何按照该层次结构放置副本（例如，“不同机架中的三个副本”）。

#### 设备

设备是`ceph-osd`可以存储数据的单个守护程序。通常，您将为集群中的每个OSD守护程序定义一个。设备由ID（非负整数）和名称标识，通常`osd.N`在哪里`N`是设备ID。

设备还可以具有与它们关联的_设备类_（例如 `hdd`或`ssd`），从而可以通过挤压规则方便地将它们作为目标。

#### 类型和铲斗

桶是层次结构中内部节点（主机，机架，行等）的CRUSH术语。CRUSH映射定义了一系列用于描述这些节点的_类型_。默认情况下，这些类型包括：

* osd（或设备）
* 主办
* 机壳
* 架
* 行
* du
* 荚
* 房间
* 数据中心
* 区域
* 根

大多数群集仅使用其中少数几个类型，并且可以根据需要定义其他类型。

该层次结构是通过`osd`在叶子处的设备（通常为type ），具有非设备类型的内部节点以及type的根节点 构建的`root`。例如，

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-0c657a3914dc0ac118dea82d5e61a60cb275d827.png)

层次结构中的每个节点（设备或存储桶）都有 与之关联的_权重_，指示设备或层次结构子树应存储的总数据中的相对比例。权重设置在叶子上，指示设备的大小，并从那里自动对树进行求和，这样默认节点的权重将是它下面包含的所有设备的总和。通常，权重以TB为单位。

您可以使用以下方法简单查看群集的CRUSH层次结构，包括权重：

```text
ceph osd crush tree
```

#### 规则

规则定义有关如何在层次结构中的设备之间分配数据的策略。

CRUSH规则定义了放置和复制策略或分发策略，使您可以精确指定CRUSH如何放置对象副本。例如，您可能创建一条规则，该规则选择一对目标以进行2路镜像，另一条规则在两个不同的数据中心中选择三个目标以进行3路镜像，再创建一条规则以擦除六个存储设备上的编码。有关CRUSH规则的详细讨论，请参阅[CRUSH控制的，可伸缩的，去中心化的复制数据放置，](https://ceph.com/wp-content/uploads/2016/08/weil-crush-sc06.pdf)尤其是**第3.2节**。

在几乎所有情况下，都可以通过CLI来指定CRUSH规则，方法是指定规则所用的_池类型_（复制或擦除编码），_故障域_以及_设备类（_可选）。在极少数情况下，必须通过手动编辑CRUSH映射来手工编写规则。

您可以使用以下命令查看为集群定义了哪些规则：

```text
ceph osd crush rule ls
```

您可以使用以下方法查看规则的内容：

```text
ceph osd crush rule dump
```

#### 设备类

每个设备可以选择具有与之关联的_类_。默认情况下，OSD在启动时会根据其支持的设备类型自动将其类设置为 hdd，ssd或nvme。

可以使用以下命令显式设置一个或多个OSD的设备类：

```text
ceph osd crush set-device-class <class> <osd-name> [...]
```

设置设备类别后，除非使用以下方法取消设置旧类别，否则无法将其更改为另一个类别：

```text
ceph osd crush rm-device-class <osd-name> [...]
```

这使管理员可以设置设备类，而无需在OSD重新启动时或通过其他脚本更改类。

可以使用以下方式创建针对特定设备类别的放置规则：

```text
ceph osd crush rule create-replicated <rule-name> <root> <failure-domain> <class>
```

然后可以将池更改为在以下情况下使用新规则：

```text
ceph osd pool set <pool-name> crush_rule <rule-name>
```

通过为使用中的每个仅包含该类设备的设备类创建“影子” CRUSH层次结构来实现设备类。然后，规则可以在影子层次结构上分布数据。这种方法的一个优点是，它与旧的Ceph客户端完全向后兼容。您可以使用以下阴影项查看CRUSH层次结构：

```text
ceph osd crush tree --show-shadow
```

对于在Luminous之前创建的较旧的群集，这些群集依赖于手动制作的CRUSH映射来维护每个设备类型的层次结构，可以使用一个 _重新分类_工具来帮助过渡到设备类而无需触发数据移动（请参阅[从旧SSD规则迁移到设备类](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map-edits/#crush-reclassify)）。 。

#### 权重集

甲_重量组_是一种替代组权重计算数据放置时使用的。与CRUSH映射中的每个设备关联的正常权重是根据设备大小设置的，并指示_应将_多少数据存储在何处。但是，由于CRUSH基于伪随机放置过程，因此与理想分布总是存在一些差异，就像掷骰子60次不会导致准确掷出10个和10个6一样。权重集使群集可以根据群集的详细信息（层次结构，池等）进行数值优化，以实现平衡的分布。

支持两种类型的权重集：

> 1. 甲**compat的**重量组为单替代组权重的集群中的每个设备和节点的。这不适用于校正所有异常（例如，不同池的放置组可能具有不同的大小并具有不同的负载水平，但平衡器将对其进行大多数处理）。但是，兼容权重集具有与Ceph的早期版本_向后兼容_的巨大优势 ，这意味着即使权重集是在Luminous v12.2.z中首次引入的，较旧的客户端（例如firefly）仍可以连接到当使用组合权重集平衡数据时进行聚类。
> 2. 甲**每个缓冲池的**重量集是因为它允许放置到每个数据池来优化更加灵活。此外，可以针对每个放置位置调整权重，从而使优化器可以校正数据向权重较小的设备（相对于其同级设备）的细微偏斜（这种影响通常仅在非常大的集群中才会出现，但可能会导致平衡问题） ）。

使用权重集时，与层次结构中每个节点关联的权重在以下命令中显示为单独的列（标记为 `(compat)`或池名称）：

```text
ceph osd crush tree
```

当同时使用_compat_和_per-pool_权重集时，特定池的数据放置将使用其自己的per-pool权重集（如果存在）。如果没有，它将使用设置的配重值。如果两者都不存在，它将使用正常的CRUSH权重。

尽管可以手动设置和操作砝码组，但建议启用_平衡器_模块以自动进行设置。

### 修改CRUSH映射

#### 添加/移动

要在正在运行的群集的CRUSH映射中添加或移动OSD，请执行以下操作：

```text
ceph osd crush set {name} {weight} root={root} [{bucket-type}={bucket-name} ...]
```

哪里：

`name`描述

OSD的全名。类型

串需要

是例

`osd.0`

`weight`描述

OSD的CRUSH权重，通常以太字节（TB）为单位。类型

双需要

是例

`2.0`

`root`描述

OSD所在树的根节点（通常为`default`）类型

键/值对。需要

是例

`root=default`

`bucket-type`描述

您可以在CRUSH层次结构中指定OSD的位置。类型

键/值对。需要

没有例

`datacenter=dc1 room=room1 row=foo rack=bar host=foo-bar-1`

以下示例将添加`osd.0`到层次结构中，或将OSD从先前的位置移出。

```text
ceph osd crush set osd.0 1.0 root=default datacenter=dc1 room=room1 row=foo rack=bar host=foo-bar-1
```

#### 调整OSD权重

要在正在运行的群集的CRUSH映射中调整OSD的压缩重量，请执行以下操作：

```text
ceph osd crush reweight {name} {weight}
```

哪里：

`name`描述

OSD的全名。类型

串需要

是例

`osd.0`

`weight`描述

OSD的CRUSH权重。类型

双需要

是例

`2.0`

#### 删除

要从正在运行的群集的CRUSH映射中删除OSD，请执行以下操作：

```text
ceph osd crush remove {name}
```

哪里：

`name`描述

OSD的全名。类型

串需要

是例

`osd.0`

#### 添加一个桶

要在正在运行的集群的CRUSH映射中添加存储桶，请执行以下 命令：`ceph osd crush add-bucket`

```text
ceph osd crush add-bucket {bucket-name} {bucket-type}
```

哪里：

`bucket-name`描述

存储桶的全名。类型

串需要

是例

`rack12`

`bucket-type`描述

铲斗的类型。该类型必须已经存在于层次结构中。类型

串需要

是例

`rack`

以下示例将`rack12`存储桶添加到层次结构中：

```text
ceph osd crush add-bucket rack12 rack
```

#### 移动桶

要将存储桶移动到CRUSH地图层次结构中的其他位置或位置，请执行以下操作：

```text
ceph osd crush move {bucket-name} {bucket-type}={bucket-name}, [...]
```

哪里：

`bucket-name`描述

要移动/重新定位的铲斗的名称。类型

串需要

是例

`foo-bar-1`

`bucket-type`描述

您可以在CRUSH层次结构中指定存储桶的位置。类型

键/值对。需要

没有例

`datacenter=dc1 room=room1 row=foo rack=bar host=foo-bar-1`

#### 删除桶

要从CRUSH映射层次结构中删除存储桶，请执行以下操作：

```text
ceph osd crush remove {bucket-name}
```

注意 

从CRUSH层次结构中将其删除之前，存储桶必须为空。

哪里：

`bucket-name`描述

您要删除的存储桶的名称。类型

串需要

是例

`rack12`

以下示例`rack12`从层次结构中删除存储桶：

```text
ceph osd crush remove rack12
```

#### 创建一个兼容体重集

要创建_兼容的体重组_：

```text
ceph osd crush weight-set create-compat
```

兼容重量组的重量可以通过以下方式调整：

```text
ceph osd crush weight-set reweight-compat {name} {weight}
```

可以用以下方法破坏组合重量组：

```text
ceph osd crush weight-set rm-compat
```

#### 创建每个池的权重集

要为特定池创建权重集，请执行以下操作：

```text
ceph osd crush weight-set create {pool-name} {mode}
```

注意 

每池权重集要求所有服务器和守护程序都运行Luminous v12.2.z或更高版本。

哪里：

`pool-name`描述

RADOS池的名称类型

串需要

是例

`rbd`

`mode`描述

无论是`flat`或`positional`。阿_平_重组具有用于每个设备或桶的单个权重。甲 _位置_重组具有用于在所得到的位置的映射各位置的潜在的不同权重。例如，如果池的副本数为3，则位置权重集将为每个设备和存储桶具有三个权重。类型

串需要

是例

`flat`

调整重量集中的物品的重量：

```text
ceph osd crush weight-set reweight {pool-name} {item-name} {weight [...]}
```

要列出现有的权重集，请执行以下操作：

```text
ceph osd crush weight-set ls
```

要删除砝码组，请执行以下操作：

```text
ceph osd crush weight-set rm {pool-name}
```

#### 为复制池创建规则

对于复制池，创建CRUSH规则时的主要决策是故障域的类型。例如，如果选择的故障域`host`，则CRUSH将确保数据的每个副本都存储在不同的主机上。如果`rack` 选择，则每个副本将存储在不同的机架中。您选择哪种故障域主要取决于群集的大小以及层次结构的结构。

通常，整个集群层次结构嵌套在名为的根节点下`default`。如果您已自定义层次结构，则可能要创建一个嵌套在层次结构中其他节点上的规则。与该节点关联的类型无关紧要（不必是`root`节点）。

还可以创建一个规则，将数据放置位置限制为特定_类别_的设备。默认情况下，Ceph OSD会根据所使用设备的基础类型自动将其自身分类为`hdd`或`ssd`。这些类也可以自定义。

要创建复制规则，请执行以下操作：

```text
ceph osd crush rule create-replicated {name} {root} {failure-domain-type} [{class}]
```

哪里：

`name`描述

规则名称类型

串需要

是例

`rbd-rule`

`root`描述

应在其下放置数据的节点的名称。类型

串需要

是例

`default`

`failure-domain-type`描述

我们应在其中分离副本的CRUSH节点的类型。类型

串需要

是例

`rack`

`class`描述

设备类别数据应放在上面。类型

串需要

没有例

`ssd`

#### 为擦除编码池创建规则

对于采用擦除编码的池，需要做出与复制池相同的基本决策：故障域是什么，层次结构中的哪个节点将数据放置在（通常为`default`），并且放置位置将限制在特定设备上类。但是，删除代码池的创建方式略有不同，因为需要根据所使用的删除代码仔细构建它们。因此，您必须在_擦除代码配置文件中_包含此信息。使用配置文件创建池时，将根据该规则显式或自动创建CRUSH规则。

擦除代码配置文件可以列出：

```text
ceph osd erasure-code-profile ls
```

现有个人资料可以通过以下方式查看：

```text
ceph osd erasure-code-profile get {profile-name}
```

通常，绝对不要修改配置文件。而是在创建新池或为现有池创建新规则时创建并使用新配置文件。

擦除码配置文件由一组键=值对组成。其中大多数控制擦除代码的行为，该擦除代码正在对池中的数据进行编码。`crush-`但是，那些以开头的文件会影响所创建的CRUSH规则。

感兴趣的擦除码配置文件属性为：

> * **rush-root**：将数据放置在\[default：`default`\] 下的CRUSH节点的名称。
> * **rush-failure-domain**：CRUSH类型，用于跨\[default：`host`\] 分隔擦除编码的碎片。
> * **rush-device-class**：用于放置数据的设备类\[默认：无，表示使用了所有设备\]。
> * **k**和**m**（对于`lrc`插件，为**l**）：它们确定擦除代码分片的数量，从而影响最终的CRUSH规则。

定义配置文件后，您可以使用以下方法创建CRUSH规则：

```text
ceph osd crush rule create-erasure {name} {profile-name}
```

#### 删除规则

可以通过以下方式删除池未使用的规则：

```text
ceph osd crush rule rm {rule-name}
```

### 可调项

随着时间的推移，我们已经（并将继续改进）用于计算数据放置的CRUSH算法。为了支持行为的更改，我们引入了一系列可调选项，用于控制使用算法的传统版本还是改进版本。

为了使用更新的可调参数，客户端和服务器都必须支持新版本的CRUSH。因此，我们创建 `profiles`了以引入它们的Ceph版本命名的。例如，`firefly`萤火虫版本中首先支持该可调参数，并且不适用于较旧的（例如，饺子）客户端。一旦将给定的一组可调参数从传统的默认行为更改为后，`ceph-mon`和`ceph-osd`将阻止不支持新的CRUSH功能的旧客户端连接到群集。

#### ARGONAUT（旧版）

argonaut和较早版本使用的旧式CRUSH行为在大多数集群上都可以正常工作，只要没有太多OSD被标出即可。

#### 短尾（CRUSH\_TUNABLES2）

短尾可调配置文件修复了一些关键的不良行为：

> * 对于在叶存储桶中具有少量设备的层次结构，某些PG映射到少于所需数量的副本。这通常发生在具有“主机”节点的层次结构中，每个节点下方嵌套有少量（1-3）OSD。
> * 对于大型群集，少量百分比的PG映射到少于所需数量的OSD。当层次结构有多个层（例如，行，机架，主机，osd）时，这种情况更为普遍。
> * 标记出某些OSD后，数据倾向于重新分配给附近的OSD，而不是整个层次结构。

新的可调项是：

> * `choose_local_tries`：本地重试次数。旧值是2，最佳值为0。
> * `choose_local_fallback_tries`：旧值是5，最佳值为0。
> * `choose_total_tries`：选择项目的总尝试次数。旧值是19，后续测试表明值50更适合典型群集。对于非常大的群集，可能需要更大的值。
> * `chooseleaf_descend_once`：递归的selectleaf尝试将重试，还是仅尝试一次并允许原始展示位置重试。旧版默认值为0，最佳值为1。

迁移影响：

> * 从argonaut变为bobtail可调参数会触发少量的数据移动。在已填充数据的群集上请多加注意。

#### 萤火虫（CRUSH\_TUNABLES3）

萤火虫可调配置文件解决了`chooseleaf`CRUSH规则行为的问题，当标记了太多的OSD时，该问题往往导致PG映射的结果太少。

新的可调参数是：

> * `chooseleaf_vary_r`：基于父级已经进行了多少次尝试，递归的selectleaf尝试是否将从r的非零值开始。旧版默认值为0，但使用此值CRUSH有时无法找到映射。最佳值（在计算成本和正确性方面）为1。

迁移影响：

> * 对于具有大量现有数据的现有集群，将其从0更改为1将导致大量数据移动。值4或5将允许CRUSH查找有效的映射，但将减少数据移动。

#### STRAW\_CALC\_VERSION可调参数（也由FIREFLY引入）

计算和存储在CRUSH映射中的`straw`铲斗内部重量存在一些问题。具体来说，当存在CRUSH权重为0的项或权重和某些重复权重的混合体时，CRUSH会错误地分配数据（即，不与权重成正比）。

新的可调参数是：

> * `straw_calc_version`：值0保留旧的内部重量计算；值为1可修复该行为。

迁移影响：

> * _如果_集群遇到了问题之一，那么移至straw\_calc\_version 1，然后调整秸秆存储桶（通过添加，删除或重新加权项目，或使用reweight-all命令）可能会触发少量到中等数量的数据移动。

该可调选项非常特殊，因为它绝对不会影响客户端所需的内核版本。

#### 锤子（CRUSH\_V4）

音锤可调配置文件仅通过更改配置文件就不会影响现有CRUSH映射的映射。然而：

> * `straw2`支持新的存储桶类型（）。新的 `straw2`存储桶类型修复了原始`straw`存储桶中的一些限制 。具体而言，旧的`straw`存储桶会更改一些在调整权重时应已更改的映射，同时`straw2`达到了仅更改与更改了权重的存储桶项之间的映射的原始目标。
> * `straw2` 是所有新创建的存储桶的默认设置。

迁移影响：

> * 将存储桶类型从更改`straw`为`straw2`会导致相当少量的数据移动，具体取决于存储桶项目权重之间的差异。当权重相同时，没有数据将移动，并且当项目权重显着变化时，将有更多的移动。

#### 宝石（CRUSH\_TUNABLES5）

珠宝可调配置文件改善了CRUSH的整体性能，从而在将OSD标记出群集时，更改的映射显着减少。

新的可调参数是：

> * `chooseleaf_stable`：递归选择叶尝试是否将为内部循环使用更好的值，从而在标记OSD时大大减少映射更改的次数。旧值是0，而新值1使用新方法。

迁移影响：

> * 在现有群集上更改此值将导致大量数据移动，因为几乎每个PG映射都可能更改。

#### 哪些客户端版本支持CRUSH\_TUNABLES 

> * argonaut系列，v0.48.1或更高版本
> * v0.49或更高版本
> * Linux内核版本v3.6或更高版本（用于文件系统和RBD内核客户端）

#### 哪些客户端版本支持CRUSH\_TUNABLES2 

> * v0.55或更高版本，包括短尾系列（v0.56.x）
> * Linux内核版本v3.9或更高版本（用于文件系统和RBD内核客户端）

#### 哪些客户端版本支持CRUSH\_TUNABLES3 

> * v0.78（萤火虫）或更高版本
> * Linux内核版本v3.15或更高版本（用于文件系统和RBD内核客户端）

#### 哪些客户端版本支持CRUSH\_V4 

> * v0.94（锤）或更高版本
> * Linux内核版本v4.1或更高版本（用于文件系统和RBD内核客户端）

#### 哪些客户端版本支持CRUSH\_TUNABLES5 

> * v10.0.2（jewel）或更高版本
> * Linux内核版本v4.5或更高版本（用于文件系统和RBD内核客户端）

#### 当可调参数不是最优时警告

从v0.74版本开始，如果当前的CRUSH可调参数未包含`default`配置文件中的所有最佳值，则Ceph将发出运行状况警告（有关 配置文件的含义，请参见下文`default`）。要使此警告消失，您有两种选择：

1. 在现有集群上调整可调项。请注意，这将导致一些数据移动（可能多达10％）。这是首选的方法，但是在数据移动可能影响性能的生产群集上应格外小心。您可以使用以下方法启用最佳可调参数：

   ```text
   ceph osd crush tunables optimal
   ```

   如果事情进展不顺利（例如，过多的负载）并且没有取得太多进展，或者存在客户端兼容性问题（旧的内核cephfs或rbd客户端，或预bobtail librados客户端），则可以使用以下方法切换回：

   ```text
   ceph osd crush tunables legacy
   ```

2. 您可以通过在ceph.conf `[mon]`节中添加以下选项来消除警告，而无需对CRUSH进行任何更改：

   ```text
   mon warn on legacy crush tunables = false
   ```

   为了使更改生效，您将需要重新启动监视器，或通过以下方式将选项应用于正在运行的监视器：

   ```text
   ceph tell mon.\* config set mon_warn_on_legacy_crush_tunables false
   ```

#### 有几个重要的点

> * 调整这些值将导致某些PG在存储节点之间移动。如果Ceph集群已经存储了很多数据，请为一部分数据做好准备。
> * 在`ceph-osd`和`ceph-mon`守护进程就会开始要求新连接的功能位，因为他们得到了更新的地图。但是，已经连接的客户端实际上已经成为父级，并且如果它们不支持新功能，则会表现异常。
> * 如果将CRUSH可调参数设置为非旧值，然后又更改回默认值，`ceph-osd`则不需要守护程序来支持该功能。但是，OSD对等过程需要检查和了解旧地图。因此，`ceph-osd`即使集群先前使用了非传统的CRUSH值，也不应运行该守护程序的旧版本，即使映射的最新版本已切换回使用旧式默认值。

#### 调整CRUSH 

调整挤压可调参数的最简单方法是更改​​为已知轮廓。那些是：

> * `legacy`：来自argonaut和更早版本的旧行为。
> * `argonaut`：原始argonaut版本支持的旧值
> * `bobtail`：bobtail版本支持的值
> * `firefly`：firefly版本支持的值
> * `hammer`：锤子释放所支持的值
> * `jewel`：珠宝发布支持的值
> * `optimal`：当前Ceph版本的最佳（即最佳）值
> * `default`：从头开始安装的新集群的默认值。这些值取决于Ceph的当前版本，是经过硬编码的，通常是最佳值和旧值的组合。这些值通常与`optimal`以前的LTS版本或我们最近为之发行的最新版本的配置文件相匹配，但更多用户需要拥有最新的客户端。

您可以使用以下命令在正在运行的集群上选择配置文件：

```text
ceph osd crush tunables {PROFILE}
```

请注意，这可能会导致某些数据移动。

### 主亲和力

当Ceph客户端读取或写入数据时，它总是联系代理集中的主要OSD。对于set ，是主要的。有时，与其他OSD相比，OSD不太适合充当主服务器（例如，它的磁盘或控制器慢）。为了在最大程度地利用硬件的同时防止性能瓶颈（特别是在读取操作上），可以设置Ceph OSD的主要关联性，以使CRUSH不太可能在行动集中将OSD用作主要因素。`[2, 3, 4]osd.2`

```text
ceph osd primary-affinity <osd-id> <weight>
```

`1`默认情况下_，_主要亲缘关系是默认的（_即_ OSD可以充当主要亲和力）。您可以从设置OSD主要范围`0-1`，其中`0`表示OSD **不能**用作主要对象，并且`1`意味着OSD可以用作主要对象。当重量，它不太可能CRUSH将选择Ceph的OSD守护进程充当主。`< 1`

