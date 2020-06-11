# 内存分析

## 内存分析

Ceph MON，OSD和MDS可以使用生成堆配置文件 `tcmalloc`。要生成堆概要文件，请确保已 `google-perftools`安装：

```text
sudo apt-get install google-perftools
```

探查器将输出转储到您的目录（即 ）。有关详细信息，请参见[记录和调试](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/log-and-debug)。要使用Google的性能工具查看事件探查器日志，请执行以下操作：`log file/var/log/ceph`

```text
google-pprof --text {path-to-daemon}  {log-path/filename}
```

例如：

```text
$ ceph tell osd.0 heap start_profiler
$ ceph tell osd.0 heap dump
osd.0 tcmalloc heap stats:------------------------------------------------
MALLOC:        2632288 (    2.5 MiB) Bytes in use by application
MALLOC: +       499712 (    0.5 MiB) Bytes in page heap freelist
MALLOC: +       543800 (    0.5 MiB) Bytes in central cache freelist
MALLOC: +       327680 (    0.3 MiB) Bytes in transfer cache freelist
MALLOC: +      1239400 (    1.2 MiB) Bytes in thread cache freelists
MALLOC: +      1142936 (    1.1 MiB) Bytes in malloc metadata
MALLOC:   ------------
MALLOC: =      6385816 (    6.1 MiB) Actual memory used (physical + swap)
MALLOC: +            0 (    0.0 MiB) Bytes released to OS (aka unmapped)
MALLOC:   ------------
MALLOC: =      6385816 (    6.1 MiB) Virtual address space used
MALLOC:
MALLOC:            231              Spans in use
MALLOC:             56              Thread heaps in use
MALLOC:           8192              Tcmalloc page size
------------------------------------------------
Call ReleaseFreeMemory() to release freelist memory to the OS (via madvise()).
Bytes released to the OS take up virtual address space but no physical memory.
$ google-pprof --text \
               /usr/bin/ceph-osd  \
               /var/log/ceph/ceph-osd.0.profile.0001.heap
 Total: 3.7 MB
 1.9  51.1%  51.1%      1.9  51.1% ceph::log::Log::create_entry
 1.8  47.3%  98.4%      1.8  47.3% std::string::_Rep::_S_create
 0.0   0.4%  98.9%      0.0   0.6% SimpleMessenger::add_accept_pipe
 0.0   0.4%  99.2%      0.0   0.6% decode_message
 ...
```

同一守护程序上的另一个堆转储将添加另一个文件。与以前的堆转储进行比较以显示间隔中增长的内容很方便。例如：

```text
$ google-pprof --text --base out/osd.0.profile.0001.heap \
      ceph-osd out/osd.0.profile.0003.heap
 Total: 0.2 MB
 0.1  50.3%  50.3%      0.1  50.3% ceph::log::Log::create_entry
 0.1  46.6%  96.8%      0.1  46.6% std::string::_Rep::_S_create
 0.0   0.9%  97.7%      0.0  26.1% ReplicatedPG::do_op
 0.0   0.8%  98.5%      0.0   0.8% __gnu_cxx::new_allocator::allocate
```

有关更多详细信息，请参阅[Google Heap Profiler](http://goog-perftools.sourceforge.net/doc/heap_profiler.html)。

一旦安装了堆探查器，就启动集群并开始使用堆探查器。您可以在运行时启用或禁用堆分析器，或者确保它连续运行。对于以下命令行用法，请`{daemon-type}`用`mon`， `osd`或`mds`替换，并`{daemon-id}`用OSD编号或MON或MDS ID 替换。

### 启动PROFILER 

要启动堆分析器，请执行以下操作：

```text
ceph tell {daemon-type}.{daemon-id} heap start_profiler
```

例如：

```text
ceph tell osd.1 heap start_profiler
```

另外，如果`CEPH_HEAP_PROFILER_INIT=true`在环境中找到该变量，则可以在守护程序开始运行时启动配置文件。

### 打印统计

要打印统计信息，请执行以下操作：

```text
ceph  tell {daemon-type}.{daemon-id} heap stats
```

例如：

```text
ceph tell osd.0 heap stats
```

注意 

打印统计信息不需要运行分析器，也不会将堆分配信息转储到文件中。

### 转储堆信息

要转储堆信息，请执行以下操作：

```text
ceph tell {daemon-type}.{daemon-id} heap dump
```

例如：

```text
ceph tell mds.a heap dump
```

注意 

转储堆信息仅在分析器运行时有效。

### 释放内存

要释放`tcmalloc`已分配但Ceph守护进程本身未使用的内存，请执行以下操作：

```text
ceph tell {daemon-type}{daemon-id} heap release
```

例如：

```text
ceph tell osd.2 heap release
```

### 停止探查器

要停止堆分析器，请执行以下操作：

```text
ceph tell {daemon-type}.{daemon-id} heap stop_profiler
```

例如：

```text
ceph tell osd.0 heap stop_profiler
```

