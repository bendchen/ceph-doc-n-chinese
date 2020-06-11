# CPU分析

## CPU分析

如果您从源代码构建Ceph并编译了Ceph以与[oprofile](http://oprofile.sourceforge.net/about/)一起使用，则 可以分析Ceph的CPU使用情况。有关详细信息，请参见[安装Oprofile](https://docs.ceph.com/docs/nautilus/dev/cpu-profiler)。

### 初始化OPROFILE的

第一次使用时`oprofile`，需要对其进行初始化。找到与`vmlinux`您现在正在运行的内核相对应的 映像。

```text
ls /boot
sudo opcontrol --init
sudo opcontrol --setup --vmlinux={path-to-image} --separate=library --callgraph=6
```

### 启动OPROFILE的

要开始`oprofile`执行以下命令：

```text
opcontrol --start
```

一旦开始`oprofile`，您可以使用Ceph运行一些测试。

### 停止OPROFILE的

要停止`oprofile`执行以下命令：

```text
opcontrol --stop
```

### 检索OPROFILE结果

要检索最高`cmon`结果，请执行以下命令：

```text
opreport -gal ./cmon | less
```

要检索`cmon`带有调用图的顶部结果，请执行以下命令：

```text
opreport -cal ./cmon | less
```

重要 

查看结果后，您应该先重置`oprofile`然后再运行。重置`oprofile`将从会话目录中删除数据。

### 重置OPROFILE的

要重置`oprofile`，请执行以下命令：

```text
sudo opcontrol --reset
```

重要 

您应该`oprofile`在分析数据后重置，以免混淆来自不同测试的结果。

