# PG故障排除

## PG故障排除

### 放置组永远不会干净

当您创建集群并且集群仍处于`active`， `active+remapped`或`active+degraded`状态而从未达到 `active+clean`状态时，您的配置可能会出现问题。

您可能需要查看[Pool，PG和CRUSH Config Reference中的设置](https://docs.ceph.com/docs/nautilus/rados/configuration/pool-pg-config-ref) 并进行适当的调整。

通常，应使用一个以上的OSD和大于1个对象副本的池大小来运行群集。

#### 一个节点集群

Ceph不再提供在单个节点上运行的文档，因为您永远不会在单个节点上部署专为分布式计算而设计的系统。另外，由于Linux内核本身存在问题，除非在客户端上使用VM，否则在包含Ceph守护程序的单个节点上安装客户端内核模块可能会导致死锁。尽管有本文所述的局限性，您仍可以在1节点配置中试用Ceph。

如果您试图在单个节点上创建集群，则必须在创建监视器和OSD之前将Ceph配置文件中设置的默认值从（含义 或）更改为（含义）。这告诉Ceph，一个OSD可以与同一主机上的另一个OSD对等。如果您尝试设置一个大于1的群集，则Ceph将尝试根据设置将一个OSD的PG与另一个节点，机箱，机架，行甚至数据中心上的另一个OSD的PG对等。 。`osd crush chooseleaf type1hostnode0osdosd crush chooseleaf type0`

小费 

不要将内核客户端与Ceph存储群集直接安装在同一节点上，因为可能会发生内核冲突。但是，您可以在单个节点上的虚拟机（VM）中挂载内核客户端。

如果要使用单个磁盘创建OSD，则必须首先手动为数据创建目录。例如：

```text
ceph-deploy osd create --data {disk} {host}
```

#### OSD少于副本

如果您将两个OSD调到`up`和`in`状态，但仍然看不到展示位置组，则可以将其 设置为大于。`active + cleanosd pool default size2`

有几种方法可以解决这种情况。如果要在具有两个副本的状态下操作集群，可以将设置为 ，以便可以在状态下写入对象。您也可以将 设置设置为，以便只有两个存储的副本（原始副本和一个副本），在这种情况下，群集应达到某种 状态。`active + degradedosd pool default min size2active + degradedosd pool default size2active + clean`

注意 

您可以在运行时进行更改。如果您在Ceph配置文件中进行更改，则可能需要重新启动集群。

#### 池大小= 

如果将设置为，则将只有该对象的一个​​副本。OSD依靠其他OSD来告诉他们应该拥有哪些对象。如果第一个OSD具有对象的副本，并且没有第二个副本，则没有第二个OSD可以告诉第一个OSD它应该具有该副本。对于映射到第一个OSD的每个放置组（请参阅参考资料 ），可以通过运行以下命令来强制第一个OSD注意到其所需的放置组：`osd pool default size1ceph pg dump`

```text
ceph osd force-create-pg <pgid>
```

#### CRUSH映射错误

另一个放置组不干净的候选对象涉及您的CRUSH映射中的错误。

### 卡住安置组

放置组在失败后进入“降级”或“对等”状态是正常的。通常，这些状态指示故障恢复过程中的正常进程。但是，如果放置组长时间处于这些状态之一，则可能表示存在较大问题。因此，当放置组陷入非最佳状态时，监视器将发出警告。具体来说，我们检查以下内容：

* `inactive`-展示位置组`active`的时间过长（即，它无法处理读/写请求）。
* `unclean`-展示位置组`clean`的时间过长（即，它无法从以前的故障中完全恢复过来）。
* `stale`-放置组状态尚未由a更新`ceph-osd`，表明存储此放置组的所有节点都可能为`down`。

您可以使用以下之一明确列出停留的放置组：

```text
ceph pg dump_stuck stale
ceph pg dump_stuck inactive
ceph pg dump_stuck unclean
```

对于卡住的`stale`放置组，通常需要`ceph-osd`重新运行正确的守护程序。对于卡住的`inactive`放置组，通常是一个对等问题（请参阅[放置组向下-对等失败](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-pg/#failures-osd-peering)）。对于卡住的`unclean`放置组，通常存在一些阻碍恢复完成的事情，例如未找到的对象（请参见未 [找到的对象](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-pg/#failures-osd-unfound)）；

### 放置组向下-对等失败

在某些情况下，对`ceph-osd` 等进程可能会遇到问题，从而导致PG变得无法使用。例如，可能报告：`ceph health`

```text
ceph health detail
HEALTH_ERR 7 pgs degraded; 12 pgs down; 12 pgs peering; 1 pgs recovering; 6 pgs stuck unclean; 114/3300 degraded (3.455%); 1/3 in osds are down
...
pg 0.5 is down+peering
pg 1.4 is down+peering
...
osd.1 is down since epoch 69, last address 192.168.106.220:6801/8651
```

我们可以查询集群以确定PG为何标记`down`为：

```text
ceph pg 0.5 query
```

```text
{ "state": "down+peering",
  ...
  "recovery_state": [
       { "name": "Started\/Primary\/Peering\/GetInfo",
         "enter_time": "2012-03-06 14:40:16.169679",
         "requested_info_from": []},
       { "name": "Started\/Primary\/Peering",
         "enter_time": "2012-03-06 14:40:16.169659",
         "probing_osds": [
               0,
               1],
         "blocked": "peering is blocked due to down osds",
         "down_osds_we_would_probe": [
               1],
         "peering_blocked_by": [
               { "osd": 1,
                 "current_lost_at": 0,
                 "comment": "starting or marking this osd lost may let us proceed"}]},
       { "name": "Started",
         "enter_time": "2012-03-06 14:40:16.169513"}
   ]
}
```

本`recovery_state`节告诉我们，由于关闭了`ceph-osd`守护程序，导致对等连接被阻止`osd.1`。在这种情况下，我们可以重新开始，`ceph-osd` 一切都会恢复。

或者，如果发生灾难性故障`osd.1`（例如，磁盘故障），我们可以告诉集群它是这种情况，`lost`并尽力应对。

重要 

这很危险，因为群集无法保证数据的其他副本是一致的和最新的。

指示Ceph继续进行：

```text
ceph osd lost 1
```

恢复将继续。

### UNFOUND对象

在某些失败组合下，Ceph可能会抱怨以下 `unfound`物体：

```text
ceph health detail
HEALTH_WARN 1 pgs degraded; 78/3778 unfound (2.065%)
pg 2.4 is active+degraded, 78 unfound
```

这意味着存储集群知道存在某些对象（或现有对象的较新副本），但尚未找到它们的副本。对于数据位于ceph-osds 1和2上的PG可能如何实现的一个示例：

* 1下降
* 2独自处理一些写操作
* 1出现
* 1和2重复，并且1上缺少的对象排队等待恢复。
* 在复制新对象之前，2掉线了。

现在1知道这些对象存在，但是没有`ceph-osd`人拥有副本。在这种情况下，这些对象的IO将被阻止，群集将希望出现故障的节点很快回来；假定这比将IO错误返回给用户更好。

首先，您可以使用以下方法确定找不到哪些对象：

```text
ceph pg 2.4 list_unfound [starting offset, in json]
```

```text
{ "offset": { "oid": "",
     "key": "",
     "snapid": 0,
     "hash": 0,
     "max": 0},
 "num_missing": 0,
 "num_unfound": 0,
 "objects": [
    { "oid": "object 1",
      "key": "",
      "hash": 0,
      "max": 0 },
    ...
 ],
 "more": 0}
```

如果有太多对象无法在单个结果中列出，则该`more` 字段为true，您可以查询更多。（最终，命令行工具会将其隐藏起来，但还没有。）

其次，您可以确定已探查哪些OSD或可能包含数据：

```text
ceph pg 2.4 query
```

```text
"recovery_state": [
     { "name": "Started\/Primary\/Active",
       "enter_time": "2012-03-06 15:15:46.713212",
       "might_have_unfound": [
             { "osd": 1,
               "status": "osd is down"}]},
```

例如，在这种情况下，集群知道`osd.1`可能有数据，但是它是`down`。可能的状态的完整范围包括：

* 已经探查
* 查询
* OSD关闭
* 尚未查询（尚未）

有时，集群仅花一些时间来查询可能的位置。

可能存在对象未列出的其他位置。例如，如果停止了ceph-osd并将其从集群中取出，集群将完全恢复，并且由于将来发生的某些故障集最终会导致找不到对象，因此它不会将使用已久的ceph-osd视为一个可能要考虑的位置。（但是，这种情况不太可能。）

如果已查询所有可能的位置并且仍然丢失了对象，则可能必须放弃丢失的对象。同样，在异常的异常组合允许集群了解写入本身已恢复之前执行的写入的情况下，这也是可能的。要将“未找到”的对象标记为“丢失”：

```text
ceph pg 2.5 mark_unfound_lost revert|delete
```

最后一个参数指定集群如何处理丢失的对象。

“删除”选项将完全忘记它们。

“还原”选项（不适用于擦除编码池）将回滚到该对象的先前版本，或者（如果是新对象）将其完全忘记。请谨慎使用，因为它可能会使期望该对象存在的应用程序感到困惑。

### 无家可归者安置组

具有给定放置组副本的所有OSD都有可能失败。如果是这种情况，则该对象存储的子集不可用，并且该监视器将不会收到这些放置组的状态更新。为了检测到这种情况，监视器将其主OSD失败的所有放置组标记为`stale`。例如：

```text
ceph health
HEALTH_WARN 24 pgs stale; 3/300 in osds are down
```

您可以通过以下方式确定哪些展示位置组是`stale`，以及最后存储它们的OSD 是什么：

```text
ceph health detail
HEALTH_WARN 24 pgs stale; 3/300 in osds are down
...
pg 2.5 is stuck stale+active+remapped, last acting [2,0]
...
osd.10 is down since epoch 23, last address 192.168.106.220:6800/11080
osd.11 is down since epoch 13, last address 192.168.106.220:6803/11539
osd.12 is down since epoch 24, last address 192.168.106.220:6806/11861
```

例如，如果我们要使展示位置组2.5重新在线，则可以告诉我们它最后一次由`osd.0`和管理`osd.2`。重新启动这些`ceph-osd` 守护程序将使集群可以恢复该放置组（可能还有许多其他）。

### 只有少数的OSD接收数据

如果集群中有许多节点，而只有少数几个节点接收数据， [请检查](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups#get-the-number-of-placement-groups)池中的放置组数。由于展示位置组已映射到OSD，因此少数展示位置组将不会在整个群集中分布。尝试创建一个放置组计数为OSD数量倍数的池。有关详情，请参见[展示位置组](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups)。池的默认放置组计数没有用，但是您可以[在此处](https://docs.ceph.com/docs/nautilus/rados/configuration/pool-pg-config-ref)更改它。

### 不能将数据写入

如果群集已启动，但某些OSD已关闭并且无法写入数据，请检查以确保为该放置组运行的OSD数量最少。如果您没有运行最小数量的OSD，Ceph将不允许您写入数据，因为不能保证Ceph可以复制您的数据。有关详细信息[，](https://docs.ceph.com/docs/nautilus/rados/configuration/pool-pg-config-ref)请参见 “ [池，PG和CRUSH配置参考](https://docs.ceph.com/docs/nautilus/rados/configuration/pool-pg-config-ref) ”。`osd pool default min size`

### PG不一致

如果收到状态，则可能是由于清理过程中的错误而发生的。与往常一样，我们可以通过以下方式识别不一致的展示位置组：`active + clean + inconsistent`

```text
$ ceph health detail
HEALTH_ERR 1 pgs inconsistent; 2 scrub errors
pg 0.6 is active+clean+inconsistent, acting [0,1,2]
2 scrub errors
```

或者，如果您更喜欢以编程方式检查输出：

```text
$ rados list-inconsistent-pg rbd
["0.6"]
```

只有一个一致的状态，但在最坏的情况下，我们可能会在多个对象中发现的多个视角中存在不同的不一致之处。如果`foo`PG中命名的对象`0.6`被截断，我们将有：

```text
$ rados list-inconsistent-obj 0.6 --format=json-pretty
```

```text
{
    "epoch": 14,
    "inconsistents": [
        {
            "object": {
                "name": "foo",
                "nspace": "",
                "locator": "",
                "snap": "head",
                "version": 1
            },
            "errors": [
                "data_digest_mismatch",
                "size_mismatch"
            ],
            "union_shard_errors": [
                "data_digest_mismatch_info",
                "size_mismatch_info"
            ],
            "selected_object_info": "0:602f83fe:::foo:head(16'1 client.4110.0:1 dirty|data_digest|omap_digest s 968 uv 1 dd e978e67f od ffffffff alloc_hint [0 0 0])",
            "shards": [
                {
                    "osd": 0,
                    "errors": [],
                    "size": 968,
                    "omap_digest": "0xffffffff",
                    "data_digest": "0xe978e67f"
                },
                {
                    "osd": 1,
                    "errors": [],
                    "size": 968,
                    "omap_digest": "0xffffffff",
                    "data_digest": "0xe978e67f"
                },
                {
                    "osd": 2,
                    "errors": [
                        "data_digest_mismatch_info",
                        "size_mismatch_info"
                    ],
                    "size": 0,
                    "omap_digest": "0xffffffff",
                    "data_digest": "0xffffffff"
                }
            ]
        }
    ]
}
```

在这种情况下，我们可以从输出中学习：

* 唯一不一致的对象被命名为`foo`，并且它的头部存在不一致。
* 不一致分为两类：
  * `errors`：这些错误表示分片之间的不一致，而没有确定哪个分片不好。检查`errors`分 片数组中的（如果有），以查明问题所在。
    * `data_digest_mismatch`：从OSD.2读取的副本的摘要与OSD.0和OSD.1的摘要不同
    * `size_mismatch`：从OSD.2读取的副本的大小为0，而OSD.0和OSD.1报告的大小为968。
  * `union_shard_errors`：数组中所有特定分片的`errors`并集 `shards`。在`errors`对于给定碎片有问题设置。它们包括类似的错误`read_error`。在`errors`结束的 `oi`指示与比较`selected_object_info`。查看 `shards`阵列以确定哪个分片具有哪个错误。
    * `data_digest_mismatch_info`：存储在对象信息中的摘要不是 `0xffffffff`，这是从OSD.2读取的分片计算得出的
    * `size_mismatch_info`：存储在对象信息中的大小与从OSD.2读取的大小不同。后者为0。

您可以通过执行以下操作来修复不一致的展示位置组：

```text
ceph pg repair {placement-group-ID}
```

从而用权威的副本覆盖不良副本。在大多数情况下，Ceph可以使用一些预定义的标准从所有可用副本中选择权威副本。但这并不总是有效。例如，存储的数据摘要可能会丢失，并且在选择权威副本时将忽略所计算的摘要。因此，请谨慎使用以上命令。

如果shard `read_error`的`errors`属性中列出了，则可能由于磁盘错误而导致不一致。您可能要检查该OSD使用的磁盘。

如果由于时钟偏斜而定期收到状态，则可以考虑将Monitor主机上的[NTP](https://en.wikipedia.org/wiki/Network_Time_Protocol)守护程序配置为对等主机。有关更多详细信息，请参见[网络时间协议](http://www.ntp.org/)和Ceph [时钟设置](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-config-ref/#clock)。`active + clean + inconsistent`

### 擦除编码的PG无效+干净

当CRUSH找不到足够的OSD映射到PG时，它将显示为 `2147483647`ITEM\_NONE或。例如：`no OSD found`

```text
[2,1,6,0,5,8,2147483647,7,4]
```

#### 没有足够的OSD 

如果Ceph集群只有8个OSD，并且擦除编码池需要9个，那么它将显示。您可以创建另一个需要较少OSD的擦除编码池：

```text
ceph osd erasure-code-profile set myprofile k=5 m=3
ceph osd pool create erasurepool 16 16 erasure myprofile
```

或添加新的OSD，PG将自动使用它们。

#### 不能满足CRUSH约束

如果群集具有足够的OSD，则CRUSH规则可能会施加无法满足的约束。如果两个主机上有10个OSD，并且CRUSH规则要求在同一PG中不使用来自同一主机的两个OSD，则映射可能会失败，因为只会找到两个OSD。您可以通过显示（“转储”）规则来检查约束：

```text
$ ceph osd crush rule ls
[
    "replicated_rule",
    "erasurepool"]
$ ceph osd crush rule dump erasurepool
{ "rule_id": 1,
  "rule_name": "erasurepool",
  "ruleset": 1,
  "type": 3,
  "min_size": 3,
  "max_size": 20,
  "steps": [
        { "op": "take",
          "item": -1,
          "item_name": "default"},
        { "op": "chooseleaf_indep",
          "num": 0,
          "type": "host"},
        { "op": "emit"}]}
```

您可以通过创建一个新的池来解决问题，其中允许PG将OSD驻留在同一主机上，并具有以下功能：

```text
ceph osd erasure-code-profile set myprofile crush-failure-domain=osd
ceph osd pool create erasurepool 16 16 erasure myprofile
```

#### CRUSH放弃太早了

如果Ceph集群只有足够的OSD来映射PG（例如，一个集群总共有9个OSD和一个擦除编码池，每个PG需要9个OSD），则CRUSH可能会在找到映射之前放弃。可以通过以下方式解决：

* 降低擦除编码池的要求，以便每个PG使用更少的OSD（这需要创建另一个池，因为不能动态修改擦除码配置文件）。
* 向集群添加更多OSD（不需要修改擦除编码池，它将自动变得干净）
* 使用手工CRUSH规则尝试更多次才能找到良好的映射。可以通过设置`set_choose_tries`为大于默认值的值来完成此操作。

`crushtool`从群集中提取粉碎映射后，首先应该验证问题所在，这样您的实验就不会修改Ceph群集，而只能在本地文件上运行：

```text
$ ceph osd crush rule dump erasurepool
{ "rule_name": "erasurepool",
  "ruleset": 1,
  "type": 3,
  "min_size": 3,
  "max_size": 20,
  "steps": [
        { "op": "take",
          "item": -1,
          "item_name": "default"},
        { "op": "chooseleaf_indep",
          "num": 0,
          "type": "host"},
        { "op": "emit"}]}
$ ceph osd getcrushmap > crush.map
got crush map from osdmap epoch 13
$ crushtool -i crush.map --test --show-bad-mappings \
   --rule 1 \
   --num-rep 9 \
   --min-x 1 --max-x $((1024 * 1024))
bad mapping rule 8 x 43 num_rep 9 result [3,2,7,1,2147483647,8,5,6,0]
bad mapping rule 8 x 79 num_rep 9 result [6,0,2,1,4,7,2147483647,5,8]
bad mapping rule 8 x 173 num_rep 9 result [0,4,6,8,2,1,3,7,2147483647]
```

`--num-rep`删除代码CRUSH规则所需的OSD数量在哪里，`--rule`是所`ruleset`显示字段的值。该测试将尝试映射一百万个值（即，由定义的范围），并且必须显示至少一个错误的映射。如果未输出任何内容，则表示所有映射均已成功，您可以就此停止：问题出在其他地方。`ceph osd crush rule dump[--min-x,--max-x]`

可以通过反编译美感贴图来编辑CRUSH规则：

```text
$ crushtool --decompile crush.map > crush.txt
```

并将以下行添加到规则中：

```text
step set_choose_tries 100
```

文件的相关部分`crush.txt`应类似于：

```text
rule erasurepool {
        ruleset 1
        type erasure
        min_size 3
        max_size 20
        step set_chooseleaf_tries 5
        step set_choose_tries 100
        step take default
        step chooseleaf indep 0 type host
        step emit
}
```

然后可以对其进行编译和再次测试：

```text
$ crushtool --compile crush.txt -o better-crush.map
```

当所有映射都成功时，可以使用以下`--show-choose-tries`选项显示找到所有映射所需的尝试次数的直方图 `crushtool`：

```text
$ crushtool -i better-crush.map --test --show-bad-mappings \
   --show-choose-tries \
   --rule 1 \
   --num-rep 9 \
   --min-x 1 --max-x $((1024 * 1024))
...
11:        42
12:        44
13:        54
14:        45
15:        35
16:        34
17:        30
18:        25
19:        19
20:        22
21:        20
22:        17
23:        13
24:        16
25:        13
26:        11
27:        11
28:        13
29:        11
30:        10
31:         6
32:         5
33:        10
34:         3
35:         7
36:         5
37:         2
38:         5
39:         5
40:         2
41:         5
42:         4
43:         1
44:         2
45:         2
46:         3
47:         1
48:         0
...
102:         0
103:         1
104:         0
...
```

尝试了11次尝试映射了42个PG，12次尝试映射了44个PG等。尝试次数最多的是`set_choose_tries`防止不良映射的最小值（即，在上面的输出中为103次，因为对于任何PG来说，尝试次数都不超过103次）进行映射）。

