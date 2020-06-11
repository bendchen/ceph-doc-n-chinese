# 使用PG-UPMAP

## 使用PG-UPMAP 

从Luminous v12.2.z开始，OSDMap中有一个新的_pg-upmap_异常表，该表允许集群将特定PG显式映射到特定OSD。在大多数情况下，这使群集可以将数据分布微调到跨OSD的完美分布的PG。

此新机制的主要警告是它要求所有客户端都了解_OSDMap中_的新_pg-upmap_结构。

### 启用

要允许使用该功能，您必须通过以下方式告诉集群它仅需要支持发光（和更新）的客户端：

```text
ceph osd set-require-min-compat-client luminous
```

如果将任何发光的客户端或守护程序连接到监视器，则该命令将失败。您可以查看正在使用哪些客户端版本：

```text
ceph features
```

### 平衡器模块

用于ceph-mgr 的新平衡器模块将自动平衡每个OSD的PG数量。看到`Balancer`

### 离线优化

使用内置的离线优化器更新Upmap条目`osdmaptool`。

1. 获取您的osdmap的最新副本：

   ```text
   ceph osd getmap -o om
   ```

2. 运行优化器：

   ```text
   osdmaptool om --upmap out.txt [--upmap-pool <pool>]
            [--upmap-max <max-optimizations>] [--upmap-deviation <max-deviation>]
            [--upmap-active]
   ```

   强烈建议分别对每个池或一组类似使用的池进行优化。您可以`--upmap-pool`多次指定选项。“相似池”是指映射到相同设备并存储相同类型数据的池（例如，RBD图像池，是； RGW索引池和RGW数据池，否）。

   该`max-optimizations`值是运行中要标识的最大上映射条目数。与ceph-mgr balancer模块一样，默认值为10，但是如果要进行离线优化，则应使用更大的数字。如果找不到任何其他更改，它将尽早停止（即，当池分配完美时）。

   该`max-deviation`值默认为5。如果OSD PG计数与计算的目标数量相差小于或等于此数量，将被认为是理想的。

   该`--upmap-active`选项在向上映射模式下模拟活动平衡器的行为。它会一直循环直到OSD平衡为止，并报告几轮以及每轮花费多长时间。轮次使用的时间表明ceph-mgr在尝试计算下一个优化计划时将消耗CPU负载。

3. 应用更改：

   ```text
   source out.txt
   ```

   `out.txt`在上面的示例中，建议的更改被写入输出文件。这些是正常的ceph CLI命令，可以运行这些命令以将更改应用到集群。

可以根据需要将上述步骤重复多次，以实现每组池的PG完美分配。

你可以看到有关该工具是通过传递做一些（血淋淋的）细节有更 给。`--debug-osd 10--debug-crush 10osdmaptool`

