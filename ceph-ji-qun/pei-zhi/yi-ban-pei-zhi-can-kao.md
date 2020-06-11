# 一般配置参考

## 一般配置参考

`fsid`描述

文件系统ID。每个群集一个。类型

UUID需要

没有。默认

不适用 通常由部署工具生成。

`admin socket`描述

套接字，用于在守护程序上执行管理命令，而与Ceph Monitors是否已建立仲裁无关。类型

串需要

没有默认

`/var/run/ceph/$cluster-$name.asok`

`pid file`描述

mon，osd或mds将在其中写入其PID的文件。例如，`/var/run/$cluster/$type.$id.pid` 将为`mon`ID `a`在`ceph`群集中运行的/var/run/ceph/mon.a.pid创建文件。在当守护程序停止正常被删除。如果未守护进程（即使用 或选项运行），则不会创建。`pid file-f-dpid file`类型

串需要

没有默认

没有

`chdir`描述

一旦启动并运行，目录Ceph守护进程就会更改为。`/`建议使用默认目录。类型

串需要

没有默认

`/`

`max open files`描述

如果设置，则在启动[Ceph存储群集](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-storage-cluster)时，Ceph将 在操作系统级别（即，文件描述符的最大数量）进行设置。它有助于防止Ceph OSD守护进程用尽文件描述符。`max open fds`类型

64位整数需要

没有默认

`0`

`fatal signal handlers`描述

如果已设置，我们将为SEGV，ABRT，BUS，ILL，FPE，XCPU，XFSZ，SYS信号安装信号处理程序，以生成有用的日志消息类型

布尔型默认

`true`

