# RGW动态存储桶索引重新分片

## RGW动态存储桶索引重新分片

L的新版本。

大的存储桶索引可能会导致性能问题。为了解决这个问题，我们引入了桶索引分片。在发光之前，需要脱机更改存储区分片（重新分片）的数量。从Luminous开始，我们支持在线存储分区重新分片。

每个存储区索引碎片都可以有效地处理其条目，直到达到一定的阈值条目数量为止。如果超过此阈值，则系统可能会遇到性能问题。动态重新分片功能会检测到这种情况，并自动增加存储桶索引使用的分片数量，从而减少每个存储桶索引分片中条目的数量。此过程对用户是透明的。

检测过程运行：

1. 当新对象添加到存储桶时，
2. 在后台进程中定期扫描所有存储桶。

为了处理系统中未更新的现有存储桶，需要后台处理。需要重新分片的存储桶被添加到重新分片队列中，并计划在以后重新分片。重新分片线​​程在后台运行并一次执行计划的重新分片任务。

### 多站点

在多站点环境中不支持动态重新分片。

### 配置

启用/禁用动态存储区索引重新分片：

* `rgw_dynamic_resharding`：true / false，默认值：true

控制重新分片过程的配置选项：

* `rgw_reshard_num_logs`：重新分片队列的分片数，默认值：16
* `rgw_reshard_bucket_lock_duration`：在重新分片期间锁定存储桶obj的持续时间（以秒为单位），默认值：120秒
* `rgw_max_objs_per_shard`：触发​​重新分片之前，每个存储区索引分片的最大对象数，默认值：100000个对象
* `rgw_reshard_thread_interval`：两轮重新分片队列处理之间的最长时间（以秒为单位），默认值：600秒

### 管理员命令

#### 将存储桶添加到重新分片队列中

```text
# radosgw-admin reshard add --bucket <bucket_name> --num-shards <new number of shards>
```

#### 列出重新分片队列

```text
# radosgw-admin reshard list
```

#### 在重新分片队列上处理任务

```text
# radosgw-admin reshard process
```

#### 铲斗重新分片状态

```text
# radosgw-admin reshard status --bucket <bucket_name>
```

输出是每个分片包含3个对象（reshard\_status，new\_bucket\_instance\_id，num\_shards）的json数组。

例如，不同动态重新分片阶段的输出如下所示：

`1. Before resharding occurred:`

```text
[
  {
      "reshard_status": "not-resharding",
      "new_bucket_instance_id": "",
      "num_shards": -1
  }
]
```

`2. During resharding:`

```text
[
  {
      "reshard_status": "in-progress",
      "new_bucket_instance_id": "1179f470-2ebf-4630-8ec3-c9922da887fd.8652.1",
      "num_shards": 2
  },
  {
      "reshard_status": "in-progress",
      "new_bucket_instance_id": "1179f470-2ebf-4630-8ec3-c9922da887fd.8652.1",
      "num_shards": 2
  }
]
```

`3, After resharding completed:`

```text
[
  {
      "reshard_status": "not-resharding",
      "new_bucket_instance_id": "",
      "num_shards": -1
  },
  {
      "reshard_status": "not-resharding",
      "new_bucket_instance_id": "",
      "num_shards": -1
  }
]
```

#### 取消挂起的存储桶重新分片

注意：正在进行的存储区重新分片操作无法取消。

```text
# radosgw-admin reshard cancel --bucket <bucket_name>
```

#### 手动即时存储区重新分片

```text
# radosgw-admin bucket reshard --bucket <bucket_name> --num-shards <new number of shards>
```

### 故障排除

Luminous 12.2.11和Mimic 13.2.5之前的群集保留在过时的存储桶实例条目中，这些条目不会自动清除。此问题还影响了LifeCycle策略，该策略不再应用于重新分片的存储桶。使用几个radosgw-admin命令可以解决这两个问题。

#### 陈旧的实例管理

列出集群中准备清理的陈旧实例。

```text
# radosgw-admin reshard stale-instances list
```

清理集群中的陈旧实例。注意：这些实例的清除只能在单个站点群集上完成。

```text
# radosgw-admin reshard stale-instances rm
```

#### 生命周期修复

对于具有重新分片实例的集群，很可能旧的生命周期进程会在重新分片期间更改存储桶实例时标记并删除生命周期处理。虽然这是针对较新的群集（来自Mimic 13.2.6和Luminous 12.2.12）修复的，但具有生命周期策略并经过重新分片的较旧存储桶将必须手动修复。

这样做的命令是：

```text
# radosgw-admin lc reshard fix --bucket {bucketname}
```

作为便利包装，如果`--bucket`删除了该参数，则此命令将尝试为集群中的所有存储桶修复生命周期策略。

#### 对象过期修复程序

较早的群集上受Swift对象到期的对象可能已从日志池中删除，并且在重新存储分区后再也不会删除。如果它们的到期时间是在集群升级之前，就会发生这种情况，但是如果它们的到期时间是在升级之后，则可以正确处理对象。要管理这些过期的对象，radosgw-admin提供了两个子命令。

清单：

```text
# radosgw-admin objects expire-stale list --bucket {bucketname}
```

以JSON格式显示对象名称和到期时间的列表。

删除中：

```text
# radosgw-admin objects expire-stale rm --bucket {bucketname}
```

启动此类对象的删除，以JSON格式显示对象名称，到期时间和删除状态的列表。

