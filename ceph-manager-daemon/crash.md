# crash

## crash模块

崩溃模块收集有关守护程序崩溃转储的信息，并将其存储在Ceph集群中，以供以后分析。

默认情况下，守护进程崩溃转储被转储到/ var / lib / ceph / crash中。可以使用“ crash dir”选项进行配置。崩溃目录由时间和日期以及随机生成的UUID命名，并包含元数据文件“ meta”和最新的日志文件，并且“ crash\_id”相同。此模块允许有关这些转储的元数据保留在监视器的存储中。

### 启用

该_碰撞_模块启用了：

```text
ceph mgr module enable crash
```

### 命令

```text
ceph crash post -i <metafile>
```

保存故障转储。元数据文件是存储在崩溃目录中的JSON Blob `meta`。通常，可以使用调用ceph命令，并将从stdin中读取。`-i -`

```text
ceph rm <crashid>
```

删除特定的故障转储。

```text
ceph crash ls
```

列出所有新的和存档的崩溃信息的时间戳/ uuid崩溃标识。

```text
ceph crash ls-new
```

列出所有新崩溃信息的时间戳/ uuid崩溃标识。

```text
ceph crash stat
```

显示按年龄分组的已保存崩溃信息摘要。

```text
ceph crash info <crashid>
```

显示已保存的崩溃的所有详细信息。

```text
ceph crash prune <keep>
```

删除保存时间早于“保留”天的崩溃。&lt;keep&gt;必须为整数。

```text
ceph crash archive <crashid>
```

存档崩溃报告，以便不再对其进行`RECENT_CRASH`运行状况检查，并且也不会出现在输出中（它仍将出现在输出中）。`crash ls-newcrash ls`

```text
ceph crash archive-all
```

存档所有新的崩溃报告。

### 选项

* `mgr/crash/warn_recent_interval`\[默认值：2周\]控制“最近”的构成，以`RECENT_CRASH`发出健康警告。
* `mgr/crash/retain_interval` \[默认值：1年\]控制崩溃报告在集群被自动清除之前保留的时间。

