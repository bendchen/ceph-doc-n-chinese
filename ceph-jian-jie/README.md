# CEPH简介

## CEPH简介

无论您是要向Cloud Platforms提供Ceph对象存储和/或 Ceph块设备服务，部署Ceph文件系统还是出于其他目的使用Ceph，所有Ceph Storage Cluster部署都要 从设置每个 Ceph节点，您的网络和Ceph存储开始簇。一个Ceph存储群集至少需要一个Ceph监视器，Ceph管理器和Ceph OSD（对象存储守护程序）。运行Ceph文件系统客户端时，也需要Ceph Metadata Server。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-409784e9076840f895f8cbd328a523961cda0d87.png)

* **监视器**：Ceph Monitor（`ceph-mon`）维护集群状态的映射，包括监视器映射，管理器映射，OSD映射和CRUSH映射。这些映射是Ceph守护程序相互协调所需的关键群集状态。监视器还负责管理守护程序和客户端之间的身份验证。通常至少需要三个监视器才能实现冗余和高可用性。
* **管理器**：Ceph Manager守护进程（`ceph-mgr`）负责跟踪运行时指标和Ceph集群的当前状态，包括存储利用率，当前性能指标和系统负载。Ceph Manager守护进程还托管基于python的模块，以管理和公开Ceph集群信息，包括基于Web的Ceph仪表板和 REST API。高可用性通常至少需要两个管理器。
* **Ceph OSD**：Ceph OSD（对象存储守护程序， `ceph-osd`）存储数据，处理数据复制，恢复，重新平衡，并通过检查其他Ceph OSD守护程序的心跳来向Ceph监视器和管理器提供一些监视信息。通常至少需要3个Ceph OSD才能实现冗余和高可用性。
* **MDS**：Ceph元数据服务器（MDS，`ceph-mds`）代表Ceph文件系统存储元数据（即，Ceph块设备和Ceph对象存储不使用MDS）。Ceph的元数据服务器允许POSIX文件系统的用户来执行基本的命令（如 `ls`，`find`没有放置在一个Ceph存储集群的巨大负担，等等）。

Ceph将数据作为对象存储在逻辑存储池中。使用 CRUSH算法，Ceph计算哪个放置组应包含该对象，并进一步计算哪个Ceph OSD守护程序应存储该放置组。CRUSH算法使Ceph存储集群能够动态扩展，重新平衡和恢复。

