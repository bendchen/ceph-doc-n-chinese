# 消息设定

## 消息

### 常规设置

`ms tcp nodelay`描述

在Messenger的TCP会话上禁用nagle的算法。类型

布尔型需要

没有默认

`true`

`ms initial backoff`描述

故障重新连接之前要等待的初始时间。类型

双需要

没有默认

`.2`

`ms max backoff`描述

故障重新连接之前等待的最长时间。类型

双需要

没有默认

`15.0`

`ms nocrc`描述

在网络消息上禁用crc。如果CPU受限制，可能会提高性能。类型

布尔型需要

没有默认

`false`

`ms die on bad msg`描述

调试选项；不配置。类型

布尔型需要

没有默认

`false`

`ms dispatch throttle bytes`描述

限制等待发送的邮件的总大小。类型

64位无符号整数需要

没有默认

`100 << 20`

`ms bind ipv6`描述

如果希望守护程序绑定到IPv6地址而不是IPv4地址，请启用。（如果您指定守护程序或集群IP，则不需要。）类型

布尔型需要

没有默认

`false`

`ms rwthread stack bytes`描述

堆栈大小的调试选项；不配置。类型

64位无符号整数需要

没有默认

`1024 << 10`

`ms tcp read timeout`描述

控制Messenger在关闭空闲连接之前将等待多长时间（以秒为单位）。类型

64位无符号整数需要

没有默认

`900`

`ms inject socket failures`描述

调试选项；不配置。类型

64位无符号整数需要

没有默认

`0`

### 异步MESSENGER选项

`ms async transport type`描述

异步Messenger使用的传输类型。可能是`posix`，`dpdk` 或`rdma`。Posix使用标准的TCP / IP网络，并且是默认设置。其他传输方式可能是实验性的，支持可能会受到限制。类型

串需要

没有默认

`posix`

`ms async op threads`描述

每个Async Messenger实例使用的工作线程的初始数量。应该至少等于最大副本数，但是如果CPU核心数量少和/或在单个服务器上托管许多OSD，则可以减少副本数。类型

64位无符号整数需要

没有默认

`3`

`ms async max op threads`描述

每个Async Messenger实例使用的最大工作线程数。当计算机的CPU数量有限时，请设置为较低的值；当CPU的利用率不足时（即，一个或多个CPU在I / O操作期间始终处于100％负载），请设置为较低的值。类型

64位无符号整数需要

没有默认

`5`

`ms async send inline`描述

直接从生成消息的线程发送消息，而不是从Async Messenger线程排队和发送消息。已知此选项会降低具有大量CPU内核的系统的性能，因此默认情况下处于禁用状态。类型

布尔型需要

没有默认

`false`

