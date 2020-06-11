# SYSTEMD

## SYSTEMD 

作为激活过程的一部分（使用[activate](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/activate/#ceph-volume-lvm-activate) 或[activate](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/activate/#ceph-volume-simple-activate)），将启用systemd单元，该单元将使用OSD id和uuid作为其名称的一部分。这些单元将在系统启动时运行，并将继续通过其子命令实现来激活其相应的卷。

用于激活的API有点松散，它只需要两部分：要使用的子命令和任何由破折号分隔的额外元信息。此约定使单位看起来像：

```text
ceph-volume@{command}-{extra metadata}
```

该_额外的元数据_可以是需要的子命令执行的处理可能需要什么。在[lvm](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/#ceph-volume-lvm)和 [simple](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/#ceph-volume-simple)的情况下，两者都希望消耗[OSD id](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-id)和[OSD uuid](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-uuid)，但这并不是硬性要求，这只是实现子命令的方式。

systemd将命令和多余的元数据都保留为单元_“实例名称”_的一部分 。例如，对于`lvm`子命令，ID为0的OSD 如下所示：

```text
systemctl enable ceph-volume@lvm-0-0A3E1ED2-DA8A-4F0E-AA95-61DEC71768D6
```

启用的单元是[systemd oneshot](https://docs.ceph.com/docs/nautilus/glossary/#term-systemd-oneshot)服务，旨在在准备好使用本地文件系统后在引导时启动。

### 失败和重试

系统联机时出现故障是很常见的。设备有时不完全可用，这种不可预知的行为可能会导致OSD无法使用。

有两个可配置的环境变量用于设置重试行为：

* `CEPH_VOLUME_SYSTEMD_TRIES`：默认为30
* `CEPH_VOLUME_SYSTEMD_INTERVAL`：默认为5

的_“尝试”_是一个数字，其设定的时间单位将试图放弃之前激活的OSD的最大量。

该_“间隔”_是以秒为确定在启动OSD发起另一个尝试之前的等待时间的值。

