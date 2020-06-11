# 数据平衡

## 平衡器

该_平衡器_可以优化跨越的OSD的PGs的放置，以便自动地或者在监督方式实现的均衡分布。

### 状态

平衡器的当前状态可以随时通过以下方式检查：

```text
ceph balancer status
```

### 自动平衡

可以使用默认设置通过以下方式启用自动平衡：

```text
ceph balancer on
```

可以使用以下方法再次关闭平衡器：

```text
ceph balancer off
```

这将使用`crush-compat`与较旧的客户端向后兼容的模式，并且随着时间的流逝对数据分布进行细微更改，以确保OSD被平等地利用。

### 节流

如果集群降级（例如，由于OSD发生故障且系统尚未自愈），则不会对PG分配进行任何调整。

当群集运行状况良好时，平衡器将限制其更改，以使放错位置（即，需要移动）的PG的百分比低于（默认）5％的阈值。该 `target_max_misplaced_ratio`阈值可以进行调整：

```text
ceph config set mgr target_max_misplaced_ratio .07   # 7%
```

### 模式

当前支持两种平衡器模式：

1. **暗恋**。CRUSH兼容模式使用compat weight-set功能（在Luminous中引入）来管理CRUSH层次结构中设备的替代权重集。正常权重应保持设置为设备的大小，以反映我们要存储在设备上的目标数据量。然后，平衡器优化权重设置值，以较小的增量向上或向下调整权重设置值，以实现与目标分布尽可能接近的分布。（由于PG放置是一个伪随机过程，因此放置中存在自然的变化；通过优化权重，我们抵消了该自然变化。）

   值得注意的是，此模式与较旧的客户端_完全向后兼容_：当OSDMap和CRUSH映射与较旧的客户端共享时，我们将优化的权重表示为“实际”权重。

   此模式的主要限制是，如果层次结构的子树共享任何OSD，则平衡器无法处理具有不同放置规则的多个CRUSH层次结构。（通常不是这种情况，并且通常也不建议这样做，因为很难管理共享OSD上的空间利用率。）

2. **向上映射**。从Luminous开始，OSDMap可以存储单个OSD的显式映射，作为常规CRUSH放置计算的例外。这些上映射条目提供了对PG映射的细粒度控制。此CRUSH模式将优化各个PG的位置，以实现平衡分配。在大多数情况下，这种分布是“完美的”，每个OSD上的PG数量相等（+/- 1 PG，因为它们可能不会平均分配）。

   请注意，使用upmap要求所有客户端均为Luminous或更高版本。

默认模式是`crush-compat`。可以通过以下方式调整模式：

```text
ceph balancer mode upmap
```

要么：

```text
ceph balancer mode crush-compat
```

### 监督优化

平衡器操作分为几个不同的阶段：

1. 建设_计划_
2. 评估数据分发的质量，无论是针对当前的PG分发，还是执行_计划_后可能产生的PG分发
3. 执行_计划_

要评估和分配当前分布，请执行以下操作：

```text
ceph balancer eval
```

您还可以使用以下方法评估单个池的分布：

```text
ceph balancer eval <pool-name>
```

评估的更多细节可以通过以下方式看到：

```text
ceph balancer eval-verbose ...
```

平衡器可以使用当前配置的模式通过以下方式生成计划：

```text
ceph balancer optimize <plan-name>
```

该名称由用户提供，可以是任何有用的标识字符串。可以通过以下方式查看计划的内容：

```text
ceph balancer show <plan-name>
```

所有计划都可以显示：

```text
ceph balancer ls
```

旧计划可以通过以下方式丢弃：

```text
ceph balancer rm <plan-name>
```

当前记录的计划显示为status命令的一部分：

```text
ceph balancer status
```

执行计划后可能产生的分配质量可通过以下公式计算：

```text
ceph balancer eval <plan-name>
```

假设该计划有望改善分布（即，其得分低于当前集群状态），则用户可以执行以下计划：

```text
ceph balancer execute <plan-name>
```

