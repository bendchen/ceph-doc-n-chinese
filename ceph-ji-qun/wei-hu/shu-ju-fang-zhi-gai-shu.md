# 数据放置概述

## 数据放置概述

Ceph在整个RADOS集群中动态存储，复制和重新平衡数据对象。由于许多用户在无数OSD上出于不同目的将对象存储在不同的池中，因此Ceph操作需要一些数据放置计划。Ceph中的主要数据放置计划概念包括：

* **池：** Ceph将数据存储在池中，池是用于存储对象的逻辑组。池管理放置组的数量，副本的数量以及池的CRUSH规则。要将数据存储在池中，必须具有经过身份验证的用户，该用户对该池具有权限。Ceph可以快照池。有关其他详细信息，请参见[池](https://docs.ceph.com/docs/nautilus/rados/operations/pools)。
* **放置组：** Ceph将对象映射到放置组（PG）。放置组（PG）是逻辑对象池的碎片或片段，将对象作为一个组放置到OSD中。当Ceph将数据存储在OSD中时，放置组会减少每个对象元数据的数量。大量的放置组（例如，每个OSD为100个）可导致更好的平衡。有关其他详细信息，请参见 [展示位置组](https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups)。
* **CRUSH Maps：** CRUSH是Ceph进行扩展的重要组成部分，它没有性能瓶颈，没有可扩展性的限制，也没有单点故障。CRUSH映射为CRUSH算法提供了群集的物理拓扑，以确定应将对象及其副本的数据存储在何处，以及如何跨故障域进行存储以提高数据安全性。有关其他详细信息，请参见[CRUSH Maps](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map)。
* **平衡器：**平衡器是一项功能，它将自动优化设备之间PG的分配，以实现平衡的数据分配，从而最大化可存储在群集中的数据量，并在OSD之间平均分配工作量。

最初设置测试集群时，可以使用默认值。一旦开始计划大型Ceph集群，请参考池，放置组和CRUSH进行数据放置操作。

