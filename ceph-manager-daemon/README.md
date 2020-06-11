# CEPH MANAGER DAEMON

The [Ceph Manager](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-manager) daemon \(ceph-mgr\) runs alongside monitor daemons, to provide additional monitoring and interfaces to external monitoring and management systems.

Since the 12.x \(_luminous_\) Ceph release, the ceph-mgr daemon is required for normal operations. The ceph-mgr daemon is an optional component in the 11.x \(_kraken_\) Ceph release.

By default, the manager daemon requires no additional configuration, beyond ensuring it is running. If there is no mgr daemon running, you will see a health warning to that effect, and some of the other information in the output of ceph status will be missing or stale until a mgr is started.

Use your normal deployment tools, such as ceph-ansible or ceph-deploy, to set up ceph-mgr daemons on each of your mon nodes. It is not mandatory to place mgr daemons on the same nodes as mons, but it is almost always sensible.

## CEPH MANAGER守护程序[¶](https://docs.ceph.com/docs/nautilus/mgr/#ceph-manager-daemon)

该[Ceph的经理](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-manager)守护进程（CEPH-MGR）一起运行监控守护进程，向外部监测和管理系统提供额外的监测和接口。

从12.x（_发光的_）Ceph版本开始，正常操作需要使用ceph-mgr守护程序。_ceph_ -mgr守护程序是11.x（_kraken_）Ceph发行版中的可选组件。

默认情况下，管理器守护程序不需要其他配置即可确保运行。如果没有运行mgr守护程序，您将看到相应的运行状况警告，并且在启动mgr之前，ceph status输出中的某些其他信息将丢失或陈旧。

使用普通的部署工具（例如ceph-ansible或ceph-deploy）在每个mon节点上设置ceph-mgr守护程序。将mgr守护程序与mons放在同一节点上不是强制性的，但几乎总是明智的。

