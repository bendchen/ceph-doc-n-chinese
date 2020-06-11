# 控制命令参考

## 控制命令

### 监控命令

监视命令是使用ceph实用程序发出的：

```text
ceph [-m monhost] {command}
```

该命令通常（尽管并非总是）采用以下形式：

```text
ceph {subsystem} {command}
```

### 系统命令

执行以下操作以显示集群的当前状态。

```text
ceph -s
ceph status
```

执行以下操作以显示集群状态和主要事件的运行摘要。

```text
ceph -w
```

执行以下操作以显示监视器法定人数，包括哪些监视器正在参与以及哪个监视器是领导者。

```text
ceph quorum_status
```

执行以下操作以查询单个监视器的状态，包括它是否在仲裁中。

```text
ceph [-m monhost] mon_status
```

### 认证子系统

要为OSD添加密钥环，请执行以下操作：

```text
ceph auth add {osd} {--in-file|-i} {path-to-osd-keyring}
```

要列出集群的键及其功能，请执行以下操作：

```text
ceph auth ls
```

### 放置组子系统

要显示所有展示位置组的统计信息，请执行以下操作：

```text
ceph pg dump [--format {format}]
```

有效格式为`plain`（默认）和`json`。

要显示处于指定状态的所有展示位置组的统计信息，请执行以下操作：

```text
ceph pg dump_stuck inactive|unclean|stale|undersized|degraded [--format {format}] [-t|--threshold {seconds}]
```

`--format`可能是`plain`（默认）或`json`

`--threshold` 定义“卡住”的秒数（默认值：300）

**不活动的**放置组无法处理读取或写入，因为它们正在等待包含最新数据的OSD返回。

**不干净的**放置组包含未复制所需次数的对象。他们应该正在恢复。

**陈旧的**放置组处于未知状态-承载它们的OSD暂时未向监视集群报告（由配置 `mon_osd_report_timeout`）。

删除“丢失”的对象或将它们恢复为先前的状态（以前的版本），或者如果刚刚创建则将其删除。

```text
ceph pg {pgid} mark_unfound_lost revert|delete
```

### OSD子系统

查询OSD子系统状态。

```text
ceph osd stat
```

将最新OSD映射的副本写入文件。参见 [osdmaptool](https://docs.ceph.com/docs/nautilus/man/8/osdmaptool/#osdmaptool)。

```text
ceph osd getmap -o file
```

将最新OSD映射的Crush映射的副本写入文件。

```text
ceph osd getcrushmap -o file
```

前述功能等同于

```text
ceph osd getmap -o /tmp/osdmap
osdmaptool /tmp/osdmap --export-crush file
```

转储OSD映射。有效的格式`-f`是`plain`和`json`。如果未提供任何 `--format`选项，则OSD映射将以纯文本格式转储。

```text
ceph osd dump [--format {format}]
```

将OSD映射转储为一棵树，每个OSD一行包含权重和状态。

```text
ceph osd tree [--format {format}]
```

找出特定对象在系统中的存储位置或存储位置：

```text
ceph osd map <pool-name> <object-name>
```

在指定位置添加或移动具有给定ID /名称/重量的新物品（OSD）。

```text
ceph osd crush set {id} {weight} [{loc1} [{loc2} ...]]
```

从CRUSH映射中删除现有项目（OSD）。

```text
ceph osd crush remove {name}
```

从CRUSH映射中删除现有的存储桶。

```text
ceph osd crush remove {bucket-name}
```

将现有存储桶从层次结构中的一个位置移动到另一位置。

```text
ceph osd crush move {id} {loc1} [{loc2} ...]
```

将给定的项目的重量设置`{name}`为`{weight}`。

```text
ceph osd crush reweight {name} {weight}
```

将OSD标记为丢失。这可能会导致永久数据丢失。请谨慎使用。

```text
ceph osd lost {id} [--yes-i-really-mean-it]
```

创建一个新的OSD。如果未给出UUID，则它将在OSD启动时自动设置。

```text
ceph osd create [{uuid}]
```

删除给定的OSD。

```text
ceph osd rm [{id}...]
```

在OSD映射中查询当前的max\_osd参数。

```text
ceph osd getmaxosd
```

导入给定的美眉贴图。

```text
ceph osd setcrushmap -i file
```

`max_osd`在OSD映射中设置参数。在扩展存储集群时，这是必需的。

```text
ceph osd setmaxosd
```

`{osd-num}`记下OSD 。

```text
ceph osd down {osd-num}
```

`{osd-num}`从分发中标记OSD （即未分配数据）。

```text
ceph osd out {osd-num}
```

标记`{osd-num}`分配（即分配的数据）。

```text
ceph osd in {osd-num}
```

在OSD映射中设置或清除暂停标志。如果设置，则不会将IO请求发送到任何OSD。通过取消暂停清除标志将导致重新发送待处理的请求。

```text
ceph osd pause
ceph osd unpause
```

将的权重设置`{osd-num}`为`{weight}`。具有相同权重的两个OSD将接收大约相同数量的I / O请求，并存储大约相同数量的数据。 在OSD上设置优先权重。该值在0到1的范围内，并强制CRUSH替换原本存放在该驱动器上的数据的（1-权重）。它不会更改分配给在粉碎图中OSD上方的铲斗的权重，并且是在正常的CRUSH分配不能正确计算的情况下的一种纠正措施。例如，如果您的OSD之一为90％，其他OSD为50％，则可以减小此权重以尝试补偿。`ceph osd reweight`

```text
ceph osd reweight {osd-num} {weight}
```

通过减少过度使用的OSD的重量来重新加权所有OSD。默认情况下，它将对平均利用率为120％的OSD向下调整权重，但是如果包括阈值，它将使用该百分比代替。

```text
ceph osd reweight-by-utilization [threshold]
```

描述按利用率调整权重。

```text
ceph osd test-reweight-by-utilization
```

向/从黑名单添加/删除地址。添加地址时，您可以指定将其列入黑名单的时间（以秒为单位）；否则，它将默认为1小时。禁止将列入黑名单的地址连接到任何OSD。黑名单通常用于防止滞后的元数据服务器对OSD上的数据进行不良更改。

这些命令通常仅对故障测试有用，因为黑名单通常是自动维护的，不需要手动干预。

```text
ceph osd blacklist add ADDRESS[:source_port] [TIME]
ceph osd blacklist rm ADDRESS[:source_port]
```

创建/删除池的快照。

```text
ceph osd pool mksnap {pool-name} {snap-name}
ceph osd pool rmsnap {pool-name} {snap-name}
```

创建/删除/重命名存储池。

```text
ceph osd pool create {pool-name} pg_num [pgp_num]
ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]
ceph osd pool rename {old-name} {new-name}
```

更改池设置。

```text
ceph osd pool set {pool-name} {field} {value}
```

有效字段为：

> * `size`：设置池中数据的副本数。
> * `pg_num`：展示位置组号。
> * `pgp_num`：计算pg位置时的有效数字。
> * `crush_rule`：映射放置的规则号。

获取池设置的值。

```text
ceph osd pool get {pool-name} {field}
```

有效字段为：

> * `pg_num`：展示位置组号。
> * `pgp_num`：计算展示位置时的有效展示位置组数。

将清理命令发送到OSD `{osd-num}`。要将命令发送到所有OSD，请使用`*`。

```text
ceph osd scrub {osd-num}
```

将修复命令发送到OSD.N。要将命令发送到所有OSD，请使用`*`。

```text
ceph osd repair N
```

针对OSD.N运行简单的吞吐量基准，写入每个的`TOTAL_DATA_BYTES` 写入请求`BYTES_PER_WRITE`。默认情况下，测试以4 MB的增量写入总共1 GB。基准测试是非破坏性的，不会覆盖现有的实时OSD数据，但可能会暂时影响同时访问OSD的客户端的性能。

```text
ceph tell osd.N bench [TOTAL_DATA_BYTES] [BYTES_PER_WRITE]
```

要在基准测试运行之间清除OSD的缓存，请使用'cache drop'命令

```text
ceph tell osd.N cache drop
```

要获取OSD的缓存统计信息，请使用“缓存状态”命令

```text
ceph tell osd.N cache status
```

### MDS子系统

更改正在运行的mds上的配置参数。

```text
ceph tell mds.{mds-id} config set {setting} {value}
```

例：

```text
ceph tell mds.0 config set debug_ms 1
```

启用调试消息。

```text
ceph mds stat
```

显示所有元数据服务器的状态。

```text
ceph mds fail 0
```

将活动的MDS标记为失败，如果存在则触发故障转移到备用数据库。

去做 

`ceph mds` 子命令缺少文档：set，dump，getmap，stop，setmap

### MON子系统

显示监控器统计信息：

```text
ceph mon stat

e2: 3 mons at {a=127.0.0.1:40000/0,b=127.0.0.1:40001/0,c=127.0.0.1:40002/0}, election epoch 6, quorum 0,1,2 a,b,c
```

在`quorum`末列出清单监控是当前法定人数的一部分的节点。

也可以直接使用：

```text
ceph quorum_status -f json-pretty
```

```text
{
    "election_epoch": 6,
    "quorum": [
        0,
        1,
        2
    ],
    "quorum_names": [
        "a",
        "b",
        "c"
    ],
    "quorum_leader_name": "a",
    "monmap": {
        "epoch": 2,
        "fsid": "ba807e74-b64f-4b72-b43f-597dfe60ddbc",
        "modified": "2016-12-26 14:42:09.288066",
        "created": "2016-12-26 14:42:03.573585",
        "features": {
            "persistent": [
                "kraken"
            ],
            "optional": []
        },
        "mons": [
            {
                "rank": 0,
                "name": "a",
                "addr": "127.0.0.1:40000\/0",
                "public_addr": "127.0.0.1:40000\/0"
            },
            {
                "rank": 1,
                "name": "b",
                "addr": "127.0.0.1:40001\/0",
                "public_addr": "127.0.0.1:40001\/0"
            },
            {
                "rank": 2,
                "name": "c",
                "addr": "127.0.0.1:40002\/0",
                "public_addr": "127.0.0.1:40002\/0"
            }
        ]
    }
}
```

以上将阻止，直到达到法定人数。

对于仅连接到显示器的状态（用于 选择）：`-m HOST:PORT`

```text
ceph mon_status -f json-pretty
```

```text
{
    "name": "b",
    "rank": 1,
    "state": "peon",
    "election_epoch": 6,
    "quorum": [
        0,
        1,
        2
    ],
    "features": {
        "required_con": "9025616074522624",
        "required_mon": [
            "kraken"
        ],
        "quorum_con": "1152921504336314367",
        "quorum_mon": [
            "kraken"
        ]
    },
    "outside_quorum": [],
    "extra_probe_peers": [],
    "sync_provider": [],
    "monmap": {
        "epoch": 2,
        "fsid": "ba807e74-b64f-4b72-b43f-597dfe60ddbc",
        "modified": "2016-12-26 14:42:09.288066",
        "created": "2016-12-26 14:42:03.573585",
        "features": {
            "persistent": [
                "kraken"
            ],
            "optional": []
        },
        "mons": [
            {
                "rank": 0,
                "name": "a",
                "addr": "127.0.0.1:40000\/0",
                "public_addr": "127.0.0.1:40000\/0"
            },
            {
                "rank": 1,
                "name": "b",
                "addr": "127.0.0.1:40001\/0",
                "public_addr": "127.0.0.1:40001\/0"
            },
            {
                "rank": 2,
                "name": "c",
                "addr": "127.0.0.1:40002\/0",
                "public_addr": "127.0.0.1:40002\/0"
            }
        ]
    }
}
```

监视器状态的转储：

```text
ceph mon dump

dumped monmap epoch 2
epoch 2
fsid ba807e74-b64f-4b72-b43f-597dfe60ddbc
last_changed 2016-12-26 14:42:09.288066
created 2016-12-26 14:42:03.573585
0: 127.0.0.1:40000/0 mon.a
1: 127.0.0.1:40001/0 mon.b
2: 127.0.0.1:40002/0 mon.c
```

