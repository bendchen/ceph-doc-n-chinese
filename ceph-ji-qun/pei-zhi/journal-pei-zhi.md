# journal配置

## 日志配置参考

Ceph OSD使用日志的原因有两个：速度和一致性。

* **速度：**日志使Ceph OSD守护进程能够快速提交少量写入。Ceph顺序地向日志写入较小的随机I / O，这通过允许备用文件系统有更多时间合并写入，从而倾向于加快突发性工作负载。但是，Ceph OSD Daemon的日志会导致尖峰的性能，因为短时间的高速写操作会导致文件系统追上日志，而随后的时间段没有任何写进度。
* **一致性：** Ceph OSD守护进程需要一个文件系统接口，以保证原子复合操作。Ceph OSD守护程序将操作的描述写入日志并将该操作应用于文件系统。这样可以对对象进行原子更新（例如，放置组元数据）。每隔几秒钟之间和 -the Ceph的OSD守护程序停止写入和同步与文件系统日志，允许Ceph的OSD后台程序从日志修剪操作和重复使用的空间。失败时，Ceph OSD守护程序将从上一次同步操作开始重播日志。`filestore max sync intervalfilestore min sync interval`

Ceph OSD守护程序支持以下日志设置：

`journal dio`描述

启用直接I / O到日志。需要设置为。`journal block aligntrue`类型

布尔型需要

是的，使用时`aio`。默认

`true`

`journal aio`

版本0.61中的更改​​：墨鱼描述

允许`libaio`用于异步写入日志。需要设置为。`journal diotrue`类型

布尔型需要

没有。默认

版本0.61和更高版本`true`。版本0.60和更早版本`false`。

`journal block align`描述

块对齐写入操作。`dio`和必需`aio`。类型

布尔型需要

是，当使用`dio`和时`aio`。默认

`true`

`journal max write bytes`描述

日志一次可以写入的最大字节数。类型

整数需要

没有默认

`10 << 20`

`journal max write entries`描述

日记一次可以写入的最大条目数。类型

整数需要

没有默认

`100`

`journal queue max ops`描述

任一时刻队列中允许的最大操作数。类型

整数需要

没有默认

`500`

`journal queue max bytes`描述

任一时刻队列中允许的最大字节数。类型

整数需要

没有默认

`10 << 20`

`journal align min size`描述

对齐大于指定最小值的数据有效负载。类型

整数需要

没有默认

`64 << 10`

`journal zero on create`描述

导致文件存储`0`在期间用覆盖整个日志 `mkfs`。类型

布尔型需要

没有默认

`false`

